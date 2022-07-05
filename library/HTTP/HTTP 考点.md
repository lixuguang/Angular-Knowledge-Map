# `HTTP` 考点

## `URI`
`URI` ，是 `uniform resource identifier` ，统一资源标识符，用来唯一的标识一个资源。

`Web` 上可用的每种资源如 `HTML` 文档、图像、视频片段、程序等都是一个来 `URI` 来定位的
`URI` 一般由三部组成：
- ①访问资源的命名机制
- ②存放资源的主机名
- ③资源自身的名称，由路径表示，着重强调于资源。

`URL` 是 `uniform resource locator` ，统一资源定位器，它是一种具体的 `URI` ，即 `URL` 可以用来标识一个资源，而且还指明了如何 `locate` 这个资源。

## `MIME`
`MIME` (Multipurpose Internet Mail Extensions) 是描述消息内容类型的标准，用来表示文档、文件或字节流的性质和格式。
`MIME` 的组成结构非常简单，由类型与子类型两个字符串中间用 `/` 分隔而组成，**不允许有空格**。 `type` 表示可以被分多个子类的独立类别， `subtype` 表示细分后的每个类型。

`MIME` 类型**对大小写不敏感**，但是传统写法都是**小写**。

> 两种主要的 `MIME` 类型在默认类型中扮演了重要的角色：
> `text/plain` 表示文本文件的默认值。
> `application/octet-stream` 表示所有其他情况的默认值。

### 常见的 `MIME` 类型
- 超文本标记语言文本 `.html`、`.html`：`text/html`
- 普通文本 .txt： `text/plain`
- RTF 文本 .rtf： `application/rtf`
- GIF 图形 .gif： `image/gif`
- JPEG 图形 `.jpeg`、`.jpg`： `image/jpeg`
- au 声音文件 .au： `audio/basic`
- MIDI 音乐文件 mid、.midi： `audio/midi`、`audio/x-midi`
- RealAudio 音乐文件 .ra、.ram： `audio/x-pn-realaudio`
- MPEG 文件 .mpg、.mpeg： `video/mpeg`
- AVI 文件 .avi： `video/x-msvideo`
- GZIP 文件 .gz： `application/x-gzip`
- TAR 文件 .tar： `application/x-tar`

## `HTTP` 请求和响应报文
`http` 协议是一个**应用层**协议，其报文分为 `请求报文` 和 `响应报文` 

`http` 报文结构为：

- 起始行: 对报文进行描述
- 头部 : 向报文中添加了一些附加信息，是一个名/值的列表，头部和协议配合工作，共同决定了客户端和服务器能做什么事情, 例如： `Content-Length`（主体长度）， `Content-Type`（主体类型）等。
- 主体 : 包含数据的主体部分

### 请求报文
#### 起始行
在请求报文中，起始行包括了3个部分：

- 请求的方法（ POST ）
- 请求的URL( /cgi-bin/qqshow_user_props_info )
- 协议类型及版本( HTTP/1.1 )

#### 头部
- `Client-IP` ：提供了运行客户端的机器的IP地址
- `From` ：提供了客户端用户的E-mail地址
- `Host` ：给出了接收请求的服务器的主机名和端口号
- `Referer` ：提供了包含当前请求URI的文档的URL
- `UA-Color` ：提供了与客户端显示器的显示颜色有关的信息
- `UA-CPU` ：给出了客户端CPU的类型或制造商
- `UA-OS` ：给出了运行在客户端机器上的操作系统名称及版本
- `User-Agent` ：将发起请求的应用程序名称告知服务器       
- `Accept` ：告诉服务器能够发送哪些媒体类型
- `Accept-Charset` ：告诉服务器能够发送哪些字符集
- `Accept-Encoding` ：告诉服务器能够发送哪些编码方式
- `Accept-Language` ：告诉服务器能够发送哪些语言
- `TE` ：告诉服务器可以使用那些扩展传输编码
- `Expect` ：允许客户端列出某请求所要求的服务器行为
- `Range` ：如果服务器支持范围请求，就请求资源的指定范围
- `Cookie` ：客户端用它向服务器传送数据
- `Cookie2` ：用来说明请求端支持的cookie版本

### 响应报文
#### 起始行
在响应报文中，起始行包括了3个部分：

