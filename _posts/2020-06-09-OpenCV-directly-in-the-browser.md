---
layout: post
title:  【译】直接在游览器中使用OpenCV(webassembly + webworker)
date:   2020-06-09 19:00:00 +0800
categories: 开发者
tag: 开发者
---

* content
{:toc}

[原文](https://aralroca.com/blog/opencv-in-the-web)

我们将会看到如何直接在游览器中使用 OpenCV 库！要做到这点，我们需要把 OpenCV 编译成 webassembly，之后在 webworker 中运行它。

### 什么是 OpenCV

OpenCV 是最受欢迎的计算机视觉库，并自1999年以来一直存在！它的作用是提供用户友好且高效的开发环境。它是 Intel 用 C 和 C++ 编写的库。

OpenCV 还可以使用英特尔的嵌入式性能原语，这是英特尔特有的一组低级例程。

借助 OpenVC 你可以开发以下的内容：

- 2D 和 3D 工具包；
- 自运动估计(Egomotion estimation)；
- 面部识别系统；
- 手势识别；
- 人机交互(HCI)；
- 移动机器人(Mobile robotics)；
- 运动理解(Motion understanding)；
- 对象识别(Object identification)；
- 细分与识别(Segmentation and recognition)；
- 立体视觉：2台摄像机的深度感知(Stereopsis stereo vision: depth perception from 2 cameras)；
- 运动结构(SFM)；
- 运动追踪；
- 增强现实；

### 为什么在浏览器中

能够直接在浏览器运行计算机视觉算法，使我们能够将成本转移到客户端设备，从而节省了服务器的许多成本。

想象一下，你想从图片中获得葡萄酒标签的特征。有很多方法可以做到它。如果我们为我们的服务寻找最符合人体工程学的方式，我们将吧酒标签检测的一部分逻辑移动到浏览器中。然后，当我们获取请求到服务器，我们只需要发送最终的载体。这样，我们能避免在服务器上处理图像。

即使它是私人公司使用的嵌入式应用，我们可以把所有的逻辑在浏览器中。

### 开始一个新的Next.js项目

我们将在 React 中使用 Next.js 框架，以简化项目的设置和使用。但是，相同的方法也可以应用于带有 Angular，Vue.js，Svelte ...或 vanilla.js 的项目。

首先，让我们使用以下命令创建一个新的 Next.js 项目：

```bash
yarn create next-app
```

在你填写你的项目名称后，使用 `yarn dev` 启动本地环境。现在我们已经准备好在 Next.js 项目中使用 OpenCV。

### Compile OpenCV into Webassembly

要将OpenCV编译为webassembly，我们可以遵循以下官方文档：

- [https://docs.opencv.org/3.4.10/d4/da1/tutorial_js_setup.html](https://docs.opencv.org/3.4.10/d4/da1/tutorial_js_setup.html)

但是，我会告诉你我采用的步骤：

首先克隆 OpenCV 存储库：

```bash
git clone https://github.com/opencv/opencv.git
```

现在，一旦我们克隆了 repo 目录，就让我们使用 Docker 进行编译！

Linux / Mac:

```bash
docker run --rm --workdir /code -v "$PWD":/code "trzeci/emscripten:latest" python ./platforms/js/build_js.py build
```

Windows:

```bash
docker run --rm --workdir /code -v "$(get-location):/code" "trzeci/emscripten:latest" python ./platforms/js/build_js.py build
```

现在需要等待了，可能需要15分钟。

完成后，将生成的文件复制到项目中，将其移至 `/public`。

public
├── favicon.ico
├── js
+│   ├── opencv.js
└── vercel.svg

初始的内容将如下所示：

```js
/**
 *  Here we will check from time to time if we can access the OpenCV
 *  functions. We will return in a callback if it's been resolved
 *  well (true) or if there has been a timeout (false).
 */
function waitForOpencv(callbackFn, waitTimeMs = 30000, stepTimeMs = 100) {
  if (cv.Mat) callbackFn(true)

  let timeSpentMs = 0
  const interval = setInterval(() => {
    const limitReached = timeSpentMs > waitTimeMs
    if (cv.Mat || limitReached) {
      clearInterval(interval)
      return callbackFn(!limitReached)
    } else {
      timeSpentMs += stepTimeMs
    }
  }, stepTimeMs)
}

/**
 * This exists to capture all the events that are thrown out of the worker
 * into the worker. Without this, there would be no communication possible
 * with the project.
 */
onmessage = function (e) {
  switch (e.data.msg) {
    case 'load': {
      // Import Webassembly script
      self.importScripts('./opencv.js')
      waitForOpencv(function (success) {
        if (success) postMessage({ msg: e.data.msg })
        else throw new Error('Error on loading OpenCV')
      })
      break
    }
    default:
      break
  }
}
```

#### 在我们的项目中加载worker

好的，现在我们可以在项目中创建与 worker 通信的服务。为此，我们将创建一个 `services` 目录，将文件放入其中。

services
+└── cv.js

创建文件后，我们将输入以下初始代码，这将使我们能够将 OpenCV 加载到我们的项目中：

```js
class CV {
  /**
   * We will use this method privately to communicate with the worker and
   * return a promise with the result of the event. This way we can call
   * the worker asynchronously.
   */
  _dispatch(event) {
    const { msg } = event
    this._status[msg] = ['loading']
    this.worker.postMessage(event)
    return new Promise((res, rej) => {
      let interval = setInterval(() => {
        const status = this._status[msg]
        if (status[0] === 'done') res(status[1])
        if (status[0] === 'error') rej(status[1])
        if (status[0] !== 'loading') {
          delete this._status[msg]
          clearInterval(interval)
        }
      }, 50)
    })
  }

  /**
   * First, we will load the worker and capture the onmessage
   * and onerror events to always know the status of the event
   * we have triggered.
   *
   * Then, we are going to call the 'load' event, as we've just
   * implemented it so that the worker can capture it.
   */
  load() {
    this._status = {}
    this.worker = new Worker('/js/cv.worker.js') // load worker

    // Capture events and save [status, event] inside the _status object
    this.worker.onmessage = (e) => (this._status[e.data.msg] = ['done', e])
    this.worker.onerror = (e) => (this._status[e.data.msg] = ['error', e])
    return this._dispatch({ msg: 'load' })
  }
}

// Export the same instant everywhere
export default new CV()
```

#### 使用Service

由于我们是直接导出实例，因此可以将其导入到页面或组件中。

例如，我们可以在 onClick 事件上加载它：

```js
async function onClick() {
  await cv.load()
  // Ready to use OpenCV on our component
}
```

### 在游览器中使用OpenCV

现在我们已经设法在浏览器中加载了 OpenCV 库，我们将看到如何从库中运行一些实用程序。

当然，你可以使用 OpenCV 做很多事情。我将在这里展示一个简单的示例。然后，你的工作就是阅读官方文档并学习如何使用 OpenCV。

我们将使用一个简单的图像处理作为示例，使用相机拍摄照片并将其处理为灰度图。尽管看似简单，但这是我们使用 OpenCV 实现的第一个“hello world”。

```js
import { useEffect, useRef, useState } from 'react'
import cv from '../services/cv'

// We'll limit the processing size to 200px.
const maxVideoSize = 200

/**
 * What we're going to render is:
 *
 * 1. A video component so the user can see what's on the camera.
 *
 * 2. A button to generate an image of the video, load OpenCV and
 * process the image.
 *
 * 3. A canvas to allow us to capture the image of the video and
 * show it to the user.
 */
export default function Page() {
  const [processing, updateProcessing] = useState(false)
  const videoElement = useRef(null)
  const canvasEl = useRef(null)

  /**
   * In the onClick event we'll capture a frame within
   * the video to pass it to our service.
   */
  async function onClick() {
    updateProcessing(true)

    const ctx = canvasEl.current.getContext('2d')
    ctx.drawImage(videoElement.current, 0, 0, maxVideoSize, maxVideoSize)
    const image = ctx.getImageData(0, 0, maxVideoSize, maxVideoSize)
    // Load the model
    await cv.load()
    // Processing image
    const processedImage = await cv.imageProcessing(image)
    // Render the processed image to the canvas
    ctx.putImageData(processedImage.data.payload, 0, 0)
    updateProcessing(false)
  }

  /**
   * In the useEffect hook we'll load the video
   * element to show what's on camera.
   */
  useEffect(() => {
    async function initCamara() {
      videoElement.current.width = maxVideoSize
      videoElement.current.height = maxVideoSize

      if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        const stream = await navigator.mediaDevices.getUserMedia({
          audio: false,
          video: {
            facingMode: 'user',
            width: maxVideoSize,
            height: maxVideoSize,
          },
        })
        videoElement.current.srcObject = stream

        return new Promise((resolve) => {
          videoElement.current.onloadedmetadata = () => {
            resolve(videoElement.current)
          }
        })
      }
      const errorMessage =
        'This browser does not support video capture, or this device does not have a camera'
      alert(errorMessage)
      return Promise.reject(errorMessage)
    }

    async function load() {
      const videoLoaded = await initCamara()
      videoLoaded.play()
      return videoLoaded
    }

    load()
  }, [])

  return (
    <div
      style={{
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center',
        flexDirection: 'column',
      }}
    >
      <video className="video" playsInline ref={videoElement} />
      <button
        disabled={processing}
        style={{ width: maxVideoSize, padding: 10 }}
        onClick={onClick}
      >
        {processing ? 'Processing...' : 'Take a photo'}
      </button>
      <canvas
        ref={canvasEl}
        width={maxVideoSize}
        height={maxVideoSize}
      ></canvas>
    </div>
  )
}
```

在我们的 service 中：

```js
class CV {
  // ...previous service code here...

  /**
   * We are going to use the _dispatch event we created before to
   * call the postMessage with the msg and the image as payload.
   *
   * Thanks to what we've implemented in the _dispatch, this will
   * return a promise with the processed image.
   */
  imageProcessing(payload) {
    return this._dispatch({ msg: 'imageProcessing', payload })
  }
}
```

在我们的 worker 中：

```js
// ...previous worker code here...

/**
 * With OpenCV we have to work with the images as cv.Mat (matrices),
 * so you'll have to transform the ImageData to it.
 */
function imageProcessing({ msg, payload }) {
  const img = cv.matFromImageData(payload)
  let result = new cv.Mat()

  // This converts the image to a greyscale.
  cv.cvtColor(img, result, cv.COLOR_BGR2GRAY)
  postMessage({ msg, payload: imageDataFromMat(result) })
}

/**
 * This function converts again from cv.Mat to ImageData
 */
function imageDataFromMat(mat) {
  // converts the mat type to cv.CV_8U
  const img = new cv.Mat()
  const depth = mat.type() % 8
  const scale =
    depth <= cv.CV_8S ? 1.0 : depth <= cv.CV_32S ? 1.0 / 256.0 : 255.0
  const shift = depth === cv.CV_8S || depth === cv.CV_16S ? 128.0 : 0.0
  mat.convertTo(img, cv.CV_8U, scale, shift)

  // converts the img type to cv.CV_8UC4
  switch (img.type()) {
    case cv.CV_8UC1:
      cv.cvtColor(img, img, cv.COLOR_GRAY2RGBA)
      break
    case cv.CV_8UC3:
      cv.cvtColor(img, img, cv.COLOR_RGB2RGBA)
      break
    case cv.CV_8UC4:
      break
    default:
      throw new Error(
        'Bad number of channels (Source image must have 1, 3 or 4 channels)'
      )
  }
  const clampedArray = new ImageData(
    new Uint8ClampedArray(img.data),
    img.cols,
    img.rows
  )
  img.delete()
  return clampedArray
}

onmessage = function (e) {
  switch (e.data.msg) {
    // ...previous onmessage code here...
    case 'imageProcessing':
      return imageProcessing(e.data)
    default:
      break
  }
}
```

[结果](https://aralroca.com/images/blog-images/28.gif):

尽管我们已经以非常简单的方式处理了图像，并且无需使用 OpenCV 就可以完成图像处理，但这是我们使用 OpenCV 的“hello world”。它为更复杂的事物打开了大门。

### 结论

我们已经看到了如何在浏览器中使用最常用的库来实现计算机视觉。我们已经看到了如何将OpenCV编译为webassembly并在程序中使它以不阻塞UI的方式以获得良好的性能。我希望即使你从未听说过该库，现在也可以尝试一下。

### 代码

如果你想看看，我已经在GitHub上传了本文的代码。

- CODE -> [https://github.com/vinissimus/opencv-js-webworker](https://github.com/vinissimus/opencv-js-webworker)
- DEMO -> [https://vinissimus.github.io/opencv-js-webworker/](https://vinissimus.github.io/opencv-js-webworker/)

要查看在Vue.js中实现的更复杂的示例，请查看另一个库：

- [https://github.com/latsic/imgalign](https://github.com/latsic/imgalign)

### 参考文献

- https://docs.opencv.org/3.4.10/d4/da1/tutorial_js_setup.html
- https://docs.opencv.org/master/de/d06/tutorial_js_basic_ops.html
- https://en.wikipedia.org/wiki/OpenCV
- https://github.com/latsic/imgalign
- https://opencv.org/