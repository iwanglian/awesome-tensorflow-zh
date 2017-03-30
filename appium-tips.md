## 前言
数据分析离不开数据，爬数据为分析提供弹药.
去年了解两个微信自动抢红包的插件。
1. 利用 Android的辅助功能，监控通知和消息列表，当有`红包`特征的文字出现时，模拟点击，打开红包
2. 使用 xposed 框架，注入相关函数里，直接调用抢红包的函数，又快又狠
方式一速度慢，但开发简单直接。方案二，当遇到微信升级，函数变化，就需要重新反编译微信来分析。



在爬数据领域，也是这两个思路。我们从第一个模拟点击思路出发。有以下几种方案，
1. 辅助功能（类似抢红包插件），Android开发
2. pc端模拟点击微信客户端，使用模拟精灵
3. Android端使用触动精灵的脚本
4. 使用moneyrunner
5. 使用UIAutomator

## Appium 介绍
查询了大量资料，发现Appium具备上述方案的所有优点。
1. 可以模拟点击坐标，可以获取ui层次，直接点击控件
2. 支持多语言开发，特别是 python
3. 支持切换到webview,并且获取渲染后的 dom树和源码
3. 版本一直在演进，提供越来越多特性

## Appium 安装
### macOS
1. 官网下载  dmg 安装包
2. 下载安装 JDK, Android SDK, 并且设置环境变量 
3. GUI操作，填写 android sdk地址，使用 / 开头的绝对地址
3. 或者使用 命令行， 进入 `/Applications/Appium.app/Contents/Resources/node_modules/appium` ；
3. node . -p 4492 -bp 2251 -U 32456  ，即启动server
5. python用 virtaulenv创建一个新环境，`pip install Appium-Python-Client`

### Windows
1. 官网下载 exe安装包
2. 安装node 6.2.1, 注意 node7会报错
3. 安装python 2.7
4. 安装 jdk7
5. 安装android studio
6. 同上

### Linux
1. 保证网络是直连的，不要通过代理（否则在 npm安装appium时会出错）
2. 安装 nvm, 用nvm 安装 nodejs, npm
3. nodejs安装appium-doctor, 查看缺少的依赖项
4. 安装Android studio, 将目录下的 jre目录放入 环境变量 JAVA_HOME， jre/bin加入PATH; 下载的SDK路径放入 ANDROID_HOME中，tools, platform-tools加入PATH
5. 安装 appium, 此时网络可以有代理。
6. appium -p 4492 -bp 2251 -U 32456  ，即启动server
6. 同上

> **注意**
> Windows系统下 adb 连接手机过多时会异常，另外 chromeDriver也会异常，抓取不到部分网页
> 建议使用 Linux系统



## 执行脚本
Appium本身是nodejs的工程，但支持各种语言，最方便的就是 python, 利用 python 全面的功能。这里以控制 
Android手机为例。


### 初始化
webdriver初始化时，必须填上正确的值。几个需要注意的：
1. platformVersion  Android版本号，appium根据此值，决定对手机使用哪一套指令框架
2. app              apk在电脑主机的路径，填上此值，当手机没有安装时，会自动安装App
3. appPackage       被自动执行的App的 `package name`,例如 `com.tencent.mm`
4. appActivity      首先打开的Android页面，如微信首页`.ui.LauncherUI`
5. deviceName       在`adb devices` 看到的设备标识
6. noReset          启动时不重置应用，这个很重要，如果不设置，每次就会把上次执行的登录和用户信息全部抹掉
7. unicodeKeyboard  输入汉字和全角符号必备，appium会为手机安装unicode输入法，当有输入时，自动切换到此输入法
8. resetKeyboard    执行完成后，恢复原输入法。 appium unicode输入法不会弹出键盘，不适合人机交互
9. recreateChromeDriverSessions  下次切换到webview时，是否重建，填 `True`, 否则下次使用webview时不能正常工作
10. chromeOptions    chromeDriver的选项，如 {"androidProcess": 'com.tencent.mm:tools'} ,即要使用的webview所在的进程


启动时使用的端口号，与appium执行的 -p 端口号一致即可。

这样就可以一台pc开多个appium, 指定不同的端口（-p 和 -bp都要不同），然后连接多台手机。
