说到HTTP请求，首先就想到GET、POST，一般在获取数据的时候用GET，在需要给服务器提交数据的时候使用POST。有过api接口联调经验的同学可能遇到过，客户端明明是post提交的数据，可服务端就是获取不到数据。这里就涉及到post提交数据的方式，如果客户端--服务端没有约定一致的方式来post数据，很有可能就导致服务端接收不到参数。下面就介绍一下post提交数据的正确姿势。

我们知道，HTTP协议是以 ASCII 码传输，建立在 TCP/IP 协议之上的应用层规范。规范把 HTTP 请求分为三个部分：请求行、请求头、消息主体。类似于：

```http
<method> <request-URL> <version>
<headers>

<entity-body>
```

HTTP协议规定 POST 提交的数据必须放在消息主体（entity-body）中，但协议并没有规定数据必须使用什么编码方式。实际上，开发者完全可以自己决定消息主体的格式，只要最后发送的 HTTP 请求满足上面的格式就可以。

但是，数据发送出去，还要服务端解析成功才有意义。一般服务端语言如 php、python 等，都内置了自动解析常见数据格式的功能。服务端通常是根据请求头（headers）中的 Content-Type 字段来获知请求中的消息主体是用何种方式编码，再对主体进行解析。所以说 POST 提交数据的方法，包含了 Content-Type 和消息主体编码方式两部分。下面就正式开始介绍。

### Content-Type: application/x-www-form-urlencoded

这应该是最常见的 POST 提交数据的方式了。浏览器的原生 <form\> 表单，如果不设置 enctype 属性，那么最终就会以 application/x-www-form-urlencoded 方式提交数据。请求类似于下面这样：

```http
POST / HTTP/1.1
Host: foo.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 13

say=Hi&to=Mom
```
首先，Content-Type 被指定为 application/x-www-form-urlencoded；其次，提交的数据按照 say=Hi&to=Mom 的方式进行编码，key 和 val 都进行了 URL 转码。大部分服务端语言都对这种方式有很好的支持。例如 PHP 中，$_POST['say'] 可以获取到 say 的值，$_POST['to'] 可以得到 to 的值。
很多时候，我们用 Ajax 提交数据时，也是使用这种方式。例如 JQuery 和 QWrap 的 Ajax，Content-Type 默认值都是「application/x-www-form-urlencoded」。

### Content-Type:  multipart/form-data
这又是一个常见的 POST 数据提交的方式。我们使用表单上传文件时，必须让 <form\> 表单的 enctype 等于 multipart/form-data。直接来看一个请求示例：

```http
POST http://www.example.com HTTP/1.1
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA

------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="text"

title
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="file"; filename="chrome.png"
Content-Type: image/png

PNG ... content of chrome.png ...
------WebKitFormBoundaryrGKCBY7qhFd3TrwA--
```

这个例子稍微复杂点。首先生成了一个 boundary 用于分割不同的字段，为了避免与正文内容重复，boundary 很长很复杂。然后 Content-Type 里指明了数据是以 multipart/form-data 来编码，本次请求的 boundary 是什么内容。消息主体里按照字段个数又分为多个结构类似的部分，每部分都是以 --boundary 开始，紧接着是内容描述信息，然后是回车，最后是字段具体内容（文本或二进制）。如果传输的是文件，还要包含文件名和文件类型信息。消息主体最后以 --boundary-- 标示结束。这种方式一般用来上传文件，各大服务端语言对它也有着良好的支持。

上面提到的这两种 POST 数据的方式，都是浏览器原生支持的，而且现阶段标准中原生 <form\> 表单也只支持这两种方式（通过 <form\> 元素的 enctype 属性指定，默认为 application/x-www-form-urlencoded。其实 enctype 还支持 text/plain，不过用得非常少）。

###  Content-Type:  application/json
application/json 这个 Content-Type 作为响应头大家肯定不陌生。同样，现在也越来越多的人把它作为请求头，用来告诉服务端消息主体是序列化后的 JSON 字符串。JSON 格式支持比键值对复杂得多的结构化数据，可以比较清晰的展现数据层次结构。android , iOS都有相应的http库，请求的时候注意设置header  Content-Type 为：application/json。

Google 的 AngularJS 中的 Ajax 功能，默认就是提交 JSON 字符串。例如下面这段代码：
```javascript
var data = {'title':'test', 'sub' : [1,2,3]};
$http.post(url, data).success(function(result) {
    ...
});
```

最终发送的请求是：

```http
POST http://www.example.com HTTP/1.1 
Content-Type: application/json;charset=utf-8

{"title":"test","sub":[1,2,3]}
```
使用application/json这种方式，可以方便的提交复杂的结构化数据，特别适合 RESTful 的接口。各大抓包工具如 Chrome 自带的开发者工具、Firebug、Fiddler，都会以树形结构展示 JSON 数据，非常友好。但也有些服务端语言还没有支持这种方式，例如 php 就无法通过 $_POST 对象从上面的请求中获得内容。这时候，需要自己动手处理下：在请求头中 Content-Type 为 application/json 时，从 php://input 里获得原始输入流，再 json_decode 成对象。列如：
```php
$postData = file_get_contents('php://input');
$data = json_decode($postData, true);
```

