---
title: "Electron Note"
tags:
  - Electron
  - note
categories:
  - ["Electron"]
  - ["Note"]
date: 2020-03-21 16:55:25
draft: false
toc: false
images:
math: true
---

Note
<!--more-->

# Hello World

在项目文件夹下新建两个文件:  `main.html` & `main.js`

```html
<!--main.html-->

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello World</title>
</head>

<body>
    <h1 id="title">
        Hello World !
    </h1>
</body>

</html>
```



``` js
// main.js

const electron = require('electron'); 		 // 引入electron模块

const app = electron.app; 					 // 创建electron引用
const BrowserWindow = electron.BrowserWindow; // 创建窗口引用

let mainWindow = null; 					// 声明要打开的主窗口

app.on('ready', () => {
    mainWindow = new BrowserWindow({ 	// 设置打开的窗口大小
        width: 800,
        heigth: 500
    });
    mainWindow.loadFile('main.html'); 	// 加载html页面
    mainWindow.on('closed', () => {		// 监听窗口关闭事件
        mainWindow = null				// 一定要把窗口设置为null,否则会一直占内存
    });									// 如同C语言申请内存后一定要free释放内存
});
```

然后打开cmd命令行, cd到项目的根目录, 执行命令

```shell
npm init
```

项目根目录就会生成一个 `package.json` 文件

```json
{
  "name": "Hello-World",
  "version": "1.0.0",
  "description": "",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "electron ."	// 这句要自己添加
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

然后就可以在 `cmd命令行` 中执行命令来启动项目

```shell
npm start 或者 electron .
```

效果如下:

![Hello world](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Electron-Note/1.png)


也可以自己添加 `main.css` 编写样式, 让界面更好看.

所有命名不固定.



# 主进程与渲染进程

### 主进程

1. Electron 运行 `package.json` 的 `main属性` 的进程被称为主进程

2. 每个应用只有一个主进程
3. **作用**:
    1. 管理原生GUI , 典型的窗口(BrowserWindow , Tray, Dock, Menu)
    2. 创建渲染进程
    3. 控制应用生命周期 (app)
4. **模块**:
    1. **(常用): **app , BrowserWindow , ipcMain , Menu , Tray , MenuItem , dialog , Notification , webContents , autoUpdater , globalShortcut , **clipboard , crashReporter**
    2. SystemPreferences , TouchBar , netLog , powerMonitor , inAppPurchase , net , powerSaveBlocker , contentTracing , BrowserView ,  session , protocol , Screen , **shell , nativelmage**

### 渲染进程

1. 展示web页面的进程称为渲染进程
2. 一个应用可以有多个渲染进程
3. **作用**: 通过Node.js,  Electron提供的API可以跟系统底层打交道
4. **常用模块**: 
    1. **(常用): **ipcRenderer , remote , desktopCapture , **clipboard , crashReporter** 
    2. webFrame , **shell , nativelmage**

## 进程间通信

通信工具: **IPC通信模块**

- Electron 提供了IPC通信模块, 主进程的 `ipcMain` 和渲染进程的 `ipcRenderer` 
- `ipcMain` 和 `ipcRenderer` 都是`EventEmitter` 对象

### 从渲染进程到主进程 (render to main)

- Callbake写法:
    - ipcRenderer.send( channer, ...args)      // 渲染进程中发送
    - ipcMain.on(channel, handler)                // 主进程中响应

```js
// Render-process.js
const {ipcRenderer} = require('electron');
ipcRenderer.send('通信频段名', 0或N个参数);


// Main-process.js
...
function handleIPC() {
    ipcMain.on('通信频段名', (err, 0或N个参数) => {
        // do something to reply
    });
}

setTimeout(handleIPC, 500);
...
```



- Promise写法 (Electron 7.0之后, 处理请求 + 响应模式):
    - ipcRenderer.invoke(channel, ...args)    // 渲染进程中发送
    - ipcMain.handle(channel, handler)        // 主进程中响应

### 从主进程到渲染进程 (main to render)

- ipcRenderer.on(channel, handler)    // 渲染进程中响应
- webContents.send(channel)              // 主进程中发送

```js
// Main-process.js
...
mainWindow = new BrowserWindow({......});
mainWindow.webContents.send('通信频段名');
...


// Render-process.js
const {ipcRenderer} = require('electron');
ipcRenderer.on('通信频段名', (err, 0或N个参数) => {
    // do something to reply
})
```



### 从渲染进程到渲染进程 (render to render)   页面间通信
页面之间的通信主要做两件事情: 1. 通知事件;  2. 数据共享

#### 通知事件

- ipcRenderer.sendTo( webContentsId, channel, ...args )

```js
// Main-process.js
const {app, BrowserWindow, Notification, ipcMain} = require('electron');
let win1 = null;
let win2 = null;
app.on('ready', () => {
    win1 = new BrowserWindow({ 
        width:500, 
        heigth:500, 
        webPreferences:{nodeIntegration:true} 
    });
    win1.loadFile('./win1.html');
    win2 = new BrowserWindow({ 
        width:500, 
        heigth:500, 
        webPreferences:{nodeIntegration:true} 
    });
    win2.loadFile('./win2.html');
    
    global.sharedObject = { win2WebContentsId: win2.webContents.id }	//把win2的id放在全局对象中
});
```

```js
// render1-process.js
// sender
const {ipcRenderer, remote} = require('electron')
let win2Id = remote.getGlobal('sharedObject').win2WenContentsId		//获取win2的id
ipcRenderer.sendTo(win2Id, '通信频段名', 0或N个参数)		// 与win2进行通信
```

```js
// render2-process.js
// responser
const {ipcRenderer} = require('electron')
ipcRenderer.on('通信频段名', (err, 0或N个参数) => {	// 响应
    // do something to reply
});
```



#### 数据共享

- 使用web技术( localStorage , sessionStorage , indexedDB )



### 调试

#### 渲染进程的调试

1. 用代码打开 Chromiun的开发者工具

```js
let win = new BrowserWindow();
win.webContents.openDevTools();
```

2. 输入命令行 `electron .` 之后, 在窗口按下快捷键 `Ctrl + Shift + i`



#### 主进程的调试

Electron 主进程是一个 Node.js 进程。Node.js 在 8 之后引入了 `--inspect` 参数用于调试，同样也适用于 Electron 主进程：

```sh
electron . --inspect
```

默认会监听 9229 端口，应用启动后，在 Chrome 浏览器（或其他基于 Chromium 开发的浏览器）中打开 `chrome://inspect` 即可看到对应的调试会话，点击会话链接即可打开 devtools 进行调试。

