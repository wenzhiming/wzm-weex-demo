## IDE

推荐使用VSCode。

## 安装依赖

Weex 官方提供了weex-cli 的脚手架工具来辅助开发和调试。

首先，你需要 Node.js 和 Weex CLI。

安装 Node.js 方式多种多样，最简单的方式是在 [Node.js 官网](https://nodejs.org/en/?spm=a2c7j.-zh-guide-develop-setup-develop-environment.0.0.82e61a8eOyxGdn) 下载可执行程序直接安装即可。

更多安装方式可参考 [Node.js 官方信息](https://nodejs.org/en/download/?spm=a2c7j.-zh-guide-develop-setup-develop-environment.0.0.82e61a8eOyxGdn)

> TIP
通常，安装了 Node.js 环境，npm 包管理工具也随之安装了。因此，直接使用 npm 来安装 `weex-toolkit` , 你也可以通过 yarn 来进行安装。
国内的开发者推荐将npm镜像切换至 Taobao NPM 镜像 `https://registry.npm.taobao.org` 。

运行下面的命令安装最新的beta版本工具：

### OSX环境

```bash
$ sudo chmod -R 777 /usr/local/lib/node_modules/
$ npm i -g weex-toolkit // 安装weex工具包
$ weex -v // 查看当前weex工具版本
```

### Windows环境

```bash
$ npm i -g weex-toolkit 
$ weex -v // 查看当前weex工具版本
```

安装结束后你可以直接使用 `weex help` 命令验证是否安装成功，它会显示 weex 支持的所有指令，同时，你也可以通过 `weex doctor` 命令检查你的本地开发环境。

## 初始化项目
然后初始化 Weex 项目：

```bash
$ weex create weex-demo
```

执行完命令后，在 weex-demo 目录中已经为我们生成了标准项目结构。

## 开发
进入项目所在路径，如果你在生成项目的时候选择了自动安装依赖，在进入项目后只需直接运行 `npm start` 就可以将项目完整跑起来，否则，你需要预先在项目中运行一下 `npm install` 安装项目所需依赖。  

`npm start`运行后工具会启动一个本地的 web 服务，监听 `8081` 端口。你可以打开 `http://localhost:8081` 查看页面在 Web 下的渲染效果。 源代码在 `src/` 目录中，你可以像一个普通的 Vue.js 项目一样来开发.


## 编译和运行
默认情况下 `weex create` 命令并不初始化 iOS 和 Android 项目，你可以通过执行 `weex platform add` 来添加特定平台的项目。

```bash
weex platform add ios
weex platform add android
```

由于网络环境的不同，安装过程可能需要一些时间，请耐心等待。如果安装失败，请确保自己的网络环境畅通。

为了能在本地机器上打开 Android 和 iOS 项目，你应该配置好客户端的开发环境。对于 iOS，你应该安装并且配置好 Xcode。对于 Android，你应该安装并且配置好 Android Studio。当开发环境准备就绪后，运行下面的命令，可以在模拟器或真实设备上启动应用：

```bash
weex run ios
weex run android
weex run web
```
>tips:
>配置Android环境时推荐JDK选择`JDK8`，我自己的电脑上JDK环境默认是`JDK14`,结果编译报错了，切换到JDK8就正常了。

### 编译异常
1. `weex run ios` 运行失败 ：
Failed to locate 'instruments' 
xcrun: error: unable to find utility "instruments", not a developer tool or in PATH
![在这里插入图片描述](https://img-blog.csdnimg.cn/0138945eab6f41a39f1827499181a441.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAemhpbWluZ3dlbg==,size_20,color_FFFFFF,t_70,g_se,x_16)
命令行运行失败可能是weex打包工具weex-toolkit的问题，官方一直没有推出修复版。我们尝试其他办法运行项目，比如直接用xcode 运行是否可以呢？于是试试用xcode打开WeexDemo.xcworkspace，点击运行，发现还是报错 Pods-XXXXX.debug.xcconfig.xcconfig: unable to open file:
![在这里插入图片描述](https://img-blog.csdnimg.cn/1f7a7a6c24ef4fc7b12676238a2f5374.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAemhpbWluZ3dlbg==,size_19,color_FFFFFF,t_70,g_se,x_16)
这个是一个常见错误，解决办法很简单：
我们先cd到iOS目录下，在命令行输入：
1，sudo gem install cocoapods --pre
2，pod install

然后运行结果依然是报错的：
![在这里插入图片描述](https://img-blog.csdnimg.cn/4075364e608b47cbb6ec9103c2208671.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAemhpbWluZ3dlbg==,size_20,color_FFFFFF,t_70,g_se,x_16)
分析可得知，是无法远程下载 weex-sdk 0.18.0版本。
我们先看看项目模板中自动生成的podfile内容：
```bash
source 'git@github.com/CocoaPods/Specs.git'
platform :ios, '8.0'
#inhibit_all_warnings!

def common
	pod 'WeexSDK'
    pod 'WeexPluginLoader'
    pod 'SDWebImage', '3.7.5'
    pod 'SocketRocket', '0.4.2'
end

target 'WeexDemo' do
    common
end

target 'WeexUITestDemo' do
    common
end
```
发现`WeexSDK`没有指定版本，而模板工程生成的时候关联的 weex-sdk 0.18.0在远程仓库已经失效了。查weex官方github主页可知最新的sdk版本是0.30.0。而且ios平台默认为8.0太低了，可以升级到9.0。
于是修改Podfile如下：

```bash
source 'git@github.com/CocoaPods/Specs.git'
platform :ios, '9.0'
#inhibit_all_warnings!

def common
	pod 'WeexSDK','0.30.0'
    pod 'WeexPluginLoader'
    pod 'SDWebImage', '3.7.5'
    pod 'SocketRocket', '0.4.2'
end

target 'WeexDemo' do
    common
end

target 'WeexUITestDemo' do
    common
end
```
这时候运行 `weex run ios` 命令依旧是跑不起来的，但是打开xcode 选择真机或模拟器，点击run就可以正常运行项目了。
## 调试
weex-toolkit 还提供了强大的调试功能，只需要执行：

```bash
weex debug
```

这条命令会启动一个调试服务，并且在 Chrome （目前只支持基于 V8 引擎的桌面浏览器） 中打开调试页面。详细用法请参考 [weex-toolkit](http://doc.weex.io/zh/guide/develop/weex_cli.html#%E7%B3%BB%E7%BB%9F%E7%BB%84%E4%BB%B6)的文档。

## How to create your own template

See [How-to-create-your-own-template](https://github.com/weex-templates/How-to-create-your-own-template).

## License

Copyright 2022. WenZhiming

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   <http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
