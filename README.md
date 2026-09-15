# Shadowrocket 配置

免费 iOS 客户端测试版：[sing-box MT 模板与教程](SING-BOX.md)、[Karing 原生分流与 DNS 教程](KARING.md)。均需自己的节点；Karing 保留首页手动选节点，DNS 须按教程另外设置，不能只导入分流就认定防泄漏。

Android 用户请看 [Clash Meta for Android 测试模板和说明](ANDROID.md)。Android 版必须在本地添加自己的节点/订阅，不能原样下载就代理上网。

面向已有可用代理节点的 Shadowrocket 用户。建议使用 2.2.92 或更新版本。

## 导入与选择节点

### 一键导入（iPhone / iPad）

[点击导入 Shadowrocket 配置](https://lowertop.github.io/Shadowrocket-First/redirect.html?url=shadowrocket%3A%2F%2Fconfig%2Fadd%2Fhttps%3A%2F%2Fraw.githubusercontent.com%2Fissachigga%2Fshadowrocket-config%2Fmain%2Fshadowrocket.conf)

先安装 Shadowrocket，建议用 Safari 打开；允许跳转至 Shadowrocket，再按软件提示完成导入。若没有自动跳转，点击跳转页的“一键安装”按钮。导入后检查当前配置是否选中、全局路由是否为“配置”。先备份原配置。

说明：GitHub README 对自定义协议链接有限制，因此按钮经过第三方 LOWERTOP 的公开跳转页，页面可能变化或不可达；不附带节点或凭据，也没有为本仓库添加点击追踪。

不想经过第三方页面，可复制下列完整链接到 Safari 地址栏打开（需手机实测）：

```text
shadowrocket://config/add/https://raw.githubusercontent.com/issachigga/shadowrocket-config/main/shadowrocket.conf
```

### 手动导入备用地址

配置下载地址：

https://raw.githubusercontent.com/issachigga/shadowrocket-config/main/shadowrocket.conf

在 Shadowrocket「配置」中添加上述 URL，下载后选中配置；首页「全局路由」选择「配置」。先备份原配置。

**在首页手动选择节点。** 主组「🚀 节点选择」只保留 PROXY，国外服务组全部引用主组。不要在服务组中单独添加节点或 DIRECT。这样网页与裸 `#proxy` DNS 使用同一个首页代理选择，实际出口仍需实测确认。关闭“测试并选择最快服务器”等自动选择功能（若开启）。

节点和订阅由每个使用者自己添加，配置更新不会替你提供节点。首次导入需检查所有远程规则是否下载成功。

## 分流和 DNS

- Apple、国内媒体、小米、国内规则命中的网站直连；微软、Google、GitHub、AI 和其他国外服务走首页节点。
- 未命中的目标默认代理；中国 IP 直连规则使用 no-resolve，不为地域判断主动触发额外 DNS 查询。
- 主 DNS：Cloudflare DoH，经代理；备用：Google DoH，经代理。
- 直连域名 DNS：阿里、腾讯 DoH，不使用系统 DNS 覆盖。
- 节点入口域名：阿里、腾讯 DoH，独立于国外网站解析，避免代理入口解析循环。国内解析商仍能看到它负责的查询。
- 禁止系统 DNS 回退；直连解析失败允许经国外备用 DNS 处理，不保证国内 DNS 故障时国内网站仍可用。
- IPv6 开启，prefer-ipv6 关闭。IPv6-only 目标能否访问还取决于节点出口和网络。
- 节点不支持 UDP 时拒绝，不改成直连；通话、游戏可能因此失败。

DoH 加密传输，不保证解析商不记录查询。直连网站仍可能通过 IP、TLS 信息暴露目标。配置不提供绝对匿名或全设备 kill switch 保证。关闭 VPN 后系统恢复直连，不属于本配置覆盖范围。

## QUIC 可选设置

默认 `block-quic = always-allow`，不额外屏蔽 QUIC。若代理网页、视频不稳定，可在纯文本编辑中改为 `block-quic = all-proxy`：只阻断走代理连接的 QUIC，DIRECT 连接保留。不是关闭所有 UDP；少数不能回落 TCP 的应用可能失败。整份更新会覆盖此个人修改，更新后需重新设置。

参考：[维护者配置说明](https://github.com/LOWERTOP/Shadowrocket/blob/main/lazy.conf)。不额外附带模块。

## 更新

配置含 update-url，指向本仓库 main 分支同一文件。可在配置界面手动更新；自动更新需在软件设置中启用，iOS 后台调度不保证准时执行。

远程规则来自第三方仓库，经 jsDelivr CDN 获取，由 Shadowrocket 下载/缓存。检查刷新状态；CDN 可能延迟、不可达或上游变化，不承诺自动更新一定成功。若 raw GitHub 无法下载，先使用已有可用配置/节点，不删除原配置。

整份配置更新可能覆盖个人 DNS、规则、QUIC 及策略组修改。备份后再更新。请勿将本地节点订阅、CA 证书私钥、密码或带凭据日志提交至公开仓库。

## 测试状态与验证

v0.1 仅做静态检查与远程资源可用性检查，**尚未在 iPhone 做 DNS 出口、IPv6 或故障回退实测，不保证零泄漏**。应用自带 DoH/DoQ、其他模块和特殊网络可能改变实际行为。

1. VPN 开启、配置模式下测试国内和国外域名，查看连接记录、命中策略及 DNS 出口。
2. 首页切换节点，重启相关应用/连接，确认新连接与 DNS 出口跟随。
3. VPN 保持开启、选定节点故障时，国外请求应失败，不应国内 DNS 回退或直连。
4. 分别验证 IPv4 和 IPv6；DNS 测试网页只可辅助判断解析商，不能证明路径。必要时结合路由器抓包。
5. 如失败，切回备份配置。分享日志前脱敏节点凭据、订阅、证书和私人访问记录。

## 上游规则

[blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)、[ACL4SSR](https://github.com/ACL4SSR/ACL4SSR)、[Aethersailor/Custom_OpenClash_Rules](https://github.com/Aethersailor/Custom_OpenClash_Rules)。规则通过链接引用，各自遵循上游许可；本项目不复制完整规则库。
