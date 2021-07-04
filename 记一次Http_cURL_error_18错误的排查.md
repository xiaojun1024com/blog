# 记一次Http  cURL error 18错误的排查

### 现象

有一个合作方提供的服务接口，我们每天会定时的调用接口读取数据。开始几个月一切正常，后来发现一个月中偶尔会有几天获取不到数据，接口异常；程序重试那几天的请求，依然是错误，无法获取到数据。

### 排查

查看日志发现，请求对方接口时curl出错，提示：

> GuzzleHttp\Exception\RequestException
> cURL error 18: transfer closed with 397918 bytes remaining to read

于是断定是对方服务出错了，告知对方要求修复故障。对方排查后回复：接口可以正常访问，没有问题，并附上了Postman请求接口获取到了数据的截图。

一阵诧异：明明curl都出错了，怎么可能请求正常，难道是网络问题？打开Postman，测试了一下，果真数据正常返回了。那为什么php curl 读取接口失败呢？

细读了一遍错误日志`cURL error 18: transfer closed with 397918 bytes remaining to read `，以及查看Postman请求数据发现：Postman中`Http Response`得到的数据只有`40kb`，而程序错误日志中却是`397918 bytes`。进一步观察发现，所有获取不到数据的日期，都是因为那些天数据量比较大，需要读取数据还没完成，http连接就被断开了。

可为什么Postman可以正常获取呢，原来Postman Http Request时默认使用了`Accept-Encoding: gzip, deflate, br`，使得服务端会对数据进行压缩传输，传输数据在40kb左右；而php程序中curl header未设置 `Accept-Encoding: gzip, deflate, br`，传输数据在400kb左右，数据大、传输比较慢，数据还没完成，http连接就被断开了，从而接口读取数据异常。

### 解决

定位到问题了，那就好处理了，在实例化`GuzzleHttp\Client`时，加上`gzip,deflate`即可。代码如下：

```php
$httpClient = new \GuzzleHttp\Client(['headers' => ['Accept-Encoding' => 'gzip,deflate']]);
```

### 总结

`Accept-Encoding`表示Http响应是否进行压缩，一般的浏览器(Postman也是一样的)在访问网页时，是默认在请求头中加入
`Accept-Encoding: gzip, deflate` ，表示这个请求的内容希望被压缩，压缩的目的是为了减少网络流量，
但是这个只是协议，只能是要求而不是强制的，如果服务器不支持压缩或者没有开启压缩,则不能起到作用，
如果服务器也是支持压缩或者开启压缩，则会在响应头中加入Content-Encoding: gzip 头部。

php curl  `GuzzleHttp\Client` 中，默认没有`Accept-Encoding: gzip, deflate`，如果需要gzip传输，需要单独加上，服务器才会支持gzip。



#### 参考：

[Accept-Encoding]: https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Encodin
[Content-Length]: https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Length

