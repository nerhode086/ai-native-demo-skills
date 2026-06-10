# 功能交付契约

当交付计划需要在多个 skill 或 subagent 之间稳定传递时，使用本 reference。

## Feature Request 输入

```json
{
  "request_id": "string",
  "title": "string",
  "background": "string",
  "business_goal": "string",
  "raw_requirement": "string",
  "expected_owner": ["planning", "client", "server", "art", "qa"],
  "acceptance_criteria": ["string"],
  "related_systems": ["string"],
  "related_configs": ["string"],
  "related_protocols": ["string"],
  "known_constraints": ["string"],
  "historical_references": ["string"]
}
```

## Subskill 路由

| 需求 | 路由到 |
| --- | --- |
| 范围或 Owner 边界不清 | `requirement-pre-review` |
| 功能可能复用通用系统 | `generic-component-matching` |
| 双端编码前需要协议契约 | `protocol-contract` |
| 需要 QA 覆盖、测试数据或回归范围 | `qa-testcase` |
| 触及资产、支付、结算、奖励、排行、合服或一致性 | `high-risk-review` |
| 需要用历史 case 评估 skill 输出 | `skill-evaluation-backtest` |

## Handoff Packet

传给每个 subagent：

```json
{
  "source_requirement": "string",
  "known_context": ["string"],
  "scope": "string",
  "expected_artifacts": ["string"],
  "dependencies": ["string"],
  "manual_review_gates": ["string"],
  "output_format": "markdown|json"
}
```

每个 subagent 返回：

```json
{
  "artifact_summary": "string",
  "decisions": ["string"],
  "open_questions": ["string"],
  "risks": ["string"],
  "next_dependencies": ["string"]
}
```
