# ClashMi / Clash Meta for Android 使用说明（测试模板）

面向 [ClashMi](https://clashmi.app/guide/)（iPhone / iPad / Android）和 [MetaCubeX/ClashMetaForAndroid](https://github.com/MetaCubeX/ClashMetaForAndroid) 的新版 mihomo 内核。两者共用下方 YAML，文件名保留不变以免旧下载链接失效。不保证旧版 Clash for Android 或其他内核兼容。

[下载 YAML 模板](https://raw.githubusercontent.com/issachigga/shadowrocket-config/main/clash-meta-android.yaml)

## 必须先添加自己的节点

这个公开文件不含节点，也不会复用客户端其他配置中的节点。**原样导入无法代理上网**：主组默认 REJECT，防止无节点时默认直连。

推荐操作：下载 YAML，在本地编辑，取消 `proxy-providers` 示例整段注释，替换 `url` 中的占位地址为自己的 **Clash 格式**订阅。保留 `proxy: DIRECT` 可避免首次下载订阅依赖尚未加载的代理；订阅服务器可能需要可直连，打不开时需另行调整或手动添加节点。不要启用健康检查来自动选节点，本模板只有 select 主组。

或者将 `proxies: []` 替换为自己的完整 Clash 节点列表。不要把原始 ss://、vless://、trojan:// 链接直接当 YAML 节点填入；不要通过不可信在线转换站处理私人订阅。

编辑后通过客户端「配置」的本地文件导入功能导入、选中；使用规则模式，启动 VPN，在「🚀 节点选择」中手动选择实际节点（不是首页节点），其余国外组统一引用它。Apple、国内媒体、小米、国内规则命中目标直连，未知目标默认代理。

## 新手教程：填订阅、导入和选节点

1. 点击上方「下载 YAML 模板」，保存为 `clash-meta-android.yaml`，用纯文本编辑器打开。不要用 Word 编辑。
2. 找到 `# proxy-providers:` 开头的示例，删除该段每行开头的 `# `，保留原来的缩进。改成下面这样，只替换 `url` 的内容：

```yaml
proxy-providers:
  MySubscription:
    type: http
    url: "在这里填自己的 Clash 格式订阅地址"
    path: ./proxy-providers/my-subscription.yaml
    interval: 86400
    proxy: DIRECT
    health-check:
      enable: false
```

3. 保留 `proxies: []` 和下面所有分组、DNS、规则，不要把机场整份配置覆盖进来。`include-all: true` 会让主组选到这个 provider 中的节点。
4. 保存文件，在客户端配置页面通过本地文件导入，选中这份配置。ClashMi 的额外设置见下一节；具体按钮名称可能随版本不同。
5. 更新代理集合/订阅，确认 `MySubscription` 已下载成功；首次加载可能自动下载。这里更新的是模板内的代理集合，不是另一份机场配置。看不到节点时检查日志、订阅格式和下载是否成功。
6. 启动客户端 VPN，使用「规则」模式，在代理页面的「🚀 节点选择」里选择真实节点，不能停留在 `REJECT`。其他国外分组统一跟随它，不会自动替你选最快节点。
7. 测试国内网站、Apple 服务和国外网站，再切换一次主组节点，确认国外连接跟随切换；DNS/IPv6 测试按文末说明进行。

订阅必须能返回 Clash/mihomo 支持的节点 YAML；只有一串 `ss://` 等节点链接的普通订阅不能直接填。向服务商索取 Clash 格式订阅，不要把私人链接交给不可信转换网站。示例使用直连下载订阅，如果订阅地址本身被阻断，需要先在其他可用网络下载或本地手动添加节点。

每位朋友都填自己的订阅，或使用服务商明确允许共享的订阅。**订阅地址相当于凭证：不要发到公开 issue、README、截图或仓库。**这里只自动更新订阅节点和远程规则；完整模板更新仍需自行合并，避免覆盖私人订阅。

## ClashMi：iPhone / iPad 的额外步骤

1. 先按上方教程在本地 YAML 填好私人订阅，再从「我的配置」的添加入口导入文件并选中。若当前版本只提供粘贴配置内容的入口，可粘贴本地完整 YAML；不要把公开模板链接当成已带节点的订阅。官方快速教程中的「添加配置链接」适用于你已有可访问的完整配置地址，不会自动合并另一份机场配置。
2. 检查「核心设置」及当前配置的覆写选项。**保留模板的 DNS、规则和策略组，不启用会替换它们的内置/自定义覆写**；仅关闭总覆写未必够，应检查最终生效配置。若从完整配置 URL 添加，官方提供 `overwrite=false` 参数用于关闭默认覆写；本地导入则检查界面中的实际开关。
3. 到「核心设置 → TUN」启用 TUN；本模板没有 `tun` 段，可按官方 FAQ 开启 TUN 的覆写并启用，然后重连。TUN 覆写与 DNS/规则覆写是不同部分，不要为了启用 TUN 而同时覆盖模板 DNS。允许系统添加 VPN 配置。
4. 使用「规则」模式，在「代理」或「面板」中的「🚀 节点选择」手选真实节点；其余国外组跟随它。不要切换到全局模式后仍以为国内和 Apple 保持直连。
5. 查看生效配置/日志：默认 DNS 应仍是带 `#🚀 节点选择` 的国外 DoH，节点入口和直连 DNS 应是国内 DoH；规则末尾应仍为 `MATCH,🚀 节点选择`。不要只看 YAML 原文件而忽略客户端覆写结果。
6. 切换主组节点后，关闭旧连接或重启相关 App 再测试。已有视频/长连接可能暂时沿用旧出口，不代表新节点选择没有生效。

iOS 端的 VPN 排除、其他 DNS 设置及应用自带解析仍可能绕过预期路径。不要随意照搬桌面 `auto-route`、Android 的系统 VPN 阻断开关或一整套网络排除参数。Android ClashMi 同样需检查覆写和 TUN，但系统断线保护以 Android 实际支持为准。

如果 iOS VPN 启动后立刻断开，先看日志。官方 FAQ 提醒 iOS VPN 扩展有内存限制，并建议减少地域规则或使用 MRS。此版仍沿用原有 classical YAML 规则 provider，尚未做 iOS 内存实测；若出现该问题，不要反复重连，先切回备份并保留脱敏日志，再针对规则做精简。

官方参考：[快速使用](https://clashmi.app/guide/)、[覆写、TUN 与 iOS 故障说明](https://clashmi.app/guide/faq)。

## DNS 与 IPv6

- 使用 fake-ip 模式。默认查询使用 Cloudflare/Google DoH，并明确绑定「🚀 节点选择」。
- `proxy-server-nameserver` 独立用国内 DoH 解析节点入口，避免循环依赖。
- `direct-nameserver` 用国内 DoH 解析 DIRECT 出口的域名；默认查询阶段可能仍通过国外 DoH，因此不是“所有国内域名查询只发给国内 DNS”的承诺。
- DNS 服务器均使用 IP 形式，bootstrap 也配置加密 DNS，不添加系统 DNS 或国内 fallback 到默认国外查询。
- IPv6 开启，不添加强制优先 IPv6 的选项。IPv6 目标的可达性依赖节点出口。
- 不禁用 QUIC，不强制覆写节点 UDP 或跳过 TLS 证书校验。节点须具备相应协议能力；本模板不提供 UDP 不支持时的 DIRECT 兜底。
- 不复制桌面 tun.auto-route 配置，移动端 VPN/TUN 由客户端管理。应用排除、分应用代理、系统私人 DNS、其他 VPN 和 ClashMi 覆写等仍可能改变实际效果，请检查客户端设置。

内网域名不保证能由公共 DoH 解析。需要访问 NAS/PVE/OpenWrt 的内网域名时，按自己的网络在本地设置专用解析规则；公开版不包含你的路由器 IP、私有域名或订阅。

## 更新和隐私

规则 provider 每 86400 秒检查更新，由客户端运行时执行，不保证准时。provider 下载为 DIRECT；规则下载不可达可能影响首次启动，务必检查日志。规则来源为 blackmatrix7 的 Clash 专用 YAML，远程规则和地域数据库可能误分类。

这份文件是公开模板，**不要把加入私人订阅后的版本提交至公开仓库**。建议以本地配置使用，不要将 public raw URL 设置成个人完整配置的自动更新来源，否则更新会清除你的节点和订阅修改。更新模板需在本地保留私人 provider/proxies 后合并。

DoH 服务商可看到其负责的查询；加密不等于匿名。默认国外查询没有系统 fallback，但应用自带 DNS、VPN 排除和故障行为仍需实测。关闭 VPN 后网络恢复直连。若需要断线保护，可检查 Android 系统“始终开启 VPN / 阻止未使用 VPN 的连接”功能，设备/客户端支持因系统而异，可能影响局域网或其他应用。

## 已完成与未完成测试

已检查 YAML 语法、分组/规则引用及远程规则地址可下载，并按 ClashMi 官方教程补充客户端设置。**没有 ClashMi iOS/Android 实机或此 YAML 的内核运行测试，也没有验证 DNS、IPv6 或节点故障时零泄漏。**

添加自己的节点后，测试正常连接、切换主组节点、节点故障（VPN 保持开启）三个场景；分别检查 IPv4/IPv6、DNS 服务商和连接日志。泄漏测试网页可辅助，但不能单独证明解析路径。出问题先切回原配置，分享日志前脱敏订阅和密码。

配置字段依据：[mihomo DNS 文档](https://wiki.metacubex.one/config/dns/)、[代理集合](https://wiki.metacubex.one/config/proxy-providers/)、[策略组](https://wiki.metacubex.one/config/proxy-groups/)。
