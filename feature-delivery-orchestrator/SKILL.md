---
name: feature-delivery-orchestrator
description: "将传统游戏功能需求编排为适合 AI agent 并行执行的交付计划。用于 Codex 需要把原始策划需求转成结构化范围、Owner 边界、subagent 分工、通用组件复用、协议/配置/Mock/测试任务、风险评审、待澄清问题和客户端、服务端、策划、美术、QA 联调清单的场景。"
---

# Feature Delivery Orchestrator

使用本 skill 将一条原始游戏功能需求转成可协作执行的研发计划。始终保留人类 Owner 的最终判断权：输出判断依据、问题、风险和 Review 门禁，不替代最终验收。

## 输入

优先收集以下信息；缺失但影响判断时先提出问题：

- 原始需求、背景、业务目标、验收标准。
- 预期参与角色：planning、client、server、art、qa。
- 相关系统、配置、协议、历史需求、已知约束。
- 期望输出格式：默认 Markdown；需要交给工具或下游 agent 时输出 JSON。

## 工作流

1. 将需求拆成玩家意图、触发方式、状态变化、奖励/消耗、UI 反馈和验收信号。
2. 识别 planning、client、server、art、qa、ops_or_data 的执行边界。
3. 按需要路由到子 skill：
   - 边界不清时使用 `requirement-pre-review`。
   - 需要判断复用时使用 `generic-component-matching`。
   - 双端开发前使用 `protocol-contract`。
   - 测试前置时使用 `qa-testcase`。
   - 涉及资产、支付、结算、奖励、邮件、排行榜、合服、数据修复或一致性时使用 `high-risk-review`。
4. 生成 subagent 分工，确保写入范围、交付物、依赖和 Review 门禁互不冲突。
5. 在实现任务前先给出配置、协议、Mock 和测试建议。
6. 高风险链路一律标记 `manual_review_required`。
7. 最后输出联调 checklist、待澄清问题和下一步动作。

## 输出格式

Markdown 输出使用以下小节：

```text
需求摘要
玩家行为链路
执行边界
通用组件匹配
协议与配置影响
Subagent 工作计划
风险与人工 Review
待澄清问题
测试与 Mock 计划
联调 Checklist
下一步动作
```

JSON 输出使用以下结构：

```json
{
  "summary": "string",
  "player_flow": ["string"],
  "execution_boundary": {
    "planning": ["string"],
    "client": ["string"],
    "server": ["string"],
    "art": ["string"],
    "qa": ["string"],
    "ops_or_data": ["string"]
  },
  "matched_components": [
    {
      "component": "string",
      "confidence": 0.0,
      "reason": "string",
      "need_extension": false
    }
  ],
  "protocol_impact": ["string"],
  "config_impact": ["string"],
  "subagent_work_plan": [
    {
      "agent": "string",
      "scope": "string",
      "deliverables": ["string"],
      "dependencies": ["string"]
    }
  ],
  "risk_points": [
    {
      "risk_type": "string",
      "level": "low|medium|high",
      "reason": "string",
      "manual_review_required": true
    }
  ],
  "clarification_questions": ["string"],
  "test_cases": ["string"],
  "integration_checklist": ["string"],
  "manual_review_required": true
}
```

如需稳定传递给其他 agent，读取 `references/contracts.md`。

## 规则库

- 先判断可复用系统，再提出一次性实现。
- 协议、配置、Mock、测试用例是前置交付物，不是联调后补丁。
- 组件匹配和风险判断必须给出置信度与依据。
- 将“已知工作”和“会影响正确性的待澄清问题”分开。
- 涉及资产、支付、战斗结算、奖励发放、邮件附件、活动领奖、排行榜结算、合服、数据修复、数据一致性时，不允许 AI 绕过人工确认。
- 不要让多个 subagent 同时修改同一目录、文件或模块，除非指定集成 Owner。

## 正例

### 采集任务

输入：“新增采集石头任务，采集后获得石料，并在左侧任务面板显示进度。”

输出重点：识别客户端面板与交互反馈、服务端采集校验/发奖/任务推进、策划配置、QA 重复点击/断线/背包满/完成验证；匹配 `InteractionObject`、`RewardSystem`、`TaskCondition`，必要时匹配 `DropSystem`。

### 装备升级

输入：“装备升级提升力量并重算战力。”

输出重点：匹配 `LevelUp`、`CostSystem`、`AttributeSystem`；标记材料扣除、满级、属性刷新、战力重算、日志流水和人工 Review。

### 历史入口恢复

输入：“恢复活动入口按钮展示。”

输出重点：不能只判为客户端 UI；必须检查服务端开关字段、活动状态、配置门禁、历史屏蔽原因、红点、兼容性和 QA 回归。

## 反例与禁止事项

- 不要在确认数据源、状态门禁、配置门禁和历史屏蔽原因前，将入口、按钮、红点或面板改动判为客户端单点任务。
- 不要在命名协议/配置/测试影响和人工 Review 门禁前，直接输出实现任务。
