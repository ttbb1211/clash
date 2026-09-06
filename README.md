# clash

自用 Clash / sing-box 规则配置仓库，为自建订阅转换后端（subconverter）提供远程配置（远程分流规则）。

## 在用文件

| 文件 | 用途 | 引用方式 |
|---|---|---|
| [ACL4SSR_Online_Full.ini](https://raw.githubusercontent.com/ttbb1211/clash/refs/heads/main/ACL4SSR_Online_Full.ini) | Clash / mihomo（OpenClash）订阅转换配置 | 订阅转换的 SUBCONFIG 参数 |
| [ACL4SSR_Online_singbox.ini](https://raw.githubusercontent.com/ttbb1211/clash/refs/heads/main/ACL4SSR_Online_singbox.ini) | sing-box 订阅转换配置 | 订阅转换的 SBSUBCONFIG 参数 |

> 其余文件（`ACL4SSR_Online.ini`、`qichiyu.ini`、`wukong.ini`、`*.list`）为历史留存，当前不再使用。

## 规则说明

两个配置均基于 [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR) 在线规则版修改，`ruleset` 指向的上游 `.list` 文件在每次订阅转换时实时拉取。

### ACL4SSR_Online_Full.ini（Clash 主配置）

在 ACL4SSR_Online_Full 原版基础上做了如下定制（2026-09-06，提交 0f40d36）：

- 删除不需要的分流：广告拦截（`BanProgramAD` 等去广告规则）、网易云音乐、微软 Bing
- 微软云盘（OneDrive）并入 Ⓜ️ 微软服务
- 国内媒体、哔哩哔哩 → 🎯 全球直连
- 巴哈姆特 → 🌍 国外媒体
- 电报消息 → 📲 电报信息（组名与 ACL4SSR 标准一致，避免策略组引用缺失）
- Ai平台 → 💬 OpenAi

保留的分流组：🚀 节点选择、♻️ 自动选择（url-test 自动测速）、📲 电报信息、📹 油管视频、📺 奈飞视频、💬 OpenAi、🌍 国外媒体、🎮 游戏平台、Ⓜ️ 微软服务、📢 谷歌FCM、🍎 苹果服务、🎯 全球直连、🐟 漏网之鱼。

### ACL4SSR_Online_singbox.ini（sing-box 配置）

- 针对 sing-box 1.12+ 移除了 GEOIP 规则（`GEOIP,CN` / `GEOIP,LAN`），避免导入报错
- 精简了部分分流（去广告相关规则已注释）
- 漏网之鱼默认走代理线路

## ⚠️ 重要提醒

- **请勿用 ACL4SSR 上游原版覆盖 `ACL4SSR_Online_Full.ini`**，上游原版会把已删除的广告拦截/网易音乐等规则加回来，且部分策略组名与订阅端不匹配会导致 Clash 内核校验失败。原版备份保存在本地 `D:\SubOps\rules\ACL4SSR_Online_Full_原版备份.ini`（不入库）。
- 该配置被以下订阅服务引用，修改后无需重新部署订阅前端（转换时实时拉取）：
  - Global-Reach 主订阅（SUBCONFIG / SBSUBCONFIG 环境变量）
  - cf-workers-sub 备用订阅（SUBCONFIG 环境变量）

## 引用地址

```
# Clash（OpenClash 等）
https://raw.githubusercontent.com/ttbb1211/clash/refs/heads/main/ACL4SSR_Online_Full.ini

# sing-box
https://raw.githubusercontent.com/ttbb1211/clash/refs/heads/main/ACL4SSR_Online_singbox.ini
```
