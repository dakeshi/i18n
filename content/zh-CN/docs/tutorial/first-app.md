# 打造你第一个 Electron 应用

Electron 可以让你使用纯 JavaScript 调用丰富的原生(操作系统) APIs 来创造桌面应用。 你可以把它看作一个专注于桌面应用的 Node. js 的变体，而不是 Web 服务器。

这不意味着 Electron 是某个图形用户界面（GUI）库的 JavaScript 版本。 相反，Electron 使用 web 页面作为它的 GUI，所以你能把它看作成一个被 JavaScript 控制的，精简版的 Chromium 浏览器。

** 注意 **: 此示例还有可用的仓库, 您可以 [ 立即下载并运行 ](#trying-this-example)。

从开发的角度来看, Electron application 本质上是一个 Node. js 应用程序。 应用启动的入口是一个与 Node.js 模块相同的 `package.json` 文件。 一个最基本的 Electron 应用一般来说会有如下的目录结构：

```text
your-app/
├── package.json
├── main.js
└── index.html
```

为你的新Electron应用创建一个新的空文件夹。 打开你的命令行工具，然后从该文件夹运行`npm init`

```sh
npm init
```

npm 会帮助你创建一个基本的 `package.json` 文件。 其中的 `main` 字段所表示的脚本为应用的启动脚本，它将会在主进程中执行。 如下片段是一个 `package.json` 的示例：

```json
{
  "name": "your-app",
  "version": "0.1.0",
  "main": "main.js"
}
```

**注意**：如果 `main` 字段没有在 `package.json` 中出现，那么 Electron 将会尝试加载 `index.js` 文件（就像 Node.js 自身那样）。 如果你实际开发的是一个简单的 Node 应用，那么你需要添加一个 `start` 脚本来指引 `node` 去执行当前的 package：

```json
{
  "name": "your-app",
  "version": "0.1.0",
  "main": "main.js",
  "scripts": {
    "start": "node ."
  }
}
```

把这个 Node 应用转换成一个 Electron 应用也是非常简单的，我们只不过是把 `node` 运行时替换成了 `electron` 运行时。

```json
{
  "name": "your-app",
  "version": "0.1.0",
  "main": "main.js",
  "scripts": {
    "start": "electron ."
  }
}
```

## 安装 Electron

现在，您需要安装`electron`。 我们推荐的安装方法是把它作为您 app 中的开发依赖项，这使您可以在不同的 app 中使用不同的 Electron 版本。 在您的app所在文件夹中运行下面的命令：

```sh
npm install --save-dev electron
```

除此之外，也有其他安装 Electron 的途径。 请咨询[安装指南](installation.md)来了解如何用代理、镜像和自定义缓存。

## 开发一个简易的 Electron

Electron apps 是使用与 Node.js 开发中相同的原理和方法在 JavaScript 中开发的。 All APIs and features found in Electron are accessible through the `electron` module, which can be required like any other Node.js module:

```javascript
const electron = require('electron')
```

The `electron` module exposes features in namespaces. As examples, the lifecycle of the application is managed through `electron.app`, windows can be created using the `electron.BrowserWindow` class. 一个简单的`main.js`文件可能仅等待应用程序准备就绪后打开一个窗口：

```javascript
const {app, BrowserWindow} = require('electron')
const path = require('path')
const url = require('url')

function createWindow () {
  // Create the browser window.
  win = new BrowserWindow({width: 800, height: 600})

  // 然后加载应用的 index.html。
  win.loadURL(url.format({
    pathname: path.join(__dirname, 'index.html'),
    protocol: 'file:',
    slashes: true
  }))
}

app.on('ready', createWindow)
```

The `main.js` should create windows and handle all the system events your application might encounter. A more complete version of the above example might open developer tools, handle the window being closed, or re-create windows on macOS if the user clicks on the app's icon in the dock.

```javascript
const {app, BrowserWindow} = require('electron')
const path = require('path')
const url = require('url')

// 保持一个对于 window 对象的全局引用，如果你不这样做，
// 当 JavaScript 对象被垃圾回收， window 会被自动地关闭
let win

function createWindow () {
  // 创建浏览器窗口。
  win = new BrowserWindow({width: 800, height: 600})

  // 然后加载应用的 index.html。
  win.loadURL(url.format({
    pathname: path.join(__dirname, 'index.html'),
    protocol: 'file:',
    slashes: true
  }))

  // 打开开发者工具。
  win.webContents.openDevTools()

  // 当 window 被关闭，这个事件会被触发。
  win.on('closed', () => {
    // 取消引用 window 对象，如果你的应用支持多窗口的话，
    // 通常会把多个 window 对象存放在一个数组里面，
    // 与此同时，你应该删除相应的元素。
    win = null
  })
}

// Electron 会在初始化后并准备
// 创建浏览器窗口时，调用这个函数。
// 部分 API 在 ready 事件触发后才能使用。
app.on('ready', createWindow)

// 当全部窗口关闭时退出。
app.on('window-all-closed', () => {
  // 在 macOS 上，除非用户用 Cmd + Q 确定地退出，
  // 否则绝大部分应用及其菜单栏会保持激活。
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  // 在macOS上，当单击dock图标并且没有其他窗口打开时，
  // 通常在应用程序中重新创建一个窗口。
  if (win === null) {
    createWindow()
  }
})

// 在这文件，你可以续写应用剩下主进程代码。
// 也可以拆分成几个文件，然后用 require 导入。
```

最后，你想展示的 `index.html`：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using node <script>document.write(process.versions.node)</script>,
    Chrome <script>document.write(process.versions.chrome)</script>,
    and Electron <script>document.write(process.versions.electron)</script>.
  </body>
</html>
```

## 启动你的应用

Once you've created your initial `main.js`, `index.html`, and `package.json` files, you can try your app by running `npm start` from your application's directory.

## 尝试此例

复制并运行这个库 [`electron/electron-quick-start`](https://github.com/electron/electron-quick-start)。

**注意**：运行此需要[Git](https://git-scm.com)

```sh
# 克隆这仓库
$ git clone https://github.com/electron/electron-quick-start
# 进入仓库
$ cd electron-quick-start
# 安装依赖库
$ npm install
# 运行应用
$ npm start
```

For a list of boilerplates and tools to kick-start your development process, see the [Boilerplates and CLIs documentation](./boilerplates-and-clis.md).