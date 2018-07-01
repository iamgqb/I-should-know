### XSS - Corss-Site Scripting

分为 存储型XSS、 反射型XSS、DOM-Based XSS；这些都是从利用XSS的角度讲。

危害都是一样的，会导致页面执行不可预期的外部脚本。

防御手段总的来讲是两点：
  1. 在服务端，输出时要记得escape。
  2. 在客户端，使用安全的API，比如慎用innerHTML。

*下面链接有更详细的规则，分别针对服务端的处理与客户端的处理（甚至包含了HTTP响应头*
https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet

https://www.owasp.org/index.php/DOM_based_XSS_Prevention_Cheat_Sheet


### XSRF - Corss-Site Request Forgery

由于 http 请求默认都会带上目标网站的一些验证信息，比如cookie，ip地址等等。这就导致如果在三方(攻击者)网站上进行了某些操作，比如点击链接，或是某些自动执行脚本，从而导致用户在不知情的情况下对另外网站的数据进行了操作。

如果网站本身存在XSS漏洞，那么该网站甚至可以变相成为XSRF攻击的温床。

一般陷入迷惑的地方：

1. 一般用户信息都存在cookie，那么一个加密的或者secure cookie能否防御XSRF呢？

并不行，因为正常情况下所有cookie都会随请求发送，secure cookie只是必须在https下而已。
例外：[sameSite](https://github.com/iamgqb/I-should-know/blob/master/Cookie%20%E4%B8%8E%20Storage.md#cookie) 是一种完全可以防御 XSRF 的方法，但是需要浏览器支持。

2. 那么网站若只接受 POST 请求修改数据，是否可以防御XSRF？

并不行，攻击者依旧可以构造表单的方式，诱导受害者点击。但是GET的确不是一种安全的增删改数据的方式。

3. 上https

是否上https与XSRF并没有关系，不过能上https的网站都应该上https(考虑钱...

一般防御XSRF的方式：

0. 任何情况下都应先保证网站不被 XSS

1. 检查请求头中的 referer 与 origin 字段

为什么是这两个字段，因为这两个字段不能被 js 修改 https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_header_name

2. 由服务端生成一个同步的token，在需要的时候随请求发送

*更多防御 XSRF 的方式*

https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet

*参考，配合goggle信息足够多*

https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)

