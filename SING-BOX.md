# sing-box MT（iOS）测试模板

[下载 sing-box-ios.json](https://raw.githubusercontent.com/issachigga/shadowrocket-config/main/sing-box-ios.json)

面向官方 **sing-box MT**，以 sing-box **1.14.x** 配置接口编写。旧版 VT、不同内核版本不能直接保证兼容。此文件是完整配置的公开骨架，不含可用节点、订阅凭证、MITM 或去广告模块。

## 先添加节点，不能原样使用

官方 sing-box 原生使用 JSON，**没有 Clash 的 `proxy-providers` 自动合并节点功能**。不能把 Clash YAML、普通节点 URI 或订阅 URL 直接塞进此模板，也不会借用另一份配置里的节点。

1. 下载模板，用纯文本编辑器打开并另存一个私人副本。不要用 Word；JSON 不允许注释、末尾多余逗号。
2. 向服务商取得兼容你当前版本的完整 sing-box 配置，在本地找到其中的 `outbounds`。
3. 只复制真正的代理节点对象，例如 `shadowsocks`、`trojan`、`vless`、`hysteria2`、`tuic`；不复制原配置的 selector、urltest、direct、block、dns 和其他分组。完整保留节点的协议、端口、密码、TLS、Reality、传输等参数，不要为了导入方便删掉它们。
4. 用这些节点对象替换模板中标签为 `未配置节点` 的 socks 对象。这个对象只指向本机 `127.0.0.1:1`，不是可用服务器；请不要把它当真实节点。
5. 每个节点的 `tag` 必须唯一，不能与模板的分组或 `DIRECT` 重名。若节点的 `server` 是域名，在节点对象中添加/替换 `domain_resolver` 为 `{"server":"国内DoH"}`，单独解析代理入口，避免靠代理解析自身入口的循环依赖。节点 server 是 IP 则不需要此项。若原节点有特殊 detour/链式依赖，不属于简单复制场景，必须连同依赖单独核对。
6. 找到主组 `🚀 节点选择`，把它的 `outbounds` 改成节点的 tag 列表，把 `default` 改成其中一个真实 tag。不要在主组加入 DIRECT。下面示例只演示分组，不是完整配置：

```json
{
  "type": "selector",
  "tag": "🚀 节点选择",
  "outbounds": ["我的节点A", "我的节点B"],
  "default": "我的节点A",
  "interrupt_exist_connections": true
}
```

7. 在 sing-box MT 配置/Profile 页面通过本地文件导入这个私人 JSON，选中并启动 VPN。具体按钮名称以客户端版本为准。
8. 在客户端提供的分组/面板入口选择 `🚀 节点选择` 中的真实节点。selector 由 Clash API 控制，模板接口仅监听本机 `127.0.0.1:9090`，不自动下载第三方面板；若客户端没有可用面板，可先在本地编辑主组的 `default`，重启配置后检查实际选择。缓存的旧选择可能优先于 default。

节点的 `domain_resolver` 示意（添加在节点对象内，不能替代节点的其他字段）：

```json
"domain_resolver": { "server": "国内DoH" }
```

不要开启自动最快节点。国外服务组仅引用主组；切换主组会中断旧连接，使应用重新连接，可能暂时影响下载、通话和视频播放。模板未提供全局直连/全局代理模式切换规则，请按本模板分流使用，不要把面板的模式按钮误当作已实现的功能。

## 分流和 DNS

- Apple、小米、网易云音乐、国内域名规则和中国 IP 规则命中目标直连；国外服务分组在国内规则前，未知目标最终走主组。
- 国内/Apple 域名规则使用阿里 DoH；默认和国外域名使用 Cloudflare DoH，且显式 `detour` 到主组。没有系统 DNS 或国内备用用于国外默认查询。
- 未知域名即使最终解析到中国 IP，首次查询也仍走国外 DoH，不会先向国内 DNS 查询来判断位置。
- 节点入口域名通过每个节点的 `domain_resolver` 单独使用国内 DoH。规则文件下载是独立维护流量，直连 CDN 并通过国内 DoH 解析 CDN 域名；这是与访问国外网站的 DNS 不同的边界。
- 使用真实 IP + DNS reverse mapping + 协议 sniff，不使用 Clash fake-ip。域名识别并非始终可用，ECH、应用自带 DNS 和直接 IP 连接等可能影响分类；两种客户端的规则来源不完全相同，不承诺与 Shadowrocket 逐条等价。
- IPv4/IPv6 均配置 TUN 地址，不强制优先 IPv6，不屏蔽 AAAA。IPv6 出口取决于节点，TUN/VPN 的实际接管范围还取决于 iOS。
- 默认不禁用 QUIC，不强制 UDP 能力，不使用 UDP 失败直连兜底，不关闭服务器 TLS 验证。

为了第一版路径清晰，默认国内/国外各使用一个 DoH 上游；上游故障时对应查询可能失败。可在本地把国内服务器 IP 改为 `1.12.12.12`，国外改为 `1.0.0.1` 或 `8.8.8.8`，但保留 HTTPS、证书验证和原有 detour。此处没有自动上游容灾承诺。

局域网 IP 直连，但公共 DoH 无法保证解析你的内网域名。NAS/PVE/OpenWrt 应按自己的网络添加本地专用解析；共享版不含路由器地址。VPN 排除、系统/应用自带 DNS、VPN 断开后的直连都不能靠本文件消除。

## 怎么更新

远程规则使用 [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat) 的 sing-box SRS，通过公开 CDN 下载，每 `1d` 检查更新，缓存开启。应用暂停/关闭时不保证准时；首次规则下载失败可能阻止启动，请检查日志。国内地域规则可能误分类，不是隐私证明。

**节点不会像 Clash provider 那样自动更新。**订阅变化时重新在本地复制节点、同步主组 tag；不要用机场整份订阅自动覆盖我们的 DNS 和规则。模板升级也要在私人副本中合并，保留自己的节点。若需要“一条订阅自动换节点又保留全部规则”，需要另做本地配置生成流程，本版不假装具备此功能。

订阅链接、节点密码、UUID 和 CA 私钥都不要上传公开仓库、issue 或截图。控制 API 只绑定本机，不要改成 `0.0.0.0`；可在本地添加独立 secret，并在面板连接中填写相同 secret，不要公开私人值。

## 验证范围

本版是测试模板，不是“已经实测零泄漏”的成品。已通过官方 sing-box 1.14.0 的 `check`，核对 17 个 outbound 引用；16 个远程 SRS 地址下载成功，并经官方内核反编译检查通过。**没有启动完整 VPN、没有 iOS 实机、Karing 魔改内核或 DNS/IPv6 故障泄漏测试。**

自己加入节点后测试正常连接、主组切换、节点故障且 VPN 保持开启三个场景；检查 DNS/IPv4/IPv6 和日志。测试网页只能辅助，不能证明每个 App 的解析路径。出问题切回旧配置，提供日志前脱敏。

依据：[JSON 配置](https://sing-box.sagernet.org/configuration/)、[HTTPS DNS](https://sing-box.sagernet.org/configuration/dns/server/https/)、[selector](https://sing-box.sagernet.org/configuration/outbound/selector/)、[远程规则](https://sing-box.sagernet.org/configuration/rule-set/)。
