---
layout: post
title:  "[Template]Electron + React + webpackの導入(備忘録)"
date:   2017-06-10 20:01:14 +0900
tags: electron react webpack installation
permalink: /electron-react/installation
---
```
npm: 5.0.3
electron: 1.6.11
```

Electronの開発に必要なReact(UIライブラリ)、Webpack(モジュールハンドラ)、
Babel(JSXトランスパイル)、Photpn(CSS)などのモジュールを導入する。
ここでポイントは、Webpackでトランスパイルした後に作成されるディレクトリを正しく設定することである。
トランスパイル後が"dist/"ディレクトリ下に作成されるため、index.htmlやpackage.jsonに指定する必要がある。

#### Electronのひな型作成

"Electron Quick Start"というElectron開発の最小構成のテンプレがあるので、
git cloneとnpm initしておく
```
$ git clone https://github.com/electron/electron-quick-start
$ npm init -y
```

#### モジュールのインストール
```
$ npm install electron --save-dev
$ npm install connors/photon --save
$ npm install webpack --save-dev
$ npm install css-loader style-loader --save-dev
$ npm install babel-core babel-loader babel-preset-es2015 babel-preset-react --save-dev
$ npm install react react-dom react-router --save
```

#### package.jsonを用意する

以下のJSONをpackage.jsonに記述し(上記の「モジュールのインストール」ですでに追加されているが、package.jsonに記述した後に、npm installでも導入可能)
```
$ npm installでdependencies,devDependenciesに書かれているモジュールを一括でインストールできる
```
"main"には、トランスパイル後のディレクトリ(dist/main/index.js)を指定する
{% highlight javascript %}
{
  "name": "Template",
  "version": "1.0.0",
  "description": "",
  "main": "dist/main/index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": ""
  },
  "keywords": [],
  "author": "htwata",
  "license": "ISC",
  "homepage": "https://bitbucket.org/dev_htwata/gentil#readme",
  "devDependencies": {
    "babel-core": "^6.25.0",
    "babel-loader": "^7.1.1",
    "babel-preset-es2015": "^6.24.1",
    "babel-preset-react": "^6.24.1",
    "css-loader": "^0.28.4",
    "electron": "^1.6.11",
    "style-loader": "^0.18.2",
    "webpack": "^3.4.1"
  },
  "dependencies": {
    "photon": "github:connors/photon",
    "react": "^15.6.1",
    "react-dom": "^15.6.1"
  },
  "scripts": {
    "build": "webpack",
    "watch": "webpack --watch",
    "start": "electron ."
  }
}
{% endhighlight %}


#### Webpack・.babelrcの設定
・webpack.config.js
{% highlight javascript %}

module.exports = {
  target: "electron",
  node: {
    __dirname: false,
    __filename: false
  },
  resolve: {
    extensions: [".js", ".jsx"]
  },
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        loader: "babel-loader"
      },
      {
        test: /\.css$/,
        loaders: [ "style-loader", "css-loader?modules" ]
      }
    ]
  },
  entry: {
    "main/index": "./src/main/index.js",
    "renderer/app": "./src/renderer/app.jsx"
  },
  output: {
    filename: "dist/[name].js"
  },
  devtool: "source-map"
};
{% endhighlight %}


・.babelrc
```
{
  "presets": [
    "es2015", "react"
  ]
}


#### ひな型ファイルの作成
以下のファイルを作る
・index.html
{% highlight html %}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    <!-- All of the Node.js APIs are available in this renderer process. -->
    We are using Node.js <script>document.write(process.versions.node)</script>,
    Chromium <script>document.write(process.versions.chrome)</script>,
    and Electron <script>document.write(process.versions.electron)</script>.
    <div class="window">
      <div id="app" class="window-content"></div>
    </div>
    <script>
      require("./dist/renderer/app.js")
    </script>
  </body>
</html>
{% endhighlight %}

・src/main/index.js(メインプロセス、main.jsからsrc/main/index.jsに設置)
ここでの"dirname"は、dist/main/をさしており、index.htmlはプロジェクトのルートディレクトリにおくので、
以下のように変更する
```
”pathname: path.join(__dirname, 'index.html'),”
```
↓
```
”pathname: path.join(__dirname, '../../index.html')
```

・src/main/index.js
{% highlight javascript %}
const electron = require('electron')
// Module to control application life.
const app = electron.app
// Module to create native browser window.
const BrowserWindow = electron.BrowserWindow

const path = require('path')
const url = require('url')

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let mainWindow

function createWindow () {
  // Create the browser window.
  mainWindow = new BrowserWindow({width: 800, height: 600})

  // and load the index.html of the app.
  mainWindow.loadURL(url.format({
    //pathname: path.join(__dirname, 'index.html'),
    pathname: path.join(__dirname, '../../index.html'),
    protocol: 'file:',
    slashes: true
  }))

  // Open the DevTools.
  // mainWindow.webContents.openDevTools()

  // Emitted when the window is closed.
  mainWindow.on('closed', function () {
    // Dereference the window object, usually you would store windows
    // in an array if your app supports multi windows, this is the time
    // when you should delete the corresponding element.
    mainWindow = null
  })
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.on('ready', createWindow)

// Quit when all windows are closed.
app.on('window-all-closed', function () {
  // On OS X it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', function () {
  // On OS X it's common to re-create a window in the app when the
  // dock icon is clicked and there are no other windows open.
  if (mainWindow === null) {
    createWindow()
  }
})
{% endhighlight %}

・src/renderer/app.js
```
import React from "react";
import { render } from "react-dom";

render(<div>Hello, Mario = DAN</div>, document.getElementById("app"));
```

{% highlight javascript %}

{% endhighlight %}

#### webpackによるトランスパイルと実行

```
$ ./node_modules/.bin/webpack
$ ./node_modules/.bin/electron .
```
