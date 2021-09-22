# node后端接收表单文件

[tag]:node|file|腾讯云
[create]:2019-03-05

## **安装依赖**
```
npm install --save express multer formidable body-parser
```

## app.js

## 使用multer保存表单图片
```
const express = require('express');
const app = express();
const bodyParser = require('body-parser');
const multer = require('multer');
const upload = multer({ dest: 'uploads/' }); // 保存在当前目录下的uploads文件夹
// 如果调用multer时，不传入文件夹名称，那么文件将不会写入磁盘，而是在内存中

// file为表单中字段的属性名
app.post('/img', upload.single('file'), (req, res) => {
    console.log(req.file); // 此时req对象中就多了个file字段
    res.json({
        message: 'I got it.'
    });
});

const server = app.listen(3000, () => {
    console.log('服务挂载在3000端口');
});
```

## 使用formidable保存表单图片
```
const express = require('express');
const app = express();
const bodyParser = require('body-parser');
const formidable = require('formidable');

// file为表单中字段的属性名
app.post('/img', (req, res) => {
    const form = new formidable.IncomingForm();
    // 设置保存未知可以使用 form.uploadDir = "/my/dir";
    form.parse(req, function(err, fileds, files) {
        if (err) {
            res.json({
                message: 'I lose it.'
            });
        } else {
            console.log(files);
            res.json({
                message: 'I got it.'
            });
        }
    });
});

const server = app.listen(3000, () => {
    console.log('服务挂载在3000端口');
});
```

## 将上传的文件保存到腾讯云cos上
```
const express = require('express');
const app = express();
const bodyParser = require('body-parser');
const multer = require('multer');
const upload = multer();
const COS = require('cos-nodejs-sdk-v5'); // 这个依赖另外安装
const config = require('./config'); // 保存cos的敏感信息
const fs = require('fs');
const cos = new COS({
  SecretId: config.secretId,
  SecretKey: config.secretKey
});

// file为表单中字段的属性名
app.post('/img', upload.single('file'), (req, res) => {
    cos.putObject({
        Bucket: config.bucket,
        Region: config.region,
        Key: 'node_upload/' + req.file.filename,
        Body: req.file.buffer
    }, (err, data) => {
        // 上传之后最好清理掉文件引用，以免造成内存泄漏
        req.file = undefined;
        if (err) {
            res.json({
                message: 'I lose it.'
            });
        } else {
            res.json({
                filePath: 'https://' + data.Location
            });
        }
    });
});

const server = app.listen(3000, () => {
    console.log('服务挂载在3000端口');
});
```
