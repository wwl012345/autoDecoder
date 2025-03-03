## 0x01.第一次使用

1. 第一次使用建议使用样例进行加解密流程的https://mp.weixin.qq.com/s/B-lBbVpJsPdCp1pjz2Rxdg

2. 加解密的时候核对一下三个地方的参数:

`加解密的域名` 只有出现该域名的请求包会进行加解密操作(`autoDecoder`选项卡与此参数有关)

`明文关键字` 当请求体内出现该关键字即进行加解密操作(换行符为分隔符，支持多个关键字)

`密文关键字` 当请求体内出现该关键字即不进行加解密操作(换行符为分隔符，支持多个关键字)

<img width="380" alt="image" src="https://user-images.githubusercontent.com/48286013/187074998-90ddce7f-7b65-4721-8803-2c5e82e16295.png">

3. 有代码基础的建议使用接口进行加解密，可以自定义请求的内容，自由替换；工具本身自带加解密满足不了现在大部分应用的加解密需求。



## 0x02.在`repeater`模块出现解密错误问题

进入`Burp Suite`的`logger`模块(或者`logger++`模块)查看当前请求的实际数据包(在`repeater`内的数据包被autoDecoder处理过，所以会提示解密失败)，进而分析是什么原因

1. 可能使用了工具自带的加解密，目前工具仅支持请求包、响应包同时加解密，一旦响应包为非加密的，就会报解密的错误，而使用接口加解密，就可以很好解决此问题
2. 使用了接口加解密，考虑代码编写问题，如函数的入参、函数的返回值等错误



## 0x03.接口脚本参数

参考示例见[flasktestheader.py](https://github.com/f0ng/autoDecoder/blob/main/flasktestheader.py)(处理请求头)与[flasktest.py](https://github.com/f0ng/autoDecoder/blob/main/flasktest.py)(DES加解密示例)脚本

参数解释如下：

### 函数入参为

#### 1.`dataBody` 请求体，传参为flask框架传参，示例：

正常POST格式(application/x-www-form-urlencoded)

`ImmutableMultiDict([('dataBody', 'a=1&b=2&c=3')])`(原始数据为a=1&b=2&c=3)

JSON格式(application/json)

`ImmutableMultiDict([('dataBody', '{"f0ng":"onlysecurity"}')])`(原始数据为{"f0ng":"onlysecurity"})等



#### 2.`dataHeaders `请求头，示例：

```java
ImmutableMultiDict([('dataBody', '{"id":"1"}'), ('dataHeaders', 'POST /0828/testsql.php HTTP/1.1\r\nHost: 10.211.55.4\r\nUser-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:104.0) Gecko/20100101 Firefox/104.0\r\nAccept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8\r\nAccept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2\r\nAccept-Encoding: gzip, deflate\r\nConnection: keep-alive\r\nUpgrade-Insecure-Requests: 1\r\nContent-Type: application/x-www-form-urlencoded\r\nContent-Length: 39\r\n')])
```

原始数据为（数据包的换行分隔符为`\r\n`）

```
POST /0828/testsql.php HTTP/1.1
Host: 10.211.55.4
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:104.0) Gecko/20100101 Firefox/104.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 39

{"id":"1"}
```

获取相应参数的代码如下（flask框架的读取参数方法）：

```
body = request.form.get('dataBody')  # 获取  post 参数 必需
headers = request.form.get('dataHeaders')  # 获取  post 参数  可选
```

#### 3.`requestorresponse`标识为请求包还是响应包(可选为request和response)，示例：
```Java
ImmutableMultiDict([('dataBody', 'dCtLdlmk7wI='), ('requestorresponse', 'response')])
```


### 函数出参为

#### 3.`body`(当未勾选对请求头加密时)

传入的为请求体，传出的为处理后的请求体

```
I9z1fsH5QQ2NUbJi/7a8lw==
```



#### 4.`headers + "\r\n\r\n\r\n\r\n" + body`(当勾选对请求头加密时)

传入的为请求头与请求体，传出的为处理后的请求头与请求体（这里未处理请求体，所以请求体是未处理过的）

这里的`\r\n\r\n\r\n\r\n`是为了区分开headers与body，是必需的

```
POST /0828/testsql.php HTTP/1.1
Host: 10.211.55.4
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:104.0) Gecko/20100101 Firefox/104.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 39
aaaa:bbbb
f0ng:test



{"id":"1"}
```



