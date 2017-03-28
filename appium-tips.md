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
5. 写 python 脚本执行

### Windows
1. 官网下载 exe安装包
2. 安装node 6.2.1, 注意 node7会报错
3. 安装python 2.7
4. 安装 jdk7
5. 安装android studio
6. 同上
