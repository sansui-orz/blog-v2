# express构建小程序登录以及验证流程

[tag]:express|miniapp|auth
[create]:2019-03-05

## 小程序端

app.js
```
const login = new Promise((resolve, reject) => {
    wx.login({
        success: e => {
            wx.request({
                url: 'http://localhost:3000/login',
                method: 'POST',
                data: {
                    code: e.code
                },
                success: res => {
                    resolve(res.data.token);
                },
                fail: reject
            });
        },
        fail:: reject
    });
});

// 将登录请求放在最前面，可以省去页面编译的时间
App({
    ...
});
```

## node后端
### 安装依赖

```
npm install --save express jsonwebtoken request-promise body-parser
```
index.js
```
const express = require('express');
const app = express();
const bodyParser = require('body-parser');
const jwt = require('jsonwebtoken');
const request = require('request-promise');
const tokenSecret = 'tokenSecret';

app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

// 校验是否带有合法token
app.all('*', (req, res, next) => {
    try {
        res.header('Access-Control-Allow-Origin', '*');
        res.header('Access-control-Allow-Headers', 'XX-Token');
        res.header('Access-Control-Allow-Methods', 'GET,POST,DELETE,PUT,OPTIONS,HEAD,FETCH');
        if (req.url === '/login') {
            next();
        } else {
            // 小程序无法携带cookie所以将token放在了head里面
            jwt.verify(req.headers['xx-token'], tokenSecret, (err, result) => {
                if (err) {
                    res.json({
                        message: '登陆失败'
                    });
                } else {
                    req.__openid__ = result.data.openid;
                    next();
                }
            });
        }
    } catch(err) {
        res.json({
            message: '登录失败'
        });
    }
});

app.post('/login', async (req, res) => {
    try {
        const response = await request({
            method: 'POST',
            url: 'https://api.weixin.qq.com/sns/jscode2session?',
            formData: {
                appid: config.appid,
                secret: config.appSecret,
                js_code: req.body.code,
                grant_type: 'authorization_code'
            }
        });

        let data = JSON.parse(response);

        const token = jwt.sign({
            exp: Math.floor(Date.now() / 1000) + (2 * 60 * 60), // 两小时过期
            data: {
                openid: data.openid,
                session_key: data.session_key
            }
        }, tokenSecret);

        res.json({
            token: token
        });
    } catch(err) {
        res.json({
            message: '登陆失败'
        });
    }
});

const server = app.listen(3000, () => {
    console.log('服务挂载在3000端口');
});
```