- 协议类型及版本 ( HTTP/1.1 ）
- 状态码( 200 )
- 状态码的文字描述( OK )

#### 头部
- `Age` ： (从最初创建开始)响应持续时间
- `Public` ： 服务器为其资源支持的请求方法列表
- `Retry-After` ： 如果资源不可用的话，在此日期或时间重试
- `Server` ： 服务器应用程序软件的名称和版本
- `Title` ： 对HTML文档来说，就是HTML文档的源端给出的标题
- `Warning` ： 比原因短语更详细一些的警告报文
- `Accept-Ranges` ： 对此资源来说，服务器可接受的范围类型
- `Vary` ： 服务器会根据这些首部的内容挑选出最适合的资源版本发送给客户端
- `Proxy-Authenticate` ： 来自代理的对客户端的质询列表
- `Set-Cookie` ： 在客户端设置数据，以便服务器对客户端进行标识
- `Set-Cookie2` ： 与Set-Cookie类似
- `WWW-Authenticate` ： 来自服务器的对客户端的质询列表

## HTTP 请求方法和状态码
### 状态码
状态码有**三位**数字组成，第一个数字定义了响应的类别，共分**五**种类别:

- 1xx：指示信息--表示请求已接收，继续处理
- 2xx：成功--表示请求已被成功接收、理解、接受
- 3xx：重定向--要完成请求必须进行更进一步的操作
- 4xx：客户端错误--请求有语法错误或请求无法实现
- 5xx：服务器端错误--服务器未能实现合法的请求

### HTTP请求方法
根据 `HTTP` 标准， `HTTP` 请求可以使用多种请求方法。
- `HTTP1.0` 定义了三种请求方法： `GET` , `POST` 和 `HEAD` 方法。
- `HTTP1.1` 新增了五种请求方法： `OPTIONS` , `PUT` , `DELETE` , `TRACE` 和 `CONNECT` 方法。

- `HEAD`    类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头- `HEAD`    类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头
- `GET`     请求指定的页面信息，并返回实体主体。
- `POST`    向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。
- `PUT`     从客户端向服务器传送的数据取代指定的文档的内容。
- `DELETE`  请求服务器删除指定的页面。
- `CONNECT` HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。 OPTIONS 允许客户端查看服务器的性能。 
- `TRACE` 回显服务器收到的请求，主要用于测试或诊断。

## XHR 上传和下载数据
`XMLHttpRequest` （ 简称 `xhr` ），是**浏览器**提供的 `javascript` 对象，主要用于在后台与服务器交换数据。简言之，通过该对象，可以请求服务器上的数据资源。而 `jQuery` 框架中的 `Ajax` 就是基与 `xhr` 对象的封装。

**拆分解释：**

`XML` ：是可扩展标记语言，是各种应用程序之间进行数据传输的常用工具，不过现在常用的 `json` 对象
`Http` ：是超文本传输协议，是一个简单的请求-响应协议，它指定了客户端可能发送给服务器什么样的消息以及得到什么样的响应
`Request` ：译为请求，即服务器接口中的 `Request` （简称req）对象，是客户端向服务器发送的请求

**组合解释：**

通过 `Http` 协议将客户端请求的数据通过 `x-www-form-urlencoded` 格式数据发送给服务器，等待服务器响应数据。

### 作用
`XMLHttpRequest` 对象用于在后台与服务器交换数据

- 在不重新加载页面的情况下更新网页，即网页的局部刷新功能
- 在页面已加载后从服务器请求数据，即可以向服务器请求资源
- 在页面已加载后从服务器接收资源，即可以从服务器获取资源
- 在后台向服务器发送数据，如：将表单中的数据发送给服务器，等待服务器处理并响应

### `XMLHttpRequest` 与 `Ajax` 的关系
- `Ajax` 是一种技术方案，但并不是一种新技术
- `Ajax` 的核心依赖是浏览器提供的 `XMLHttpRequest` 对象

> “使用 XMLHttpRequest 对象发起一个 Ajax 请求”

### `XMLHttpRequest` 的使用
#### 步骤
1. 实例化 XMLHttpRequest 对象
2. 建立一个HTTP请求
  1. 可选择的查看和设定头信息
3. 传递参数
4. 监听服务器
5. 响应处理

```javascript
var xhr = new XMLHttpRequest(); // 1. 实例化 XMLHttpRequest 对象
xhr.open(method, url, async, username, password) // 2. 建立一个HTTP请求
xhr.send(body) // 3. 传递参数 参数body表示将通过该请求发送的数据，如果不传递数据，可以设置为null或者省略
xhr.onreadystatechange = function(){ // 4. 监听服务器
	if( xhr.readyState === 4 && xhr.status === 200 ){
		// 5. 响应处理
		var data = JSON.parse(xhr.responseText)
	}
}
```

##### open参数
|参数	|是否必须	|描述|
|-|-|-|
|method|	是	|HTTP请求方式：GET、POST，大小写不敏感|
|url|	是	|请求的URL地址字符串，大部分浏览器仅支持同源策略|
|asyns|	是	|指定请求是否为异步方式，默认为true。如果为false，当状态改变时会立即调用onreadystatechange属性指定的回调函数|
|username|	否	|如果服务器需要验证，该参数指定用户名，如果未指定，当服务器需要验证时，会弹出验证窗口|
|password|	否	|验证信息中的密码部分，如果用户名为空，则该值将被忽略|

##### readyState状态
|值	|状态	|描述|
|-|-|-|
|0	|UNSENT	|XMLHttpRequest 对象已被创建，但尚未调用 open方法。|
|1	|OPENED	open() |方法已经被调用。|
|2	|HEADERS_RECEIVED	|send() 方法已经被调用，响应头也已经被接收。|
|3	|LOADING	|数据接收中，此时 response 属性中已经包含部分数据。|
|4	|DONE	|Ajax 请求完成，这意味着数据传输已经彻底完成或失败。|

##### 响应信息
|响应信息	|说明|
|-|-|
|responseBody	|将响应信息正文以 Unsigned Byte 数组形式返回|
|responseStream	|以 ADO Stream 对象的形式返回响应信息|
|responseText	|将响应信息作为字符串返回|
|responseXML	|将响应信息格式化为 XML 文档格式返回|

### XMLHttpRequest Level2新特性
#### 旧版 XMLHttpRequest 的缺点
1. 只支持文本数据传输，无法用来读取和上传文件
2. 传送和接收数据时，没有进度信息，只能提示有没有完成
3. 受到"同域限制"（`Same Origin Policy`），只能向同一域名的服务器请求数据。

#### XMLHttpRequest Level2 新功能
##### 可以设置 `HTTP` 请求的时限 
```ts
xhr.timeout = 3000
xhr.ontimeout = function(event){
	alert('timeout')
}
```
##### 可以使用 `FormData` 对象管理表单数据
为了方便**表单处理**，以及**文件上传**， `HTML5` 新增了一个 `FormData` 对象，可以用来快速获取表单中的值，并且可以模拟表单操作。
```ts
// 1. 实例化FormData对象, form为可选参数, 为html表单元素form对象
var form = documen.querySelector('#form1');
var formData = new FormData(form?: HTMLFormElement);

// 2. 向FormData对象添加数据
/* 参数	是否必须	说明
name	是	键值key，即名称
value	是	键值value，即值
Blob	可选	文件名称*/
formData.append(name: string, value: string | Blob, fileName?: string);

// 3. 发送数据给服务器
xhr.send(formData);
```

###### 方法
1. 添加 : append
2. 修改 : set
3. 获取 : get
4. 删除 : delete
5. 取得所有 : getAll
6. 判断是否有 : has

##### 可以上传文件
```ts
var formData = new FormData();
formData.append('file', files[0]); // 画面要先上传文件 <input type='file' id='files'>
var xhr new XMLHttpRequest();
xhr.open('POST','URL');
xhr.setRequestHeader('Content-Type', 'multipart/form-data');
xhr.send(formData);
xhr.upload.onprogress = function(e){
	if(e.lengthComputable){
		var percent = Math.ceil((e.loaded / e.total)*100);
		// 后续是 画面处理
	}
}
xhr.onreadystatechange = function(){
	if( xhr.readyState === 4 && xhr.status === 200 ){
		// 5. 响应处理
		var data = JSON.parse(xhr.responseText)
	}
}
```
##### 可以获取数据传输的进度信息
```ts
xhr.upload.onprogress = function(e){
	if(e.lengthComputable){
		var percent = Math.ceil((e.loaded / e.total)*100);
		// 后续是 画面处理
	}
}
```

## XHR 流式传输

## XHR 定时轮询和长轮询区别与优缺点
### 定时轮询
从服务器检索更新的最简单的策略之一是让客户端进行定期检查：客户端可以以周期性间隔（轮询服务器）启动后台XHR请求，以检查更新。如果新数据在服务器上可用，则在响应中返回，否则响应为空。
```ts
function checkUpdates(url) {
  var xhr = new XMLHttpRequest();
  xhr.open('GET', url);
  xhr.onload = function() { ... }; // 1
  xhr.send();
}

setInterval(function() { checkUpdates('/updates') }, 60000); // 2
```

#### 问题
1. 每个 `XHR` 请求都是一个独立的 `HTTP` 请求，平均来说， `HTTP` 的请求头可能会引起大约 800 字节的开销 (不带HTTP cookie)。
2. 是如果有轮询期间有新的可用消息，客户端是不会马上收到此新消息，而是要等到下一次轮询的时候，才能获取最新数据。
3. 轮询消耗资源

### 长轮询
```ts
function checkUpdates(url) {
  var xhr = new XMLHttpRequest();
  xhr.open('GET', url);
  xhr.onload = function() { // 1
    ...
    checkUpdates('/updates'); // 2
  };
  xhr.send();
}

checkUpdates('/updates'); // 3
```
## HttpClient

## CORS
新版本的 `XMLHttpRequest` 对象，**可以向不同域名的服务器发出 `HTTP` 请求**。这叫做 "跨域资源共享"（Cross-origin resource sharing，简称 `CORS` ）。
```ts
xhr.open('GET', 'http://other.server/and/path/to/script');

```