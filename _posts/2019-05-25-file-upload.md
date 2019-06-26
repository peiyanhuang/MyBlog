---
layout: post
title:  文件的分片及断点上传
date:   2019-05-25 19:00:00 +0800
categories: 开发者
tag: JS
---

* content
{:toc}

上传大文件时，如果不重新配置 nginx 的 http 超时时间和 client_max_body_size 请求 body 的大小，很容易报 413 或 502 的状态码。

分片上传可以解决以上的问题。分片上传，就是将所要上传的文件，按照一定的大小，将整个文件分隔成多个数据块（我们称之为Part）来进行分别上传，上传完之后再由服务端对所有上传的文件进行汇总整合成原始的文件。分片上传不仅可以避免因网络环境不好导致的一直需要从文件起始位置还是上传的问题，还能使用多线程对不同分块数据进行并发发送，提高发送效率，降低发送时间。

### 分片上传的流程

1. 先对文件进行MD5的加密, 这样有两个好处, 即可以对文件进行唯一的标识, 为秒传做准备, 也可以为后台进行文件完整性的校验进行比对
2. 拿到MD5值以后, 要查询一下, 这个文件是否已经上传过了, 如果上传过了, 就不用再次重复上传, 也就是能够秒传, 网盘里的秒传, 原理也是一样的
3. 对文件进行切片, 假如文件是500M, 一个切片大小我们定义为50M, 那么整个文件就为分为100次上传
4. 向后台请求一个接口, 接口里面的数据是该文件已经上传过的文件块, 为什么要有这个请求呢? 我们经常用网盘, 网盘里面有续传的功能, 一个文件传到一半, 由于各种原因, 不想再传了, 那么再次上传的时候, 服务器应该保留我之前上传过的文件块, 跳过这些已经上传过的块, 再次上传其他文件块, 当然续传方案有很多, 目前来看, 单独发一次请求, 这样效率最高
5. 开始对未上传过的块进行POST上传
6. 当上传成功后, 通知服务器进行文件的合并, 至此, 上传完成!

### 主要代码

- 获取文件 md5 值，

```js
import md5 from 'md5';

export function md5File (file) {
	return new Promise((resolve, reject) => {
		const fileName = file.name
		// 读取文件:
    const reader = new FileReader();
    reader.onload = function(e) {
      const data = e.target.result;
      resolve(md5(`${fileName}${data}`));
    };
    reader.onerror = function(e) {
    	reject(e)
    }
    reader.readAsArrayBuffer(file);
	});
}
```

- 文件分片，这也很简单

```js
export function sliceFile (file, fileMD5 blockSize = 1024 * 1024) {
	const fileSize = file.size;
	const blockNum = Math.ceil(fileSize / blockSize);
	for (let i = 0; i < blockNum; i++) {
		let end = (i + 1) * blockSize >= fileSize ? fileSize : (i + 1) * blockSize
    let fileChunks = file.slice(i * blockSize, end)
    
    // upload(fileMD5, fileChunks, blockNum, i)
	}
}
```

### 断点续传

实现分片上传后断点续传也很简单了。

客户端发送身份+文件 md5，以及该文件的总分片数，校验已上传的分片。服务端返回该文件已经上传了的分片索引数组，这样就可以可以实现断点续传。我们设定分片大小是固定的，如果文件 md5 不变，那么它分片的结果也不会变。