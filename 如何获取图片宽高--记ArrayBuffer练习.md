
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
> http://jsfiddle.net/g4yjpx79/

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
> http://jsfiddle.net/pjLk7xnu

#### 不借助 img 元素
上面都是借助了浏览器的 img 元素来获取宽高，但如果我们不在浏览器环境中呢，比如在 nodejs 中

***下面只是为了练习 ArrayBuffer 的操作，因此依旧在浏览器中，但原理是一样的***

##### 从图片文件结构中获取宽高

不同格式的图片它所存储的结构不同，因此，若要获取不同格式图片的宽高，不得不逐一分析它们的结构，但不同后缀的图片格式也可能属于同一种图片结构，比如jpg，jpeg，jpe都属于jpeg。

同时，由于文件名并不包含在文件结构中，所以一个叫 test.png 的图片它可能是 jpeg 格式的图片。因此在分析一个图片前还要确定它是否属于某个格式。

###### Magic Number
或叫 file signature，它是一段用来表示该二进制文件是何种结构的标记

一些常见的 (signature)[https://en.wikipedia.org/wiki/List_of_file_signatures]

比如 PNG 的文件开头必定是 `0x89 0x50 0x4E 0x47 0x0D 0x0A 0x1A 0x0A`，在解析PNG图片前，可以先校验这个8位来确定接下来的格式的确是PNG的存储结构。

###### PNG

从PNG的规格文件来看，PNG这种文件结构相对简单，它由8字节的signature和一系列的`Chunk`构成。 https://www.w3.org/TR/PNG/#11Chunks

`Chunk` 的结构也比较清晰，4字节的dataLength + 4字节的chunkType + 变长Data + 4字节的CRC 即构成一个 `Chunk`，如果length为0，则 Data 不存在。

![image](https://user-images.githubusercontent.com/2213424/42680863-c943ab82-86b8-11e8-9663-dafdba5bac13.png)

> A valid PNG datastream shall begin with a PNG signature, immediately followed by an IHDR chunk, then one or more IDAT chunks, and shall end with an IEND chunk. Only one IHDR chunk and one IEND chunk are allowed in a PNG datastream.

而 PNG 的宽高信息则存储在 `IHDR Chunk` 中， `IHDR Chunk` 紧跟在signature后，因此我们可以非常方便地获得 `IDHR Chunk`：
```javascript
    const CHUNKLENGTHBYTES = 4; // chunk 数据长度为4字节
    const TYPEBYTES = 4; // 4字节表示 chunk 类型
    const CRCBYTES = 4; // 4字节 CRC校验
    
    const chunkDataLegth = new DataView(fileBuffer).getUint32(8); // 8 为8位signature偏移
    
    const IDHRChunkBuffer = fileBuffer.slice(8, 8 + CHUNKLENGTHBYTES + TYPEBYTES + chunkDataLegth + CRCBYTES); // 8 为8位signature偏移
```

根据 `IDHR Chunk` 的字节序，可以得知数据字节的前4位表示宽，紧接着4位表示高

![image](https://user-images.githubusercontent.com/2213424/42755259-04586b18-892a-11e8-9efc-fefd1fc5fd05.png)

因此也很容易获得 width 与 height

```javascript
    const dv = new DataView(IDHRChunkBuffer);
    const width = dv.getUint32(4 + 4); // 4位length + 4位chunk type
    const height = dv.getUnit32(4 + 4 + 4); // 4位length + 4位chunk type + 4位width
```
> http://jsfiddle.net/uf8aby2n/


###### JPEG

JPEG 的格式比 PNG 复杂很多，首先它并没有像 PNG 那样清晰的 signature，虽然 wiki 上指出 JEPG 的 magic number 应为 `ff d8 ff`, 但实际上，根据 JPEG 但结构描述，JPEG 由一系列的片段构成，每个片段以一个 `marker` 开头，而 `marker` 以 0xFF 开头，紧接着一字节表示该 `marker` 的类型。根据 `marker` 类型的不同，后续的数据也不同，如果该 `marker` 包含数据，则接下去的两字节代表该片段所含数据的长度；如果不包含数据，则该片段结束。所以在读取每一个片段时都需判断它是否以 0xFF 开头，是否该文件的确是以JPEG结构存储的。

> A JPEG image consists of a sequence of segments, each beginning with a marker, each of which begins with a 0xFF byte followed by a byte indicating what kind of marker it is. Some markers consist of just those two bytes; others are followed by two bytes (high then low) indicating the length of marker-specific payload data that follows. (The length includes the two bytes for the length, but not the two bytes for the marker.)

wiki 上所示的 marker 类型 
![image](https://user-images.githubusercontent.com/2213424/42756605-76bdb50a-892f-11e8-9f71-081bd2c0d12a.png)

> https://en.wikipedia.org/wiki/JPEG#Syntax_and_structure

从上表中可知，我们要查询的宽高包含在 `SOF0` 与 `SOF2` 两个类型的 `marker` 中。同时我们在搜索这两个 `mraker` 时，要注意有些 `marker` 是不包含数据的，跳过的时候要注意长度计算。

```javascript
    const dv = new DataView(fileBuffer);
    let offset = 2; // 跳过 SOI

    while (offset < fileBuffer.byteLength) {

        const m1 = dv.getUint8(offset);
        if (m1 !== 0xFF) { // marker 一定以 0xFF 开头
            throw 'this is not a JPEG image';
        }
        
        const m2 = dv.getUint8(offset + 1);
        if (m2 === 0xC2 || m2 === 0xC0) { // 目标 marker 的类型
            return fileBuffer.slice(offset, dv.getUint16(offset + 2) + offset + 2);
        } else {
            if (208 <= m2 && m2 <= 215) { // RSTn 不包含数据，只需跳过 2 字节
                offset += 2;
            } else {
                offset += dv.getUint16(offset + 2) + 2;
            }
        }
    }
    
    throw 'can`t find sof0 or sof2'
```

获得了 `SOF0` 或者 `SOF2` 之后，一直没具体查到，这个两块内部的结构是怎么样的... 直到看到爆栈上的这个 C 的回答

https://stackoverflow.com/questions/18264357/how-to-get-the-width-height-of-jpeg-file-without-using-library

![image](https://user-images.githubusercontent.com/2213424/42757314-248be650-8932-11e8-8242-6a52b76162f7.png)

数据偏移量为3，只有两字节来表示宽高

```javascript
    const width = new DataView(sof02Buffer).getUint16(7);
    const height = new DataView(sof02Buffer).getUint16(5);
```

> http://jsfiddle.net/n48retxv/


##### 最后
虽然浏览器提供了 TypedArray 的视图类型，但是在实际操作一些流的时候，感觉还是 DataView 视图来得更方便。毕竟对于大多数文件结构来讲，各种长度类型的数据都可能存在。

##### 资料

https://en.wikipedia.org/wiki/Magic_number_(programming)

https://en.wikipedia.org/wiki/List_of_file_signatures

https://en.wikipedia.org/wiki/Portable_Network_Graphics

https://www.w3.org/TR/PNG/

https://en.wikipedia.org/wiki/JPEG

https://stackoverflow.com/questions/18264357/how-to-get-the-width-height-of-jpeg-file-without-using-library





