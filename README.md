# Vue 3 + Typescript + Vite + electron hello world


### step 1 创建项目

```shell
yarn create vite
```

### step 2 添加 electron 依赖

```shell
yarn add -D concurrently cross-env electron electron-builder wait-on
```

### step 3 安装vue相关依赖

```shell
yarn install
```

### step 4 添加 electron 编译配置

```json
"build": {
  "appId": "com.my-website.my-app",
  "productName": "MyApp",
  "copyright": "Copyright © 2019 ${author}",
  "mac": {
    "category": "public.app-category.utilities"
  },
  "nsis": {
    "oneClick": false,
    "allowToChangeInstallationDirectory": true
  },
  "files": [
    "dist/**/*",
    "electron/**/*"
  ],
  "directories": {
    "buildResources": "assets",
    "output": "dist_electron"
  }
}
```

### step 5 添加 yarn/npm 脚本

```json
"scripts": {
    "dev": "vite",
    "build": "vue-tsc --noEmit && vite build",
    "serve": "vite preview",
    "electron": "wait-on tcp:3000 && cross-env IS_DEV=true electron .",
    "electron:pack": "electron-builder --dir",
    "electron:dev": "concurrently -k \"cross-env BROWSER=none yarn dev\" \"yarn electron\"",
    "electron:builder": "electron-builder",
    "build:for:electron": "vue-tsc --noEmit && cross-env ELECTRON=true vite build",
    "app:build": "yarn build:for:electron && yarn electron:builder"
  },
```

### step 6 修改vite.config.ts/js 配置

```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  base: process.env.ELECTRON=="true" ? './' : ".",
  plugins: [vue()]
})
```

### step 7 编写 electron 相关代码

```javascript
// electron/electron.js
const path = require('path');
const { app, BrowserWindow } = require('electron');

const isDev = process.env.IS_DEV == "true" ? true : false;

function createWindow() {
  // Create the browser window.
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      nodeIntegration: true,
    },
  });

  // and load the index.html of the app.
  // win.loadFile("index.html");
  mainWindow.loadURL(
    isDev
      ? 'http://localhost:3000'
      : `file://${path.join(__dirname, '../dist/index.html')}`
  );
  // Open the DevTools.
  if (isDev) {
    mainWindow.webContents.openDevTools();
  }
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.whenReady().then(() => {
  createWindow()
  app.on('activate', function () {
    // On macOS it's common to re-create a window in the app when the
    // dock icon is clicked and there are no other windows open.
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
});

// Quit when all windows are closed, except on macOS. There, it's common
// for applications and their menu bar to stay active until the user quits
// explicitly with Cmd + Q.
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});


// electron/preload.js

// All of the Node.js APIs are available in the preload process.
// It has the same sandbox as a Chrome extension.
window.addEventListener('DOMContentLoaded', () => {
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector)
    if (element) element.innerText = text
  }

  for (const dependency of ['chrome', 'node', 'electron']) {
    replaceText(`${dependency}-version`, process.versions[dependency])
  }
})
```

### step end 

开发
```shell
yarn electron:dev
```

构建

```shell
yarn app:build
```

单纯调试网页

```shell
yarn dev
```

原文链接
https://dev.to/brojenuel/vite-vue-3-electron-5h4o

git代理

```shell
git config --global http.proxy "socks5://127.0.0.1:7891"
git config --global https.proxy "socks5://127.0.0.1:7891"
# 端口号在代理软件上查看
```

