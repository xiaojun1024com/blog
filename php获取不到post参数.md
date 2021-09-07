在与客户端调试接口时，定义了如下接口：

接口地址：domain/post/update

请求方式：POST

| 参数              | 类型   | 必填 | 示例                                 | 说明                 |
| ----------------- | ------ | ---- | ------------------------------------ | -------------------- |
| id                | string | 是   | 1001 | 要更改的ID   |
| remark            | string | 是   | testcontent              | 标记的内容        |

然后服务端在通过$_POST['id']， $_POST['remark']获取参数时，Android客户端提交上来的参数一直获取不到，iOS客户端提交的参数正常获取。与Android客户端确认，确实是通过post提交的参数。

于是抓包对比：
*Android客户端的请求*
```shell
POST /post/update HTTP/1.1
Host: test.xxx.com
Content-Type: application/json
Cookie: PHPSESSID=mpm4aon96fm2bebc1jqud72g91
Content-Length: 30

{"id":"1001","remark":"testcontent"}
```
*iOS客户端的请求*
```shell
POST /post/update HTTP/1.1
Host: test.xxx.com
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=mpm9uidf96fm2bebc1jqud72g91
Content-Length: 20

id=1001&remark=testcontent
```
对比一下一目了然，虽然Android客户端是通过post提交的参数，但是Content-Type为：application/json，是通过json提交的，这样php通过$_POST 是无法获取到参数的。$_POST只能获取到Content-Type为：application/x-www-form-urlencoded和multipart/form-data 提交上来的数据。Content-Type：application/json提交上来的数据，php可以如下处理：
```php
$postData = file_get_contents('php://input');
$data = json_decode($postData, true);
```

所以在定义接口文档的时候，不能仅仅只定义说是post请求，还要明确定义Content-Type类型，明确请求数据格式。

Content-Type类型的具体介绍：[post提交数据的3种常见方式](https://www.xiaojun1024.com/archives/167 "post提交数据的3种常见方式")
