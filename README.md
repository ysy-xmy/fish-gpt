# chatgpt-vue

使用 Vue3 + Typescript + Tailwind CSS 框架，调用 OpenAI 的 `gpt-3.5-turbo` 模型 API 实现的简单聊天对话，支持连续对话。

![preview](img/preview.gif)

## 快速开始

>注意：本项目没有使用任何代理，API 在前端发送请求，能否连通基于你当前浏览器的所处的网络环境。如果你需要在服务端发送 API 请求的 ChatGPT，可以查看我的另一个项目 [chatgpt-nuxt](https://github.com/lianginx/chatgpt-nuxt)，点击这里 [在线体验](http://ai.in-x.cc/)。

在开始之前，请确保您已正确安装 Node.js 运行时环境。如果您还没有安装 Node.js，请 [点击这里下载](https://nodejs.org/zh-cn/)。

使用 ChatGPT 需要先申请 API Key，已注册但还没有 API Key 的用户可以 [前往这里生成](https://platform.openai.com/account/api-keys)。

准备就绪后，进入项目根目录执行以下命令运行项目：

```bash
npm i
npm run dev
```

或者

```bash
yarn
yarn dev
```

运行成功时通常显示（请以具体为准）：

```bash
VITE v3.2.5  ready in 294 ms

➜  Local:   http://localhost:5173/
➜  Network: use --host to expose
```

