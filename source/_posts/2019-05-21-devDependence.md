---
title: devDependence
date: 2019-05-21 22:33:19
tags:
---

# Node的dependencies与devDependencies

### 从npm说起
>NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：
>- 允许用户从NPM服务器下载别人编写的第三方包到本地使用。
>- 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
>- 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。

### package.json
管理本地包的配置文件，在文件中需要配置一些基础信息，以及项目依赖包的信息。

### dependencies与devDependencies登场
package.json之中便出现了我们这次讨论的主角，dependencies与devDependencies。
```
npm install xxx --save // 安装后写入dependencies
npm install xxx --save-dev // 安装后写入devDependencies
```
上述是命令行安装依赖时候使用的命令  
那么这两行看似相近的命令到底有什么区别呢？顾名思义：
#### dependencies
运行时刻需要的包
#### devDependencies
开发时候需要用到的包（比如一些处理编译/测试框架的包)，会在执行npm install或者npm link时安装  
***
例如我们的项目使用ES6来编写，但是最后为了兼容性，一般我们会输出编译后的js文件到lib目录供别人使用  
那么我们编译相关的babel之类的包便是本地发布时需要使用的包，而对于使用方来说并不需要，因此这些包就可以放到devDependencies里  
一般在真正发布线上的时候，脚本命令也会使用
```
npm install --prod
```
这样只会在服务器安装运行时刻需要依赖的包而已。
后续我还会讨论项目中遇到的一些依赖安装的坑，下次再见啦