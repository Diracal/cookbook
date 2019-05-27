# 使用pre-signed URLs上传文件 [![Slack](https://slack.min.io/slack?type=svg)](https://slack.min.io)

使用presigned URLs,你可以让浏览器将文件直接上传到与S3兼容的云存储服务(S3)，而不需要暴露S3服务的认证信息给这个用户。

这个指南介绍了如何使用[MinIO JavaScript Library](https://github.com/minio/minio-js)的[`presignedPutObject`](https://docs.min.io/docs/javascript-client-api-reference#presignedPutObject) API生成一个pre-signed URL。这是一个通过JavaScript示例展示的，在示例中，Express Node.js服务器公开一个endpoint去生成一个pre-signed URL以及客户端web应用使用URL上传一个文件到MinIO Server。

1. [创建Server](#createserver) 
2. [创建客户端Web应用](#createclient)

## <a name="createserver"></a>1. 创建服务器

该服务器由一个[Express](https://expressjs.com) Node.js服务器组成，该服务器公开一个名为`/ presignedUrl`的端点。这个endpoint使用`Minio.Client`对象生成一个短期的预签名URL，可用于将文件上载到MinIO Server.

```js
// 为了使用MinIO JavaScript API生成pre-signed URL，用示例化一个`Minio.Client`对象开始并为你的服务器传递值。
// 下面的示例使用play.min.io:9000的值

const Minio = require('minio')

var client = new Minio.Client({
    endPoint: 'play.min.io',
    port: 9000,
    secure: true,
    accessKey: 'Q3AM3UQ867SPQQA43P2F',
    secretKey: 'zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG'
})

// 示例化一个`express` server并且公开一个叫做`/presignedUrl`的endpoint作为通过叫做`name`的查询参数接受文件名字的`GET请求`。
//对于这个endpoint的实现来说，调用`Minio.Client`示例的[`presignedPutObject`](https://docs.min.io/docs/javascript-client-api-reference#presignedPutObject)来生成一个pre-signed URL，并且返回响应的URL：

// express是一个小巧的Http server封装，不过这对任何HTTP server都管用。
const server = require('express')()

server.get('/presignedUrl', (req, res) => {
    client.presignedPutObject('uploads', req.query.name, (err, url) => {
        if (err) throw err
        res.end(url)
    })
})

server.get('/', (req, res) => {
    res.sendFile(__dirname + '/index.html');
})

server.listen(8080)
```

## <a name="createclient"></a>2. 创建客户端web应用

客户端web application用户界面包括一个选择器参数，允许用户选择上传文件，以及一个调用叫做`upload`的`onclick`处理程序的按钮。

```html
<input type="file" id="selector" multiple>
<button onclick="upload()">Upload</button>

<div id="status">No uploads</div>

<script src="//code.jquery.com/jquery-3.1.0.min.js"></script>
<script type="text/javascript">

//`upload`遍历所有选中的文件并调用一个名为`retrieveNewURL`的辅助函数。
 function upload() {
   [$('#selector')[0].files].forEach(fileObj => {
     var file = fileObj[0]
     // 从服务器获取一个URL
     retrieveNewURL(file, url => {
       // 上传文件到服务器
       uploadFile(file, url)
     })
   })
 }

 // `retrieveNewURL`接受当前的文件名字并且调用`/presignedURL`端点
 // 生成一个pre-signed URL用于上传文件：
 function retrieveNewURL(file, cb) {
   $.get(`/presignedUrl?name=${file.name}`, (url) => {
     cb(url)
   })
 }

 //``uploadFile``接受当前的文件名字和pre-signed URL。然后它可以调用`XMLHttpRequest()`
 // 使用下列的URL来上传文件到S3的`play/min/io:9000`。
 function uploadFile(file, url) {
     var xhr = new XMLHttpRequest ()
     xhr.open('PUT', url, true)
     xhr.send(file)
     xhr.onload = () => {
       if (xhr.status == 200) {
         $('#status').text(`Uploaded ${file.name}.`)
       }
     }
 }

</script>
```

**注意：** 这个web应用使用[jQuery](http://jquery.com/)。
