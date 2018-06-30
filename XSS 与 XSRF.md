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
