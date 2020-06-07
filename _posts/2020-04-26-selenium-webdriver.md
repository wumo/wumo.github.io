---
title:  Selenium WebDriver浏览器自动测试
---

Selenium可以通过特定的webdriver与主流浏览器进行程序化交互，这种能力可以用于自动测试浏览器程序，也可以作为运行浏览器脚本的容器。

使用Selenium需要下载对应浏览器的WebDriver，如Chrome需要[ChromeDriver](https://chromedriver.chromium.org/downloads)，FireFox需要[geckodriver](https://github.com/mozilla/geckodriver/releases)，而Edge浏览器也有对应的[webdriver](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/)。 WebDriver的版本需要与目标浏览器的版本兼容，否则出现无法启动或连接浏览器的错误。

<!--description-->

## 使用Selenium进行自动化测试的步骤：

1. 在目标操作系统上安装目标浏览器。
以github actions的runner为例，在ubuntu-latest上安装chrome的命令如下：
```bash
sudo apt-get update
sudo apt-get install -y software-properties-common
sudo add-apt-repository ppa:canonical-chromium-builds/stage -y
sudo apt-get update
sudo apt-get install chromium-browser -y
```
在windows-latest上**安装firefox的命令如下**：
```bash
choco install firefox -y
```
github actions runner可能已经安装了你需要的浏览器，查阅[software-installed-on-github-hosted-runners](https://help.github.com/en/actions/reference/software-installed-on-github-hosted-runners)来确认。
2. 确保WebDriver在程序可访问的路径之上，一般可以将下载的
WebDriver放置`Path`环境变量或当前目录。github actions runner 也可能已经下载了流行的WebDriver。
3. 添加Selenium相应程序语言的binding。
```kotlin
implementation("org.seleniumhq.selenium:selenium-java:3.141.59")
```
4. 以`headless`模式启动测试浏览器。因为目标操作系统不一定装有图形组件又或者为性能考虑，通常需要以`headless`模式启动测试浏览器，不同的浏览器启动`headless`模式的参数不同，如chrome的参数如下：
```kotlin
val options = ChromeOptions()
options.addArguments("headless")
```
而FireFox的参数如下：
```kotlin
val firefoxBinary = FirefoxBinary()
firefoxBinary.addCommandLineOptions("--headless")
val options = FirefoxOptions()
options.binary = firefoxBinary
```

## Selenium的备忘
* 一般使用css selector浏览DOM树，css selector的常用语法如下：
```
tag.class[attribute=value]
```
如`div.blue-btn[id=msg-confirm]`就可以匹配`<div class="blue-btn" id="msg-confirm">`这样的标签。

* 通过css selector找到的`WebElement`具有以下方法：
`click()`用于点击标签，但有时某些标签无法点击，则需要通过运行javascript来进行点击：
```kotlin
val executor = driver as JavascriptExecutor
executor.executeScript("arguments[0].scrollIntoView();", webElement)
executor.executeScript("arguments[0].click();", webElement)
```
`sendKeys()`和`clear()`用于向文本框输入字符或者清除字符。
`getCssValue()`和`getText()`可以获取标签属性值和文本内容