# Clash 配置模板

四份文件都是独立的完整模板，选择对应版本导入，无需叠加覆写文件。Nikki 的界面覆写设置需与下文保持一致。

| 文件 | 用途 | 游戏 / ASMR |
| --- | --- | --- |
| `static-simple.yaml` | Mihomo 通用模板；TUN 接管由客户端设置 | 保留国服游戏与四个 ASMR 域名 |
| `openclash-simple.yaml` | OpenClash：Fake-IP 混合模式，TCP 转发 + UDP TUN，区域绕过大陆 | 保留 |
| `nikki-simple.yaml` | Nikki 常规模式：Fake-IP，TCP Redirect + UDP TUN，由 Nikki 管理路由和防火墙 | 保留 |
| `stash-simple.yaml` | Stash iOS 3.6+，使用 Stash 的 VPN、DNS 和测速字段 | 移除专用分流与游戏 DNS 过滤 |

## 使用

1. 将所选模板的 `tag_url`、`wd_url` 替换为 Clash YAML 节点订阅地址。
2. 两个订阅都通过 `use: [tag, wd]` 引入。只使用一个订阅时，删除另一个 provider，并修改 `HandSelect`、`FallBack` 的 `use` 列表。
3. OpenClash 中选择 Fake-IP 混合模式并启用区域绕过大陆；该模板的端口对应提供的运行配置。OpenClash 的界面设置仍会覆写运行配置。
4. Stash 使用专用模板直接导入。DNS 使用 `follow-rule` 和 `geosite:cn`；节点域名通过独立 DNS 解析，避免递归。

## Nikki 导入

`openclash-simple.yaml` 的 Mihomo 语法、分组和分流可以复用，但它的端口、TUN 设备名和 DNS 监听按 OpenClash 设置。Nikki 使用独立的 `nikki-simple.yaml`，保留原文件的业务规则、分组、节点筛选、Apple 推送和游戏例外，包括原文件现有的 `proxylite`。

1. 在 `nikki-simple.yaml` 的两个 provider 中填写真实订阅，上传到 Nikki 的配置文件页面并选中。
2. 使用 Nikki 常规模式，关闭“仅内核”；TCP 选 Redirect，UDP 选 TUN，开启 IPv4 代理和 IPv4 DNS 劫持。模板自身不设置自动路由，不能在“仅内核”模式下独立接管流量。
3. 对照下表调整 Nikki 覆写设置。端口、IPv6、TUN、DNS 等会被界面设置覆盖，不能只修改 YAML。
4. 保留模板中的 DNS 服务器、策略和 Fake-IP 过滤列表，不启用对应列表的替换开关；保留原规则和分组，不添加额外覆写规则。大陆 IP 可由模板规则直连；如需复现原 OpenClash 的区域绕过，可另在 Nikki 代理设置中开启大陆 IPv4 绕过。
5. 切换前停用 Momo 等其他透明代理服务，再启用 Nikki。当前 Momo 使用的 DNS 1053 和混合端口 7890 与此模板相同，不能同时监听。当前 dnsmasq 已转发到 1053，切换时保留该设置，避免强制延长 Fake-IP TTL。

| Nikki 设置 | 对应 UCI 字段 | 值 |
| --- | --- | --- |
| 全局 IPv6、DNS IPv6 | `nikki.mixin.ipv6`、`nikki.mixin.dns_ipv6` | `0` / 关闭 |
| IPv6 代理、IPv6 DNS 劫持 | `nikki.proxy.ipv6_proxy`、`nikki.proxy.ipv6_dns_hijack` | `0` / 关闭 |
| TUN 设备、协议栈 | `nikki.mixin.tun_device`、`nikki.mixin.tun_stack` | `nikki`、`system` |
| TUN DNS 劫持列表覆写 | `nikki.mixin.tun_dns_hijack` | `0` / 关闭；由 Nikki 防火墙劫持 DNS |
| DNS 监听 | `nikki.mixin.dns_listen` | `0.0.0.0:1053` |
| DNS 遵循规则 | `nikki.mixin.dns_respect_rules` | `1` / 开启 |
| 直连 DNS 遵循策略 | `nikki.mixin.dns_direct_nameserver_follow_policy` | `1` / 开启 |
| 策略选择、Fake-IP 缓存 | `nikki.mixin.selection_cache`、`nikki.mixin.fake_ip_cache` | `1` / 开启 |
| DNS 服务器、DNS 策略、Fake-IP 过滤列表覆写 | `nikki.mixin.dns_nameserver`、`nikki.mixin.dns_nameserver_policy`、`nikki.mixin.fake_ip_filter` | `0` / 关闭，沿用文件内容 |

