# quantumultx-rules

Quantumult X 远程分流规则（Surge list 格式，QX 兼容 DOMAIN / DOMAIN-SUFFIX / DOMAIN-KEYWORD）。

配套配置：大陆白名单模式，国外流量分两层——普通走代理组自由切换，风控严格（AI/支付）走固定节点。

## 文件一览

### 个人维护层（加域名/加 IP 直接编辑这几个文件）

| 文件 | 用途 | 常见场景 |
|---|---|---|
| [user-strict.list](https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/user-strict.list) | **风控专线**补充 | 新出的 AI 服务、对 IP 敏感的站点，强制走固定节点 |
| [user-proxy.list](https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/user-proxy.list) | **普通国外**补充 | general.list 没收录的小众国外站、自建服务、API |
| [user-direct.list](https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/user-direct.list) | **自定义直连**补充 | ChinaMax 漏掉的国内域名/IP，强制直连 |

**编辑规则**（行尾的策略名是占位符，QX 导入时被 force-policy 覆盖，不用改）：

```
host-suffix, example.com, Proxy      # 该域名及其所有子域名
host, api.example.com, Proxy         # 仅精确匹配这一个域名
host-keyword, example, Proxy         # URL 中含 example 即命中（慎用，范围大）
ip-cidr, 203.0.113.0/24, Proxy       # IPv4 网段
ip6-cidr, 2001:db8::/32, Proxy       # IPv6 网段
```

提交后 QX 端生效方式：分流页 → **全部更新**（或等 24h 自动同步，建议手机端 `update-interval` 设 86400）。

### 上游同步层（从原作者仓库自动拉取，无需手动维护）

| 在配置中的 tag | 上游列表 | 走向 |
|---|---|---|
| 严格风控 | [levi-dev-bot/strict-ai.list](https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/strict-ai.list)（VPSDance 每日同步 + Repcz 补充） | 风控专线 |
| PayPal / Stripe | blackmatrix7/ios_rule_script | 风控专线 |
| 普通国外列表 | [levi-dev-bot/general.list](https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/general.list)（改自 ConnersHua/RuleGo） | 普通国外 |
| 大陆白名单 | blackmatrix7 ChinaMax（12 万+条域名+IP） | direct |
| 规则修正 / 微信直连 / 苹果服务 | ddgksf2013 Unbreak、blackmatrix7 WeChat/Apple | direct |

互斥性：strict-ai 里的条目已从 general 剔除；user-strict 优先级最高，可覆盖一切。

### 每日自动生成（strict-ai.list）

GitHub Actions 每天 04:00（北京时间）跑 `scripts/merge-strict-ai.py`：

1. `ai-core.list`（手工核心 164 域名+IP/ASN 段，保底，不过滤）→
2. L1 VPSDance `rules/quantumult-x/global.list`（境外聚合版，已剔除国内文件）→
3. L2 Repcz/Tool `QuantumultX/Rules/AI.list` 差集（cursor.sh/groq.com/dify.ai 等）→
4. `exclude-cn.txt` 命中/`.cn` 后缀一律剔除（只作用上游层，核心层不受影响）→
5. 有变化才提交；L1 拉取失败保留旧文件。

> 本仓库是**大陆白名单**模式（命中国内名单→直连，其余走代理），所以“国内直连补口”是往 ChinaMax / user-direct.list 加条目；与家宽仓库（levi-dev-bot/clash，国外白名单）方向相反，勿混淆。

## Quantumult X 配置示例

```ini
[filter_remote]
; —— 个人层（优先）——
https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/user-strict.list, tag=风控补充, force-policy=风控专线, enabled=true
https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/user-proxy.list, tag=国外补充, force-policy=普通国外, enabled=true
https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/user-direct.list, tag=直连补充, force-policy=direct, enabled=true
; —— 上游层 ——
https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/strict-ai.list, tag=严格风控, force-policy=风控专线, enabled=true
https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/general.list, tag=普通国外列表, force-policy=普通国外, enabled=true

[filter_local]
final, 兜底
```

`风控专线` 为 static 策略组，下挂 available 区域组（美/日/新），锁定固定 IP 不漂移。

## 维护

- **user-*.list**：随时编辑提交，QX 下次更新即生效。
- **strict-ai.list**：数据源每日自动同步 + 手工补充，直接编辑本仓库。
- **general.list**：基于 ConnersHua/RuleGo 快照手工剔除 AI 类目，上游更新后手动同步。
