# 解决puppeteer无法在docker中运行

[tag]:docker|爬虫|puppeteer
[create]:2021-03-21

最近开发遇到个需要爬取第三方数据的需求，所有用上了puppeteer，但是在部署的时候发现爬取的时候会报错:

```text
Error: Failed to launch the browser process!
/app/node_modules/puppeteer/.local-chromium/linux-722234/chrome-linux/chrome: error while loading shared libraries: libX11-xcb.so.1: cannot open shared object file: No such file or directory


TROUBLESHOOTING: https://github.com/puppeteer/puppeteer/blob/master/docs/troubleshooting.md

    at onClose (/app/node_modules/puppeteer/lib/Launcher.js:750:14)
    at Interface.helper.addEventListener (/app/node_modules/puppeteer/lib/Launcher.js:739:50)
    at Interface.emit (events.js:187:15)
    at Interface.close (readline.js:379:8)
    at Socket.onend (readline.js:157:10)
    at Socket.emit (events.js:187:15)
    at endReadableNT (_stream_readable.js:1081:12)
    at process._tickCallback (internal/process/next_tick.js:63:19)
```

报这个错误是因为加载无头浏览器失败。需要安装chrome内核。

## 解决方法

需要在docker构建的时候将chrome环境一并安装。修改后的简易打包代码如下:

```sh
FROM node:10.6.0
ARG APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=1
RUN apt-get update
RUN apt-get install -y wget gnupg ca-certificates procps libxss1 --fix-missing
RUN wget -q -O - http://dl.google.com/linux/linux_signing_key.pub | apt-key add -
RUN sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list'
RUN apt-get update
RUN apt-get install -y google-chrome-stable
RUN rm -rf /var/lib/apt/lists/*
# RUN wget https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh -O /usr/sbin/wait-for-it.sh
COPY ./wait-for-it.sh /usr/sbin/wait-for-it.sh
RUN chmod +x /usr/sbin/wait-for-it.sh

COPY . /app
WORKDIR /app

RUN npm install

RUN npm install puppeteer@2.0.0

EXPOSE 3000

CMD node index.js
```

这里可以将多个RUN命令合成一个运行，但是如果运行时错误，为了更方便看到错误出在哪里，所以将其拆开来最好。

这里指定puppeteer版本为2.0.0是因为我的服务跑在node10.6的环境下，如果你使用的版本比较高，记得修改这个版本以及node版本。

最终部署之后就可以跑了。

[线上demo](https://github.com/sansui-orz/docker-puppeteer)