HTTP / SOCKS / 混合 / Redirect / TProxy 端口采用 Nikki 默认的 `8080 / 1080 / 7890 / 7891 / 7892`。API 默认使用 9090，由 Nikki 的 API 密钥覆盖文件中的 `CHANGE_ME`；HTTP/SOCKS 鉴权也使用 Nikki 的界面设置。不要将占位密钥作为正式密钥使用。

中国域名规则集改名为 `cn_domain`，显式定义并使用独立缓存路径，不依赖 OpenClash 注入。节点缓存使用 `./providers/proxy/`，全部规则缓存使用 `./providers/rule/`，相对路径均基于 `/etc/nikki/run/`。开启 Fake-IP 持久化；从 Momo 切换时两套核心不共享映射，客户端应清空 DNS 缓存或重连网络。

Nikki 版移除了 Mihomo 1.19.32 已不再支持的 `global-client-fingerprint`，TLS 指纹沿用订阅节点各自的 `client-fingerprint`；原 OpenClash 文件未修改。

## 分流差异

- 通用版与 OpenClash 版将 `asmr-300.com`、`asmr-200.com`、`asmr-100.com`、`asmr.one` 及子域名交给全局手动策略组。
- 删除已被国内域名集覆盖的 QQ / WeGame 补丁，保留 `games_cn` 和 `tencentgames.com` 游戏例外。Riot 国际域名恢复正常代理分流。
- 四份模板均保留国内域名、国内 IP 和私有地址直连兜底。Stash 移除游戏 / ASMR 专用规则后，这些网站仍按普通规则分流。
- OpenClash 版显式定义 `oc-cn-domain`，与插件自动注入的名称、路径和更新周期一致，避免再加载一份 `cn_domain`。通用版与 Stash 版使用自己的 `cn_domain`，不依赖插件注入。
- OpenClash 版使用完整 `apple` 规则集，将 iCloud 与账户服务纳入 Apple 分组（默认 DIRECT）；`push.apple.com` 及子域名优先直连。已有策略选择会被保存，重新导入后需确认 Apple 分组选为 DIRECT，并更新 Apple 规则集。
- OpenClash 版另为 APNs 推送域名使用国内 DNS 并排除 Fake-IP，用于排查 iCloud for Windows 初始化时的推送注册失败。应用后需重启 OpenClash、清理 Windows DNS 缓存并重启 iCloud 后台；该调整尚未经过实际注册验证。
- OpenClash 与 Stash 专用版将 YouTube 合并到 Google / Proxy 分流；通用版仍保留独立的 YouTube 策略组。Stash 版另移除 DLsite 专用规则和订阅。
- Stash 版不包含路由器端口、TUN、嗅探和 Mihomo 专用 DNS 字段。订阅节点的前缀如有需要，在订阅源中设置。

通用业务规则变更时，需同步四份模板。`tag-simple.yaml` 是本地运行配置样本，不随模板提交。

## 验证范围

已检查 YAML 解析、规则集与策略组引用、无未使用的规则集、业务规则优先级、节点筛选正则和三份模板的预期差异。未进行 OpenClash / Stash 实机导入或实际连通测试。

2026-11-03：Nikki 版保留原 OpenClash 文件的 33 个策略组和 44 条分流规则，规则列表仅将 `oc-cn-domain` 引用改为 `cn_domain`。35 个规则集引用和缓存路径均已检查。使用路由器上的 Mihomo 1.19.32 执行 `-t`，原始模板以及在隔离 UCI 目录按上表设置、经设备实际 Nikki 覆写脚本合并后的配置均通过；未启用 Nikki，未验证真实订阅下载、规则更新和 Nikki 实际接管流量。

参考：[Nikki](https://github.com/nikkinikki-org/OpenWrt-nikki)、[Nikki 覆写实现](https://github.com/nikkinikki-org/OpenWrt-nikki/blob/main/nikki/files/ucode/mixin.uc)、[Stash DNS](https://stash.wiki/features/dns-server)、[Stash 代理集](https://stash.wiki/en/proxy-protocols/proxy-providers)、[Stash 规则集](https://stash.wiki/en/rules/rule-set)。
