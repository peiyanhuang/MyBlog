---
layout: post
title:  node express 文件上传
date:   2019-02-15 19:00:00 +0800
categories: Node
tag: Node
---

* content
{:toc}

### 文件上传

来看下客户端上传的简易代码：

```jsx
// upload.vue
<div class="hcell-file">
    导入文件：
    <input type="file" name="file" ref="files" />>
    <input type="button" value="上传文件" @click="upload"/>
</div>

upload() {
    let files = this.$refs.files.files[0];
    let params = new FormData();
    params.append('file', files);
    axios.post('/question/upload', params).then(res => {
        console.log(res);
    });
}
```

### express upload server

Multer 是一个 node.js 中间件，用于处理 `multipart/form-data` 类型的表单数据，它主要用于上传文件。

```js
const express = require('express');
const router = express.Router();
const multer  = require('multer');
const upload = multer({dest: './upload_tmp'});

router.post('/upload', upload.single('file'), function (req, res) {
    var file = req.file;
    console.log('文件类型：%s', file.mimetype);
    console.log('原始文件名：%s', file.originalname);
    console.log('文件大小：%s', file.size);
    console.log('文件路径：', file.path);
    res.json({code: 1});
});
```

下面是 Multer 的 API：

#### multer(opts)

| Key | Description |
| :-- | :-- |
| dest or storage | 在哪里存储文件 |
| fileFilter | 文件过滤器，控制哪些文件可以被接受 |
| limits | 限制上传的数据 |
| preservePath | 保存包含文件名的完整文件路径 |

如果省略 `options` 对象，这些文件将保存在内存中，永远不会写入磁盘。

为了避免命名冲突，Multer 会修改上传的文件名。这个重命名功能可以根据您的需要定制。

* `.single(fieldname)`

接受一个以 `fieldname` 命名的文件。这个文件的信息保存在 `req.file`。

* `.array(fieldname[, maxCount])`

接受一个以 `fieldname` 命名的文件数组。可以配置 `maxCount` 来限制上传的最大数量。这些文件的信息保存在 req.files。

* `.fields(fields)`

接受指定 `fields` 的混合文件。`fields` 应该是一个对象数组，应该具有 `name` 和可选的 `maxCount` 属性。

```js
// Example:
[
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 8 }
]
```

* `.none()`

只接受文本域。如果任何文件上传到这个模式，将发生 `LIMIT_UNEXPECTED_FILE` 错误。这和 `.fields([])` 的效果一样。

* `.any()`

接受一切上传的文件。文件数组将保存在 req.files。

**注意:** 确保你总是处理了用户的文件上传。 永远不要将 multer 作为全局中间件使用，因为恶意用户可以上传文件到一个你没有预料到的路由，应该只在你需要处理上传文件的路由上使用。

#### 文件信息

每个文件具有下面的信息:

Key | Description | Note
--- | --- | ---
`fieldname` | Field name 由表单指定 |
`originalname` | 用户计算机上的文件的名称 |
`encoding` | 文件编码 |
`mimetype` | 文件的 MIME 类型 |
`size` | 文件大小（字节单位） |
`destination` | 保存路径 | `DiskStorage`
`filename` | 保存在 `destination` 中的文件名 | `DiskStorage`
`path` | 已上传文件的完整路径 | `DiskStorage`
`buffer` | 一个存放了整个文件的 `Buffer`  | `MemoryStorage`

#### `storage`

* 磁盘存储引擎 (`DiskStorage`)

磁盘存储引擎可以让你控制文件的存储。

```javascript
var storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, '/tmp/my-uploads')
  },
  filename: function (req, file, cb) {
    cb(null, file.fieldname + '-' + Date.now())
  }
})

var upload = multer({ storage: storage })
```

有两个选项可用，`destination` 和 `filename`。他们都是用来确定文件存储位置的函数。

`destination` 是用来确定上传的文件应该存储在哪个文件夹中。也可以提供一个 `string` (例如 `'/tmp/uploads'`)。如果没有设置 `destination`，则使用操作系统默认的临时文件夹。

**注意:** 如果你提供的 `destination` 是一个函数，你需要负责创建文件夹。当提供一个字符串，multer 将确保这个文件夹是你创建的。

`filename` 用于确定文件夹中的文件名的确定。 如果没有设置 `filename`，每个文件将设置为一个随机文件名，并且是没有扩展名的。

**注意:** Multer 不会为你添加任何扩展名，你的程序应该返回一个完整的文件名。

每个函数都传递了请求对象 (`req`) 和一些关于这个文件的信息 (`file`)，有助于你的决定。

注意 `req.body` 可能还没有完全填充，这取决于向客户端发送字段和文件到服务器的顺序。

* 内存存储引擎 (`MemoryStorage`)

内存存储引擎将文件存储在内存中的 `Buffer` 对象，它没有任何选项。

```javascript
var storage = multer.memoryStorage()
var upload = multer({ storage: storage })
```

当使用内存存储引擎，文件信息将包含一个 `buffer` 字段，里面包含了整个文件数据。

**警告**: 当你使用内存存储，上传非常大的文件，或者非常多的小文件，会导致你的应用程序内存溢出。

#### 其他

还有些其他的属性，可以参考文档[Multer](https://github.com/expressjs/multer/blob/master/README.md)