---
name: skill-evaluation-backtest
description: "用历史 golden cases 回测游戏研发工作流中的 AI-agent skill，并评估输出质量。用于 Codex 需要在更新团队 skill 前，检查执行边界准确率、通用组件命中、高风险召回、测试覆盖、澄清问题质量、可执行性、幻觉、规则缺口或回归风险的场景。"
---

# Skill Evaluation Backtest

使用本 skill 将候选 skill 输出与历史 golden case 对比，生成可复现评分表、失败原因和规则更新建议。只评估质量；除非用户要求，不要静默改写候选输出。

## 输入

支持文件夹、Markdown 表格、JSON 数组或粘贴内容：

```json
{
  "evaluation_id": "string",
  "skill_name": "string",
  "candidate_version": "string",
  "cases": [
    {
      "case_id": "string",
      "raw_requirement": "string",
      "golden_answer": {
        "execution_boundary": {},
        "matched_components": [],
        "risk_points": [],
        "test_cases": [],
        "clarification_questions": []
      },
      "candidate_output": {},
      "tags": ["string"]
    }
  ],
  "metric_weights": {},
  "pass_thresholds": {}
}
```

没有自定义指标时使用默认规则库。只有候选输出而没有 golden answer 时，切换为 gap analysis，并说明这不是回测。

## 工作流

1. 将每个 case 归一化为共享需求结构：原始需求、预期边界、组件、风险、测试、问题和证据。
2. 将候选输出归一化到相同标签；只有语义等价时才映射同义词。
3. 按 case 评分，再按权重聚合。
4. 高风险漏报视为 critical failure，即使总分较高也不能通过。
5. 惩罚幻觉：编造系统、配置、协议、数据源、Reviewer、分数、约束或未由输入/标注支持的实现结论。
6. 输出可执行失败原因：缺失规则、触发弱、措辞歧义、输出 schema 漂移、过度扩展。
7. 推荐规则补充、示例补充或反例补充；将建议与已确认分数分开。
8. 用户要求 JSON 时只输出 JSON；否则输出 Markdown。

## 输出格式

Markdown 使用以下小节：

```text
回测摘要
指标评分表
Case 结果
Critical Failures
幻觉与过度扩展
回归说明
建议补充规则
下一批评估集
决策
```

`决策` 只能取：

- `pass`：达到阈值且没有 critical failure。
- `needs_rule_update`：接近阈值或存在可修复缺口。
- `fail`：存在高风险漏报、严重幻觉或明显 schema 漂移。

JSON 输出：

```json
{
  "evaluation_id": "string",
  "skill_name": "string",
  "decision": "pass|needs_rule_update|fail",
  "aggregate_scores": {
    "execution_boundary_accuracy": 0.0,
    "component_hit_rate": 0.0,
    "high_risk_recall": 0.0,
    "test_coverage": 0.0,
    "clarification_quality": 0.0,
    "executability": 0.0,
    "hallucination_penalty": 0.0,
    "weighted_total": 0.0
  },
  "case_results": [
    {
      "case_id": "string",
      "passed": false,
      "scores": {},
      "false_negatives": ["string"],
      "false_positives": ["string"],
      "hallucinations": ["string"],
      "failure_reason": "string",
      "recommended_rule_updates": ["string"]
    }
  ],
  "critical_failures": ["string"],
  "recommended_rule_updates": ["string"]
}
```

## 规则库

- 默认权重：执行边界准确率 20%，组件命中率 20%，高风险召回 30%，测试覆盖 10%，澄清问题质量 10%，可执行性 5%，幻觉惩罚 5%。
- 默认通过阈值：边界准确率 >= 80%，组件命中率 >= 75%，高风险召回 >= 90%，测试覆盖 >= 70%，严重幻觉 = 0。
- Golden case 长期应覆盖：UI 入口、采集任务、掉落来源飘字、合成、装备升级、坐骑属性转化、技能学习、活动领取、邮件附件、战斗结算、Roguelike 退出、随机奖励、支付回调、排行结算、合服重名、合服排行重建、战力异常、客户端等服务端、配置缺失、弱网重复请求。
- 候选输出漏掉资产、支付、战斗结算、奖励、邮件附件、活动领取、任务进度、属性/战力、排行、合服、迁移、弱网、重试、幂等或数据一致性时，计为高风险 false negative。
- 候选输出编造代码模块、配置、协议、数据存储、Reviewer、分数或约束时，计为 hallucination。
- 不奖励冗长；只给具体、可执行、有证据的输出加分。
- 高风险类别优先召回率；实现和协议声明优先精确率。

## 正例

### 高风险召回

Golden case 是“支付回调发钻石”，候选只识别支付但漏掉重复回调处理。输出应给部分分，标记关键机制缺口，并建议给被测 skill 增加幂等和平台重试示例。

### UI 入口边界

Golden case 是“恢复活动入口”，候选只写客户端 UI。输出应指出漏掉服务端状态、配置门禁、历史屏蔽、红点和 QA 回归；决策通常为 `needs_rule_update` 或 `fail`。

### 幻觉

输入没有协议名，候选声称已有 `ActivityClaimV3`。输出应扣幻觉分，要求改成假设或待澄清问题，并建议增加禁止编造协议名的反例。

## 反例与禁止事项

- 没有 golden answer 或人工接受基线时，不要输出数值回测分；改为 gap analysis。
- 不要把隐藏 golden answer 原文复制进被评估 skill。建议应抽象为可迁移规则、触发词和示例。
