## IOX

[iox | Github](https://github.com/EddieIvan01/iox)

### 正向连接

一层正向连接

```cmd
# 边界机器
iox.exe proxy -l [port]

iox.exe proxy -l 1000 // Socks5 Server
```

二层正向连接

```cmd
# 边界机器
iox.exe fwd -l [port] -r [next machine ip:port]

iox.exe fwd -l 1000 -r 10.10.2.4:2000

# 二层机器
iox.exe proxy -l [port]

iox.exe proxy -l 2000 // Socks5 Server
```

三层正向连接

```cmd
# 边界机器
iox.exe fwd -l [port] -r [next machine ip:port]

iox.exe fwd -l 1000 -r 10.10.2.4:2000

# 二层机器
iox.exe fwd -l [port] -r [next machine ip:port]

iox.exe proxy -l 2000 -r 10.10.3.4:3000

# 三层机器
iox.exe proxy -l [port]

iox.exe proxy -l 3000 // Socks5 Server
```

### 反向连接

一层反向连接

```cmd
# VPS
iox.exe proxy -l [remote port] -l [local port]

iox.exe proxy -l 1000 -l 1080 // Socks5 Server

# 边界机器
iox.exe proxy -r VPS[ip:port]

iox.exe proxy -r 1.1.1.1:1000
```

二层反向连接

```cmd
# VPS
iox.exe proxy -l [remote port] -l [local port]

iox.exe proxy -l 1000 -l 1080 // Socks5 Server

# 边界机器
iox.exe fwd -l [local port] -r VPS[ip:port]

iox.exe fwd -l 2000 -r 1.1.1.1:1000
```

三层反向连接

```cmd
# VPS
iox.exe proxy -l [remote port] -l [local port]

iox.exe proxy -l 1000 -l 1080 // Socks5 Server

# 边界机器
iox.exe fwd -l [local port] -r VPS[ip:port]

iox.exe fwd -l 2000 -r 8.8.8.8:1000

# 二层机器
iox.exe fwd -l [local port] -r [upper machine ip:port]

iox.exe fwd -l 3000 -r 1.1.1.1:2000
```

## FRP

[frp | Github](https://github.com/fatedier/frp)

### 配置解析

#### frps.toml

[frps_full_example.toml | Github](https://github.com/fatedier/frp/blob/dev/conf/frps_full_example.toml)

```toml
bindAddr = "0.0.0.0"
bindPort = [port]
# 验证 token
auth.token = "token"

# 默认为 127.0.0.1，如果需要公网访问，需要修改为 0.0.0.0。
webServer.addr = "0.0.0.0"
webServer.port = [port]

# dashboard 用户名密码，可选，默认为空
webServer.user = "[username]"
webServer.password = "[password]"
```

#### frpc.toml

[frpc_full_example.toml |Github](https://github.com/fatedier/frp/blob/dev/conf/frpc_full_example.toml)

```toml
# frps.toml
bindAddr = "ip" # Default 0.0.0.0
bindPort = [port]
auth.token = "[token]"

# frpc.toml
serverAddr = "ip"
serverPort = [port]
# 验证 token
auth.token = "token"
# frpc 内置的 Admin UI 
webServer.addr = "[server ip]"
webServer.port = [port]
webServer.user = "[username]"
webServer.password = "[password]"

[[proxies]]
name = "plugin_socks5"
type = "tcp"
remotePort = [port]
transport.useEncryption = true
transport.useCompression = true
[proxies.plugin]
type = "socks5"
username = "[username]"
password = "[password]"

[[proxies]]
name = "plugin_http_proxy"
type = "tcp"
remotePort = [port]
transport.useEncryption = true
transport.useCompression = true
[proxies.plugin]
type = "http_proxy"
httpUser = "[username]"
httpPassword = "[password]"
```

### 配置校验

```cmd
frps verify -c frps.toml
# frps: the configuration file frps.toml syntax is ok

frpc verify -c frpc.toml 
# frpc: the configuration file frpc.toml syntax is ok
```

### 正向

二层代理

```toml
# frps.toml
bindAddr = "0.0.0.0"
bindPort = 7000

# frpc.toml
serverAddr = "10.10.1.3"
serverPort = 7000

[[proxies]]
name = "plugin_socks5"
type = "tcp"
remotePort = 3000 # 边界机器会在 frpc 连接之后开放该代理端口
transport.useEncryption = true
transport.useCompression = true
[proxies.plugin]
type = "socks5"
```

三层代理

```toml
# 二层代理
# frps.toml
bindAddr = "0.0.0.0"
bindPort = 7000

# frpc.toml
serverAddr = "10.10.1.2"
serverPort = 7000

[[proxies]]
name = "plugin_socks5"
type = "tcp"
remotePort = 3000 # 边界机器会在 frpc 连接之后开放该代理端口
transport.useEncryption = true
transport.useCompression = true
[proxies.plugin]
type = "socks5"

# 在二层的基础上新增 frps.toml
# frps.toml
bindAddr = "0.0.0.0"
bindPort = 7000

# 三层
# frpc.toml
serverAddr = "10.10.1.3"
serverPort = 7000

[[proxies]]
name = "plugin_socks5"
type = "tcp"
remotePort = 3000 # 边界机器会在 frpc 连接之后开放该代理端口
transport.useEncryption = true
transport.useCompression = true
[proxies.plugin]
type = "socks5"
```

