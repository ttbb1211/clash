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

- 删除不需要的分流：广告拦截（`BanAD`）、网易云音乐、微软 Bing
- 微软云盘（OneDrive）并入 Ⓜ️ 微软服务
- 国内媒体、哔哩哔哩 → 🎯 全球直连
- 巴哈姆特 → 🌍 国外媒体
- 电报消息 → 📲 电报信息（组名与 ACL4SSR 标准一致，避免策略组引用缺失）
- Ai平台 → 💬 OpenAi

后续补充（2026-09-12，提交 `8bb286b` / `a18ac52` / `904826a` / `41b97e2`）：

- 新增 🇭🇰 香港均衡 负载均衡组，🌍 国外媒体 默认走它
- 该组放在策略组列表末尾（避免被其他组名匹配干扰）

保留的分流组：🚀 节点选择、♻️ 自动选择（url-test 自动测速）、🇭🇰 香港均衡（load-balance 轮询）、📲 电报信息、📹 油管视频、📺 奈飞视频、💬 OpenAi、🌍 国外媒体、🎮 游戏平台、Ⓜ️ 微软服务、📢 谷歌FCM、🍎 苹果服务、🎯 全球直连、🛑 全球拦截、🍃 应用净化、🐟 漏网之鱼。

#### 🇭🇰 香港均衡（负载均衡组）

把香港 cdt 线路的多个节点分摊流量，避免单节点跑满或被限速。

| 项 | 值 |
|---|---|
| 组类型 | `load-balance`（需 mihomo / Clash.Meta 内核，原版 Clash Premium 不支持） |
| 节点过滤 | `(?i)香港cdt` —— 只收节点名含「香港cdt」的线路，大小写不敏感 |
| 健康检查 | `http://www.gstatic.com/generate_204` |
| 检查间隔 | 300 秒 |
| 均衡策略 | `round-robin`（真轮询：新连接轮流分配到组内不同节点） |
| 引用位置 | 🚀 节点选择 第 3 顺位；🌍 国外媒体 **第 1 顺位（默认走均衡）** |
| 在配置中的位置 | 策略组列表**末尾** |

对应配置行（`ACL4SSR_Online_Full.ini` 第 63 行）：

```ini
custom_proxy_group=🇭🇰 香港均衡`load-balance`(?i)香港cdt`http://www.gstatic.com/generate_204`300`!!strategy=round-robin
```

> ⚠️ **末尾的 `` `!!strategy=round-robin `` 不可删除、也不可改成 5 段写法。** 这行是 6 段写法（多一段 `!!key=value`），依赖**自建转换后端的定制补丁**：后端会先剥离末尾所有 `!!` 开头的段放进扩展参数，再按「倒数第二段=test_url、最后一段=interval」解析，因此产出的 Clash 配置为 `strategy: round-robin` + `url: http://www.gstatic.com/generate_204` + `interval: 300`。
>
> 若换成**官方原版** subconverter（无该补丁），这行会被解析成 `url: 300`、`interval: 0`，`strategy` 退化为默认的 `consistent-hashing`；若按上游源码「修正」成 5 段，则**均衡策略直接丢失**，只剩按目标地址哈希（同一网站固定走一个节点）。
>
> ⚠️ 节点过滤 `(?i)香港cdt` 是硬编码，**依赖节点命名**。改名或换供应商后该组会变空，需同步调整过滤词。
>
> ℹ️ `round-robin` 与 `consistent-hashing` 的差别：前者新连接轮流分配，单站点也能同时用上多个节点；后者按目标地址哈希，同一网站（youtube.com 等）的全部流量固定走一个节点，压不满多个节点。

### ACL4SSR_Online_singbox.ini（sing-box 配置）

- 针对 sing-box 1.12+ 移除了 GEOIP 规则（`GEOIP,CN` / `GEOIP,LAN`），避免导入报错
- 精简了部分分流（去广告相关规则已注释）
- 漏网之鱼默认走代理线路
- **不含负载均衡组**：sing-box 内核只有 `selector` / `urltest` 两种策略出站，没有 `load-balance` 类型，均衡能力仅在 Clash / mihomo 侧生效
- AI 分流组名为 `💬 Ai平台`（Clash 侧为 `💬 OpenAi`），两端命名不同属历史差异

## ⚠️ 重要提醒

- **请勿用 ACL4SSR 上游原版覆盖 `ACL4SSR_Online_Full.ini`**，上游原版会把已删除的广告拦截/网易音乐等规则加回来，且部分策略组名与订阅端不匹配会导致 Clash 内核校验失败。原版备份保存在本地 `D:\运维-订阅系统\rules\ACL4SSR_Online_Full_原版备份.ini`（不入库）。
- 该配置被以下订阅服务引用，修改后无需重新部署订阅前端（转换时实时拉取）：
  - Global-Reach 主订阅（SUBCONFIG / SBSUBCONFIG 环境变量）
  - cf-workers-sub 备用订阅（SUBCONFIG 环境变量）
- **自建转换后端是定制编译版**（`subconverter:hot` 镜像，源码含「INI 末尾 `!!key=value` 段剥离」补丁），`ACL4SSR_Online_Full.ini` 第 63 行的 6 段写法依赖该补丁。换成官方原版镜像或按上游源码「修正」该行都会导致均衡组参数错位或策略丢失，详见上文「🇭🇰 香港均衡」小节的警告。

## 引用地址

```
# Clash（OpenClash 等）
https://raw.githubusercontent.com/ttbb1211/clash/refs/heads/main/ACL4SSR_Online_Full.ini

# sing-box
https://raw.githubusercontent.com/ttbb1211/clash/refs/heads/main/ACL4SSR_Online_singbox.ini
```
