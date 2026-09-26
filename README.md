# Surge 配置

自用 Surge 托管配置，适用于 Surge iOS / Mac 5 及以上版本。

托管链接：

```
https://raw.githubusercontent.com/Feb17/proxy/main/Surge.conf
```

## 使用方法

### 1. 安装托管配置

Surge → 配置 → 从 URL 下载配置 → 粘贴上面的链接。之后 Surge 每 24 小时自动检查更新（`interval=86400`）。

### 2. 填入订阅链接（不要写进仓库）

本仓库是**公开**的。订阅链接一旦写进 `Surge.conf` 并推送，任何人都能拿到你的节点。
仓库里的 `✈️ 我的节点` 只是占位符，真实订阅链接放在设备本地：

在 Surge 中新建一个本地配置，`[General]`、`[Ponte]`、`[Rule]` 引用托管配置（随仓库自动更新），只有 `[Proxy Group]` 保存在本地：

```ini
[General]
#!include Surge.conf

[Ponte]
#!include Surge.conf

[Proxy Group]
# 把 Surge.conf 中 [Proxy Group] 整段复制到这里，
# 只把「✈️ 我的节点」的 policy-path 换成你的真实订阅链接

[Rule]
#!include Surge.conf
```

- `Surge.conf` 为托管配置在 Surge 配置列表中的文件名，如不同请替换。
- 也可以在 Surge 中对托管配置直接「创建关联配置」，效果相同。
- 仓库中的 `[Proxy Group]` 有改动（如新增策略组）时，需要手动同步到本地。
- 如果不小心把订阅链接推送到了仓库，删除提交没有用（历史记录仍可见），请立即到机场后台**重置订阅链接**。

## 策略组

| 策略组 | 默认 | 说明 |
| --- | --- | --- |
| Proxy | 🚀 自动优选 | 主出口，其余策略组的「Proxy」选项即指向这里 |
| Apple | DIRECT | 苹果服务；APNs 推送单独走 Proxy |
| Intelligence | 🇺🇸 美国节点 | ChatGPT / Claude / Gemini 等，不提供香港节点（不支持该地区） |
| Telegram | 🚀 自动优选 | |
| Netflix / Disney+ / YouTube / Spotify / GlobalMedia | Proxy | 流媒体 |
| TikTok | 🇯🇵 日本节点 | 不提供香港节点（TikTok 已退出香港） |
| BiliBili | DIRECT | 可切换港台节点看番剧 |
| Microsoft / Gamer / Emby | DIRECT | |
| 🚀 自动优选 | smart | 港 / 美 / 日 / 台 / 新加坡节点中自动择优 |
| 🇭🇰 🇺🇸 🇯🇵 🇨🇳 🇰🇷 🇸🇬 地区组 | smart | 按节点名称自动归类，在主界面隐藏 |
| ✈️ 我的节点 | select | 订阅中的全部节点 |

## 相比原配置的优化

**性能**

- 大陆域名规则由 `ChinaMax_All.list`（约 12.4 万条 RULE-SET，3.3 MB）拆分为 `ChinaMax_Domain.list`（DOMAIN-SET）+ `ChinaMax.list`（IP 等规则）。DOMAIN-SET 专为大体量纯域名设计，匹配更快、内存占用更低，对 iOS 网络扩展的内存限制尤其友好。
- 境外规则 `Proxy_All_No_Resolve.list` 同样拆分为 DOMAIN-SET + RULE-SET。
- `RULE-SET,LAN` 包含 IP 规则，放在最前面会让每个请求都先做一次 DNS 解析。现在把它移到所有域名规则之后、`GEOIP,CN` 之前；最前面保留带 `no-resolve` 的局域网网段，直接访问 IP 的请求照样直连。

**地区节点筛选**

- 原正则 `(?i)US` 会把 Russia、Australia、Belarus、Cyprus，甚至「Status」这类信息节点误判为美国节点；`TW` 会误匹配 Network；`🇨🇳` 会把大陆中转节点归入台湾组。
- 新正则要求地区缩写前后不能是英文字母，并补充了繁体、城市名（洛杉矶、东京、首尔等）和 `HKG` / `USA` / `JPN` 写法。35 个常见节点名测试用例：原正则正确 24 个，新正则全部正确。

**分流修正**

- Intelligence：ChatGPT、Claude、Gemini 均不支持香港，原来默认的 Proxy（自动优选）经常选到香港节点导致无法使用。现在默认美国节点，并移除香港选项。
- TikTok：已退出香港市场，默认改为日本节点，并移除香港选项。
- AI 规则集收录了 `pool.ntp.org`，NTP 授时会被送进代理节点（节点不支持 UDP 时直接被拒绝），现已修正为直连。

**网络设置**

- 加密 DNS 增加腾讯 DoH（`doh.pub`），与阿里 DoH 互为备份。
- `skip-proxy` 补充 `100.64.0.0/10`（CGNAT / Tailscale）、IPv6 私有网段和 `captive.apple.com`（公共 Wi-Fi 认证页）。
- `always-real-ip` 补充 Windows 联网检测、STUN / TURN 打洞域名，减少 Fake IP 导致的联机、语音通话问题。
- GeoIP 数据库改用 raw 直链，少一次 github.com 跳转。
- 删去对智能策略组无效的 `update-interval` 等冗余参数。

## 规则来源

- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)
- [EAlyce/conf](https://github.com/EAlyce/conf)（AI 规则）
- [mrbruce516/apns-fix](https://github.com/mrbruce516/apns-fix)（APNs 推送）
- [Hackl0us/GeoIP2-CN](https://github.com/Hackl0us/GeoIP2-CN)（GeoIP 数据库）
