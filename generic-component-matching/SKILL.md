---
name: generic-component-matching
description: "从原始游戏功能需求中识别可复用通用组件，并根据 reference 中的组件目录输出复用、扩展或适配建议。用于 Codex 需要分析需求、避免一次性硬编码、匹配 InteractionObject、RewardSystem、LevelUp、AttributeSystem 等已有或新增组件，并输出 Markdown 或 JSON 组件匹配建议的场景。"
---

# Generic Component Matching

使用本 skill 将具象游戏需求转成通用组件匹配结果。先考虑复用、扩展或适配层，再提出新逻辑。

组件索引维护在 `references/component-catalog.md`，每个组件的详细说明维护在 `references/components/<ComponentName>.md`。新增通用组件时优先新增一个组件 md，并把它登记到索引；不要改动本文件，除非匹配流程、输出 schema 或全局红线发生变化。

## 输入

- 原始需求、业务目标、玩家链路、验收标准。
- 已有系统、配置、协议、历史案例、已知约束。
- 受影响角色：planning、client、server、art、qa。
- 输出格式：默认 Markdown；用户要求时输出 JSON。

## 工作流

1. 提取需求中的动词、名词、触发器、消耗、奖励、状态、展示和持久化需求。
2. 读取 `references/component-catalog.md`，先用索引筛出候选组件。
3. 只加载候选组件对应的 `references/components/<ComponentName>.md`，再给出命中证据、复用方式和缺口。
4. 复用优先级为 `direct_config`、`extend_component`、`adapter_layer`、`compose_components`，最后才是 `new_component_candidate`。
5. 识别协议、配置、数据、UI、QA 和历史兼容影响。
6. 列出禁止硬编码的部分，例如道具 ID、奖励数值、等级阈值、可见性开关、公式常量。
7. 涉及资产、支付、结算、奖励、邮件附件、排行、合服、数据修复、属性或战力时，标记人工 Review。
8. 默认输出 Markdown；JSON 模式下保留所有字段，未知内容用空数组。

## 输出格式

Markdown 输出：

- 需求摘要。
- 组件匹配表：组件、置信度、证据、复用方式、是否需要扩展。
- 角色影响：策划、客户端、服务端、QA。
- 硬编码风险与可复用替代方案。
- 待澄清问题。
- 高风险人工 Review 说明。

JSON 输出：

```json
{
  "summary": "string",
  "matched_components": [
    {
      "component": "string",
      "confidence": "high|medium|low",
      "evidence": ["string"],
      "reuse_mode": "direct_config|extend_component|adapter_layer|compose_components|new_component_candidate",
      "need_extension": false,
      "avoid_hardcode": ["string"]
    }
  ],
  "role_impact": {
    "planning": ["string"],
    "client": ["string"],
    "server": ["string"],
    "qa": ["string"]
  },
  "clarification_questions": ["string"],
  "manual_review_required": false,
  "manual_review_reason": ["string"]
}
```

## 核心规则

- 候选组件先看 `references/component-catalog.md`；组件命中、适用场景、配置/协议关注点和典型组合以对应的 `references/components/<ComponentName>.md` 为准。
- 多步骤玩法应拆成组件组合，不要压成单一 Owner 模块。
- 需求命中多个组件时，分别说明主组件、辅助组件和数据流向。
- 未在组件目录中找到合适组件时，先输出 `new_component_candidate`，并说明为什么已有组件不足。
- 新增组件时新建一个组件 md，并在索引中登记；只有输出格式、匹配流程或人工 Review 红线变化时才改本文件。

## 正例

### 采集任务

读取索引后，只加载 `InteractionObject`、`RewardSystem`、`TaskCondition`，必要时加载 `DropSystem`。匹配采集点、矿石发放和任务进度，并询问采集配置、奖励组、任务条件 ID、重复采集和断线处理。

### 装备升级

匹配 `LevelUp`、`CostSystem`、`AttributeSystem`，必要时用 `RewardSystem` 包装升级结果。不要在升级代码里硬编码属性增量；要求等级、消耗、属性增量、满级和日志配置化。

### 赛季排行奖励

匹配 `RankingSystem`、`RewardSystem`、`MailSystem`，可能匹配 `SettlementSystem`。要求快照时间、排名区间、奖励组、邮件模板、幂等键和人工 Review。

## 反例与禁止事项

- 不要把“调整入口 UI 展示”判为纯客户端，除非确认服务端状态、活动配置、历史屏蔽标记和解锁条件都无影响。
- 不要为了一个活动奖励写内联道具 ID 或服务端分支；优先走 `ActivitySystem` + `RewardSystem` 配置。
- 不要编造已有 API、表名或协议字段；证据不足时写假设和问题。
- 不要为了新增一个组件而改写本 skill 主流程；优先新增独立组件 md，并更新索引。