另外，可以在命令行参数中指定端口号，实现同时调试多个应用中的多个进程而不产生冲突：

```sh
electron . --inspect=1234
```

##### 步骤

1.开启命令行开关

启动electron的时候需要带上inspect开关，并配置调试端口.

有两个开关，分别是 `--inspect=[port]` 和 `--inspect-brk=[port]`，区别在于后者会暂停在第一行js代码

这里建议在 `package.json` 的 `script` 字段添加如下内容
    
```json
 "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "start": "electron .",
        "debug": "electron . --inspect=5858"    // 添加这行
      },
```

2.设置chrome调试器

打开chrome，然后新开一个标签进入[chrome://inspect](chrome://inspect) ，这里我们要先配置监听的端口，不然的话，Remote Target列表里是不会出现要调试的electron程序的

\>_

![chrome://inspect](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Electron-Note/chrome-inspect.png)


\>_

![chrome-port](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Electron-Note/chrome-port.png)

    
然后在项目目录下就可以直接使用命令 

```sh
npm run debug
```

就可以看到如下画面:

\>_

![chrome-run](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Electron-Note/chrome-run.png)

    
    


3.调试

点击 inspect 就可以进行调试了.

##### 在 VSCode 中调试

上述方法均会打开 devtools 界面，所有的调试操作均在 devtools 中进行。对于某些操作比如代码断点调试，可以进一步与编辑器或 IDE 相结合，提升开发体验。以下将简要介绍如何在 VSCode 进行调试。

以 Electron 官方的模板 [electron-quick-start](https://github.com/electron/electron-quick-start) 为例，首先需要为 VSCode 安装一个扩展：[Debugger for Chrome](https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-chrome)（用于调试渲染进程）。克隆代码仓库到本地并安装依赖：

```sh
git clone https://github.com/electron/electron-quick-start.git
cd electron-quick-start
npm install
```

然后在仓库中添加文件 `.vscode/launch.json`，内容如下：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Main",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "C:\\Users\\用户名\\AppData\\Roaming\\npm\\electron",
      "runtimeArgs": ["--remote-debugging-port=9222", "."],
      "windows": {
          "runtimeExecutable": "C:\\Users\\用户名\\AppData\\Roaming\\npm\\electron.cmd"
      }
    },
    {
      "name": "Renderer",
      "type": "chrome",
      "request": "attach",
      "port": 9222,
      "webRoot": "${workspaceFolder}"
    }
  ],
  "compounds": [
    {
      "name": "All",
      "configurations": ["Main", "Renderer"]
    }
  ]
}
```

> 注意: Windows系统的路径分隔符要写作 "\\\\" 
>
> if  普通安装electron 
>
> ​	then 把"用户名"改成"你系统的用户名";
>
> else 自定义安装electron 
>
> ​	then 找到你的 electron.cmd; 
>
> ​			 复制路径; 
>
> ​			 到 json中修改;
>
> 
>
> if  Linux用户
>
> ​	then 系统.路径分隔符 = "/"
>
> if  (项目根目录/node_modules/.bin/electron).isExist
>
> ​	then 
>
> ```json
>  "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/electron",
>  "windows": {
>      "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/electron.cmd"
>  }
> ```



然后在 VSCode 左侧选择 debug 面板，启动 `All` 这一项开始调试，此时就可以在 `main.js` 或 `renderer.js` 文件中添加断点了：

配置文件中的一些要点解释如下：

1. `configurations` 中的两项分别对应主进程和渲染进程。`compounds` 中指定了一个组合会话 `All`，选择 `All` 将会同时启动这两个会话。
2. Renderer 配置中的 `webRoot` 参数直接使用了 `${workspaceFolder}`，是因为在这个工程中，HTML 引用的静态资源位于根目录下。实际使用的时候需要更新到对应的路径才会生效。
3. 实际开发中可能会有编译的流程，比如使用 TypeScript 配合打包工具 Webpack，最终生成的代码与源代码并不在一个路径下。这种情况下需要产出 source map 来建立映射关系。

\>_

![debug](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Electron-Note/debug.png)




### 经验技巧

- 少用remote模块

    因为每次remote会触发底层的同步IPC事件, 特别影响性能, 处理的不好容易进程卡死

- 不要用sync模式

    一旦写的不好就会整个应用卡死

- 在请求+响应的通信模式下,需要自定义超时限制

    在响应的时候需要设置一个时长限制, 当应用响应超时, 需要response一个异常的超时事件让业务处理, 然后去做对应的交互