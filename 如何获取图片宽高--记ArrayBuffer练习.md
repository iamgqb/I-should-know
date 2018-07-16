
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

#### 不借助 img 元素
上面都是借助了浏览器的 img 元素来获取宽高，但如果我们不在浏览器环境中呢，比如在 nodejs 中

*下面只是为了练习 ArrayBuffer 的操作，因此依旧在浏览器中，但原理是一样的*

**所有的信息都可以从文件字节中读取得到(文件名，修改时间等除外)**

##### 从图片文件结构中获取宽高

不同格式的图片它所存储的结构不同，因此，若要获取不同格式图片的宽高，不得不逐一分析它们的结构，但不同后缀的图片格式也可能属于同一种图片结构，比如jpg，jpeg，jpe都属于jpeg。

同时，由于文件名并不包含在文件结构中，所以一个叫 test.png 的图片它可能是 jpeg 格式的图片。因此在分析一个图片前还要确定它是否属于某个格式。

###### Magic Number
或叫 file signature，它是一段用来表示该二进制文件是何种结构的标记

一些常见的 (signature)[https://en.wikipedia.org/wiki/List_of_file_signatures]

比如 PNG 的文件开头必定是 0x89 0x50 0x4E 0x47 0x0D 0x0A 0x1A 0x0A

###### PNG

从PNG的规格文件来看，PNG这种文件结构相对简单，它由8字节的signature和一系列的`Chunk`构成。

`Chunk` 的结构也比较清晰，4字节的dataLength + 4字节的chunkType + 变长Data + 4字节的CRC 即构成一个 `Chunk`，如果length为0，则 Data 不存在。

![image](https://user-images.githubusercontent.com/2213424/42680863-c943ab82-86b8-11e8-9663-dafdba5bac13.png)

而 PNG 的宽高信息则存储在 `IHDR Chunk` 中， `IHDR Chunk` 













