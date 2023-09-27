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

### RuleProvider 支持

- 编译时需要使用 `with_ruleprovider` tag

##### 配置详解
```json5
{
    "ruleproviders": [
        {
            "tag": "rule-provider-x", // 标签，必填，用于区别不同的 rule-provider，不可重复
            "url": "", // 规则订阅链接，必填，仅支持Clash订阅规则
            "behavior": "", // 规则类型，必填，可选 domain / ipcidr / classical
            "format": "". // 规则格式，选填，可选 yaml / text，默认 yaml
            "cache_file": "/tmp/rule-provider-x.cache", // 缓存文件，选填，强烈建议填写，可以加快启动速度
            "update_interval": "4h", // 更新间隔，选填，仅填写 cache_file 有效，若当前缓存文件已经超过该时间，将会进行后台自动更新
            "request_timeout": "10s", // 请求超时时间
            "dns": "tls://223.5.5.5", // 使用自定义 DNS 请求订阅域名，格式与 proxyprovider 相同
            "request_dialer": {}, // 请求时使用的 Dial 字段配置，detour 字段无效
            "running_detour": "" // 运行时后台自动更新所使用的 outbound
        }
    ]
}
```

##### 用法

用于 Route Rule 或者 DNS Rule

假设规则有以下内容：
```yaml
payload:
  - '+.google.com'
  - '+.github.com'
```

```json5
{
    "dns": {
        "rules": [
            {
                "@rule_provider": "rule-provider-x",
                "server": "proxy-dns"
            }
        ]
    },
    "route": {
        "rules": [
            {
                "@rule_provider": "rule-provider-x",
                "outbound": "proxy-out"
            }
        ]
    }
}
```
等效于
```json5
{
    "dns": {
        "rules": [
            {
                "domain_suffix": [
                    ".google.com",
                    ".github.com"
                ],
                "server": "proxy-dns"
            }
        ]
    },
    "route": {
        "rules": [
            {
                "domain_suffix": [
                    ".google.com",
                    ".github.com"
                ],
                "outbound": "proxy-out"
            }
        ]
    }
}
```

##### 注意

- 由于 sing-box 规则支持与 Clash 可能不同，某些无法在 sing-box 上使用的规则会被**自动忽略**，请注意
- 不支持 **logical** 规则，由于规则数目可能非常庞大，设置多个 @rule_provider 靶点可能会导致内存飙升和性能问题（笛卡儿积）
- DNS Rule 不支持某些类型，如：GeoIP IP-CIDR IP-CIDR6，这是因为 sing-box 程序逻辑所决定的
- 目前支持的 Clash 规则类型：

```
Clash 类型       ==>     对于的 sing-box 配置

DOMAIN           ==> domain
DOMAIN-SUFFIX    ==> domain_suffix
DOMAIN-KEYWORD   ==> domain_keyword
GEOSITE          ==> geosite
GEOIP            ==> geoip
IP-CIDR          ==> ip_cidr
IP-CIDR6         ==> ip_cidr
SRC-IP-CIDR      ==> source_ip_cidr
SRC-PORT         ==> source_port
DST-PORT         ==> port
PROCESS-NAME     ==> process_name
PROCESS-PATH     ==> process_path
NETWORK          ==> network
```