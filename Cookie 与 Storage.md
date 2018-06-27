### Cookie

#### 设置 cookie
1. 可以通过响应返回 set-cookie 字段进行设置，一个字段只能设置一条 cookie

2. `document.cookie = "key=val;"` 的方式进行设置，可以追加 expires、domain、path 等字段；只有当 key，domain，path 一样时才会认为对同一个 cookie 进行操作，否则认为是新增 cookie。下面三种会产生三个不同的 cookie。

```javascript
document.cookie = "token=xxx; domain=github.com; path=/repo;"
document.cookie = "token=xxx; domain=github.com; path=/repo/my;"
document.cookie = "token=xxx; domain=gist.github.com; path=/repo;"
```

3. 删除一个 cookie 只需要设置该 cookie 的过期时间早于当前时间即可，当然需要注意 domain，path 是你需要删除的那个

```javascript
// 设置过期时间即可
document.cookie = "token=xxx; domain=github.com; path=/repo; expires=Thu, 01 Jan 1970 00:00:00 GMT"
```

#### 特殊字段
上面已经涉及了 domain path expires 

- secure：只有当 https 协议下才会被传输
- httpOnly：该属性只能通过服务端 set-cookie 设置，存在该属性的 cookie；无法通过 document.cookie 进行操作
- sameSite: 一个防止跨站攻击的属性，设置该属性可以防止在跨站的情况下被发送cookie

*sameSite 参考*

http://www.cnblogs.com/ziyunfei/p/5637945.html

https://www.sjoerdlangkemper.nl/2016/04/14/preventing-csrf-with-samesite-cookie-attribute/

#### Storage
...先睡觉
