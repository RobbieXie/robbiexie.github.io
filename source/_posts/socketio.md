---
title: socketio
date: 2022-05-18 21:52:41
tags: socketio
---

## EngineIO Protocol

[**Engine.IO**](https://github.com/socketio/engine.io-protocol)是[**Socket.IO**](https://github.com/socketio/socket.io)更底层的实现基础，要想完整理解Socket.IO必须对Engine.IO协议也有深刻认知。

### Engine.IO session详解

1. 传输层与Engine.IO URL建立一个连接。

2. 服务端返回一个JSON格式的

   ```
   OPEN
   ```

   握手数据，包括如下内容：

   - `sid` session id（字符串）
   - `upgrades` 可能有传输升级（字符串数组）
   - `pingTimeout` 服务端需配置ping超时时间，用于客户端判断服务端是否不可达。（数字）
   - `pingInterval` 服务端需配置ping间隔时间，用于客户端判断服务端是否不可达。（数字）

3. 客户端必须通过`pong`周期性回复服务端的`ping`。

4. 客户端和服务端可以在任意时刻交换`message`。

5. Polling方式的传输允许发送`close`来关闭socket，因为预期中会一直处于"opening"和"closing"的状态。

#### 会话示例

- Request n°1 (open packet)

```
GET /engine.io/?EIO=4&transport=polling&t=N8hyd6w
< HTTP/1.1 200 OK
< Content-Type: text/plain; charset=UTF-8
0{"sid":"lv_VI97HAXpY6yYWAAAC","upgrades":["websocket"],"pingInterval":25000,"pingTimeout":5000}
```

细节：

```
0           => "open" packet type
{"sid":...  => the handshake data
```

注意：参数`t`用于防止浏览器缓存该请求。

- Request n°2 (message in)

服务端会执行`socket.send('hey')`：

```
GET /engine.io/?EIO=4&transport=polling&t=N8hyd7H&sid=lv_VI97HAXpY6yYWAAAC
< HTTP/1.1 200 OK
< Content-Type: text/plain; charset=UTF-8
4hey
```

细节：

```
4           => "message" packet type
hey         => the actual message
```

注意：`sid`参数包括握手数据中的sid。

- Request n°3 (message out)

客户端将会执行`socket.send('hello'); socket.send('world');`：

```
POST /engine.io/?EIO=4&transport=polling&t=N8hzxke&sid=lv_VI97HAXpY6yYWAAAC
> Content-Type: text/plain; charset=UTF-8
4hello\x1e4world
< HTTP/1.1 200 OK
< Content-Type: text/plain; charset=UTF-8
ok
```

细节：

```
4           => "message" packet type
hello       => the 1st message
\x1e        => separator
4           => "message" message type
world       => the 2nd message
```

- Request n°4 (WebSocket upgrade)

```
GET /engine.io/?EIO=4&transport=websocket&sid=lv_VI97HAXpY6yYWAAAC
< HTTP/1.1 101 Switching Protocols
```

Websocket数据帧：

```
< 2probe    => probe request
> 3probe    => probe response
< 5         => "upgrade" packet type
> 4hello    => message (not concatenated)
> 4world
> 2         => "ping" packet type
< 3         => "pong" packet type
> 1         => "close" packet type
```

#### Websocket 会话示例

这种情况下，客户端只启用Websocket（没有HTTP polling）。

```
GET /engine.io/?EIO=4&transport=websocket
< HTTP/1.1 101 Switching Protocols
```

Websocket数据帧：

```
< 0{"sid":"lv_VI97HAXpY6yYWAAAC","pingInterval":25000,"pingTimeout":5000} => handshake
< 4hey
> 4hello    => message (not concatenated)
> 4world
< 2         => "ping" packet type
> 3         => "pong" packet type
> 1         => "close" packet type
```

### 路由

Engine.IO路由组成如下：

```
/engine.io/[?<query string>]
```

- `engine.io`的路径名仅可以被基于engine.io上层协议的框架修改
- 查询字符串是可选的，有6个保留字：
  - `transport`：表明传输方式。默认支持`polling`，`websocket`。
  - `j`：如果传输方式是`polling`且返回值必须是JSONP类型，则`j`必须设置为JSONP返回值的index。
  - `sid`：如果客户端要加session id，必须放在查询字符串上。
  - `EIO`：Engine.IO的版本号。
  - `t`：时间戳的hash值，用来防止浏览器缓存。

FAQ：`/engine.io`这部分路由是否可修改？ 可以，服务端在不同的子路径下拦截请求。

FAQ：是什么因素决定一个选型出现在编码了的查询字符串上？换句话说，为什么`transport`字段不在路由上面呢？ 约定是子路径仅用于让Engine.IO的服务端消除是否要处理这个请求的歧义。这当然只有Engine.IO的前缀（`/engine.io`）。

### 编码方式

有两种不同的编码方式：

- packet
- Payload

#### Packet

一个编码的packet可以是UTF-8的字符串，也可以是二进制数据。字符串形式的编码packet如下：

```
<packet type id>[<data>]
```

例如：

```
4hello
```

对于二进制数据来说，无需包含packet类型，因为只有`message`可以包括二进制数据。

##### 0 open

当新的传输通道打开会从服务端发送。

##### 1 close

请求关闭此传输通道但不会立刻关闭连接。

##### 2 ping

由服务端发送，客户端回复pong包。

样例：

- server sends: `2`
- client sends: `3`

##### 3 pong

客户端发送用于回复ping包。

##### 4 message

消息体，客户端和服务端可以通过它们的回调接口传输数据

样例一：

- 服务端发送：`4HelloWorld`
- 客户端接收并触发回调：`socket.on('message', function (data) { console.log(data); });`

样例二：

- 客户端发送：`4HelloWorld`
- 服务端接收并触发回调：`socket.on('message', function (data) { console.log(data); });`

##### 5 upgrade

在engine.io切换传输方式之前，它需要测试服务端和客户端之间是否允许这种传输方式。如果测试成功，客户端将会发送一个upgrade包给服务端并要求它刷新旧传输方式的缓存并切换到新的传输方式。

##### 6 noop

noop包主要用于当一个websocket连接将要建立时候强制一轮poll循环。

样例：

1. 客户端通过新的传输方式连接
2. 客户端发送：`2probe`
3. 服务端接收并发送：`3probe`
4. 客户端接收并发送：`5`
5. 服务端刷新环境，关闭旧传输方式并切换至新的传输方式

#### Payload

Payload是一系列packet捆绑在一起。当仅支持字符串且XHR2不支持的情况下，payload编码格式如下：

```
<packet1>\x1e<packet2>\x1e<packet3>
```

packet通过('\x1e')分割，更多信息请参考： https://en.wikipedia.org/wiki/C0_and_C1_control_codes#Field_separators

当payload支持二进制数据时，它会通过发送base64编码的字符串。为了便于解码，二进制格式的packet前会插入字符`b`。任意数量的字符串和base64编码的字符串可以被聚合发送。下面是base64编码消息的样例：

```
<packet1>\x1eb<packet2 data in b64>[...]
```

payload用于不支持帧方式的传输，比如polling协议。

- 无二进制数据样例：

```json
[
  {
    "type": "message",
    "data": "hello"
  },
  {
    "type": "message",
    "data": "€"
  }
]
```

被编码为：

```
4hello\x1e4€
```

- 二进制数据样例：

```json
[
  {
    "type": "message",
    "data": "€"
  },
  {
    "type": "message",
    "data": buffer <01 02 03 04>
  }
]
```

被编码为：

```
4€\x1ebAQIDBA==

with

4           => "message" packet type
€
\x1e        => record separator
b           => indicates a base64 packet
AQIDBA==    => buffer content encoded in base64
```

### 传输方式

Engine.IO的服务端必须支持三种传输方式：

- websocket
- server-sent events (SSE)
- Polling
  - Jsonp
  - Xhr

#### Polling

polling方式包括客户端循环GET请求访问服务端获取数据，以及客户端POST请求传输数据给服务端。

##### XHR

服务端必须支持跨域。

##### JSONP

服务端的实现必须回复合法的JavaScript。URL上面必须包含查询字符串参数`j`，并必须被应用在返回体中。`j`是一个整数：

JSONP格式：

```
`___eio[` <j> `]("` <encoded payload> `");`
```

为了确保payload被正确处理，它的转义也必须符合合法的JavaScript。使用一个JSON编码器是一个发送编码payload的良好转义方法。

如下是服务端返回的JSONP样例：

```
___eio[4]("packet data");
```

**上传数据**

客户端通过一个隐藏的`iframe`上传数据。到服务端的数据在URI中的编码格式如下：

```
d=<escaped packet payload>
```

此外根据标准转义规范，为了防止浏览器对`\n`处理的不一致，`\n`会在POST中会转义为`\\n`。

#### Server-sent events

客户端使用一个EventSource的对象用于接收数据，使用一个XMLHttpRequest的对象用于发送数据。

#### Websocket

payload编码不适用websocket，因为协议本身已经具有一个轻量级的数据帧机制。

发送payload类型的消息，只需要单独编码并连续调用`send()`发送。

### 传输升级

一个连接总是要从polling方式开始（XHR或者JSONP）。websocket要通过发送一个探测开始。加入服务端对探测有返回结果，则会发送一个upgrade包。

为了确保没有消息丢失，upgrade包在当前存在的传输通道缓存中仅会被发送一次，且此时传输被认为是处于暂停状态。

当服务端接收到upgrade包，它必须假设这是新的传输通道，并且发送当前以及存在的通道中所有的缓存内容。

客户端发送的探测是由`ping`包类型和`probe`字符串拼接成的数据。服务端返回的探测是由`pong`包类型和`probe`字符串拼接成的数据。

进一步说，升级只会考虑`polling -> x`。

### 超时时间

客户端必须使用`pingTimeout`和`pingInterval`作为握手的部分（带着`open`包）来判断服务端是否断开。

服务端发送一个`ping`包。如果在`pingTimeout`之内没有接收到包类型，则服务端会认为socket已经断开连接。如果`pong`包返回并接收成功的话，服务端会等待`pingInterval`之后再发送下一个`ping`包。

由于两个值在服务端和客户端是共享的，客户端也可以通过它们来判断服务端是否断开连接当它没有在`pingTimeout+pingInterval`之内收到任何数据。

## SocketIO Protocol

大部分内容引自出处: [blog](https://www.kevinwu0904.top/blogs/network-socketio/)。开源社区介绍 [github socket.io](https://github.com/socketio/socket.io-protocol)

### Packet格式

一个Packet包括以下字段：

- 类型（整数，详见下文）
- 名称空间（字符串）
- 可选字段：Packet内容（对象 | 数组）
- 可选字段：ACK的ID（整数）

### Packet类型

#### 0-CONNECT

该事件发生时机：

- 当客户端连接一个名称空间。客户端会发送一个用于鉴权的payload，例如：

```json
{
  "type": 0,
  "nsp": "/admin",
  "data": {
    "token": "123"
  }
}
```

- 当服务端接受来自一个名称空间的连接。请求成功后，服务端会响应一个带有Socket ID的payload，例如：

```json
{
  "type": 0,
  "nsp": "/admin",
  "data": {
    "sid": "CjdVH4TQvovi1VvgAC5Z"
  }
}
```

#### 1-DISCONNECT

该事件发生在一端想要断开名称空间的连接时。它不包括任何payload和ACK ID，例如：

```json
{
  "type": 1,
  "nsp": "/admin"
}
```

#### 2-EVENT

该事件发生在一端想要给另一端传输数据（不包括二进制数据）时。它包括payload，可能还会包括ACK ID，例如：

```json
{
  "type": 2,
  "nsp": "/",
  "data": ["hello", 1]
}
```

包括ACK ID的样例：

```json
{
  "type": 2,
  "nsp": "/admin",
  "data": ["project:delete", 123],
  "id": 456
}
```

#### 3-ACK

该事件发生在一端接收到EVENT事件或带有ACK ID的BINARY_EVENT事件。它包括对之前这个事件的ACK ID，可能还会包括payload（不包括二进制数据），例如：

```json
{
  "type": 3,
  "nsp": "/admin",
  "data": [],
  "id": 456
}
```

#### 4-CONNECT_ERROR

该事件发生在服务端拒绝一个名称空间的连接时。它包括一个"message"字段，可能还会包括一个"data"字段，例如：

```json
{
  "type": 4,
  "nsp": "/admin",
  "data": {
    "message": "Not authorized",
    "data": {
      "code": "E001",
      "label": "Invalid credentials"
    }
  }
}
```

#### 5-BINARY_EVENT

注意：BINARY_EVENT和BINARY_ACK都用于内建的解析器中，为了区别出包中是否包括二进制内容。它们不会用于其他自定义解析器中。

该事件发生在一端想要给另一端传输数据（包括二进制数据）时。它包括payload，可能还会包括ACK ID，例如：

```json
{
  "type": 5,
  "nsp": "/",
  "data": ["hello", <Buffer 01 02 03>]
}
```

包括ACK ID的样例：

```json
{
  "type": 5,
  "nsp": "/admin",
  "data": ["project:delete", <Buffer 01 02 03>],
  "id": 456
}
```

#### 6-BINARY_ACK

该事件发生在一端接收到EVENT事件或带有ACK ID的BINARY_EVENT事件。它包括对之前这个事件的ACK ID，可能还会包括payload（包括二进制数据），例如：

```json
{
  "type": 6,
  "nsp": "/admin",
  "data": [<Buffer 03 02 01>],
  "id": 456
}
```

### Packet编码

本小节描述了Socket.IO 客户端和服务端之间默认解析器的编码细节，源码实现参考：[这里](https://github.com/socketio/socket.io-parser)。

另外注意一点：Socket.IO的packet本质上是Engine.IO`message`类型的packet（关于Engine.IO参考[这里](https://github.com/socketio/engine.io-protocol)），所以编码结果发送时候会带上`4`这个数字前缀。（在HTTP long-polling的请求和响应包体中，或者在Websocket的数据帧中。）

#### 编码格式

```
<packet type>[<# of binary attachments>-][<namespace>,][<acknowledgment id>][JSON-stringified payload without binary]

+ binary attachments extracted
```

注意：

- 当名称空间不是默认的`/`时候才会出现在编码格式中。

#### 编码样例

- `/`名称空间`CONNECT`

```json
{
  "type": 0,
  "nsp": "/",
  "data": {
    "token": "123"
  }
}
```

编码为：`0{"token":"123"}`

- `/admin`名称空间的`CONNECT`

```json
{
  "type": 0,
  "nsp": "/admin",
  "data": {
    "token": "123"
  }
}
```

编码为`0/admin,{"token":"123"}`

+ `/admin`名称空间的`DISCONNECT`

```json
{
  "type": 1,
  "nsp": "/admin"
}
```

编码为`1/admin`

- `EVENT`

```json
{
  "type": 2,
  "nsp": "/",
  "data": ["hello", 1]
}
```

编码为`2["hello",1]`

- 带ACK ID的`EVENT`

```json
{
  "type": 2,
  "nsp": "/admin",
  "data": ["project:delete", 123],
  "id": 456
}
```

编码为`2/admin,456["project:delete",123]`

- `ACK`

```json
{
  "type": 3,
  "nsp": "/admin",
  "data": [],
  "id": 456
}
```

编码为`3/admin,456[]`

- `CONNECT_ERROR`

```json
{
  "type": 4,
  "nsp": "/admin",
  "data": {
    "message": "Not authorized"
  }
}
```

编码为`4/admin,{"message":"Not authorized"}`

- `BINARY_EVENT`

```json
{
  "type": 5,
  "nsp": "/",
  "data": ["hello", <Buffer 01 02 03>]
}
```

编码为`51-["hello",{"_placeholder":true,"num":0}] + <Buffer 01 02 03>`

- 带ACK ID的`BINARY_EVENT`

```json
{
  "type": 5,
  "nsp": "/admin",
  "data": ["project:delete", <Buffer 01 02 03>],
  "id": 456
}
```

编码为`51-/admin,456["project:delete",{"_placeholder":true,"num":0}] + <Buffer 01 02 03>`

- `BINARY_ACK`

```json
{
  "type": 6,
  "nsp": "/admin",
  "data": [<Buffer 03 02 01>],
  "id": 456
}
```

编码为`61-/admin,456[{"_placeholder":true,"num":0}] + <Buffer 03 02 01>`

### 协议交互

#### 连接名称空间

对于每个名称空间（包括主名称空间），客户端首先发送一个`CONNECT`，服务端回复一个带有Socket ID的`CONNECT`。

```
Client > { type: CONNECT, nsp: "/admin" }
Server > { type: CONNECT, nsp: "/admin", data: { sid: "wZX3oN0bSVIhsaknAAAI" } } (if the connection is successful)
or
Server > { type: CONNECT_ERROR, nsp: "/admin", data: { message: "Not authorized" } }
```

#### 名称空间断连

```
Client > { type: DISCONNECT, nsp: "/admin" }
```

反之亦然。同时另外一端无需任何消息回复。

#### ACK

```
Client > { type: EVENT, nsp: "/admin", data: ["hello"], id: 456 }
Server > { type: ACK, nsp: "/admin", data: [], id: 456 }
or
Server > { type: BINARY_ACK, nsp: "/admin", data: [ <Buffer 01 02 03> ], id: 456 }
```

反之亦然。

### 会话示例

这里有一份包括了Socket.IO和Engine.IO的会话示例。

- Request n°1 (open packet)

```
GET /socket.io/?EIO=4&transport=polling&t=N8hyd6w
< HTTP/1.1 200 OK
< Content-Type: text/plain; charset=UTF-8
0{"sid":"lv_VI97HAXpY6yYWAAAC","upgrades":["websocket"],"pingInterval":25000,"pingTimeout":5000}
```

细节：

```
0           => Engine.IO "open" packet type
{"sid":...  => the Engine.IO handshake data
```

注意：参数`t`用于防止浏览器缓存该请求。

- Request n°2 (namespace connection request)

```
POST /socket.io/?EIO=4&transport=polling&t=N8hyd7H&sid=lv_VI97HAXpY6yYWAAAC
< HTTP/1.1 200 OK
< Content-Type: text/plain; charset=UTF-8
40
```

细节：

```
4           => Engine.IO "message" packet type
0           => Socket.IO "CONNECT" packet type
```

- Request n°3 (namespace connection approval)

```
GET /socket.io/?EIO=4&transport=polling&t=N8hyd7H&sid=lv_VI97HAXpY6yYWAAAC
< HTTP/1.1 200 OK
< Content-Type: text/plain; charset=UTF-8
40{"sid":"wZX3oN0bSVIhsaknAAAI"}
```

- Request n°4

服务端会执行`socket.emit('hey', 'Jude')`

```
GET /socket.io/?EIO=4&transport=polling&t=N8hyd7H&sid=lv_VI97HAXpY6yYWAAAC
< HTTP/1.1 200 OK
< Content-Type: text/plain; charset=UTF-8
42["hey","Jude"]
```

细节：

```
4           => Engine.IO "message" packet type
2           => Socket.IO "EVENT" packet type
[...]       => content
```

- Request n°5 (message out)

客户端会执行`socket.emit('hello'); socket.emit('world');`

```
POST /socket.io/?EIO=4&transport=polling&t=N8hzxke&sid=lv_VI97HAXpY6yYWAAAC
> Content-Type: text/plain; charset=UTF-8
42["hello"]\x1e42["world"]
< HTTP/1.1 200 OK
< Content-Type: text/plain; charset=UTF-8
ok
```

细节：

```
4           => Engine.IO "message" packet type
2           => Socket.IO "EVENT" packet type
["hello"]   => the 1st content
\x1e        => separator
4           => Engine.IO "message" packet type
2           => Socket.IO "EVENT" packet type
["world"]   => the 2nd content
```

- Request n°6 (WebSocket upgrade)

```
GET /socket.io/?EIO=4&transport=websocket&sid=lv_VI97HAXpY6yYWAAAC
< HTTP/1.1 101 Switching Protocols
```

WebSocket数据帧：

```
< 2probe                                        => Engine.IO probe request
> 3probe                                        => Engine.IO probe response
> 5                                             => Engine.IO "upgrade" packet type
> 42["hello"]
> 42["world"]
> 40/admin,                                     => request access to the admin namespace (Socket.IO "CONNECT" packet)
< 40/admin,{"sid":"-G5j-67EZFp-q59rADQM"}       => grant access to the admin namespace
> 42/admin,1["tellme"]                          => Socket.IO "EVENT" packet with acknowledgement
< 461-/admin,1[{"_placeholder":true,"num":0}]   => Socket.IO "BINARY_ACK" packet with a placeholder
< <binary>                                      => the binary attachment (sent in the following frame)
... after a while without message
> 2                                             => Engine.IO "ping" packet type
< 3                                             => Engine.IO "pong" packet type
> 1                                             => Engine.IO "close" packet type
```

## 网络电话示例: 

```
420["netPhone", {msgId: "fe9b9760-0b15-409f-a497-1909e53355d4", signal: "call",…}]
430[{msgId: "fe9b9760-0b15-409f-a497-1909e53355d4", signal: "ack", content: ,…}]
42["netPhone", {msgId: "db2c3741-2029-4cbb-a536-ecd8baf4dbf8", signal: "call",…}]
```

### 
