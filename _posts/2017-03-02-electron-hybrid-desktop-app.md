---
layout: post
title: Electron & C++ 快速开发桌面 Web "混合"应用
tags:
- C++
- Electron
- Desktop
- Video
categories: C++
description: 使用 node-gyp 为 Electron 开发 C++ 扩展，快速对接现有 web 应用和直播 SDK
---

## 缘起
最近要做个小东西，接入网易直播 SDK，但有一个比较蛋疼的问题就是 web 推流一般都用 Flash，Flash 音频只能用 speex，然而接收端如果是苹果，一般都会选 HLS，而 HLS 只能用 AAC 音频。这样一来，要么服务端再做一次转码和重新分发（延迟+成本，所以猪厂直接不支持），要么只能选择推流端本地先编好 AAC(iOS/Mac/PC). 只能请 C++ 大哥出马了。

这活又需要对接现有的网站，包括用户、频道、互动啥的，又要得急。咋办捏？去翻了翻 SDK 提供的 Demo, 我的妈呀竟然是 QT 做的，那咋改嘛…想起了从前被 QT 和 MFC 支配的恐惧，还是换个思路吧。

前一阵子看过 Electron，正好嵌网页方便，放狗搜了一圈，发现了[node-gyp](https://github.com/nodejs/node-gyp)这高科技。可以直接调用 C++ 模块，好！就你了！

## 折腾

然后就开始了漫长的 Visual Studio 安装过程…话说 VS 这货是越来越大了，以前去食堂吃顿饭的功夫就能装完，后来去吃个烤串才能装完，再后来…好像得喝上几轮才能装完…

后来才发现似乎不装也行…… 直接这样就好了 

```

npm install -g --production windows-build-tools

npm install -g node-gyp

```

然后就可以放心大胆地写 C++ 啦，按照套路来就好了（来自 [node 官网文档](https://nodejs.org/api/addons.html) ）

```

#include <node.h>

namespace demo {

using v8::FunctionCallbackInfo;

using v8::Isolate;

using v8::Local;

using v8::Object;

using v8::String;

using v8::Value;

void Method(const FunctionCallbackInfo& args) {  

    Isolate* isolate = args.GetIsolate();  

    args.GetReturnValue().Set(String::NewFromUtf8(isolate, "world"));

}

void init(Localexports) {

    NODE_SET_METHOD(exports, "hello", Method);

}

NODE_MODULE(addon, init)

}  // namespace demo

```
这样就完成了一个可供 node 调用的 C++ 模块，只需要再加个配置文件`binding.gyp` 
```
{
  "targets": [{
    "target_name": "addon",
    "sources": [ "hello.cc"]
  }]
}
```
然后
```
node-gyp configure
npm install
```
然后直接开个js

```
var addon = require("./build/Release/addon");
console.log(addon.hello());
```
哇 就可以了 是不是很神奇……

接入 SDK 的时候有个坑，网易直播 SDK 只提供了 32bit 的库，如果是 win64 系统，默认编译也是 64位。可能会出现一个奇怪的错误提示`The system cannot find message text for message number 0x%1 in the message file for %2` 你就不能直接告诉我`Wrong Architecture`么……
Don't panic, 坐和放宽，你需要`node-gyp rebuild -target=1.4.13 -arch=ia32 -dist-url=https://atom.io/download/atom-shell` (1.4.13 是你用的 Electron 版本号)

网易直播 SDK 给的 demo 代码还比较靠谱了，照着文档写出来能编译过，这就比你们某些开源项目不知高到哪里去了 23333。
但有些细节地方还是需要处理的，

比如主播通常会外接麦克风啥的，那就需要获取音视频设备列表然后让用户选择；

还有本地要做个预览窗口，如果偷懒可以直接嵌个网易提供的 web player 直接拉流回来，但这样一来肯定有延迟啦，感觉上就会怪怪的。所以还是需要本地来播放一下。demo 里提供的方法是 QPaint, 我们既然不用 QT 就得另想办法啦，比如 VLC/FFMpeg 之类的总得嵌一个吧？

然后还有多线程之类的也要做的吧？C++ 做东西就是要考虑比较多

还得处理跟 web 的整合，比如通过 API 获取推流的地址、界面布局的整合

还得考虑到客户的业务是培训/在线教育，以后有可能还要接入其他的比如屏幕共享啦 PPT 播放啦 白板啦之类的，这就得折腾另外一套SDK了，目前还没看，还不知道能不能支持，只能开脑洞了，比如白板直接转成 HTML5 Canvas? 

哦 还有 Electron 自己的几个问题，代码不能让人直接扒了，可以通过 `asar` 之类的打个包，体积太大，估计还得用 UPX 之类的压一下，压完了还得测测什么XX卫士 XX助手 会不会报毒…

有人估计要问了这么麻烦你为啥不直接用原生…说了啊！我害 pia 嘛！
而且客户那边的工程师估计 web 居多，界面你们就可以随便折腾了啊，我只需要维护 C++ 这一块就够了…

嗯嗯 基本就是这样，我继续搬砖去了…