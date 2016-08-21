---
title: WWDC 2016 - Session 401 - What's New in Xcode App Signing 笔记
date: 2016-08-21 22:15:34
tags: [WWDC, iOS, Code Signing]
---
> 这篇 blog 来自我们内部的分享，内容比较精简，需要更多的细节信息，请参考 WWDC 2016 - 401 的视频

相信每一个开发者在初学 iOS 的时候，都有过被 Code Signing 坑过的经历，特别是当旁边没有人指导的时候，这也是当时我个人学习 iOS 的时候最困扰的地方，证书，provisioning profile, code signing 等等这些和实际的开发无关的概念，现在还记得苦苦看文档的经历。

![Image1 icon](/assets/code_signing_0.png)  
iOS证书申请和签名打包流程图，图来自[这里](http://www.pchou.info/ios/2015/12/14/ios-certification-and-code-sign.html)

Xcode 团队在 Xcode 8 中移除了 fix issue 之后还需要 fix issue 但是可能还是不能 fix issue 的 `Fix Issue` 按钮，并完全重新设计了 code signing 的交互，流程和架构，不管对于 iOS 初学者还是有经验的程序员，都能大大简化 code  signing 的流程，让我们把精力更专注于实际的业务开发上。

Xcode 8 支持两种签名方式，自动化签名 (`Automatic Singing`)和自定义签名(`Customized  Signing`) 的。下面我们说一下基础概念和这两种签名方式。

#### 1. 基础概念 (Fundamentals)
1. 证书 (Certificate)
	在 Xcode 8 之前，每个账号一般有两个证书，一个是开发证书，一个是发布证书。开发证书和发布证书都只能存在一份，所以如果有多台 Mac 开发设备，需要通过证书的导出导入来同步证书（和密钥）。在 Xcode 8 之后，支持多个**开发证书** (发布证书依然只能有一个)，也就是说，多台 mac 开发设备可以**自动**生成多份有效的开发证书（和密钥），就不再需要导出导入了。（这里有雷鸣般的掌声）
2. Provisioning profile
	用来授权给包含在 profile 里的 iOS 开发设备来安装 app。在 Xcode 8 之前，每次添加新的设备都会生成新一个新的 profile，并产生一个唯一的 id，所以在每次添加设备之后，因为 profile id 变了，需要更新并提交 project 文件，Xcode 8 以后是用文件名的方式引用，就是说添加了新的设备，只要 profile 文件名不变，就不会修改 project 文件了。这也算是一个比较方便的改进吧。（掌声）
3. Entitlement
	其实就是管理我们开启的 Capabilities，比如 IAP，Push Notifications，iCloud 等等

4. 签名的流程
	首先 Xcode 根据所择的 team 从 key chains 里选择**最新的**证书，然后根据 app identifier 选择**最新的** provisioning profile，在 build 的时候 profile 会被放在 app 包里，code sign 工具根据证书生成一个 code seal（可以理解为盖一个戳）。如果有人篡改了 app，这个戳就不 match 了，iOS 系统会阻止 app 安装。

> 想了解更多？[代码签名探析@objc.io](https://objccn.io/issue-17-2/)

#### 2. 自动化签名 (Automatic Singing)
在这种模式下，Xcode 全自动的为我们管理整个签名的流程，整个过程会在后台执行，会保证所有签名需要的文件是最新的。


![Image1 icon](/assets/code_signing_1.png)

我们所需要做的就是勾选上自动化签名，然后选择 team。剩下的 Xcode 都会接管。比如创建证书，创建和更新 profile 等等。但是当插入了一台新的 iOS 设备，Xcode 8 还是会提示是否把这台设备添加到测试设备中，如果选择是，Xcode 8 会自动添加到设备列表里，并自动更新 profile 文件。（鼓掌）

如果比较好奇 Xcode 自动为我们做了什么，可以在  Reports 里看查看 log, （鼓掌） 比如：  
![Image1 icon](/assets/code_signing_2.png)

Xcode 自动化签名只会自动化开发阶段的签名，不会修改发布的签名设置。既然这样，如何设置 release 版本的签名呢？其实我们在 Archive 的时候，Xcode 默认使用的还是开发证书做的签名，然后在 Orgnizer 里选择 export 到 App Store 发布版本的时候，会让我们重新选择 证书重新签名，这里再选择发布证书（演讲者这里说的是开发证书，应该是口误）。


#### 3. 自定义签名(Customized  Signing)
如果我们想自己管理签名所需的文件，可以选择自定义签名方式。这种模式下，Xcode 不会对签名设置做任何的修改。

操作很简单，就是取消勾选自动化签名，然后就可以对每个 build configuration 做不同的签名设置了，注意不用去 Build Setting 里设置了，直接 General 里就可以完成签名的设置了。如下图，对免费版和收费版设置不同的 profile:  
![Image1 icon](/assets/code_signing_3.png)

虽然我们设置了自定义签名，但 Xcode 并不是真的什么都不做了，相反如果签名的设置有问题， Xcode 提供更多友好和精确的提示：  
![Image1 icon](/assets/code_signing_4.png)  
![Image1 icon](/assets/code_signing_5.png)  

#### 4. 最佳实践 (Best Practices)
blah blah...  
一句话：使用自动化签名 （to make your life as easy as possible）  
blah blah...  

Enjoy!
