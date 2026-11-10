# Loon AltStore Loopback

使用 Loon 的本机回环功能，为 AltStore Classic 的设备内安装、续签提供通信通道。

本配置依据 Loon 官方文档、官方回环示例及 AltStore / LocalDevVPN 公开源码整理。它具备替代 LocalDevVPN 本机回环部分的实现条件，但本仓库当前尚未完成 Loon + AltStore 的手机端实测，请通过实际安装或续签确认兼容性。

## 要求

- Loon **3.5.3（Build 1007）或更新版本**。请查看括号内的构建号；较旧版本不支持本插件使用的 `include-tun` 和 `[IP Rewrite]`。
- AltStore Classic **2.3 或更新版本**，已完成 Remote AltServer 设置及设备配对。
- 按 AltStore 官方要求连接 Wi-Fi。

## 安装

在 Loon 的插件管理中添加以下 URL，并启用插件：

```text
https://raw.githubusercontent.com/a251231/loon-altstore-loopback/main/Loon_AltStore_Loopback.plugin
```

也可以下载 [插件文件](./Loon_AltStore_Loopback.plugin)，作为本地插件导入。

`[IP Rewrite]` 只在插件中生效，请以插件方式安装。

## 验证

1. 断开 LocalDevVPN，并启用本插件。
2. 断开后重新连接 Loon，让虚拟网卡路由重新生效。
3. 保持手机连接 Wi-Fi。
4. 打开 AltStore → Settings → Remote AltServer，检查是否显示 **Connected**。
5. 到 My Apps → Refresh All 实际续签。状态显示 Connected 只能说明本机服务可达，实际续签成功才算完成验证。

保留现有配对文件及远程服务器设置。若测试失败，关闭插件并断开 Loon，再切回原本可用的 LocalDevVPN。

## 原理与影响

```text
AltStore → 10.7.0.1 → Loon TUN → reflect → 本机开发服务
```

- `include-tun = 10.7.0.1/32`：将发往该单个地址的流量导入 Loon 的虚拟网卡。
- `10.7.0.1 = reflect`：交换数据包的源、目的 IP，保留端口和载荷，再回注本机网络协议栈。Loon 在常规代理规则之前处理这个动作。
- `ipv6-vif = off`：依照 Loon 官方回环示例，关闭 Loon 虚拟网卡的 IPv6 配置。关闭插件后重新连接 Loon，会按照主配置及其他启用插件重新计算此选项。

AltStore 的公开连接检测代码使用 `10.7.0.1:49152` 探测本机服务。LocalDevVPN 的公开实现也通过交换源、目的 IP 并回注数据包提供本机回环，因此可以据此推导 Loon 的替代方案。

该插件只提供本机通信通道；账户认证、签名、远程认证服务选择仍由 AltStore 管理。

## 排查

- **导入或解析失败**：检查 Loon Build 是否至少为 1007，并确认是以插件方式导入。
- **Not Connected**：确认 Wi-Fi 已连接、插件已启用，并重新连接 Loon。检查是否有其他插件对 `10.7.0.1` 配置了优先的 `reject` / `drop` 动作。
- **Connected 但续签失败**：记录 AltStore 完整错误；这个状态并不保证账户认证、远程服务器、签名或设备配对均正常。

## 参考来源

- [Loon 官方回环示例与 Build 1007 公告](https://t.me/LoonNews/2044)
- [Loon：include-tun 与路由优先级](https://nsloon.app/docs/General/#include-tun)
- [Loon：IP Rewrite](https://nsloon.app/docs/Plugin/#ip-rewrite)
- [AltStore：Remote AltServers](https://faq.altstore.io/altstore-classic/remote-altservers)
- [AltStore：OnDeviceClient.swift](https://github.com/altstoreio/AltStore/blob/classic/AltStore/Managing%20Apps/OnDeviceClient.swift)
- [LocalDevVPN：PacketTunnelProvider.swift](https://github.com/jkcoxson/LocalDevVPN/blob/main/TunnelProv/PacketTunnelProvider.swift)

本仓库为独立整理的配置示例，与 Loon、AltStore 或 LocalDevVPN 官方无隶属关系。
