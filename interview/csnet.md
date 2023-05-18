## 物理层

利用传输介质为数据链路层提供物理连接，实现比特流的传输

**协议数据单元** 比特

## 数据链路层

定义在物理连接基础上如何传输，管理链路的建立、维持和释放，完成差错监测（CRC 循环冗余校验），流量控制和拥塞控制（点到点）

**协议数据单元** 帧

**MAC 地址** 介质访问控制地址，网络设备的唯一物理地址

**MTU 最大传输单元** 数据链路中限制的数据帧的最大字节长度

**以太网协议** IEEE802.3，封装数据帧，头部包含：目标地址、源地址、类型：IP ｜ ARP；尾部包含 CRC 校验码

**无线** IEEE802.11 协议

**CRC 循环冗余校验**

选定一个用于校验的多项式 G(x)，并在数据尾部添加 r 个零，r 为多项式的最高次数，作为被除数，多项式位串作为除数，使用模 2 除法（不借位，异或）相除

所得余数填充在原数据 r 个零的位置，得到可校验位串，接收端将 可校验位串 除以 多项式位串，若结果为零，则传输没有出错

## 网络层

提供路由和寻址功能，确定终端互联的最佳路径

**协议数据单元** 数据包

**IP** IP 地址，逻辑地址，用于屏蔽物理地址的差异，根据 IP 协议封装上层的数据，添加目标 IP，源 IP 以及 IP 分片信息（由于 MTU 的限制，需大体积数据进行分片）等信息

**OSPF** 通过互相交换链路状态信息，计算出最短路径路由表，数据包发送根据路由表进行发送

**网络地址转换 NAT** 转化局域网 IP 和公网 IP

**子网划分 & 子网掩码** 子网掩码与 IP 地址进行与运算，计算出公网地址

**ARP** 地址解析，根据 IP 获取 MAC 地址，发送包含目标 IP 的 ARP 请求广播到网络上的所有主机，从返回的消息中获取 MAC 地址

**DHCP** 由 DHCP 服务器分配一个 IP 地址

**ICMP** 确认 IP 包是否到达目标地址，并反馈丢包原因（不可达、超时、参数问题、重定向），`ping` 指令就是借助 ICMP 协议

## 传输层

**协议数据单元** 数据段

提供端到端的通信服务，提供面向连接、可靠性、流量控制、多路复用等服务，相关协议：TCP & UDP

**Socket**

协议 IP 地址 端口号，是对 TCP/IP 协议的一种抽象封装，是应用层与 TCP/IP 协议进行交互的中间抽象和接口，端口为 16 位，即 0 ～ 65535

## 应用层

**协议数据单元** 数据

**会话层** 管理会话的开启，关闭

**表示层** 数据格式转换，数据处理、加密解密，压缩解压，编码解码

与应用结合，提供网络应用服务，管理与另一应用之间的通信，相关协议：HTTP、DNS、FTP、Telent、SSH、SMTP、POP3 等

## TCP

**面向连接、全双工**

**适用场景** 需可靠传输的场景，如文件传输

#### 如何实现可靠有序交付

**序列号 SEQ ｜确认号 ACK**

序列号用于确保数据能有序提交到应用层，表示报文中数据第一个字节的序号。

确认号表示期望收到对方的下一个字节的序号。TCP 默认使用累计确认，即只确认数据流中按序第一个丢失字节为止的字节。

**超时重传**

每发送一个报文段，对这个报文段设置一个计时器。计时器设置的重传时间到期还未收到确认，就重传这一报文

重传时间的计算：TCP 采用自适应算法，记录往返时间 RTT，基于 RTT 得到一个加权平均往返时间 RTTs，它会随 RTT 样本值的变化不断变化更新

**快重传** 拥塞控制 - 快重传

#### 流量控制

流量控制窗口（端对端的通信量控制）

#### 拥塞控制

拥塞控制窗口（全局性的网络负荷控制）

**慢开始** 拥塞窗口从零开始指数上升

**拥塞避免** 达到阈值后 +1 上升

**快重传** 接收到 3 个 ACK 相同的数据包时，立刻重新发送对应的后续数据包

**快恢复** 出现快重传后拥塞窗口减半

#### 三次握手

SYN=1 =\> SYN=1 & ACK=1 =\> ACK=1

**why?** 如果两次握手，在网络状况较差时，客户端可能发出了多次建立连接的请求，而服务端返回的可能是历史建立连接请求的答复，客户端接收到时无法判断该连接建立答复是否已经过期

#### 四次挥手

FIN=1 =\> ACK=1 =\> ... =\> FIN=1 & ACK=1 =\> ACK + TIME-WAIT(2MSL)

**MSL** 最大数据段生命周期 Maximum segment lifetime

