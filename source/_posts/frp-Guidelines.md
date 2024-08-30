---
title: frp Guidelines
categories: [Tech]
date: 2024-08-30 18:20:17
tags: tools
sticky: 
thumbnail: 
excerpt: FRP内网穿透指南
---

# 一、官方文档
[概览](https://gofrp.org/docs/overview/)
# 二、frp是什么？
`frp`是一个专注于内网穿透的高性能的反向代理应用，支持`TCP`、`UDP`、`HTTP`、`HTTPS`等多种协议。可以将内网服务以安全、便捷的方式通过具有公网`IP`节点的中转暴露到公网。
# 三、为什么使用frp？
通过在具有公网IP的节点上部署`frp`服务端，可以轻松地将内网服务穿透到公网，同时提供诸多专业的功能特性，这包括：

- 客户端服务端通信支持`TCP`、`KCP`以及`Websocket`等多种协议。
- 采用`TCP`连接流式复用，在单个连接间承载更多请求，节省连接建立时间。
- 代理组间的负载均衡。
- 端口复用，多个服务通过同一个服务端端口暴露。
- 多个原生支持的客户端插件（静态文件查看，`HTTP`、`SOCK5`代理等），便于独立使用`frp`客户端完成某些工作。
- 高度扩展性的服务端插件系统，方便结合自身需求进行功能扩展。
- 服务端和客户端UI页面。
# 四、下载
[Releases · fatedier/frp](https://github.com/fatedier/frp/releases)
# 五、工作原理
frp主要由**客户端(**`**frpc**`**)**和**服务端(**`**frps**`**)**组成，服务端通常部署在具有公网IP的机器上，客户端通常部署在需要穿透的内网服务所在的机器上。<br />内网服务由于没有公网IP，不能被非局域网内的其他用户访问。<br />用户通过访问服务端的`frps`，由`frp`负责根据请求的端口或其他信息将请求路由到对应的内网机器，从而实现通信。<br />在`frp`中一个代理对应一个需要暴露的内网服务。一个客户端支持同时配置多个代理。<br />`frp`支持多种代理类型来适配不同的使用场景。

| 类型 | 描述 |
| --- | --- |
| tcp | 单纯的`TCP`端口映射，服务端会根据不同的端口路由到不同的内网服务。 |
| udp | 单纯的`UDP`端口映射，服务端会根据不同的端口路由到不同的内网服务。 |
| http | 针对`HTTP`应用定制了一些额外的功能，例如修改`HostHeader`，增加鉴权。 |
| https | 针对`HTTPS`应用定制了一些额外的功能。 |
| stcp | 安全的`TCP`内网代理，需要在被访问者和访问者的机器上都部署`frpc`，不需要在服务端暴露端口。 |
| sudp | 安全的`UDP`内网代理，需要在被访问者和访问者的机器上都部署`frpc`，不需要在服务端暴露端口。 |
| xtcp | 点对点内网穿透代理，功能同`stcp`，但是流量不需要经过服务器中转。 |
| tcpmux | 支持服务端`TCP`端口的多路复用，通过同一个端口访问不同的内网服务。 |

# 六、配置文件
## 1.基本配置解析
配置文件是`frp`的核心组件，决定了`frp`的运行方式和运行参数。<br />`frp`目前仅支持ini格式的配置文件，`frps`和`frpc`各自支持不同的参数。<br />`frpc`：指客户端(Clinet)配置文件，`frps`：指服务端(Server)配置文件。<br />`frps`主要配置服务端的一些通用参数，`frpc`则需要额外配置每一个代理的详细配置。
```
#[common]是固定名称的段落，用于配置通用参数。
[common]
server_addr = x.x.x.x
server_port = 7000

#[ssh]代表仅在代理ssh时使用的参数，在客户端使用，代理名称唯一且不可重复，但在同一个客户端中可以配置多个代理，比如[remoteDesktop]等
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```
## 2.使用include参数
通过`includes`参数可以在主配置中包含其他配置文件，从而实现将代理配置拆分到多个文件中管理。
```
[common]
server_addr = x.x.x.x
server_port = 7000
includes = ./confd/*.ini
```
```
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```
## 3.负载均衡
可以将多个相同类型的代理加入到同一个group中，从而实现负载均衡的能力。<br />目前支持的代理类型：`tcp`,`http`,`tcpmux`
```
[test1]
type = tcp
local_port = 8080
remote_port = 80
group = web
group_key = 123

[test2]
type = tcp
local_port = 8081
remote_port = 80
group = web
group_key = 123
```
用户连接`frps`服务器的80端口，`frps`会将接收到的用户连接随机分发给其中一个存活的`proxy`。这样可以在一台`frpc`机器挂掉后仍然有其他节点能够提供服务。<br />tcp类型代理要求`group_key`相同，做权限验证，且`remote_port`相同。<br />http类型代理要求`group_key`,`custom_domains`或`subdomain`和`locations`相同。
## 4.通信加密与压缩
```
[ssh]
type = tcp
local_port = 22
remote_port = 6000
use_encryption = true
use_compression = true
```
通过设置`use_encryption`=`true`，将`frpc`与`frps`之间的通信内容加密传输，将会有效防止传输内容被截取。<br />如果传输的报文长度较长，通过设置`use_compression`=`true`对传输内容进行压缩，可以有效减小`frpc`与`frps`之间的网络流量，加快流量转发速度，但是会额外消耗一些CPU资源。
## 5.服务端完整配置
### (1)基础配置
| 参数 | 类型 | 说明 | 默认值 | 可选值 | 备注 |
| --- | --- | --- | --- | --- | --- |
| bind_addr | string | 服务端监听地址 | 0.0.0.0 |  |  |
| bind_port | int | 服务端监听端口 | 7000 |  | 接收frpc的连接 |
| bind_udp_port | int | 服务端监听UDP端口 | 0 |  | 用于辅助创建P2P连接 |
| kcp_bind_port | int | 服务端监听KCP协议端口 | 0 |  | 用于接收采用KCP连接的frpc |
| quic_bind_port | int | 服务端监听QUIC协议端口 | 0 |  | 用于接收采用QUIC连接的frpc |
| quic_keepalive_period | int | quic协议keepalive间隔，单位:秒 | 10 |  |  |
| quic_max_idle_timeout | int | quic协议的最大空闲超时时间，单位:秒 | 30 |  |  |
| quic_max_incoming_streams | int | quic协议最大并发stream数 | 100000 |  |  |
| proxy_bind_addr | string | 代理监听地址 | 同bind_addr |  | 可以使代理监听在不同的网卡地址 |
| log_file | string | 日志文件地址 | ./frps.log |  | 如果设置为console，会将日志打印在标准输出中 |
| log_level | string | 日志等级 | info | trace,debug,info,warn,error |  |
| log_max_days | int | 日志文件保留天数 | 3 |  |  |
| disable_log_color | bool | 禁用标准输出中的日志颜色 | false |  |  |
| detailed_errors_to_client | bool | 服务端返回详细错误信息给客户端 | true |  |  |
| tcp_mux_keepalive_interval | int | tcp_mux的心跳检查间隔时间 | 60 |  | 单位：秒 |
| tcp_keepalive | int | 和客户端底层TCP连接的keepalive间隔时间，单位秒 | 7200 |  | 负数不启用 |
| heartbeat_timeout | int | 服务端和客户端心跳连接的超时时间 | 90 |  | 单位：秒 |
| user_conn_timeout | int | 用户建立连接后等待客户端响应的超时时间 | 10 |  | 单位：秒 |
| udp_packet_size | int | 代理UDP服务时支持的最大包长度 | 1500 |  | 服务端和客户端的值需要一致 |
| tls_cert_file | string | TLS服务端证书文件路径 |  |  |  |
| tls_key_file | string | TLS服务端密钥文件路径 |  |  |  |
| tls_trusted_ca_file | string | TLSCA证书路径 |  |  |  |

### (2)权限配置
| 参数 | 类型 | 说明 | 默认值 | 可选值 | 备注 |
| --- | --- | --- | --- | --- | --- |
| authentication_method | string | 鉴权方式 | token | token, oidc |  |
| authenticate_heartbeats | bool | 开启心跳消息鉴权 | false |  |  |
| authenticate_new_work_conns | bool | 开启建立工作连接的鉴权 | false |  |  |
| token | string | 鉴权使用的 token 值 |  |  | 客户端需要设置一样的值才能鉴权通过 |
| oidc_issuer | string | oidc_issuer |  |  |  |
| oidc_audience | string | oidc_audience |  |  |  |
| oidc_skip_expiry_check | bool | oidc_skip_expiry_check |  |  |  |
| oidc_skip_issuer_check | bool | oidc_skip_issuer_check |  |  |  |

### (3)管理配置
| 参数 | 类型 | 说明 | 默认值 | 可选值 | 备注 |
| --- | --- | --- | --- | --- | --- |
| allow_ports | string | 允许代理绑定的服务端端口 |  |  | 格式为 1000-2000,2001,3000-4000 |
| max_pool_count | int | 最大连接池大小 | 5 |  |  |
| max_ports_per_client | int | 限制单个客户端最大同时存在的代理数 | 0 |  | 0 表示没有限制 |
| tls_only | bool | 只接受启用了 TLS 的客户端连接 | false |  |  |

### (4)Dashboard 监控
| 参数 | 类型 | 说明 | 默认值 | 可选值 | 备注 |
| --- | --- | --- | --- | --- | --- |
| dashboard_addr | string | 启用 Dashboard 监听的本地地址 | 0.0.0.0 |  |  |
| dashboard_port | int | 启用 Dashboard 监听的本地端口 | 0 |  |  |
| dashboard_user | string | HTTP BasicAuth 用户名 |  |  |  |
| dashboard_pwd | string | HTTP BasicAuth 密码 |  |  |  |
| dashboard_tls_mode | bool | 是否启用 TLS 模式 | false |  |  |
| dashboard_tls_cert_file | string | TLS 证书文件路径 |  |  |  |
| dashboard_tls_key_file | string | TLS 密钥文件路径 |  |  |  |
| enable_prometheus | bool | 是否提供 Prometheus 监控接口 | false |  | 需要同时启用了 Dashboard 才会生效 |
| asserts_dir | string | 静态资源目录 |  |  | Dashboard 使用的资源默认打包在二进制文件中，通过指定此参数使用自定义的静态资源 |
| pprof_enable | bool | 启动 Go HTTP pprof | false |  | 用于应用调试 |

### (5)HTTP & HTTPS
| 参数 | 类型 | 说明 | 默认值 | 可选值 | 备注 |
| --- | --- | --- | --- | --- | --- |
| vhost_http_port | int | 为 HTTP 类型代理监听的端口 | 0 |  | 启用后才支持 HTTP 类型的代理，默认不启用 |
| vhost_https_port | int | 为 HTTPS 类型代理监听的端口 | 0 |  | 启用后才支持 HTTPS 类型的代理，默认不启用 |
| vhost_http_timeout | int | HTTP 类型代理在服务端的 ResponseHeader 超时时间 | 60 |  |  |
| subdomain_host | string | 二级域名后缀 |  |  |  |
| custom_404_page | string | 自定义 404 错误页面地址 |  |  |  |

### (6)TCPMUX
| 参数 | 类型 | 说明 | 默认值 | 可选值 | 备注 |
| --- | --- | --- | --- | --- | --- |
| tcpmux_httpconnect_port | int | 为 TCPMUX 类型代理监听的端口 | 0 |  | 启用后才支持 TCPMUX 类型的代理，默认不启用 |
| tcpmux_passthrough | bool | 是否透传 CONNECT 请求 | false |  | 通常在本地服务是 HTTP Proxy 时使用 |

## 6.客户端完整配置
### (1)基础配置
| 参数 | 类型 | 说明 | 默认值 | 可选值 | 备注 |
| --- | --- | --- | --- | --- | --- |
| server_addr | string | 连接服务端的地址 | 0.0.0.0 |  |  |
| server_port | int | 连接服务端的端口 | 7000 |  |  |
| connect_server_local_ip | string | 连接服务端时所绑定的本地 IP |  |  |  |
| dial_server_timeout | int | 连接服务端的超时时间 | 10 |  |  |
| dial_server_keepalive | int | 和服务端底层 TCP 连接的 keepalive 间隔时间，单位秒 | 7200 |  | 负数不启用 |
| http_proxy | string | 连接服务端使用的代理地址 |  |  | 格式为 {protocol}://user:passwd@192.168.1.128:8080 protocol 目前支持 http、socks5、ntlm |
| log_file | string | 日志文件地址 | ./frpc.log |  | 如果设置为 console，会将日志打印在标准输出中 |
| log_level | string | 日志等级 | info | trace, debug, info, warn, error |  |
| log_max_days | int | 日志文件保留天数 | 3 |  |  |
| disable_log_color | bool | 禁用标准输出中的日志颜色 | false |  |  |
| pool_count | int | 连接池大小 | 0 |  |  |
| user | string | 用户名 |  |  | 设置此参数后，代理名称会被修改为 {user}.{proxyName}，避免代理名称和其他用户冲突 |
| dns_server | string | 使用 DNS 服务器地址 |  |  | 默认使用系统配置的 DNS 服务器，指定此参数可以强制替换为自定义的 DNS 服务器地址 |
| login_fail_exit | bool | 第一次登陆失败后是否退出 | true |  |  |
| protocol | string | 连接服务端的通信协议 | tcp | tcp, kcp, quic, websocket |  |
| quic_keepalive_period | int | quic 协议 keepalive 间隔，单位: 秒 | 10 |  |  |
| quic_max_idle_timeout | int | quic 协议的最大空闲超时时间，单位: 秒 | 30 |  |  |
| quic_max_incoming_streams | int | quic 协议最大并发 stream 数 | 100000 |  |  |
| tls_enable | bool | 启用 TLS 协议加密连接 | false |  |  |
| tls_cert_file | string | TLS 客户端证书文件路径 |  |  |  |
| tls_key_file | string | TLS 客户端密钥文件路径 |  |  |  |
| tls_trusted_ca_file | string | TLS CA 证书路径 |  |  |  |
| tls_server_name | string | TLS Server 名称 |  |  | 为空则使用 server_addr |
| disable_custom_tls_first_byte | bool | TLS 不发送 0x17 | false |  | 当为 true 时，不能端口复用 |
| tcp_mux_keepalive_interval | int | tcp_mux 的心跳检查间隔时间 | 60 |  | 单位：秒 |
| heartbeat_interval | int | 向服务端发送心跳包的间隔时间 | 30 |  | 建议启用 tcp_mux_keepalive_interval，将此值设置为 -1 |
| heartbeat_timeout | int | 和服务端心跳的超时时间 | 90 |  |  |
| udp_packet_size | int | 代理 UDP 服务时支持的最大包长度 | 1500 |  | 服务端和客户端的值需要一致 |
| start | string | 指定启用部分代理 |  |  | 当配置了较多代理，但是只希望启用其中部分时可以通过此参数指定，默认为全部启用 |
| meta_xxx | map | 附加元数据 |  |  | 会传递给服务端插件，提供附加能力 |

### (2)权限验证
| 参数 | 类型 | 说明 | 默认值 | 可选值 | 备注 |
| --- | --- | --- | --- | --- | --- |
| authentication_method | string | 鉴权方式 | token | token, oidc | 需要和服务端一致 |
| authenticate_heartbeats | bool | 开启心跳消息鉴权 | false |  | 需要和服务端一致 |
| authenticate_new_work_conns | bool | 开启建立工作连接的鉴权 | false |  | 需要和服务端一致 |
| token | string | 鉴权使用的 token 值 |  |  | 需要和服务端设置一样的值才能鉴权通过 |
| oidc_client_id | string | oidc_client_id |  |  |  |
| oidc_client_secret | string | oidc_client_secret |  |  |  |
| oidc_audience | string | oidc_audience |  |  |  |
| oidc_scope | string | oidc_scope |  |  |  |
| oidc_token_endpoint_url | string | oidc_token_endpoint_url |  |  |  |
| oidc_additional_xxx | map | OIDC 附加参数 |  |  | map 结构，key 需要以 oidc_additional_ 开头 |

### (3)UI
| 参数 | 类型 | 说明 | 默认值 | 可选值 | 备注 |
| --- | --- | --- | --- | --- | --- |
| admin_addr | string | 启用 AdminUI 监听的本地地址 | 0.0.0.0 |  |  |
| admin_port | int | 启用 AdminUI 监听的本地端口 | 0 |  |  |
| admin_user | string | HTTP BasicAuth 用户名 |  |  |  |
| admin_pwd | string | HTTP BasicAuth 密码 |  |  |  |
| asserts_dir | string | 静态资源目录 |  |  | AdminUI 使用的资源默认打包在二进制文件中，通过指定此参数使用自定义的静态资源 |
| pprof_enable | bool | 启动 Go HTTP pprof | false |  | 用于应用调试 |


# 七、相关指令
| Command | CommandEffect |
| --- | --- |
| ./frpc-c./frpc.ini | 通过配置文件frpc.ini启动frp |
| frpcverify-c./frpc.ini | 通过frpc校验frpc.ini的正确性 |
| frpsverify-c./frps.ini | 通过frps校验frps.ini的正确性 |
| frpc reload -c ./frpc.ini | 根据更新后的配置文件重启 |

