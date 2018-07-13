
#### 图片来自远端

一般我们都会在动态请求一张远端图片时会遇到这样的需求，请求回来后会需要进行一些计算与定位，实现方式非常简单，基本大家都写过。

```javascript
const imgEle = document.createElement('img');

imgEle.onload = function(e) {
    console.log(this.width, this.height)
}
imgEle.src = 'https://cbu01.alicdn.com/cms/upload/2016/883/878/2878388_1073447813.png';
document.body.appendChild(imgEle)
```
http://jsfiddle.net/g4yjpx79/

#### 图片来自本地

那如果我们根本就没有远端的图片地址呢，比如从本地上传一张图片

需要借助一个web API， [URL.createObjectURL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL), 可以通过它来创建一个 `File` 或 `Blob` 或 `MediaSource` 的 url 引用。于是就可以转化成上面那种方式

```javascript
const imgEle = document.createElement('img');

imgEle.onload = function(e) {
    console.log(this.width, this.height)
}
imgEle.src = URL.createObjectURL(File);
document.body.appendChild(imgEle)
```
http://jsfiddle.net/pjLk7xnu