**why?** 在客户端请求结束连接时，服务端可能还有数据需要发送，因此在确认客户端结束连接的请求后，还可以继续发送数据然后才发送释放连接的报文，最后客户端确认结束连接；TIME-WAIT 阶段是为了确保服务端接收到了最后一次确认的报文

#### 粘包问题

TCP 是面向字节流的协议，为避免浪费网络 IO，可能会组合或拆分应用层传递下来的数据到 MSS（最大分段长度）

解决粘包问题：

加入特殊标志 使用特殊标志（`0xfffffe` 或者回车符）作为头尾，明确数据边界

加入消息长度 表明多少字节是属于这个消息的，HTTP 的 `Content-Length`

#### 多路复用

让多个数据流通过一个链接进行传输。端口复用：多个应用共用一个端口进行数据传输。减少连接带来的资源开销。

## UDP

无连接、实时性、多对多

DNS、RTP（实时传输协议）

适用于实时应用（通话、视频会议、直播）

## HTTP

### HTTP 1.1

**持久连接** 不释放 TCP 连接

**分块传输** 分块传输只用于响应。响应头添加 `Transfer-Encoding: chunked`，响应数据分块，每个块在开头添加块长度信息，用 16 进制表示，块长度信息和块数据之间用 CRLF（回车换行）分隔。

**Cookie**

**管线化** HTTP 以串行的方式请求和响应，只能顺序地请求和响应，会有请求和响应的队头阻塞，管线化技术允许一个连接多个请求，请求不会阻塞、但响应仍会队头阻塞，服务器会按照发送请求的顺序响应，后请求先到达也无法响应。

同域名 6 个 TCP 连接限制

### HTTP 2

**多路复用、流机制（id）** 同域名一个 TCP 连接，通过流的并发机制解决了 HTTP 层的队头阻塞

**头部压缩** （`HPACK` 算法：同时维护一张头索引表，字段为值，发送索引键即可完成压缩）

**二进制传输**（二进制帧：`header frame | data frame`）

**服务器推送** 客户端请求一个资源时，服务端主动向其推送与该资源相关的其他资源。需要双方都支持 HTTP2。客户端可以拒绝不需要的推送资源。

TCP 层仍有队头阻塞的问题，丢包会阻塞后续请求

### HTTP 3

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/http3.5epdg8kh8hk0.webp)

**QUIC** 基于 UDP，实现可靠性传输

**快速握手** TCP 与 TLS 分层，需要分批次握手，而 QUIC 内部包含 TLS1.3，握手结合在一起，1RTT 确认双方的「连接 ID」即可完成握手

**无队头阻塞** 流之间相互独立，丢包只会阻塞这个流而不是整个连接

**连接迁移** 通过连接 ID 标记通信端点，而不是 IP 地址，当网络环境发生变化（流量=\>WIFI），仍保有上下文信息（连接 ID、TLS 密钥等），无缝复用原连接；而用 IP 标识的 TCP 会出现慢启动，导致卡顿

### HTTPS

端口 443

**SSL | TLS**

#### RSA 握手

1. 请求：协议版本、随机数、支持的加密组件
2. 响应：确认协议版本、随机数、确认加密组件、数字证书
3. 确认服务器可信、用数字证书的公钥加密发送：随机数
4. 响应：通知表示随后的信息都将用「会话秘钥」加密通信，通过协商的加密算法和三个随机数计算出会话密钥，随后进行对称加密

%%### ECDHE 握手 

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/dh-key-exchange.5qh5uv9u7r00.webp)

1. 客户端发起建立连接
2. 服务端用长期私钥 a、原根 g、质数 p 计算出长期公钥 A，将 Apg 通过 CA 证书私钥加密放在 serverConfig 里面发给客户端（serverConfig 可以缓存在客户端，方便有效期内下次 0RTT 建立连接）
3. 客户端随机选择一个短期密钥 b，用 CA 证书的公钥解密 Apg，通过 Apb 计算出对称密钥 K，gbp 计算出 B，将 B 和后续发送的用 K 加密的实际数据一起发送给服务端（在这一阶段已经可以开始发送数据，握手完毕，历经 1RTT）
4. 服务端通过 Bpb 计算出 K，以解密客户端发送的实际数据和加密响应数据%%

**混合加密** 非对称加密（交换对称密钥）& 对称加密（数据通信）

**非对称加密** RSA、Elgamal

**对称加密** DES、AES

**摘要算法** 生成数据独一无二的指纹，防止篡改，保证数据完整性。

**数字证书** 借助第三方机构，确认主机可信，携带公钥

#### 中间人攻击

HTTPS 中间人攻击（Man-in-the-Middle Attack，MITM 攻击），攻击者利用自己控制的中间网络节点来拦截、篡改或窃取双方通信的数据。

