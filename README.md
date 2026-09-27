# Surge / Stash / Shadowrocket 配置

自用代理托管配置。Surge、Stash、Shadowrocket 三份配置使用同一套策略组，分流思路一致。

| 客户端 | 托管链接 |
| --- | --- |
| Surge iOS / Mac 5+ | `https://raw.githubusercontent.com/Feb17/proxy/main/Surge.conf` |
| Stash iOS / macOS | `https://raw.githubusercontent.com/Feb17/proxy/main/Stash.yaml` |
| Shadowrocket iOS | `https://raw.githubusercontent.com/Feb17/proxy/main/Shadowrocket.conf` |

> 本仓库是**公开**的。订阅链接一旦写进仓库并推送，任何人都能拿到你的节点。Surge、Stash 配置里的订阅地址都只是占位符，真实链接只保存在设备本地（方法见下文）；Shadowrocket 的订阅在 App 首页添加，配置文件里没有订阅地址。
> 如果不小心把订阅链接推送到了仓库，删除提交没有用（历史记录仍可见），请立即到机场后台**重置订阅链接**。

## 策略组

| 策略组 | 默认 | 说明 |
| --- | --- | --- |
| Proxy | 🚀 自动优选 | 主出口，其余策略组的「Proxy」选项即指向这里。Shadowrocket 中改名为「节点选择」（原因见下文） |
| Apple | DIRECT | 苹果服务；APNs 推送单独走 Proxy |
| Intelligence | 🇺🇸 美国节点 | ChatGPT / Claude / Gemini 等，不提供香港节点（不支持该地区） |
| Telegram | 🚀 自动优选 | |
| Netflix / Disney+ / YouTube / Spotify / GlobalMedia | Proxy | 流媒体 |
| TikTok | 🇯🇵 日本节点 | 不提供香港节点（TikTok 已退出香港） |
| BiliBili | DIRECT | 可切换港台节点看番剧 |
| Microsoft / Gamer / Emby | DIRECT | |
| 🚀 自动优选 | Surge：smart / Stash、Shadowrocket：url-test | 港 / 美 / 日 / 台 / 新加坡节点中自动择优 |
| 🇭🇰 🇺🇸 🇯🇵 🇨🇳 🇰🇷 🇸🇬 地区组 | Surge：smart / Stash、Shadowrocket：url-test | 按节点名称自动归类（Surge、Shadowrocket 在主界面隐藏） |
| ✈️ 我的节点 | select | 订阅中的全部节点（Shadowrocket：App 首页的全部节点） |

## Surge

### 1. 安装托管配置

Surge → 配置 → 从 URL 下载配置 → 粘贴上面的 Surge 托管链接。之后 Surge 每 24 小时自动检查更新（`interval=86400`）。

### 2. 填入订阅链接（不要写进仓库）

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

### 相比原配置的优化

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

## Stash

