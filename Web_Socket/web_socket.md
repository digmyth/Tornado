
# WebSocket协议

### WebSocket在什么场景下使用？

页面实时展示数据
*   轮询：setInterval()前端轮询请求，性能低下
*  长轮询：把请求pending住多少秒后再返回，量大时也损耗服务器性能
*  WebSocket: 建立socket双向传输数据，高效。


### 那么什么是WebSocket
[参考链接](http://www.cnblogs.com/wupeiqi/articles/6558766.html)

先来看下http协议

http协议： 
   1 格式：请求头、请求体之间\r\n\r\n分隔，请求头或请求体内部\r\n分隔。
   2 连接：一次请求，一次响应，然后断开连接。

那么WebSocket协议是怎么样的呢？

WebSocket：
   1 格式： 请求头、请求体之间\r\n\r\n分隔，请求头或请求体内部\r\n分隔。
   2 连接： 创建socket通道后不断开，全双工(full-duplex)通信，可以相互发消息。


WebSocket实现了浏览器与服务器全双工(full-duplex)通信，能主动向浏览器发送消息，但需要浏览器支持websocket封包解包或加密解密。其本质是保持TCP连接，在浏览器和服务端通过Socket进行通信。

WebSocket特性
* WebSocket 是独立的、创建在 TCP 上的协议。
* Websocket 通过 HTTP/1.1 协议的101状态码进行握手。
* 为了创建Websocket连接，需要通过浏览器发出请求，之后服务器进行回应，这个过程通常称为“握手”（handshaking）

总结起来：WebSocket是一种在单个TCP连接上进行全双工通信的协议。使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。

### WebSokcet工作原理
首先客户端要验证服务器是否支持websocket协议，能不能一起玩耍？

 客户端发送playload data之前会发送握手字符串，服务器把握手字符串加密后返回给客户端，此时客户端也把字符串按`特定算法`加密，把客户端加密后的字符串与服务器加密后的字符串进行比较，如果一致则客户端认为服务器支持WebSocket协议通信，可以相互一起玩耍。

握手时的特定算法是什么呢？代码如下
```
#!/usr/bin/python3
import hashlib
import base64

SecKey = 'sN9cRrP/n9NdMgdcy2VJFQ=='   # browser 自动携带的随机字符串
Magic_string = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11'

def server_algorithm(SecKey):
    str = SecKey + Magic_string
    sec_str = base64.b64encode(hashlib.sha1(str.encode('utf-8')).digest())
    return sec_str

print(server_algorithm(SecKey))
```

能不能一起玩耍，官方术语就是创建Websocket连接，需要通过浏览器发出请求，之后服务器进行回应，这个过程通常称为“握手”（handshaking）.

不管怎么说，WebSocket允许服务端主动向客户端推送数据。在WebSocket API中，浏览器和服务器只需要完成一次握手，两者之间就可以创建持久性的连接，并进行双向数据传输。

什么？看不懂，一言不合上代码，下面是一个典型的Websocket握手请求.

客户端请求
```
GET / HTTP/1.1
Upgrade: websocket
Connection: Upgrade
Host: example.com
Origin: http://example.com:8002
Sec-WebSocket-Key: sN9cRrP/n9NdMgdcy2VJFQ==
Sec-WebSocket-Version: 13
```

服务器回应
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: fFBooB7FAkLlXgRSz0BT3v4hq5s=
Sec-WebSocket-Location: ws://example.com:8002/
```

字段说明
* Connection必须设置Upgrade，表示客户端希望连接升级。
* Upgrade字段必须设置Websocket，表示希望升级到Websocket协议。
* Sec-WebSocket-Key是随机的字符串，服务器端会用这些数据来构造出一个SHA-1的信息摘要。把“Sec-WebSocket-Key”加上一个特殊字符串“258EAFA5-E914-47DA-95CA-C5AB0DC85B11”，然后计算SHA-1摘要，之后进行BASE-64编码，将结果做为“Sec-WebSocket-Accept”头的值，返回给客户端。如此操作，可以尽量避免普通HTTP请求被误认为Websocket协议。
* Sec-WebSocket-Version 表示支持的Websocket版本。RFC6455要求使用的版本是13.
* Origin字段是可选的，通常用来表示在浏览器中发起此Websocket连接所在的页面，类似于Referer。但是,与Referer不同的是，Origin只包含了协议和主机名称。

### 服务器解包细节
[官方WebSocket instructions](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_servers)

注意的是客户端和服务端传输数据时，需要对数据进行封包解包。客户端有 javascript类库实现封包解包，但服务器需要手动实现。

当`conn,addr = sk.accept()`时，服务器端代码如下
```
info = conn.recv(8096)

    payload_len = info[1] & 127
    if payload_len == 126:
        extend_payload_len = info[2:4]
        mask = info[4:8]
        decoded = info[8:]
    elif payload_len == 127:
        extend_payload_len = info[2:10]
        mask = info[10:14]
        decoded = info[14:]
    else:
        extend_payload_len = None
        mask = info[2:6]
        decoded = info[6:]

    bytes_list = bytearray()
    for i in range(len(decoded)):
        chunk = decoded[i] ^ mask[i % 4]
        bytes_list.append(chunk)
    body = str(bytes_list, encoding='utf-8')
    print(body)
```

要看懂这段代码必须了解websocket解包细节.

当客户端加密发送了socket data数据时，服务端收到数据是这样的.

`b'\x81\x82\xac\xde\xdd\xf4\xdc\xae'`

需要对这样的数据解密才能拿到真的数据，跟据第二个字节后7位的值取数据.
```
0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| opcode|M| Payload len |    Extended payload length    |
|I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
|N|V|V|V|       |S|             |   (if payload len==126/127)   |
| |1|2|3|       |K|             |                               |
+-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
|     Extended payload length continued, if payload len == 127  |
+ - - - - - - - - - - - - - - - +-------------------------------+
|                               |Masking-key, if MASK set to 1  |
+-------------------------------+-------------------------------+
| Masking-key (continued)       |          Payload Data         |
+-------------------------------- - - - - - - - - - - - - - - - +
:                     Payload Data continued ...                :
+ - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
|                     Payload Data continued ...                |
+---------------------------------------------------------------+
```

```
value  = socket_data[1] & 127
value <=125  b'\x81\x82  \xac\xde\xdd\xf4\xdc\xae' # 数据在第2个节字后
value =126   b'\x81\x82\xac\xde  \xdd\xf4\xdc\xae' # next 16bit(2个字节)，数据在第4个节字后  
value =127   xxx...           # next 64bit(8个字节), 数据在第10个节字后
```

其中头32bits为掩码，真正数据还要去掉这4字节，取真正数据真不容易。

那么服务端向客户端发送数据就好封包了
```
def send_msg(conn, msg_bytes):
    """
    WebSocket服务端向客户端发送消息
    :param conn: 客户端连接到服务器端的socket对象,即： conn,address = socket.accept()
    :param msg_bytes: 向客户端发送的字节
    :return: 
    """
    import struct

    token = b"\x81"
    length = len(msg_bytes)
    if length < 126:
        token += struct.pack("B", length)
    elif length <= 0xFFFF:
        token += struct.pack("!BH", 126, length)
    else:
        token += struct.pack("!BQ", 127, length)

    msg = token + msg_bytes
    conn.send(msg)
    return True
 ```
 
好了,WebSocket所有知识都在这里了，慢慢品味知识的韵味。

基于Python socket实现的WebSocket服务端.
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import socket
import base64
import hashlib
 
 
def get_headers(data):
    """
    将请求头格式化成字典
    :param data:
    :return:
    """
    header_dict = {}
    data = str(data, encoding='utf-8')
 
    header, body = data.split('\r\n\r\n', 1)
    header_list = header.split('\r\n')
    for i in range(0, len(header_list)):
        if i == 0:
            if len(header_list[i].split(' ')) == 3:
                header_dict['method'], header_dict['url'], header_dict['protocol'] = header_list[i].split(' ')
        else:
            k, v = header_list[i].split(':', 1)
            header_dict[k] = v.strip()
    return header_dict
 
 
def send_msg(conn, msg_bytes):
    """
    WebSocket服务端向客户端发送消息
    :param conn: 客户端连接到服务器端的socket对象,即： conn,address = socket.accept()
    :param msg_bytes: 向客户端发送的字节
    :return:
    """
    import struct
 
    token = b"\x81"
    length = len(msg_bytes)
    if length < 126:
        token += struct.pack("B", length)
    elif length <= 0xFFFF:
        token += struct.pack("!BH", 126, length)
    else:
        token += struct.pack("!BQ", 127, length)
 
    msg = token + msg_bytes
    conn.send(msg)
    return True
 
 
def run():
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    sock.bind(('127.0.0.1', 8003))
    sock.listen(5)
 
    conn, address = sock.accept()
    data = conn.recv(1024)
    headers = get_headers(data)
    response_tpl = "HTTP/1.1 101 Switching Protocols\r\n" \
                   "Upgrade:websocket\r\n" \
                   "Connection:Upgrade\r\n" \
                   "Sec-WebSocket-Accept:%s\r\n" \
                   "WebSocket-Location:ws://%s%s\r\n\r\n"
 
    value = headers['Sec-WebSocket-Key'] + '258EAFA5-E914-47DA-95CA-C5AB0DC85B11'
    ac = base64.b64encode(hashlib.sha1(value.encode('utf-8')).digest())
    response_str = response_tpl % (ac.decode('utf-8'), headers['Host'], headers['url'])
    conn.send(bytes(response_str, encoding='utf-8'))
 
    while True:
        try:
            info = conn.recv(8096)
        except Exception as e:
            info = None
        if not info:
            break
        payload_len = info[1] & 127
        if payload_len == 126:
            extend_payload_len = info[2:4]
            mask = info[4:8]
            decoded = info[8:]
        elif payload_len == 127:
            extend_payload_len = info[2:10]
            mask = info[10:14]
            decoded = info[14:]
        else:
            extend_payload_len = None
            mask = info[2:6]
            decoded = info[6:]
 
        bytes_list = bytearray()
        for i in range(len(decoded)):
            chunk = decoded[i] ^ mask[i % 4]
            bytes_list.append(chunk)
        body = str(bytes_list, encoding='utf-8')
        send_msg(conn,body.encode('utf-8'))
 
    sock.close()
 
if __name__ == '__main__':
    run()
```
