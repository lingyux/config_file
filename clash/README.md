# Clash 配置模板

三份文件都是独立的完整模板，选择对应版本导入，无需叠加覆写文件。

| 文件 | 用途 | 游戏 / ASMR |
| --- | --- | --- |
| `static-simple.yaml` | Mihomo 通用模板；TUN 接管由客户端设置 | 保留国服游戏与四个 ASMR 域名 |
| `openclash-simple.yaml` | OpenClash：Fake-IP 混合模式，TCP 转发 + UDP TUN，区域绕过大陆 | 保留 |
| `stash-simple.yaml` | Stash iOS 3.6+，使用 Stash 的 VPN、DNS 和测速字段 | 移除专用分流与游戏 DNS 过滤 |

## 使用

1. 将所选模板的 `tag_url`、`wd_url` 替换为 Clash YAML 节点订阅地址。
2. 两个订阅都通过 `use: [tag, wd]` 引入。只使用一个订阅时，删除另一个 provider，并修改 `HandSelect`、`FallBack` 的 `use` 列表。
3. OpenClash 中选择 Fake-IP 混合模式并启用区域绕过大陆；该模板的端口对应提供的运行配置。OpenClash 的界面设置仍会覆写运行配置。
4. Stash 使用专用模板直接导入。DNS 使用 `follow-rule` 和 `geosite:cn`；节点域名通过独立 DNS 解析，避免递归。

## 分流差异

- 通用版与 OpenClash 版将 `asmr-300.com`、`asmr-200.com`、`asmr-100.com`、`asmr.one` 及子域名交给全局手动策略组。
- 删除已被国内域名集覆盖的 QQ / WeGame 补丁，保留 `games_cn` 和 `tencentgames.com` 游戏例外。Riot 国际域名恢复正常代理分流。
- 三份模板均保留国内域名、国内 IP 和私有地址直连兜底。Stash 移除游戏 / ASMR 专用规则后，这些网站仍按普通规则分流。
- OpenClash 版显式定义 `oc-cn-domain`，与插件自动注入的名称、路径和更新周期一致，避免再加载一份 `cn_domain`。通用版与 Stash 版使用自己的 `cn_domain`，不依赖插件注入。
- OpenClash 与 Stash 专用版将 YouTube 合并到 Google / Proxy 分流；通用版仍保留独立的 YouTube 策略组。Stash 版另移除 DLsite 专用规则和订阅。
- Stash 版不包含路由器端口、TUN、嗅探和 Mihomo 专用 DNS 字段。订阅节点的前缀如有需要，在订阅源中设置。

通用业务规则变更时，需同步三份模板。`tag-simple.yaml` 是本地运行配置样本，不随模板提交。

## 验证范围

已检查 YAML 解析、规则集与策略组引用、无未使用的规则集、业务规则优先级、节点筛选正则和三份模板的预期差异。未进行 OpenClash / Stash 实机导入或实际连通测试。

参考：[Stash DNS](https://stash.wiki/features/dns-server)、[Stash 代理集](https://stash.wiki/en/proxy-protocols/proxy-providers)、[Stash 规则集](https://stash.wiki/en/rules/rule-set)。
