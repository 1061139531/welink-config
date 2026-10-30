# welink-config

微链（WeLink）**入口清单**托管仓库。仅存放一个 `network.json`，用于入口隧道域名轮换后，客户端与「一键修复入口」仍能取到当前入口地址。

## network.json 由入口机自动维护

本文件**由入口机 `health.ps1` 推送**（计划任务 `WeLinkWsHealth`，每 45 秒一轮巡检）：

- 入口探测通过后，把 `bootstrap_peers` 写成当前可用地址（`/dns4/<域名>/tcp/443/wss/p2p/<入口PeerID>`）；
- 内容与下载通道的 `/network.json` 同源，**推送成功后 60 秒内经 gh-proxy 回读校验**；
- 无变化时不重复提交（状态记在入口机侧，不在本仓库）；
- 仅推送最小清单（`listen_port` / `ws_listen_port` / `enable_mdns` / `enable_udp` / `bootstrap_peers`），**桌面端配置里的 `turn` 段等敏感字段永不外发**。

手工改本文件不会被长期保留——下一次入口探测通过（≤45 秒）就会覆盖。应急时可临时改，但请同时修入口机。

## 为什么可以放在公网

- 入口 PeerID **固定**：`12D3KooWMPUTUdQGhGpuXtzD7QN9KT7vztzfMGhbnwZems9fsiKF`。
- 读取方（`repair-lib.ps1`）只接受 **`wss` + 该 PeerID** 的地址，且 libp2p 建链必须由该私钥应答。
- 因此本仓库内容被篡改或代理被劫持，最坏结果是「指向不可达主机」= 退化为随包清单（**DoS 上限**），无法劫持会话、也无法替换身份。

## 取源方式（客户端 / 一键修复）

- 主源：`https://gh-proxy.com/https://raw.githubusercontent.com/1061139531/welink-config/main/network.json`
- 备源：`https://api.github.com/repos/1061139531/welink-config/contents/network.json`（返回 base64 内容，读取方自动解码）

取序：远端（主源 → 备源）→ 安装包内随包清单 → 内置兜底。

## 运维注意

- 入口机推送用的 token 存放在 `C:\ProgramData\welink-entry\carrier.json`（ACL 仅 SYSTEM / Administrators），**不在本仓库、不进安装包、不写日志**。
- 本机到 `github.com` 直连不可用，故负责推送的是 `api.github.com`（可达），客户端读取走 `gh-proxy`。
- 本仓库不存放任何私钥、TURN 凭据、安装包或其他产物。
