title: Write a React Upload Component
date: 2016-01-19 15:15:34
tags:
---

这是一遍学习文档，目的：

- 了解 upload 相关技术细节。
- 了解 IE8/9 的兼容性如何解决，即所谓的渐进增强，优雅降级。

参考文档：

1. [SoAanyip/React-FileUpload](https://github.com/SoAanyip/React-FileUpload)
2. [react-component/upload](https://github.com/react-component/upload)
3. [文件上传的渐进式增强](http://www.ruanyifeng.com/blog/2012/08/file_upload.html)
4. [XMLHttpRequest Level 2 使用指南](http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)

# 上传文件

## 1. 传统形式

使用 Form，用户先选择文件，然后点击"Upload"按钮，文件开始上传。

```html
<form id="upload-form" action="upload.php" method="post" enctype="multipart/form-data" >
　　<input type="file" id="upload" name="upload" /> 
    <br />
　　<input type="submit" value="Upload" />
</form>
```

## 2. iframe上传

Form 上传是同步的，用户需等待上传结束。而且上传成功后，会跳转到 action 所指的页面。

如果要实现异步上传，在 HTML5 之前，需要使用 iframe。当用户点击提交时，动态插入 iframe 元素。

```js
var form = $("#upload-form");
　form.on('submit',function() {
　// 此处动态插入iframe元素
});
```

插入 iframe 的代码如下：

```js
// 随机生成数作为 id
var seed = Math.floor(Math.random() * 1000);
var id = "uploader-frame-" + seed;
var callback = "uploader-cb-" + seed;　　
 // iframe 的 html 代码
var iframe = $('<iframe id="' + id + '" name="' + id + '" style="display:none;">'); 
var url = form.attr('action');
form.attr('target', id).append(iframe).attr('action', url + '?iframe=' + callback);
```

最后一行，

1. 向 Form 添加 target 属性，指向动态插入的 iframe。使得上传结束后，服务器将结果返回 iframe 窗口。当前页面就不会跳转了。
2. 其次在 action 属性指定的上传网址后面添加了一个参数，使得服务器知道回调函数的名称。这样能将服务器返回的信息，从 iframe 返回到上层页面。

服务器返回的信息，格式如下：


```html
<script type="text/javascript">
    window.top.window['callback'](data);
</script>
```

所以，在当前页面定义回调：

```js
window[callback] = function(data) {　　　　
    console.log('received callback:', data);　　　　
    iframe.remove(); //removing iframe
    　　　　
    form.removeAttr('target');　　　　
    form.attr('action', url);　　　　
    window[callback] = undefined; //removing callback
};
```

## 3. ajax 上传

HTML5 提出了 XMLHttpRequest Level 2，使得 ajax 可以上传文件，为异步上传。

上传代码：

```js
form.on('submit', function() {　
    // 此处进行ajax上传
});
```

这里主要使用 FormData 对象，它能够构建类似表单的键值对。

```js
if(window.FormData) {　
　　var formData = new FormData();
　　// 建立一个upload表单项，值为上传的文件
　　formData.append('upload', document.getElementById('upload').files[0]);
　　var xhr = new XMLHttpRequest();
　　xhr.open('POST', $(this).attr('action'));
　　// 定义上传完成后的回调函数
　　xhr.onload = function () {
　　　　if (xhr.status === 200) {
　　　　　　console.log('上传成功');
　　　　} else {
　　　　　　console.log('出错了');
　　　　}
　　};
　　xhr.send(formData);
}
```

## 4. 进度条

XMLHttpRequest Level2 定义了一个 progress 事件，获取上传进度。

```html
<progress id="uploadprogress" min="0" max="100" value="0">0</progress>
```

```js
xhr.upload.onprogress = function (event) {
　　if (event.lengthComputable) {
　　　　var complete = (event.loaded / event.total * 100 | 0);
　　　　var progress = document.getElementById('uploadprogress');
　　　　progress.value = progress.innerHTML = complete;
　　}
};

```

注意，progress事件不是定义在xhr，而是定义在xhr.upload，因为这里需要区分下载和上传，下载也有一个progress事件

关于图片预览暂时不了解。

关于拖放上传暂时不了解。

# XMLHttpRequest Level 2

## 老版本 XMLHttpRequest

我们看下老版的示例代码：

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', 'example.php');
xhr.send();
```

然后监控 XMLHttpRequest 对象的状态变化：

```js
xhr.onreadystatechange = function(){
　　if ( xhr.readyState == 4 && xhr.status == 200 ) {
　　　　alert( xhr.responseText );
　　} else {
　　　　alert( xhr.statusText );
　　}
};
```

- xhr.readyState：XMLHttpRequest对象的状态，等于4表示数据已经接收完毕。
- xhr.status：服务器返回的状态码，等于200表示一切正常。
- xhr.responseText：服务器返回的文本数据
- xhr.responseXML：服务器返回的XML格式的数据
- xhr.statusText：服务器返回的状态文本。

老版本的缺点：

- 只支持文本数据的传送，无法用来读取和上传二进制文件。
- 传送和接收数据时，没有进度信息，只能提示有没有完成。
- 受到"同域限制"（Same Origin Policy），只能向同一域名的服务器请求数据。

## XMLHttpRequest Level 2

- 可以设置HTTP请求的时限。
- 可以使用FormData对象管理表单数据。
- 可以上传文件。
- 可以请求不同域名下的数据（跨域请求）。
- 可以获取服务器端的二进制数据。
- 可以获得数据传输的进度信息。

下面直接用示例代码，实际使用时注意下浏览器兼容性。

### HTTP请求的时限

```js
xhr.timeout = 3000;
xhr.ontimeout = function(event){
　alert('请求超时！');
}
```

### FormData对象

```js
var formData = new FormData();
// 添加表单项
formData.append('username', '张三');
formData.append('id', 123456);
// 直接传送 FormData 对象，和提交网页表单效果一样
xhr.send(formData);
```

也可以直接获取网页表单的值：

```js
var form = document.getElementById('myform');
var formData = new FormData(form);
formData.append('secret', '123456'); // 添加一个表单项
xhr.open('POST', form.action);
xhr.send(formData);
```

### 上传文件

```js
var formData = new FormData();
for (var i = 1; i < files.length;i++) {
　　formData.append('files[]', files[i]);
}
xhr.send(formData);
```

### 跨域资源共享（CORS）

前提是，浏览器必须支持这个功能，而且服务器端必须同意这种"跨域"。

其余写法一样。

### 接收二进制数据

两种方式

#### 老方法：改写MIMEType

即将二进制当做纯文本，再在浏览器端将字节还原成二进制数据。

```js
xhr.overrideMimeType("text/plain; charset=x-user-defined");
var binStr = xhr.responseText;

for (var i = 0, len = binStr.length; i < len; ++i) {
　　var c = binStr.charCodeAt(i);
　　var byte = c & 0xff;
}
```

> 最后一行的位运算"c & 0xff"，表示在每个字符的两个字节之中，只保留后一个字节，将前一个字节扔掉。原因是浏览器解读字符的时候，会把字符自动解读成Unicode的0xF700-0xF7ff区段。

#### responseType属性

使用新增的 responseType 属性。使用到的时候再具体了解。

进度信息上面已经写过，暂时不了解详细用法。

# React 实战

学习的目的是由浅入深，知其所以然，从最简单的开始。

先写一个简单的表单。

```js
import * as React from 'react';
import { Props } from 'react';
import { findDOMNode } from 'react-dom';

export class AjaxUpload extends React.Component<Props<any>, {}> {

  constructor() {
    super();
    this.sendForm = this.sendForm.bind(this);
  }

  private sendForm() {
    const node = findDOMNode(this.refs['file']);
    const formData = new FormData();
    formData.append('file', node);
    const xhr = new XMLHttpRequest();
    xhr.open('POST', 'url');
    xhr.send(formData);
  }

  render() {
    return (
      <form id="form1">
        <input ref="file" type="file"/>
        <input type="button"
               onClick={this.sendForm}
               value="Upload"/>
      </form>
    );
  }

}

```