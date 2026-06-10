# 通用组件索引

本文件只维护 `generic-component-matching` 的组件索引和按需加载策略。每个组件的详细说明放在 `references/components/<ComponentName>.md`。

## 按需加载策略

1. 先从需求中抽取触发器、对象、状态、消耗、奖励、展示、持久化和风险。
2. 用下方索引筛出候选组件。
3. 只读取候选组件对应的 md 文件，不要一次性加载全部组件。
4. 多组件命中时，加载主组件和必要辅助组件，输出主组件、辅助组件、数据流和责任边界。
5. 找不到合适组件时，输出 `new_component_candidate`，并说明为什么已有组件不足。

## 新增组件步骤

1. 在 `references/components/` 下新增 `<ComponentName>.md`。
2. 使用组件文件模板补齐适用场景、触发词、输入输出、配置、协议、QA、组合、不适用场景和人工 Review。
3. 在本索引的“组件索引”表里追加一行。
4. 不要修改 `SKILL.md`，除非匹配流程、输出 schema 或全局红线发生变化。

## 组件文件模板

```markdown
# ComponentName

- 适用场景：
- 常见触发词：
- 典型输入：
- 典型输出：
- 配置关注：
- 协议关注：
- QA 关注：
- 常见组合：
- 不适用场景：
- 人工 Review：
```

## 组件索引

| 组件 | 说明文件 | 快速命中线索 |
| --- | --- | --- |
| InteractionObject | `components/InteractionObject.md` | 采集、点击、打开、进入、触发、交互、场景对象 |
| DropSystem | `components/DropSystem.md` | 掉落、随机奖励、掉率、掉落池、权重、来源飘字 |
| RewardSystem | `components/RewardSystem.md` | 奖励、领取、发放、获得、补偿、首通、里程碑 |
| CostSystem | `components/CostSystem.md` | 消耗、扣除、花费、材料不足、购买、升级消耗 |
| LevelUp | `components/LevelUp.md` | 升级、升星、突破、学习、成长、解锁、进阶 |
| AttributeSystem | `components/AttributeSystem.md` | 属性、力量、攻击、生命、速度、战力、加成 |
| AttributeConvertRule | `components/AttributeConvertRule.md` | 转化、按比例、百分比、换算、来源属性影响目标属性 |
| TaskCondition | `components/TaskCondition.md` | 任务、进度、完成、条件、成就、目标、引导 |
| ComposeSystem | `components/ComposeSystem.md` | 合成、制作、碎片、配方、兑换、转换 |
| MailSystem | `components/MailSystem.md` | 邮件、附件、补偿、离线发放、批量发奖 |
| RankingSystem | `components/RankingSystem.md` | 排行、榜单、名次、赛季、结算、排名奖励 |
| ActivitySystem | `components/ActivitySystem.md` | 活动、限时、入口、日历、开启、关闭、领奖 |
| SettlementSystem | `components/SettlementSystem.md` | 结算、胜负、分数、通关、退出 run、战斗结果 |