攻击过程：
1. 利用攻击者控制的受害者和目标服务器的中间网络节点。
2. 受害者发起 HTTPS 请求，中间网络节点转发给服务器，并假装自己是受害者。
3. 服务器将证书发送给中间节点，中间节点假装自己是服务器将自己的证书发送给受害者。
4. 受害者收到证书后与中间节点建立安全连接，中间节点与服务器建立安全连接，在转发受害者请求和服务器响应的过程中，中间节点可以窃取和篡改通信数据。

解决方法：
- 安全的网络环境
- 验证证书是否受信任和未过期，使用受信任的证书颁发机构
- 证书固定，客户端将部分服务端的证书存储在本地，避免被中间节点伪造证书
- 双向认证，要求客户端和服务端都验证对方身份

### 状态码

100 继续请求、200 请求成功、204 成功处理无返回内容、206 返回部分内容（分块传输）、301 永久移动、302 暂时移动、304 未修改、307 暂时移动但重定向应采用相同的方法，400 请求错误、401 需要用户验证、403 权限不足拒绝请求、404 未找到、500 服务器未知错误、501 请求方法不支持、502 网关请求响应错误、503 服务器停机

[参考链接](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status)

### 常见首部

`Host`、`Content-Length` 统计 body 字节长度、`Connection:Keep-Alive` 持久连接、`Content-Type` 响应数据格式、`Accept` 请求声明接收的数据格式、`Content-Encoding | Accept-Encoding` 数据压缩方式

### 缓存机制

浏览器对响应数据进行缓存避免重新请求的机制，Html | Css | Js | Font | Image

`Cache-Control: max-age | no-cache | no-store | private | public | must-revalidate`

`must-revalidate` 资源过期，在向源服务器验证前，缓存不能作为后续请求的响应。`proxy-revalidate` 与前者一致，但是其只对代理生效。

`Pragma: no-cache` 与 `Cache-Control: no-cache` 一致

`Expires`

**强制缓存** 从本地缓存获取数据

**协商缓存** 请求询问缓存是否可用

`If-None-Match` | `ETag`

`If-Modified-Since / If-Unmodified-Since` | `Last-Modified`

**强 ETag** 不论实体发生多么细微的变化都会改变。

**弱 ETag** 只用于提示资源是否相同，只有资源发生根本变化才改变

ETag 生成的算法不固定：

- Nginx 由响应头的 `Last-Modified` 与 `Content-Length` 表示为十六进制组合而成
- Koa 静态文件：大小和修改时间、`string | buffer`： 哈希值和长度。

**Etag 和 LastModified 的区别**

`Last-Modified` 标注的最后修改只能精确到秒级，时间粒度更小的修改无法根据 `Last-Modified` 判断是否修改过

也有可能出现服务器无法准确获取文件修改的时间，或代理服务器时间不一致的情形让 `Last-Modified` 的可靠性下降

总的来说，Etag 的精确度更好，优先级也更高

**`GET` 和 `POST` 的区别**：

GET 获取数据｜会被缓存｜参数通过 URL 传递，长度有限制，暴露不安全｜`header` 和参数一并发送
Get 请求 URI 长度限制是浏览器的限制，http 没有限制：
- Chrome 8182 字符
- Safari 80000 字符
- Firefox 65536 字符

POST 提交数据｜一般不被缓存｜请求体参数长度无限制，不暴露，相对安全｜发送 `header` → 服务器响应 100 `continue` → 发送 `data`

**HTTP 优点**：简单 `header+body`、灵活（首部允许自定义扩展，下层实现可以随意变化）、跨平台

**HTTP 缺点**：无状态、明文传输

**Cookie**

响应首部 `set-cookie`

请求首部 `cookie`

Cookie 属性 `name | expires | path | domain | secure | httponly`

**Restful API**

**压缩**：`Content-Encoding: gzip | compress | deflate`

**HTTP 抓包**：Fiddler、Charles

## WebSocket

端口 (80 ws) | (443 wss)

#### 握手 & 挥手

**握手**

客户端通过 HTTP 请求设置 `Connection: Upgrade` 和 `Upgrade: websocket` 发起 websocket 握手，还需要设置其他字段：

- `sec-websocket-key` 客户端随机生成的 16 字节值
- `sec-websocket-version` websocket 协议版本
- `sec-websocket-protocol` 子协议
- `sec-websocket-extension` websocket 扩展

服务端同意握手，响应设置 `Connection: Upgrade` 和 `Upgrade: websocket`，HTTP 状态码 101，还需要设置其他字段：

