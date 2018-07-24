### Selenium

首先要先了解下 selenium，一个浏览器自动化测试工具，由于它才诞生了 webdriver，同时在 2.0 中，两者进行了融合。

selenium 1.0 通过向浏览器注入 js 脚本来达到自动化测试的目的。而 webdriver 直接借助 browser driver，通过调用浏览器的 native 方法来实现自动化测试。因此更加灵活与强大。

通过各类浏览器的 driver，来实现不同浏览器的自动化测试。

selenim driver： https://docs.seleniumhq.org/projects/webdriver/

历史：https://www.cnblogs.com/fnng/p/7426928.html

### Selenium-Webdriver

Selenium-Webdriver 支持多种语言编写，由于 nodejs 的发展，webdriver 自然也支持 js 来编写。

#### 安装与配置

0、建个测试目录-。-

1、首先安装 selenium-webdriver 

```
npm install selenium-webdriver
```

2、下载对应的浏览器 driver。

可以在这里找到支持的 driver： https://docs.seleniumhq.org/download/ ;
往下找到 Third Party Drivers, Bindings, and Plugins。
先下载最新版 chrome driver 来测试一下。

不同版本的 driver 与本地所安装的 chrome 版本有关，注意查看版本支持情况。（最新版的chromedriver v2.40； Supports Chrome v66-68）

为了测试方便，将下载下来的 driver 放到当前测试目录中，不然就得去配置 PATH。

#### 开始编写

javascript api doc：http://seleniumhq.github.io/selenium/docs/api/javascript/index.html

随便找个目标，比如：我要验证 http://1688.com 页面上，`请登陆`的按钮是否可以正常跳转，跳转后能否登陆。

1. 简单的开始，打开 http://1688.com

```javascript
const {Builder, By, Key, until} = require('selenium-webdriver');
(async function example() {
  let driver = await new Builder() // 通过 Builder 新建一个 WebDriver 实例
  .forBrowser('chrome') // 指定使用 chrome driver，可能的值可以看文档
  .build();
  await driver.get('https://www.1688.com/'); // get 方法可以打开一个指定的 uri
})();
```

运行脚本会发现自动打开了一个chrome窗口，并且提示 `Chrome 正受到自动测试软件的控制`。
![image](https://user-images.githubusercontent.com/2213424/43130300-fe35ca74-8f68-11e8-963d-25b269242786.png)

2. 接下来要选取 `请登陆` 这个 `a` 标签并点击

我们可以先通过chrome devtools 来确定我们要选取的元素。

![image](https://user-images.githubusercontent.com/2213424/43130755-f7348aac-8f69-11e8-8e47-ee56f12f8b85.png)

看上去 `.account-signin a` 就能选到。不过 webdriver 中的元素选择并没有 CSS 那么强大。因此需要逐级选择。

通过 By 这个类来实现对元素的定位，通过 webElement 类来实现寻找元素以及一些操作。

```javascript
const {Builder, By, Key, until} = require('selenium-webdriver');
(async function example() {
  let driver = await new Builder().forBrowser('chrome').build();
  await driver.get('https://www.1688.com/');
  await driver.findElement(By.className('account-signin')).findElement(By.tagName('a')).click(); // 先选中.account-signin, 再选中 a。然后点击
})();
```
如果你运行上面的脚本，很不幸，你可能会收到
```
UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): WebDriverError: unknown error: Element <a rel="nofollow" href="https://login.1688.com/member/signin.htm?Done=https%3A%2F%2Fwww.1688.com%2F" title="..." data-spm="d3" data-trace="cn_alibar_login" target="_self">请登录</a> is not clickable at point (217, 14). Other element would receive the click: <div id="j-identity" class="home-identity-wrapper" data-spm="201711080439001" data-sourcename="homeidentitydialog" style="display: block;">...</div>
```

它告诉你，你的点击点到了其他元素上。这是因为，目前1688访问会出现弹窗，全屏遮罩盖住了按钮。

![image](https://user-images.githubusercontent.com/2213424/43131633-1385f388-8f6c-11e8-90cc-cc0bb23264b0.png)

所以，我们点击前得先关闭这个弹窗。

重复上面的步骤，找到关闭按钮并点击。但是，为了防止某天这个弹窗下线后会导致的元素定位失败而发生的错误，需要先对弹窗的存在与否进行判断。

```javascript
const {Builder, By, Key, until} = require('selenium-webdriver');
(async function example() {
  let driver = await new Builder().forBrowser('chrome').build();
  await driver.get('https://www.1688.com/');
  if (await driver.findElement(By.id('j-identity')).isDisplayed()) { // 检查下弹窗是否可见 
      await driver.findElement(By.className('identity-close-icon')).click(); // 选中 X 按钮并点击
  }
  await driver.findElement(By.className('account-signin')).findElement(By.tagName('a')).click();
})();
```
OK，再次运行，页面已经正常跳转到登陆页面。接下来就是模拟登陆。

3. 模拟登陆

模拟登陆的操作与前面的基本一致，我们需要的是输入账号与密码的登陆框，因此我们需要判断当前是二维码登陆还是密码登陆，然后输入账号密码，并点击登陆或按回车。

然而，不幸的是，登陆框是用 iframe 包裹的，所以，这里有一个操作，需要将当前 driver 切换到 frame 中！

```javascript
const {Builder, By, Key, until} = require('selenium-webdriver');
(async function example() {
  let driver = await new Builder().forBrowser('chrome').build();
  await driver.get('https://www.1688.com/');
  if (await driver.findElement(By.id('j-identity')).isDisplayed()) {
      await driver.findElement(By.className('identity-close-icon')).click();
  }
  await driver.findElement(By.className('account-signin')).findElement(By.tagName('a')).click();
  
  await driver.switchTo().frame(0); // switchTo 会获得一个TargetLocator实例，再通过 frame 定位iframe，具体参数参看文档

  // 切换登陆框的判读与上面类同
  if (await driver.findElement(By.id('J_QRCodeLogin')).isDisplayed()) {
      await driver.findElement(By.id('J_Quick2Static')).click();
  }
  
  await driver.findElement(By.id('TPL_username_1')).sendKeys('your account'); // 找到账号输入框，输入账号
  await driver.findElement(By.id('TPL_password_1')).sendKeys('your password'); // 找到密码输入框，输入密码
  await driver.findElement(By.id('J_SubmitStatic')).click(); // 找到登陆按钮，点击
  
})();
```
一切正常的话，整个测试就完成了，最后给代码加上 catch 即可。
  

### 记一笔
目前的开发体系下，几乎所有的代码都在前端应用中，而且开发时往往在本地就能模拟出一个真实的环境，自动化测试在本地即可提前跑一遍确认，可以省去出错后不停上下环境打包的时间。 开发新功能的时候，顺带就可以把自动化写掉了。~~写功能还来不及，还写自动化？！~~
  
  
