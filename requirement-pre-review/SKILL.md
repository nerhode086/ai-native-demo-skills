---
name: requirement-pre-review
description: "在实现前预评审原始游戏策划需求。用于 AI agent 需要把粗略需求转成执行边界、影响面、风险、待澄清问题、验收条件、测试切入点，并支持 Markdown 或 JSON 输出的场景。"
---

# Requirement Pre-review

使用本 skill 将粗略策划需求转成给人类 Owner 审核的预评审包。不要实现代码、提交 PR、发布配置，也不要替代策划、QA 或 Code Review 的最终判断。

## 输入

支持自由文本、Markdown 或 JSON。推荐输入结构：

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

缺失字段只根据明确信息推断，并把不确定点写入待澄清问题。

## 工作流

1. 将需求归一化为摘要和玩家行为链路。
2. 分类需求类型：UI 展示、场景交互、奖励发放、任务推进、成长、属性、活动、排行、结算、支付、邮件、合服、数据修复等。
3. 展开真实执行边界，覆盖 planning、client、server、art、qa；只有输入明确排除时才写“无已知工作”。
4. 识别配置、协议、数据、资产、日志、埋点和兼容性影响。
5. 匹配明显通用系统，并写明置信度与依据。
6. 扫描高风险链路、弱网、重复操作和状态一致性风险。
7. 先输出阻塞正确性的澄清问题，再输出非阻塞问题。
8. 生成可验收输出：验收条件、冒烟用例、边界用例、联调清单。
9. 涉及高风险链路或 Owner 不清时，将 `manual_review_required` 设为 true。
10. 默认输出 Markdown；用户要求 JSON 时只输出合法 JSON。

## 输出格式

Markdown 使用以下小节：

```markdown
## 需求预评审
- 摘要：
- 需求类型：
- 玩家链路：

## 执行边界
- 策划：
- 客户端：
- 服务端：
- 美术：
- QA：

## 影响面
- 组件：
- 配置：
- 协议：
- 数据：
- 资产/UI：
- 日志/埋点：

## 风险
| 风险 | 等级 | 原因 | 缓解方式 |

## 待澄清问题
1.

## 可验收输出
- 验收条件：
- 测试用例：
- 联调 Checklist：
- 是否需要人工 Review：
```

JSON 使用以下结构：

```json
{
  "summary": "string",
  "requirement_types": ["string"],
  "player_flow": ["string"],
  "execution_boundary": {
    "planning": ["string"],
    "client": ["string"],
    "server": ["string"],
    "art": ["string"],
    "qa": ["string"]
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
  "data_impact": ["string"],
  "risk_points": [
    {
      "risk_type": "string",
      "level": "low|medium|high",
      "reason": "string",
      "suggestion": "string"
    }
  ],
  "clarification_questions": ["string"],
  "acceptance_criteria": ["string"],
  "test_cases": ["string"],
  "integration_checklist": ["string"],
  "manual_review_required": true
}
```

## 规则库

- UI 文案常常隐含服务端状态、协议字段、配置开关、历史入口恢复、红点、国际化、资产和 QA 回归。
- 货币、道具、邮件附件、活动奖励、战斗结算、排行奖励、支付、补偿等都需要人工 Review 和防重复机制。
- 任务进度、解锁状态、属性、战力、背包、赛季进度、Roguelike run 状态需要明确数据源和双端对齐方式。
- 改服务端状态的操作必须检查重复点击、重试、弱网、幂等键、请求顺序和客户端过期状态。
- 配置检查至少覆盖字段、ID、公式、边界值、默认值、灰度开关、兼容性和测试数据。
- 协议检查至少覆盖请求字段、响应字段、错误码、状态流转、示例数据和客户端展示字段。
- QA 覆盖至少包含正常流、边界值、重复操作、断线重连、配置缺失、背包满、等级/权限门禁、补偿或回滚路径。
- 不要编造项目 API、表名、ID 或 Owner；证据不足时写 unknown 并提问。

## 正例

### 采集石头任务

分类为场景交互、道具奖励、任务条件推进、客户端展示。边界包含策划配置采集物、任务条件与奖励，客户端采集交互与面板刷新，服务端采集校验、发奖与进度更新，QA 重复点击、断线重连、背包满和完成验证。

### 恢复隐藏入口

不要判为纯客户端。检查历史屏蔽原因、服务端功能开关、玩家资格、协议/状态来源、红点、旧配置兼容、埋点和回滚。询问由谁确认重新开放，以及哪些玩家分群可见。

### 装备升级

分类为成长、消耗、属性、战力。边界包含策划公式和消耗表，客户端预览与结果展示，服务端消耗校验和等级更新，属性/战力重算，QA 材料不足、满级、重复点击、配置缺失、断线重连和属性展示回归。

## 反例与禁止事项

- 不要把跨角色变化缩窄成单角色任务，例如只写“客户端加按钮和进度文本”。
- 不要让支付、资产、结算、排行奖励、邮件附件、合服数据或数据修复绕过人工 Review。
