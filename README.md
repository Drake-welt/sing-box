# sing-box

The universal proxy platform.

[![Packaging status](https://repology.org/badge/vertical-allrepos/sing-box.svg)](https://repology.org/project/sing-box/versions)

## Documentation

https://sing-box.sagernet.org

## Support

https://community.sagernet.org/c/sing-box/

## License

```
Copyright (C) 2022 by nekohasekai <contact-sagernet@sekai.icu>

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program. If not, see <http://www.gnu.org/licenses/>.

In addition, no derivative work may use the name or imply association
with this application without prior consent.
```

## 额外功能

---

#### 1. SideLoad 出站支持 (with_sideload)
对于 Sing-box 不支持的出站类型，可以通过侧载方式与 Sing-box 共用。只需暴露 Socks 端口，即可与 Sing-box 集成

编译时加入 tag ```with_sideload```

**!! 注意**：若 sing-box 被 kill / 发生panic后退出，侧载的程序并**不会退出**，需要**自行终止**，再重新启动sing-box

<p align="center">
  <img width="350px" src="https://raw.githubusercontent.com/yaotthaha/static/master/sideload.png">
</p>

例子：侧载 tuic 代理

Sing-box 配置：
```
{
  "tag": "sideload-out",
  "type": "sideload",
  "server": "www.example.com", // tuic 服务器地址
  "server_port": 443, // tuic 服务器端口
  "listen_port": 50001, // tuic 本地监听端口
  "listen_network": "udp", // 监听从tuic连接的协议类型，tcp/udp，留空都监听
  "socks5_proxy_port": 50023, // tuic 暴露的socks5代理端口
  "command": [ // tuic 侧启动命令：/usr/bin/tuic --server www.example.com --server-port 50001 --server-ip 127.0.0.1 --token token123 --local-port 50023
    "/usr/bin/tuic",
    "--server",
    "www.example.com",
    "--server-port",
    "50001",
    "--server-ip",
    "127.0.0.1",
    "--token",
    "token123",
    "--local-port",
    "50023"
  ],
  // Dial Fields
}
```

#### 2. Clash Dashboard 内置支持 (with_clash_dashboard)

- 编译时需要使用 `with_clash_dashboard` tag
- 编译前需要先初始化 web 文件

```
使用 yacd 作为 Clash Dashboard：make init_yacd
使用 metacubexd 作为 Clash Dashboard：make init_metacubexd
清除 web 文件：make clean_clash_dashboard
```

##### 用法

```json5
{
    "experimental": {
        "clash_api": {
            "external_controller": "0.0.0.0:9090",
            //"external_ui": "" // 无需填写
            "external_ui_buildin": true // 启用内置 Clash Dashboard
        }
    }
}
```

#### 3. URLTest Fallback 支持
按照**可用性**和**顺序**选择出站

可用：指 URL 测试存在有效结果

配置示例：
```
{
    "tag": "fallback",
    "type": "urltest",
    "outbounds": [
        "A",
        "B",
        "C"
    ],
    "fallback": {
        "enabled": true, // 开启 fallback
        "max_delay": "200ms" // 可选配置
        // 若某节点可用，但是延迟超过 max_delay，则认为该节点不可用，淘汰忽略该节点，继续匹配选择下一个节点
        // 但若所有节点均不可用，但是存在被 max_delay 规则淘汰的节点，则选择延迟最低的被淘汰节点
    }
}
```
以上配置为例子：
1. 当 A, B, C 都可用时，优选选择 A。当 A 不可用时，优选选择 B。当 A, B 都不可用时，选择 C，若 C 也不可用，则返回第一个出站：A
2. (配置了 max_delay) 当 A, C 都不可用，B 延迟超过 200ms 时（在第一轮选择时淘汰，被认为是不可用节点），则选择 B

#### 4. RandomAddr 出站支持 (with_randomaddr)

- 编译时需要使用 `with_randomaddr` tag

支持随机不同 IP:Port 连接，只需要将 Detour 设置为这个出站，即可随机使用不同的 IP:Port 组合连接，需要配合其他出站使用，~~可以躲避基于目的地址的审查~~

```json5
{
    "tag": "randomaddr-out",
    "type": "randomaddr",
    "udp": true, // 为 true 时，替换 NewPakcetConn，开启 UDP 支持
    "ignore_fqdn": false, // 为 true 时，对有 FQDN 的连接不处理
    "delete_fqdn": false, // 为 true 时，删除连接中的 FQDN
    "addresses": [ // 地址重写规则
        {
            "ip": "100.64.0.1", // IP 地址，支持 192.168.2.0/24、192.168.2.0、192.168.2.0-192.168.2.254 三种写法
            "port": 80, // 连接端口
        }
    ],
}
```

用法范例：配合 WebSocket + CloudFront CDN **（请勿滥用，后果自负）**

```json5
[
    {
        "tag": "ws-out",
        "type": "vmess",
        ...
        "transport": {
            "type": "ws",
            ...
        },
        "detour": "randomaddr-out"
    },
    {
        "tag": "randomaddr-out",
        "type": "randomaddr",
        "delete_fqdn": true,
        "addresses": [
            {
                "ip": "13.33.100.0/24",
                "port": 80
            }
        ]
    }
]
```

#### 5. Tor No Fatal 启动

```json
{
    "outbounds": [
        {
            "tag": "tor-out",
            "type": "tor",
            "no_fatal": true // 启动时将 tor outbound 启动置于后台，加快启动速度，但启动失败会导致无法使用
        }
    ]
}
```

#### 6. Geo Resource 自动更新支持

##### 用法
```json5
{
    "route": {
        "geosite": {
            "path": "/temp/geosite.db",
            "auto_update_interval": "12h" // 更新间隔，在程序运行时会间隔时间自动更新
        },
        "geoip": {
            "path": "/temp/geoip.db",
            "auto_update_interval": "12h"
        }
    }
}
```

- 支持在 Clash API 中调用 API 更新 Geo Resource

#### 7. JSTest 出站支持 (with_jstest) (*** 实验性 ***)

JSTest 出站允许用户根据 JS 脚本代码选择出站，依附 JS 脚本，用户可以自定义强大的出站选择逻辑，比如：送中节点规避，流媒体节点选择，等等。

你可以在 jstest/javascript/ 目录下找到一些示例脚本。

- 编译时需要使用 `with_jstest` tag
- JS 脚本请自行测试，慎而又慎，不要随意使用不明脚本，可能会导致安全问题或预期外的问题
- JS 脚本运行需要依赖 JS 虚拟机，内存占用可能会比较大（10-20M 左右，视脚本而定），建议使用时注意内存占用情况

- 专门告知使用送中节点的脚本的用户：请**确保 Google 定位已经正常关闭**，否则运行该脚本可能会**导致上游节点全部送中**，~~尤其是机场用户~~，运行所造成的一切后果概不负责

##### 用法
```json5
{
    "outbounds": [
        {
            "tag": "google-cn-auto-switch",
            "type": "jstest",
            "js_path": "/etc/sing-box/google_cn.js", // JS 脚本路径
            "js_base64": "", // JS 脚本 Base64 编码，若遇到某些存储脚本文件困难的情况，如：使用了移动客户端，可以使用该字段
            "interval": "60s", // 脚本执行间隔
            "interrupt_exist_connections": false // 切换时是否中断已有连接
        }
    ]
}
```

#### 8. Script 脚本支持 (with_script)

Script 脚本允许用户在程序运行时执行脚本，可以用于自定义一些功能。

- 编译时需要使用 `with_script` tag

##### 用法
```json5
{
    "scripts": [
        {
            "tag": "script-x", // 标签，必填，用于区别不同的 script，不可重复
            "command": "/path/to/script", // 脚本命令，必填，绝对路径
            "args": [], // 脚本参数，选填
            "directory": "/path/to/directory", // 脚本工作目录，选填，绝对路径
            "mode": "pre-start", // 运行模式，必填，可选列表如下
            "no_fatal": false, // 忽略脚本是否运行失败，若是运行在整个程序生命周期的脚本，则会在启动失败时退出，会在运行异常退出时程序不强制退出
            "env": { // 环境变量，选填
                "foo": "bar"
            },
            "log": {
                "enabled": false, // 是否启用日志，选填，默认 false
                "stdout_log_level": "info", // stdout 日志等级，选填，可选：trace，debug，info，warn，error，fatal，panic，默认 info
                "stderr_log_level": "error", // stderr 日志等级，选填，可选：trace，debug，info，warn，error，fatal，panic，默认 error
            }
        }
    ]
}
```

##### 运行模式
```
1. pre-start // 在启动其他服务前运行脚本
2. pre-start-service-pre-close // 运行的脚本会持续整个程序的生命周期，在启动其他服务前运行，且在关闭其他服务前停止
3. pre-start-service-post-close // 运行的脚本会持续整个程序的生命周期，在启动其他服务前运行，且在关闭其他服务后停止
4. post-start // 在启动其他服务后运行脚本
5. post-start-service-pre-close // 运行的脚本会持续整个程序的生命周期，在启动其他服务后运行，且在关闭其他服务前停止
6. post-start-service-post-close // 运行的脚本会持续整个程序的生命周期，在启动其他服务后运行，且在关闭其他服务后停止
7. pre-close // 在关闭其他服务前运行脚本
8. post-close // 在关闭其他服务后运行脚本
```
