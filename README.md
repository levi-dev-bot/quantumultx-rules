# quantumultx-rules

Quantumult X 远程分流规则（Surge list 格式，QX 兼容 DOMAIN / DOMAIN-SUFFIX / DOMAIN-KEYWORD）。

## 文件

| 文件 | 用途 | 来源 |
|---|---|---|
| [general.list](https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/general.list) | 通用代理规则（含 Meta/Google/Twitter 等日常站），**已剔除风控极严的 AI 服务** | 改自 [ConnersHua/RuleGo](https://github.com/ConnersHua/RuleGo) Surge Proxy.list |
| [strict-ai.list](https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/strict-ai.list) | 风控极严 AI 服务（OpenAI/Claude/Gemini/Grok/Copilot/Perplexity/Poe/Muse），**单独分流到固定机场/美国 IP** | 自整理 |

两表互斥无重叠：strict-ai 里的条目已从 general 剔除。

## Quantumult X 配置示例

```ini
[filter_remote]
https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/strict-ai.list, tag=风控严AI, force-policy=风控严AI专用, enabled=true
https://raw.githubusercontent.com/levi-dev-bot/quantumultx-rules/main/general.list, tag=通用代理, force-policy=Proxy, enabled=true

[filter_local]
; strict-ai 在前，优先生效；general 兜底
final, Direct
```

`风控严AI专用` 在 [policy] 里指向你专门买的服务商节点（美国 IP）。

## 维护

- general.list 基于 ConnersHua/RuleGo 2026-10-27 快照手工剔除，上游更新后需手动同步。
- strict-ai.list 新增服务直接编辑本仓库。
