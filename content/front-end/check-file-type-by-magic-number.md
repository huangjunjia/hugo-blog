---
title: "如何判断当前上传文件的类型"
date: 2023-07-03T15:51:46+08:00
draft: false
---

## 背景

在文件上传的时候，希望能够在浏览器内通过 JavaScript 对用户上传的文件的类型进行校验和限制。

## 当前方案

1. 使用 el-upload 组件的返回的 File 对象，是通过 `input[type="file"]` 文件选择框的方式来读取文件信息

```other
uid: 1648106815740
lastModified: 1647570422827
lastModifiedDate: Fri Mar 18 2022 10:27:02 GMT+0800 (中国标准时间) {}
name: "cfm测试素材.jpeg"
size: 179324
type: "image/jpeg"
webkitRelativePath: ""
```

1. 获取文件名信息，截取后缀，获取文件拓展名

显然通过文件拓展名判断文件类型是一个很不靠谱的方案，当用户手动修改文件拓展名时就能够绕开前端校验。

那是否还有其他方案可以实现呢？首先我们可以了解第一个方案：MIME 类型判断。

## MIME 是什么

[MIME 类型 - HTTP | MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)

[Media Types](https://www.iana.org/assignments/media-types/media-types.xhtml)

在 MDN 中对 MIME 的介绍如下：

> 媒体类型（通常称为 Multipurpose Internet Mail Extensions 或 MIME 类型 ）是一种标准， 用来表示文档、文件或字节流的性质和格式。 MIME类型不是传达文档类型信息的唯一方式： 1. 有时会使用名称后缀，特别是在Microsoft Windows系统上。并非所有的操作系统都认为这些后缀是有意义的 （特别是Linux和Mac OS），并且像外部MIME类型一样，不能保证它们是正确的。 2. 魔术数字。不同类型的文件的语法通过查看结构来允许文件类型推断。 例如，每个GIF文件以47 49 46 38十六进制值[GIF89]或89 50 4E 47 [.PNG]的PNG文件开头。 并非所有类型的文件都有幻数，所以这也不是100％可靠的方式。

## 浏览器获取的 MIME 类型并不可信

当对同一张图片，修改文件名的后缀时，MIME 类型发生变化

1. 获取原图片（图片类型为 jpeg，文件拓展名为 jpeg）的元信息，此时可以看到 type 为 ::image/jpeg::

    ```other
    uid: 1648106815740
    lastModified: 1647570422827
    lastModifiedDate: Fri Mar 18 2022 10:27:02 GMT+0800 (中国标准时间) {}
    name: "cfm测试素材.jpeg"
    size: 179324
    type: "image/jpeg"
    webkitRelativePath: ""
    ```

2. 修改文件拓展名修改为jpg，此时文件元信息没有发生改变

    ```other
    uid: 1648107134019
    lastModified: 1647570422827
    lastModifiedDate: Fri Mar 18 2022 10:27:02 GMT+0800 (中国标准时间) {}
    name: "cfm测试素材jpg.jpg"
    size: 179324
    type: "image/jpeg"
    webkitRelativePath: ""
    ```

3. 将文件拓展名修改为 png 后，文件元信息发生了改变，type 修改为 ::image/png::

    ```other
    uid: 1648107162025
    lastModified: 1647570422827
    lastModifiedDate: Fri Mar 18 2022 10:27:02 GMT+0800 (中国标准时间) {}
    name: "cfm测试素材png.png"
    size: 179324
    type: "image/png"
    webkitRelativePath: ""
    ```

4. 将文件拓展名修改为 gif 后，文件元信息发生了改变，type 修改为 ::image/gif::

    ```other
    uid: 1648107204277
    lastModified: 1647570422827
    lastModifiedDate: Fri Mar 18 2022 10:27:02 GMT+0800 (中国标准时间) {}
    name: "cfm测试素材gif.gif"
    size: 179324
    type: "image/gif"
    webkitRelativePath: ""
    ```

5. 移除文件拓展名，文件元信息发生了改变，type 为 空

    ```other
    uid: 1648107237503
    lastModified: 1647570422827
    lastModifiedDate: Fri Mar 18 2022 10:27:02 GMT+0800 (中国标准时间) {}
    name: "cfm测试素材"
    size: 179324
    type: ""
    webkitRelativePath: ""
    ```

可以发现当文件拓展名修改之后，对应的 MIME 类型也会发生改变，这并不是正确的结果。

在 MDN 文档中对这一个[问题](https://developer.mozilla.org/en-US/docs/Web/API/File/type)进行了描述：

> **Note:** Based on the current implementation, browsers won't actually read the bytestream of a file to determine its media type. It is assumed based on the file extension; a PNG image file renamed to .txt would give "*text/plain*" and not "*image/png*". Moreover, `file.type` is generally reliable only for common file types like images, HTML documents, audio and video. Uncommon file extensions would return an empty string. Client configuration (for instance, the Windows Registry) may result in unexpected values even for common types. **Developers are advised not to rely on this property as a sole validation scheme.**

在浏览器环境中通过 MIME 判断文件类型同样也并不是一个靠谱的方案。

根据 W3C 的规范，浏览器应该按照 MIME 嗅探规范解析文件的 MIME 类型，显而易见的是浏览器并未遵守这个规范。

[File API](https://www.w3.org/TR/FileAPI/#processing-media-types)

[MIME Sniffing Standard](https://mimesniff.spec.whatwg.org/)

## 计算机如何识别文件类型

[Files types/kinds/formats | AP CSP (article) | Khan Academy](https://www.khanacademy.org/computing/computers-and-internet/xcae6f4a7ff015e7d:computers/xcae6f4a7ff015e7d:computer-files/a/file-types-kinds-extensions)

计算机判断文件类型的方式很多，其中一种是文件的元数据的头部找到文件类型。比如 GIF 文件的元数据头部始终是字母 “GIF”，而根据 ACSII 编码转为二进制码后可以得到

```other
0100 0111 0100 1001 0100 0110
```

如果计算在文件元数据的头部发现这一串二进制编码，会认为这个文件是一个 GIF 文件。

上面讲的是计算机识别文件类型的其中一种方法，而在编程世界里，最常用的判断文件类型的方式是通过 Magic number。

[Magic number (programming) - Wikipedia](https://en.wikipedia.org/wiki/Magic_number_%28programming%29)

## Magic number

在计算机编程中，`Magic number` 一词有多种含义。它可以指的是以下一种或多种情况：

- 具有无法解释的意义或多次出现的独特值，可以（最好）用命名的常数来代替
- 用于识别文件格式或协议的恒定数字或文本值；关于文件，见[文件签名列表](https://en.wikipedia.org/wiki/List_of_file_signatures)
- 不太可能被误认为是其他含义的独特数值（例如，[全球唯一标识符](https://en.wikipedia.org/wiki/Universally_unique_identifier)）

[List of file signatures - Wikipedia](https://en.wikipedia.org/wiki/List_of_file_signatures)

[Universally unique identifier - Wikipedia](https://en.wikipedia.org/wiki/Universally_unique_identifier)

## 实现 Magic number 的获取

以 JPEG 文件的字节码举例（前 4 个字节）：

```other
FF D8 FF E0 (SOI + ADD0)
FF D8 FF E1 (SOI + ADD1)
FF D8 FF E2 (SOI + ADD2)
```

在 JS 中我们可以通过 `FileReader` 对象读取文件并通过 `File.readAsArrayBuffer(blob: Blob)` 函数获取字节码：

```javascript
var blob = files[i]; // See step 1 above
var fileReader = new FileReader();
fileReader.onloadend = function(e) {
  var arr = (new Uint8Array(e.target.result)).subarray(0, 4);
  var header = "";
  for(var i = 0; i < arr.length; i++) {
     header += arr[i].toString(16);
  }
  console.log(header);

  // Check the file signature against known types

};
fileReader.readAsArrayBuffer(blob);
```

并判断头部字节码对应的文件类型：

```javascript
switch (header) {
  case "89504e47":
    type = "image/png";
    break;
  case "47494638":
    type = "image/gif";
    break;
  case "ffd8ffe0":
  case "ffd8ffe1":
  case "ffd8ffe2":
  case "ffd8ffe3":
  case "ffd8ffe8":
    type = "image/jpeg";
    break;
  default:
    type = "unknown"; // Or you can use the blob.type as fallback
    break;
}
```

到这里，我们就可以在浏览器中正确的判断文件类型了。

## 特殊情况

并不是所有类型都支持 Magic Number 判断，比如 Json 文件，Json 是一个格式规范而不是一个文件类型，Magic Number 无法保持固定值，所以需要使用其他方式判断，如：

1. 校验文件后缀
2. 校验 Json schema 是否符合预期类型

[GitHub - colinhacks/zod: TypeScript-first schema validation with static type inference](https://github.com/colinhacks/zod)

## Demo

[Parse file type](https://codepen.io/huangjunjia/pen/YzYNbwd?editors=1011)

![Image.png](https://res.craft.do/user/full/4ea2f005-c8b7-91c3-85ec-bfa97264e94d/doc/A44598D2-12EF-4379-8FF9-9F34D6F18F1B/35c3706c-5099-d995-4854-22f3b5a21d9a/aLzslyUoIHUSZJSUcIzaG4GOAxruxwdD83SYzWLLHmgz/Image.png)

[ExpmQQW](https://codepen.io/daisy-zly/pen/ExpmQQW)

<iframe height="300" style="width: 100%;" scrolling="no" title="Parse file type" src="https://codepen.io/huangjunjia/embed/YzYNbwd?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/huangjunjia/pen/YzYNbwd">
  Parse file type</a> by Junjia Huang (<a href="https://codepen.io/huangjunjia">@huangjunjia</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>