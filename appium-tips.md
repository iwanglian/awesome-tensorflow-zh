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
初始化完成后，通过 implicitly_wait 方法指定 查找的隐性等待时间。

### WebDriver
这是脚本中最常用的对象，用于操作整个页面。

#### Android
对于native页面，使用 Android Sdk 里带的 uiautomatorviewer 工具查看，在截图里点击元素，就会显示其 NodeInfo,然后通过这个信息来查找元素。
* find_element_by_id                      通过 resource-id 来查找，会阻塞住不断重试，直到超时或找到一个
* find_elements_by_id                     找出所有符合 resource-id的元素，立即返回，需要自己用sleep控制页面加载时间
* find_element_by_accessibility_id        通过 content-desc 查找
* find_elements_by_class_name             通过 class 查找
* find_element_by_android_uiautomator     通过 Android UIAutomator 的 selector来查找，例如 "new UiSelector().text(\"订阅号\")"

打到元素后，就可以通过元素的 click()方法来点击； send_keys方法来输入文字

这里有几个关注点
1. find_element 超时没找到时，会 raise NoSuchElementException, 可以捕获该异常来判断元素是否存在
2. 判断元素是否存在，另一个方法是使用 find_elements_* , 再判断其 len
3. 对于很靠下，不在当前屏幕的元素，通过 find_element* 不能直接找到，需要 find_element_by_android_uiautomator("new UiScrollable(new UiSelector()).scrollIntoView(new UiSelector().text(\"文本\"))")， 这样会滑到界面
4. resource-id 是 Android在编译资源文件时生成的，所以每个新版本，相同位置元素的 resource-id 的值可能会变化；尽量采用 text 等别的固定不变的元素；关闭升级
5. 在升级提示、输入法提示、登录提示这种会打断自动运行，需要提前手动处理好，或者在固定场景下写异常逻辑
6. 在手动逻辑中，有些搜索没有按钮，而是通过输入法键盘右下角的`搜索`来触发，这时需要先 activate_ime_engine ，打开其它输入法，调出软键盘,然后通过坐标点击
7. 遇到异常时，尽量保留日志和手机截图，使用 get_screenshot_as_file 方法
```
                dr.activate_ime_engine(imes[0])
                el.click()
                size = dr.get_window_size()
                dr.tap([(size['width'] * 0.95, size['height'] * 0.95)])
                dr.activate_ime_engine(u'io.appium.android.ime/.UnicodeIME')
                
 ```

#### WebView
App经常会内嵌 WebView, 这时需要将 WebDriver 切换 到 webview 的 context, 才能操作 webview。

> **注意** 
>
> 有些App使用自研的 webview, 比如 x5内核。这时需要切换到系统webview, 才能被 webdriver操作。
>
> 在 App里打开网页 http://debugx5.qq.com  (要带上 http://),在 信息 一栏中，打开使用 inspect 调试的选项
>



将手机联到电脑，在chrome中访问  chrome://inspect , 就可以看到手机中打开的页面， 从这里查看元素的属性。

常用的查找方法
* find_elements_by_class_name
* find_element_by_id
* find_element_by_xpath             执行复杂的查询



还可以拿到网页的url和html源码。
在mac和linux环境下，webDriver.page_source得到的源码，是经过js执行后的。



要注意的点
1. 找到的元素的坐标跟实际不一致，其 click() 无效
2. 每个手机的坐标都可能不一样
3. 坐标信息，可以通过 手机 设置->开发者选项->指针位置 打开。部分机型需要重启生效



#### 跳转

尽量使用元素的click行为。最后方案是选择 webDriver.tap 坐标点。
每一轮执行后，请返回首页，从头开始操作。中间状态都难以预测。

1. 在各种 Exception 被捕获到时，也需要返回到首页
1. 使用 driver.startActivity 来重新打开，这个耗时较多，但更安全
2. 模拟 返回按键，直到首页。这个不用重新打开首页，速度很快。


```
        while dr.current_activity != '.ui.LauncherUI':
            dr.press_keycode(4)
            sleep(1)
```

### 附加功能
1. 安装和卸载应用，重置应用。
2. 设置手机的联网方式
3. 设置手机的gps坐标
4. 从设备上传和下载文件

### 同时运行多个设备
1. 在文档里支持多个设备，但实际使用 Linux 跑两个设备时就会adb经常重启导致失败
2. [issue](https://github.com/appium/appium/issues/3592)里建议端口号间隔10以上
3. 还可以写监控脚本，在失败时，重启所有程序


tips:
1. 由于appium、python-client依赖众多，接口很容易不兼容，总是难以定位。

```
WebDriverException: Message: Parameters were incorrect. We wanted {"required":["value"]} and you sent ["text","sessionId","id","value"]
#产生原因是 ， selenium 的版本 3.3.3过高，用pip降级到 3.3.1 即可


提示  chrome版本应该>55, 但实际 chrome版本已经是57
#产生原因，  最新appium@1.6.4带的chromedriver2.28在 linux系统找到不到chrome, 降级到 2.25即appium@1.6.3即可
```
**参考网站**

* [appium官网](http://appium.io/)
* [appium github](https://github.com/appium/appium)
* [appium-python-client](https://github.com/appium/python-client)
* [api总结](http://www.cnblogs.com/psklf/p/5368828.html)










