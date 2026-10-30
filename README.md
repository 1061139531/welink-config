# welink-config

微链（WeLink）**入口清单**托管仓库。仅存放一个 `network.json`，用于在 Cloudflare quick tunnel 域名轮换后，客户端与「一键修复入口」仍能取到当前入口地址。

## 文件说明

`network.json`：

| 字段 | 说明 |
|---|---|
| `bootstrap_peers` | **当前**入口 multiaddr（域名为最新 quick tunnel 域名） |
| `candidates` | 备用入口（慢通道，域名长期固定） |
| `generated_at` / `note` | 生成时间与说明，仅供人读 |

## 为什么可以放在公网

- 入口 PeerID **固定**：`12D3KooWMPUTUdQGhGpuXtzD7QN9KT7vztzfMGhbnwZems9fsiKF`。
- 读取方（`repair-lib.ps1`）只接受 **`wss` + 该 PeerID** 的地址，且 libp2p 建链必须由该私钥应答。
- 因此本仓库内容被篡改或代理被劫持，最坏结果是「指向不可达主机」= 退化为随包清单（**DoS 上限**），无法劫持会话、也无法替换身份。

## 取源方式

- 主源：`https://gh-proxy.com/https://raw.githubusercontent.com/1061139531/welink-config/main/network.json`
- 备源：`https://api.github.com/repos/1061139531/welink-config/contents/network.json`（返回 base64 内容，读取方会自动解码）

## 更新流程（入口域名轮换后）

1. 入口机 `health.ps1` 检测到新域名并写入 `%APPDATA%\welink\network.json`；
2. 把同一份内容同步到本仓库 `network.json`（`bootstrap_peers` 换成新域名，`generated_at` 更新）；
3. 提交并推送 `main` 分支，**≤5 分钟**内所有客户端 / 修复脚本的下次探测即生效。

> 不要提交任何私钥、TURN 凭据、安装包或其他产物；本仓库只作「地址清单」用途。
