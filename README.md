# v2ray-rules

这是一个Fork自 [v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) 的项目。在原项目基础上添加了由Systemd实现的定时更新功能，并且去掉了去广告功能（本人认为这种功能更适合由浏览器插件来进行）。

经测试直接Fork会影响Github Actions运作，故直接新建了一个项目。

自动同步时间：开机后1小时和北京时间每日中午 12 点。

# 简介

**V2Ray** 规则文件加强版，可代替 V2Ray 官方 `geoip.dat` 和 `geosite.dat` 规则文件。利用 GitHub Actions 北京时间每天早上 6 点自动构建，保证规则最新。

## 规则文件生成方式

### geoip.dat

- 通过仓库 [@v2ray/geoip](https://github.com/v2ray/geoip) 生成
- 其中 IP 地址来源于 [MaxMind 免费 IP](https://dev.maxmind.com/geoip/geoip2/geolite2/)

### geosite.dat

- 通过仓库 [@v2ray/domain-list-community](https://github.com/v2ray/domain-list-community) 生成
- **加入大量中国大陆域名、Apple 域名、Google 域名和部分海外 CDN 域名**：
  - [@felixonmars/dnsmasq-china-list/accelerated-domains.china.conf](https://github.com/felixonmars/dnsmasq-china-list/blob/master/accelerated-domains.china.conf) 加入到 `geosite:cn` 类别中
  - [@felixonmars/dnsmasq-china-list/apple.china.conf](https://github.com/felixonmars/dnsmasq-china-list/blob/master/apple.china.conf) 加入到 `geosite:geolocation-!cn` 类别中
  - [@felixonmars/dnsmasq-china-list/google.china.conf](https://github.com/felixonmars/dnsmasq-china-list/blob/master/google.china.conf) 加入到 `geosite:geolocation-!cn` 类别中
  - [@felixonmars/dnsmasq-china-list/removed-cdn.txt](https://github.com/felixonmars/dnsmasq-china-list/blob/master/removed-cdn.txt) 加入到 `geosite:geolocation-!cn` 类别中
- **加入最新 GFWList 域名**：通过仓库 [@cokebar/gfwlist2dnsmasq](https://github.com/cokebar/gfwlist2dnsmasq) 生成并加入到 `geosite:geolocation-!cn` 类别中
- **加入 Greatfire Analyzer 检测到的屏蔽域名**：通过仓库 [@wongsyrone/domain-block-list](https://github.com/wongsyrone/domain-block-list) 获取 [Greatfire Analyzer](https://zh.greatfire.org/analyzer) 检测到的屏蔽域名，并加入到 `geosite:geolocation-!cn` 类别中
- **加入更多代理域名**：通过仓库 [@ConnersHua/Profiles](https://github.com/ConnersHua/Profiles/tree/master)、[@GeQ1an/Rules](https://github.com/GeQ1an/Rules/tree/master/QuantumultX) 和 [@lhie1/Rules](https://github.com/lhie1/Rules/tree/master) 获取更多代理域名，并加入到 `geosite:geolocation-!cn`  类别中
- **加入自定义直连和代理域名**：由于上游域名列表更新缓慢或缺失某些被屏蔽的域名，所以引入自定义域名列表，主要为了解决在 DNS 解析 `A` 和 `AAAA` 记录时的 DNS 泄漏问题。[`hidden` 分支](https://github.com/Loyalsoldier/v2ray-rules-dat/tree/hidden)里有两个文件 `direct.txt` 和 `proxy.txt`，分别放置自定义的直连、代理域名，最终分别加入到 `geosite:cn` 和 `geosite:geolocation-!cn` 类别中

### 如何使用：

* Linux

```shel
git clone https://github.com/TonyAble/v2ray-rules.git && cd v2ray-rules
sudo cp v2ray-rules.* /usr/lib/systemd/system/
sudo systemctl enable v2ray-rules.timer
sudo systemctl start v2ray-rules.timer
sudo systemctl enable v2ray-rules.service
sudo systemctl start v2ray-rules.service
```

* Windows
  * 直接替换dat文件
  * 自动化脚本待开发
* Mac OS
  * 待开发

## 参考配置

### geoip.dat

跟 V2Ray 官方 `geoip.dat` 配置方式相同。

**Routing 配置方式**：

```json
"routing": {
  "rules": [
    {
      "type": "field",
      "outboundTag": "Direct",
      "ip": [
        "geoip:cn",
        "geoip:private"
      ]
    }
  ]
}
```

### geosite.dat

跟 V2Ray 官方 `geosite.dat` 配置方式相同。

**Routing 配置方式**：

```json
"routing": {
  "rules": [
    {
      "type": "field",
      "outboundTag": "Reject",
      "domain": [
        "geosite:category-ads-all"
      ]
    },
    {
      "type": "field",
      "outboundTag": "Proxy",
      "domain": [
        "geosite:geolocation-!cn"
      ]
    },
    {
      "type": "field",
      "outboundTag": "Direct",
      "domain": [
        "geosite:cn"
      ]
    }
  ]
}
```

**DNS 配置方式**：

```json
"dns": {
  "servers": [
    {
      "address": "1.1.1.1",
      "port": 53,
      "domains": [
        "geosite:geolocation-!cn"
      ]
    },
    {
      "address": "114.114.114.114",
      "port": 53,
      "domains": [
        "geosite:cn"
      ]
    },
    "8.8.8.8",
    "223.5.5.5"
  ]
}
```

### 自用 V2Ray 客户端完整配置（仅供参考，根据自身需求酌情修改）

下面为自用 V2Ray 客户端完整配置，注意事项：

- 在本机环境上 V2Ray 的 DoH 解析功能依然有问题，故并未启用。
- 下面客户端配置使 V2Ray 在本机开启 SOCKS 代理（监听 1080 端口）和 HTTP 代理（监听 2080 端口）
- BT 流量统统直连（实测依然会有部分 BT 流量走代理，尚不清楚是不是 V2Ray 的 bug。如果服务商禁止 BT 下载的话，请不要为下载软件设置代理）
- 最后，不命中任何路由规则的请求和流量，统统走代理
- `outbounds` 里的第一个大括号内的配置，即为 V2Ray 代理服务的配置。请根据自身需求进行修改，并参照 V2Ray 官网配置说明中的 [配置文件 > 文件格式 > OutboundObject](https://v2ray.com/chapter_02/01_overview.html#outboundobject) 部分进行补全

```json
{
  "log": {
    "loglevel": "warning"
  },
  "dns": {
    "servers": [
      {
        "address": "1.1.1.1",
        "domains": [
          "geosite:geolocation-!cn",
          "geosite:speedtest"
        ]
      },
      "1.1.1.1",
      "8.8.8.8",
      {
        "address": "114.114.114.114",
        "port": 53,
        "domains": [
          "geosite:cn"
        ]
      }
    ]
  },
  "inbounds": [
    {
      "protocol": "socks",
      "listen": "127.0.0.1",
      "port": 1080,
      "tag": "Socks-In",
      "settings": {
        "ip": "127.0.0.1",
        "udp": true,
        "auth": "noauth"
      },
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      }
    },
    {
      "protocol": "http",
      "listen": "127.0.0.1",
      "port": 2080,
      "tag": "Http-In",
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      }
    }
  ],
  "outbounds": [
    {
      //下面这行，协议名称为socks、shadowsocks或vmess等（记得删除这行文字说明）
      "protocol": "协议名称",
      "settings": {},
      //下面这行，必须为Proxy，对应Routing里的outboundTag（记得删除这行文字说明）
      "tag": "Proxy",
      "streamSettings": {},
      "mux": {}
    },
    {
      "protocol": "dns",
      "tag": "Dns-Out"
    },
    {
      "protocol": "freedom",
      "tag": "Direct",
      "settings": {
        "domainStrategy": "UseIPv4"
      }
    },
    {
      "protocol": "blackhole",
      "tag": "Reject",
      "settings": {
        "response": {
          "type": "http"
        }
      }
    }
  ],
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      {
        "type": "field",
        "outboundTag": "Dns-Out",
        "inboundTag": [
          "Socks-In",
          "Http-In"
        ],
        "network": "udp",
        "port": 53
      },
      {
        "type": "field",
        "outboundTag": "Direct",
        "protocol": ["bittorrent"]
      },
      {
        "type": "field",
        "outboundTag": "Proxy",
        "domain": [
          "geosite:geolocation-!cn",
          "geosite:speedtest"
        ]
      },
      {
        "type": "field",
        "outboundTag": "Direct",
        "domain": [
          "geosite:cn"
        ]
      },
      {
        "type": "field",
        "outboundTag": "Direct",
        "ip": [
          "223.5.5.5/32",
          "119.29.29.29/32",
          "180.76.76.76/32",
          "114.114.114.114/32"
        ]
      },
      {
        "type": "field",
        "outboundTag": "Proxy",
        "ip": [
          "1.1.1.1/32",
          "1.0.0.1/32",
          "8.8.8.8/32",
          "8.8.4.4/32"
        ]
      },
      {
        "type": "field",
        "outboundTag": "Direct",
        "ip": [
          "geoip:cn",
          "geoip:private"
        ]
      },
      {
        "type": "field",
        "outboundTag": "Proxy",
        "ip": [
          "0.0.0.0/0",
          "::/0"
        ]
      }
    ]
  }
}
```

## 致谢

- [@Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat)
- [@v2ray/geoip](https://github.com/v2ray/geoip)
- [@v2ray/domain-list-community](https://github.com/v2ray/domain-list-community)
- [@felixonmars/dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list)
- [@cokebar/gfwlist2dnsmasq](https://github.com/cokebar/gfwlist2dnsmasq)
- [@wongsyrone/domain-block-list](https://github.com/wongsyrone/domain-block-list)
- [@ConnersHua/Profiles](https://github.com/ConnersHua/Profiles/tree/master)
- [@GeQ1an/Rules](https://github.com/GeQ1an/Rules/tree/master/QuantumultX)
- [@lhie1/Rules](https://github.com/lhie1/Rules/tree/master)
- [MaxMind 免费 IP](https://dev.maxmind.com/geoip/geoip2/geolite2/)