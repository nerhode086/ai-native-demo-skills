---
name: high-risk-review
description: "识别游戏研发需求中的高风险链路，并在实现、上线或 AI 交接前强制输出人工 Review 清单。用于 Codex 分析资产、支付、奖励、战斗结算、邮件附件、活动领取、任务进度、属性、战力、排行榜、合服、迁移、断线恢复、弱网重试、幂等或数据一致性风险的场景。"
---

# High Risk Review

使用本 skill 发现必须由人类 Owner 控制的生产风险。输出影响面、证据、缺失机制、Reviewer、测试和上线门禁；永远不要批准或绕过高风险链路。

## 输入

支持 Markdown、需求单、PRD、协议草案、配置说明、测试报告或 JSON：

```json
{
  "request_id": "string",
  "title": "string",
  "raw_requirement": "string",
  "business_goal": "string",
  "acceptance_criteria": ["string"],
  "related_systems": ["string"],
  "related_configs": ["string"],
  "related_protocols": ["string"],
  "known_constraints": ["string"],
  "historical_references": ["string"],
  "skill_outputs": ["string"]
}
```

只有缺失信息会改变风险分类时，才要求补充协议、配置、Owner 或验收细节；否则继续分析并标记假设。

## 工作流

1. 提取玩家动作、服务端状态变化、奖励/消耗、背包变化、战斗结果、排行结果、任务进度和数据迁移影响。
2. 将每个影响匹配到规则库；高风险类别宁可多报，不要漏报。
3. 将风险分为 `high`、`medium`、`low`；资产、支付、结算、奖励、排行、合服、修复、一致性至少为 `high`。
4. 为每个 high 或 medium 风险命名必要机制：幂等键、事务边界、防重复、补偿方案、审计日志、对账、配置校验、回滚、监控、数据修复脚本 Review。
5. 按角色分配 Reviewer：feature owner、server、client、qa、planning、ops/data、payment、economy、combat、platform。
6. 生成人工 Review checklist、测试用例、上线门禁和待澄清问题。
7. 用户要求 JSON 时只输出 JSON；否则输出 Markdown。

## 输出格式

Markdown 使用以下小节：

```text
风险摘要
风险矩阵
人工 Review Checklist
必要机制
Reviewer 路由
测试用例
上线门禁
待澄清问题
决策
```

`决策` 只能取：

- `blocked_until_review`：存在 high 风险。
- `review_required`：只存在 medium 风险。
- `low_risk_with_notes`：没有强制 Review 风险。

JSON 输出：

```json
{
  "manual_review_required": true,
  "decision": "blocked_until_review|review_required|low_risk_with_notes",
  "risk_points": [
    {
      "risk_type": "string",
      "level": "low|medium|high",
      "evidence": ["string"],
      "reason": "string",
      "required_mechanisms": ["string"],
      "reviewers": ["string"],
      "test_cases": ["string"],
      "launch_gates": ["string"]
    }
  ],
  "manual_review_checklist": ["string"],
  "open_questions": ["string"],
  "assumptions": ["string"]
}
```

## 规则库

- 资产变化：货币、道具、背包、消耗、退款、补偿、库存、邮件附件、掉落、奖励都必须有幂等、事务、审计日志和对账。
- 支付：回调、订单、票据、退款、延迟发货、平台重试、拒付必须有支付/平台 Review 和重复发货测试。
- 战斗结算：奖励、分数、伤害、胜负、防作弊、回放、Roguelike run 结果、离线结算必须有确定性校验和服务端权威验证。
- 奖励发放：活动领取、任务奖励、排行奖励、随机奖励池、首通奖励、里程碑奖励必须有资格、一次性防护、配置校验和回滚方案。
- 邮件与活动：附件领取、过期、重复点击、批量发送、跨服邮件、活动窗口必须有归属、过期、重试和审计 Review。
- 任务进度：进度事件、触发器、重置、历史迁移、显示同步必须有事件 ID、防重复和双端一致性检查。
- 属性与战力：属性来源、转化、Buff、称号、伙伴、坐骑、战力重算必须有来源追踪、顺序、显示同步和回滚证据。
- 排行榜：结算、重建、同分排序、赛季切换、发奖、延迟数据必须有快照、确定性排序和运维 Review。
- 合服或数据迁移：角色重名、公会、排行、邮件、活动状态、重复 ID 必须有演练、备份、diff 报告和回滚。
- 弱网与重连：重试、重复请求、断线恢复、本地缓存、延迟响应必须有幂等键和可重放状态流转。

## 正例

### 支付回调发货

标记 `payment` 和 `asset_change` 为 high；要求 payment、economy/server Reviewer，按订单 ID 幂等，重复回调测试，审计日志，对账任务，上线前阻塞 Review。

### 活动奖励领取

标记 `reward_issue`、`activity_claim`、`weak_network_retry`；要求资格窗口、一次性防护、服务端事务、重复点击测试、时区测试、配置校验和 planning/server/QA Review。

### 合服排行重建

标记 `server_merge`、`ranking`、`data_consistency`；要求演练、快照、确定性同分排序、发奖冻结、diff 报告、回滚计划和 ops/data/server Review。

## 反例与禁止事项

- 不要在检查服务端状态、活动开关、配置门禁、历史屏蔽和 QA 回归前，把“展示入口”或“展示红点”判为纯客户端。
- 不要对资产、支付、结算、奖励、排行、合服、修复或一致性链路输出“可安全上线”；只能说明需要哪些人和哪些证据 Review。