按 [Stash 官方文档](https://stash.wiki) 的建议编写，策略组与 Surge 版保持一致。

### 1. 安装托管配置

在 iPhone / Mac 上打开一键安装链接：

```
https://link.stash.ws/install-config/raw.githubusercontent.com/Feb17/proxy/main/Stash.yaml
```

或者：Stash → 配置文件 → 从 URL 下载 → 粘贴上面的 Stash 托管链接。

配置首行带有 `#SUBSCRIBED`，Stash 会把它识别为托管配置并定时自动更新（默认 12 小时，可在设置中修改）。

### 2. 用本地覆写填入订阅链接（不要写进仓库）

Stash 的[覆写（Override）](https://stash.wiki/configuration/override)会在加载时合并进配置。字典类型的字段按键递归合并，所以覆写里只需要写订阅地址这一个字段；托管配置更新后，覆写依然生效，不用像 Surge 那样手动同步策略组。

在 Stash 的「覆写」页面新建一个本地覆写，内容如下（把 `url` 换成你的订阅链接）：

```yaml
name: 我的订阅
desc: 为托管配置填入订阅链接，仅保存在本机
proxy-providers:
  Subscription:
    url: https://你的订阅链接
```

启用这个覆写即可。

- 旧版 Stash 如果不能直接新建本地覆写，可以把上面的内容保存为 `订阅.local.stoverride`，通过「文件」App 或 AirDrop 用 Stash 打开导入。仓库的 `.gitignore` 已经忽略 `*.local.stoverride`，但更稳妥的做法是根本不要把它放进仓库目录。
- 没有启用覆写时，`Subscription` 拉不到节点，所有策略组都会退化为 DIRECT：不会报错，但所有流量都直连。
- `Subscription` 是订阅源的内部名称，覆写里的键名必须与之完全一致。

### 与 Surge 版的差异

**规则集：全部使用 domain / ipcidr 类型**

- Stash 官方建议大体量规则一律用 `domain` / `ipcidr` 类型的规则集（索引经过压缩，匹配快、省内存），避免 `classical` 类型（只能逐条顺序匹配）。iOS 网络扩展有 50 MB 内存上限，超出会被系统强制关闭。
- blackmatrix7 的 Clash 版规则里，大多数服务（Telegram、YouTube、Netflix 等）只提供 `classical` 格式，所以这些服务改用 [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat) 的 geosite / geoip 数据。这也是 Stash / mihomo 社区最常用的规则源，每个服务都有纯域名和纯 IP 两种版本。
- GlobalMedia 仍使用 blackmatrix7 的域名集（`_For_Clash` 版本使用 `+.` 前缀，符合 Stash 语法），与 Surge 版同源。
- 国内 IP 兜底改用 `geoip:cn` 的 ipcidr 规则集代替 `GEOIP,CN`，不依赖 App 内 GeoIP 数据库的版本。

**AI 规则**

- 改用 geosite `category-ai-!cn`，覆盖 ChatGPT、Claude、Gemini、Copilot、Grok、Perplexity、Cursor 等。
- Surge 版使用的 EAlyce 列表还收录了 `api.github.com`、`stripe.com`、`pool.ntp.org` 等第三方域名，以及 Cloudflare、Vultr 的整个 ASN，会把大量无关流量送进美国节点；新列表只包含 AI 服务本身的域名，Surge 版里那几条「修正苹果域名误伤」的规则也就不再需要。
- AI 规则放在 GitHub、微软、流媒体规则之前，避免 Copilot 等域名被它们截走。

**地区节点筛选**

- Surge 版正则用了 lookbehind，Stash 中无法确认支持（Go 的 RE2 引擎不支持），因此改写为等价的 RE2 兼容写法 `(^|[^A-Za-z])usa?([^A-Za-z]|$)`。
- 53 个测试用例（节点名正例 + Russia、Status、Network、剩余流量等反例）全部通过，6 个地区组的判定结果与 Surge 版完全一致。
- Stash 没有 Surge 的 smart 类型，地区组和自动优选改用 `url-test`：`tolerance: 50` 表示新节点至少快 50ms 才切换，避免出口 IP 频繁跳动；`lazy: true` 表示策略组闲置时跳过测速，更省电。
- 🚀 自动优选的筛选条件取 5 个地区组正则的并集。Surge 版的自动优选不含城市名，现在「洛杉矶 01」「Tokyo 03」这类只写城市的节点也会参与自动优选。

**其他**

- DNS 按官方建议只配两个国内服务器（阿里 + 腾讯 DoH）。需要代理的域名走 Fake IP，不会在本地解析。`*.lan` 等内网域名交给路由器 DNS 解析。
- NTP 授时按端口（UDP 123）直连，不再逐个列举授时域名。
- 未迁移的 Surge 专属设置：Ponte（Stash 的对应功能是 StashLink，在 App 内设置）、`skip-proxy`（Stash 没有这个配置项，局域网流量由规则直连）。

## Shadowrocket

以 Surge 版为基础改写（两者规则语法基本相同），配置项参考 [Shadowrocket 使用手册](https://github.com/LOWERTOP/Shadowrocket)和官方群组的懒人配置。

### 1. 安装托管配置

Shadowrocket → 配置 → 右上角 ➕ → 粘贴上面的 Shadowrocket 托管链接 → 下载，然后点击这份配置 →「使用配置」。

首页的「全局路由」要选**配置**。选「代理」时所有流量都走首页选中的节点，策略组和分流规则全部失效。

### 2. 在 App 首页添加订阅（不用改配置）

首页 → 右上角 ➕ → 类型选 `Subscribe` → 在 URL 栏填入订阅链接 → 保存。

- Shadowrocket 的订阅保存在 App 里，不写进配置文件，所以不需要占位符，也不需要 Surge 那样的关联配置。
- 地区组、🚀 自动优选和 ✈️ 我的节点都用 `policy-regex-filter` 从首页的**全部节点**（所有订阅 + 本地节点）中筛选，有多个订阅时会一起参与。
- 订阅自动更新：设置 → 订阅 → 自动后台更新（需要在系统「设置 → 通用 → 后台 App 刷新」中允许 Shadowrocket）。

### 3. 自动更新配置

配置中已写入 `update-url`。在 Shadowrocket 设置里的「配置」一项打开**自动后台更新**（间隔 1–7 天，同样依赖后台 App 刷新），也可以点击配置文件 →「更新配置」手动更新。

更新配置会用仓库版本**覆盖**本地对这份配置的所有修改。如果需要长期保留自己的改动，可以用「扩展配置」新建一份包含本配置的本地配置，把改动写在那一份里。

### 4. （可选）更换 GeoIP 数据库

Surge 版使用只含中国大陆 IP 的 Hackl0us 数据库；Shadowrocket 内置了通用的 GeoIP 数据库，也可以换成同一个：设置 → GeoLite2 数据库 → 在「国家」的 URL 位置填入下面的链接 → 更新。

```
https://github.com/Hackl0us/GeoIP2-CN/raw/release/Country.mmdb
```

### 与 Surge 版的差异

**主出口改名为「节点选择」**

- Shadowrocket 的策略名不区分大小写（官方懒人配置中的 `YouTube` 分组，在规则里写作 `YOUTUBE`），自定义的 `Proxy` 分组会与内置策略 `PROXY`（首页选中的节点）重名，所以改名为「节点选择」。
- 其余策略组的「节点选择」选项、`FINAL` 兜底都指向它，作用与 Surge 版的 Proxy 相同。

**规则集**

- 改用 blackmatrix7 为 Shadowrocket 生成的版本（`rule/Shadowrocket/`），分流顺序、策略指向与 Surge 版一致。
- Apple、GlobalMedia 由 Surge 版的 `_All_No_Resolve` 单个 RULE-SET 拆分为 `_Domain.list`（DOMAIN-SET）+ `.list`（RULE-SET），这是 blackmatrix7 对 Shadowrocket 的推荐用法。
- AI 规则与 Surge 版相同（EAlyce），修正苹果域名误伤和 NTP 授时的几条规则也一并保留。
- Surge 内置的 `RULE-SET,LAN` 改用 blackmatrix7 的 `Lan_Resolve.list`，位置不变：IP 规则不带 `no-resolve`，解析到内网 IP 的域名（如 NAS 自定义域名）也能直连。
- APNs 规则与 Stash 版一样直接写进配置（内容同 mrbruce516/apns-fix）。IPv6 网段统一写作 `IP-CIDR`，blackmatrix7 的 Shadowrocket 规则集也是这样写的。

**策略组**

- Shadowrocket 没有 smart 类型，地区组和自动优选改用 `url-test`，每 10 分钟测速一次，`tolerance=50`（新节点至少快 50ms 才切换）。
- 地区筛选正则与 Stash 版完全相同（不用 lookbehind 的写法），🚀 自动优选同样取 5 个地区组正则的并集，只写城市名的节点也会参与。
- 地区组加了 `hidden=1`，与 Surge 版一样不在分组列表中显示，但仍可在其他策略组里选择。

**网络设置**

- DNS：Shadowrocket 的 `dns-server` 可以直接填 DoH，阿里 + 腾讯 DoH 并行查询；两台都失败或超过 2 秒时回退到系统 DNS。
- 新增 `tun-excluded-routes`：局域网、组播等网段不进入 Shadowrocket 的 TUN，与 AirPlay、局域网设备发现等功能的兼容性更好。
- 新增 `block-quic = all-proxy`：走代理的连接屏蔽 QUIC，回落到 HTTP/2；直连不受影响。
- 未迁移的 Surge 专属设置：Ponte、DOMAIN-SET 的 `extended-matching`、`FINAL` 的 `dns-failed`、策略组图标 `icon-url`。

## 规则来源

- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)
- [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat)（Stash：geosite / geoip 规则集）
- [EAlyce/conf](https://github.com/EAlyce/conf)（Surge、Shadowrocket：AI 规则）
- [mrbruce516/apns-fix](https://github.com/mrbruce516/apns-fix)、[Stash 文档](https://stash.wiki/faq/ios-push-notifications)（APNs 推送）
- [Hackl0us/GeoIP2-CN](https://github.com/Hackl0us/GeoIP2-CN)（Surge、Shadowrocket 可选：GeoIP 数据库）
- [LOWERTOP/Shadowrocket](https://github.com/LOWERTOP/Shadowrocket)（Shadowrocket 使用手册与配置项说明）
