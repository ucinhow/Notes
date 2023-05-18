## 科学上网

GFW（Great Firewall），中国国家防火墙，通过 DNS 污染和针对性的 TCP 重置攻击阻止访问指定国外域名的资源。科学上网，指在未被 GFW 针对的服务器搭建代理服务，通过代理访问资源，搭建的代理服务需要具备加密混淆的效果，避免 GFW 拆包识别到访问的是黑名单 IP 导致触发 GFW 的 TCP 重置攻击。

可以根据示意图（仅用于参考）理解后续概念：
![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/net-line.6fsjn0udp1s0.webp)

BGP 协议，边界网关协议是运行于 TCP 的一种自治系统的路由协议。

AS 自治系统，具有统一路由策略的网络群组。AS 通过 BGP 协议通知自己负责的 IP 地址范围以及连接的其他 AS，BGP 路由器汇总这些信息为路由表，确定 AS 之间的路由最短路径。

直连 & 中转，直连指的是数据从终端到 GFW 不通过额外的 AS 系统，指通过终端接入的运营商 AS 系统；中转，则是通过了额外的 AS 系统，由于通过 BGP 路由 AS 系统，出现 BGP 中转的术语。

专线，分为内网专线和公网专线。

公网专线，如 CN2。Chinatelecom Next Carrier Network，可以理解为一种公网优质线路，对比家庭宽带普通线路的 AS 系统，相对不拥堵更加顺畅，因此会有更好的体验。值得注意的是，公网专线同样需要通过骨干网的国际出口传输到国外 AS，同样会经过 GFW，同样会在国际出口出现带宽拥堵。

内网专线，如 IPLC/IEPL。传输不经过公网，完全通过内网线路，不会经过 GFW。

因此 VPS 选择的优先级为：内网专线 IPLC/IEPL > 公网专线 CN2 > 公网线路。

### 代理服务搭建

#### hysteria

hysteria 是通过基于 QUIC 修改的协议实现的开源代理。

[官方文档](https://hysteria.network/zh/)

##### docker 部署

```shell
docker run -dt --network=host --name hysteria \
    -v /root/hysteria.json:/etc/hysteria.json \
    tobyxdd/hysteria -c /etc/hysteria.json server
# 如果不想使用宿主机网络(--network=host)，需要确保正确映射 Hysteria 的 UDP 端口(-p 1234:1234/udp)。
# 上述指令是将 /root/hysteria.json 文件挂载为容器的 /etc/hysteria.json 文件作为配置启动服务，需要根据实际情况调整。
```

##### 配置

json 格式文件配置，相关配置项通过 [官方文档](https://hysteria.network/zh/docs/advanced-usage/) 查询，基本配置包含 `listen` 监听端口、`cert` 证书、`key` 证书密码、`obfs` 混淆密码。

#### trojan

步骤类似，镜像部署、配置参数。。。

#### 终端客户端

### 提速优化

- 使用 BBR 拥塞控制算法，查看设置当前使用拥塞控制算法：`sysctl net.ipv4.tcp_congestion_control`
- 终端接入地转发的优化，避免多次中转、负载均衡（有待了解）

### 域名

- [Godaddy](https://www.godaddy.com/)

### HTTPS 证书

#### ACME 协议

Automatic Certificate Management Environment，用于证书颁发机构自动化地管理证书验证和管理 TLS 证书，简化证书颁发流程。需要证书的服务端可以通过 ACME 自动化地向 CA 机构申请证书。

#### acme.sh

实现 ACME 协议，用于服务端自动化申请证书的工具。支持以下的 CA 机构：

- Let's Encrypt，`acme.sh` 设置指令：`acme.sh --set-default-ca --server letsencrypt`
- Buypass，`acme.sh` 设置指令：`acme.sh --set-default-ca --server buypass`
- ZeroSSL，`acme.sh` 设置指令：`acme.sh --set-default-ca --server zerossl`

#### 证书申请

1. 安装 `socat`，`apt install socat`
2. 安装 `acme.sh`，`curl https://get.acme.sh | sh`
3. 添加软链接（方便直接调用 `acme.sh` 命令），`ln -s /root/.acme.sh/acme.sh /usr/local/bin/acme.sh`
4. 注册账号，`acme.sh --register-account -m xxx@example.com`
5. 申请证书（注意开放防火墙 80 端口），`acme.sh --issue -d 域名 -standalone -k ec-256`
6. 安装证书，`acme.sh --installcert -d 域名 -ecc --key-file /root/hysteria/server.key --fullchain-file /root/hysteria/server.crt`

#### 自签证书

生成私钥，`openssl ecparam -genkey -name prime256v1 -out ca.key`
生成证书，`openssl req -new -x509 -days 365000 -key ca.key -out ca.crt -subj "/CN=bing.com"`
