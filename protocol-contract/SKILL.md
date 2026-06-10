---
name: protocol-contract
description: "在客户端和服务端实现前定义游戏功能协议契约。用于 AI agent 需要把玩法需求转成请求字段、响应字段、错误码、状态流转、示例数据、兼容性说明和可供 Mock 使用的 API 规格，从而支持双端并行开发的场景。"
---

# Protocol Contract

使用本 skill 在开发前先稳定双端契约。协议草案应服务客户端展示、服务端校验、QA 断言和 Mock 联调，不要暴露无关内部实现。

## 输入

- 原始玩法需求、业务目标、验收标准。
- 涉及模块、相关配置表、已有协议或历史接口。
- 客户端展示需求、服务端校验需求、QA 场景。
- 是否涉及资产、奖励、支付、战斗结算、排行榜、合服、数据修复等高风险链路。
- 输出格式：默认 Markdown；用户要求时输出 JSON。

缺少关键上下文时，先列待澄清问题，再给出带假设标注的草案。

## 工作流

1. 结构化需求，提取玩家动作、服务端事实、客户端展示和验收点。
2. 定义协议边界，只覆盖本需求必要的数据交换。
3. 先写请求，再写响应；字段必须标注类型、必填、来源和校验。
4. 补齐错误码、状态流转、重复请求和断线重连规则。
5. 生成正常、异常、边界三类示例数据。
6. 输出 Mock Server 与 Mock Client 可消费的规格。
7. 标记高风险链路并要求人工 Review。
8. 按用户要求输出 Markdown 或 JSON。

## 输出格式

Markdown 使用：`摘要`、`请求`、`响应`、`错误码`、`状态流转`、`示例数据`、`Mock 规格`、`兼容性`、`人工 Review`。

JSON 输出为单个对象，不要包在代码块外添加解释：

```json
{
  "contract_name": "string",
  "version": "v1",
  "endpoint": "string",
  "method": "POST",
  "idempotency_key": "client_cmd_id|null",
  "request": [{"name": "string", "type": "string", "required": true, "source": "client|server|config", "validation": "string", "example": "any"}],
  "response": [{"name": "string", "type": "string", "required": true, "usage": "client_display|state_sync|log|qa_assertion", "example": "any"}],
  "error_codes": [{"code": "string", "reason": "string", "retryable": false, "client_message": "string", "log_level": "info|warn|error"}],
  "state_transitions": [{"from": "string", "to": "string", "trigger": "string", "invalid_transitions": ["string"]}],
  "sample_data": {"success": {}, "errors": [], "edge_cases": []},
  "mock_spec": {"scenarios": [], "latency_ms": [0, 300, 1500], "state_switches": []},
  "compatibility": ["string"],
  "manual_review_required": false,
  "clarification_questions": ["string"]
}
```

## 规则库

- 改变资产、任务、进度、排行、邮件、活动领奖或战斗结果的请求，必须包含 `client_cmd_id` 或等价幂等键。
- 错误码不能只写“失败”；必须说明触发条件、客户端表现、服务端日志和是否可重试。
- 状态流转必须覆盖成功、失败、重复请求、弱网超时、断线重连、配置缺失。
- 响应字段只包含客户端展示、状态同步、日志追踪或 QA 断言需要的内容。
- 请求字段必须区分客户端传入、服务端推导、配置读取和后台上下文。
- 示例数据必须使用稳定 ID、可重复数值和可被 Mock 复现的状态。
- 涉及奖励、支付、战斗结算、邮件附件、活动领取、排行榜、合服时，输出 `manual_review_required: true`。
- 旧协议兼容策略不明确时，默认新增可选字段和服务端默认值，不破坏旧客户端。

## 正例

### LevelUp 契约

输入：“装备升级消耗金币和材料，提升属性和战力。”

输出：`LevelUpReq(target_type,target_id,client_cmd_id)`，`LevelUpResp(new_level,cost_items,attr_delta,battle_power_delta,bag_changes)`，并覆盖材料不足、满级、重复请求和配置缺失错误码。

### 采集任务契约

输入：“采集石头获得石料并推进任务。”

输出：定义 `CollectReq(object_id,scene_id,client_cmd_id)`，响应包含掉落、背包变化、任务进度 delta 和下一状态；状态覆盖未采集、采集中、已采集、奖励已领取。

### 活动领奖契约

输入：“领取活动奖励。”

输出：给出领取协议和 Mock 规格，同时将奖励发放标记为高风险，要求服务端、策划和 QA 人工 Review。

## 反例与禁止事项

- 不要只列请求和响应字段；必须包含错误码、状态流转、示例数据和 Mock 场景。
- 不要把支付、奖励、结算或合服流程描述成 AI 可直接执行；必须人工 Review。
- 不要为了客户端方便而暴露内部临时字段、表结构或破坏兼容性的枚举改名。