- `sec-websocket-accept` key -\> base64 -\> 字符串拼接 MagicNumber -\> SHA-1 哈希 -\> base64
- `sec-websocket-protocol` 选择支持的子协议，或不设置表示不支持
- `sec-websocket-version` 选择支持的 websocket 版本，如果版本都不支持服务端应该终止握手，返回 426 状态码的响应
- `sec-websocket-extension` websocket 扩展

**挥手**

发起挥手端发送 close frame，进入 CLOSING 阶段

接收端返回 close frame，进入 CLOSED 阶段，返回的 close frame 会携带 payload，内容是状态码和调试信息

发起端接收到返回的 close frame，进入 CLOSE 阶段

#### 帧结构

- **FIN** 标志位，指明是消息的最后一个分片
- **RSV** 保留字段，websocket 扩展使用
- **opcode** 标志位，指明帧类型（分片、文本、二进制、close、ping、pong）
- **Mask** 标志位，payload 是否用掩码覆盖
- **Payload Len** payload 长度
- **Masking-key** 32 位随机数
- **Payload** 数据

#### 控制帧

**Close frame**

用于关闭 websocket 连接

**Ping frame**

Ping frame 主要用来实现 WebSocket 层 Keep-Alive, 或者用来探测对方是否仍然是可回复的状态, Ping frame 可以包含 Payload

**Pong frame**

作为 Ping frame 的响应, 接收方接收到 Ping frame 后应立即发回 Pong frame, 且 Payload 的内容需要和 Ping frame 相同, 若接收方接收到了多个 Ping frame, 还没来得及回复 Pong frame, 则只需对最后一个 Ping frame 做出回复即可

Pong frame 可以由通信方主动发出, 作为一种心跳包

#### 掩码算法

frame 的 payload 部分要用掩码覆盖，使其无法伪造为 HTTP 请求，避免代理缓存污染攻击。

**代理缓存污染攻击**

主要是利用实现不完善的代理服务器：不识别 websocket、基于套接字的目标 IP 进行路由（转发请求）但基于 host 字段的域名进行 HTTP 缓存。

攻击者使用浏览器 websocket 连接自己的服务器，进行 websocket 握手，由于代理服务器实现不完善，无法识别 websocket 握手，认为是一个短连接的 http 请求，转发请求和响应后，攻击者和自己的恶意服务器完成 websocket 连接，而代理服务器不知，攻击者通过 websocket 接口的套接字发送一个伪造的 HTTP GET 请求，host 设置为被攻击的服务器域名，由于代理服务器基于 socket 目标 ip 进行路由，将这个伪造的请求发送到了攻击者的服务器，攻击者再返回恶意的 HTTP 响应并在代理服务器设置缓存，后续正常的相同请求都会得到缓存中的污染响应。

相关论文：[Talking to Yourself for Fun and Profit](https://www.adambarth.com/papers/2011/huang-chen-barth-rescorla-jackson.pdf)

**算法实现**

随机生成 32 比特的 Masking-Key。
以字节为步长遍历 Payload, 对于 Payload 的第 i 个字节, 首先做 i MOD 4 得到 j, 则掩码覆盖后的 Payload 的第 i 个字节的值为原先 Payload 第 i 个字节与 Masking-Key 的第 j 个字节做按位异或操作。

`j = i MOD 4`
`transformed-octet-i = original-octet-i XOR masking-key-octet-j`

#### websocket 状态码

只能在 close frame 使用

**1000** 代表连接正常关闭

**1001** 代表通信方已断开 (Going AWAY), 例如服务端关机或客户端关闭网页

**1002** 代表通信方因 protocol error 关闭连接

## SMTP

Simple Mail Transfer Protocol，基于 TCP，端口号 25

SMTP 定义了邮件从用户到邮件服务商，到收件人邮件服务商的标准流程。

IMAP | POP3 协议，这两个协议定义了用户向服务商查询、分组、移动、编辑等方面的操作规范

**POP3** 995 加密

POP3 协议允许电子邮件客户端下载服务器上的邮件，但是在客户端的操作（如移动邮件、标记已读等），不会反馈到服务器上，比如通过客户端收取了邮箱中的 3 封邮件并移动到其他文件夹，邮箱服务器上的这些邮件是没有同时被移动的。

**IMAP** 143 | 993 加密

IMAP 提供 webmail 与电子邮件客户端之间的双向通信，客户端的操作都会反馈到服务器上，对邮件进行的操作，服务器上的邮件也会做相应的动作。

## DNS

DNS：

根域名服务器 `.` =\> 顶级域名服务器 `com` =\> 一级域名 `baidu.com`

优先从浏览器的 DNS 缓存和本地缓存获取 DNS 解析

DNS 使用 UDP

## RPC 远程服务调用

远程服务调用的规范，HTTP 就算是一种 RPC 协议。
