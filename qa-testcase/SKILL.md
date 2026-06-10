---
name: qa-testcase
description: "根据游戏需求、协议契约、配置表、Mock 计划、风险点或功能交付计划，生成适合 AI agent 使用的 QA 测试用例、测试数据需求、回归范围和联调 checklist。用于 Codex 需要在客户端、服务端、策划协作前置 QA 覆盖，包含正常、异常、边界、弱网、重复请求、配置错误和高风险 Review 场景。"
---

# QA Testcase

使用本 skill 在双端联调前或联调中，把游戏功能需求转成可执行 QA 覆盖。QA 和人类 Owner 保留最终判断权；本 skill 只输出覆盖面、证据、数据需求和 Review 门禁。

## 输入

缺少以下信息且会影响覆盖时，先提出问题：

- 原始需求、业务目标、玩家链路、验收标准。
- 相关客户端/服务端模块、已有系统、历史行为、功能开关。
- 协议请求/响应字段、错误码、状态流转、幂等键、Mock 示例。
- 配置表、奖励/消耗 ID、任务条件、活动窗口、限制、默认值。
- 已知风险：资产、支付、结算、奖励、邮件附件、活动领奖、排行、合服、数据修复、弱网、重复请求、数据一致性。
- 测试环境、目标平台、输出格式、优先级要求。
- 输出格式：默认 Markdown；需要下游工具或 tracker 时输出 JSON。

## 工作流

1. 将功能重述为触发条件、前置条件、玩家动作、服务端状态变化、客户端反馈、奖励/消耗和验收信号。
2. 建立覆盖矩阵：正常、异常、边界、弱网、重复请求、配置错误、兼容、回归。
3. 从协议、配置、风险和玩家链路推导用例；每个用例包含预期结果、可观察证据和 Owner。
4. 先生成测试数据需求：账号、背包、货币、配置、活动状态、任务进度、Mock 响应、日志、重置脚本。
5. 高风险链路必须标记人工 Review，并补充幂等、回滚、一致性、日志、监控证据检查。
6. 根据触达系统、共享组件、历史 bug、功能开关、红点、入口和迁移定义回归范围。
7. 输出 planning、client、server、qa、ops_or_data 的联调 checklist。
8. 只有缺失信息会改变覆盖、预期结果或风险等级时，才列为澄清问题。

## 输出格式

Markdown 使用以下小节：

```text
功能摘要
覆盖矩阵
测试用例
测试数据需求
回归范围
联调 Checklist
人工 Review
待澄清问题
```

每个测试用例包含：

```text
ID
标题
类别
优先级
前置条件
测试数据
步骤
预期结果
需采集证据
风险标签
是否需要 Owner Review
```

JSON 输出：

```json
{
  "feature_summary": "string",
  "coverage_matrix": [
    {
      "category": "normal|abnormal|boundary|weak_network|duplicate_request|config_error|compatibility|regression",
      "covered": true,
      "reason": "string"
    }
  ],
  "test_cases": [
    {
      "id": "QA-001",
      "title": "string",
      "category": "normal|abnormal|boundary|weak_network|duplicate_request|config_error|compatibility|regression",
      "priority": "P0|P1|P2",
      "preconditions": ["string"],
      "test_data": ["string"],
      "steps": ["string"],
      "expected_results": ["string"],
      "evidence_to_capture": ["string"],
      "risk_tags": ["string"],
      "owner_review_required": false
    }
  ],
  "test_data_requirements": [
    {
      "data": "string",
      "owner": "planning|client|server|qa|ops_or_data",
      "purpose": "string",
      "blocking": false
    }
  ],
  "regression_scope": ["string"],
  "integration_checklist": [
    {
      "owner": "planning|client|server|qa|ops_or_data",
      "item": "string",
      "done_signal": "string"
    }
  ],
  "manual_review_required": [
    {
      "risk_type": "string",
      "reason": "string",
      "review_owner": "string"
    }
  ],
  "clarification_questions": ["string"]
}
```

## 规则库

- 除非明确排除，至少覆盖一个正常流、一个异常流、一个边界流、一个弱网流、一个重复请求/幂等流、一个配置错误流。
- 金钱、资产、奖励、结算、排行、账号成长、持久状态、不可恢复损失优先级为 P0。
- 协议是可测契约：验证必填字段、缺字段、非法枚举、旧客户端兼容、错误码展示、状态流转顺序。
- 配置是可测契约：验证缺失 ID、关闭开关、上下限、奖励/消耗引用、活动时间窗、热加载。
- 测试数据必须标注 Owner；不要把策划、服务端、QA 或数据平台准备工作藏在步骤里。
- 共享系统回归至少考虑 `RewardSystem`、`CostSystem`、`TaskCondition`、`ActivitySystem`、`MailSystem`、`RankingSystem`、`AttributeSystem`、`LevelUp`、`SettlementSystem`、红点、入口、日志、监控。
- 高风险链路要检查日志、流水变化、前后快照、幂等、重试、回滚和对账。
- 预期结果必须可观察，例如 UI 状态、服务端状态、背包 delta、任务进度、协议响应、日志 key、指标、持久化数据。
- 不要编造精确协议字段、配置 ID、奖励 ID 或错误码；缺失时用占位并提出问题。

## 正例

### 采集任务

输出成功采集、重复点击、请求后断线、背包满、任务已完成、采集配置缺失、发奖失败、任务面板刷新等用例；数据包含已接任务账号、采集物配置、石料 ID、奖励表、接近满背包账号和成功/失败 Mock。

### 装备升级

输出正常升级、材料不足、满级、重复请求、客户端旧等级、配置缺失、属性展示刷新、战力重算用例；人工 Review 覆盖消耗扣除、属性 delta、战力公式、日志和失败回滚。

### 恢复活动入口

输出活动开启、活动关闭、服务端开关关闭、历史屏蔽用户、红点状态、弱网刷新、旧客户端兼容等用例；先确认数据源再判断是否客户端单点。

## 反例与禁止事项

- 不要只为入口、面板、红点或进度展示生成客户端视觉用例；必须先检查服务端状态、配置门禁、历史屏蔽和协议字段。
- 不要编造请求字段、响应字段、错误码、配置 ID、奖励 ID 或日志 key。
- 不要说生成用例可以替代 QA 执行、人类验收、Code Review、风险 Review 或生产验证。
