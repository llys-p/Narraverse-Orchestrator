# TASK-002A：状态字段映射证明与历史评估（验证路线 B）

## 执行者

Workbuddy

## 任务类型

只读调查 + 分析（不修改 Narraverse2.0 架构代码）。

## 背景

TASK-001 产出 SYSTEM_MAP 后，豆包在 `docs/research/ARCHITECTURE_REVIEW.md` 完成独立复核，结论：
- 路线 B（保留资产，重构核心）降级为"**首选验证假设，而非推荐路线**"；
- 新增候选 **A+**（共享数据/事实层 + 独立运行时）；
- "统一为一个 World Runtime"是未加审查的前提，需比较 **1 vs 2 个运行时**；
- Game "预设"抽象对 Module4（时钟/日程/跨日结算）被过度拉伸；
- 无跨系统 **Canon/事实采纳**是一级产品缺口；
- 浅克隆使"资产是否值得保留"缺少历史维度，要求 TASK-002 使用非浅克隆。

## 目标（验证，不是默认支持 B）

以**可以证伪**的态度，验证路线 B 的核心前提：
> "Game Mode 的事件溯源 + Actor State + 预设加载器可以作为统一运行时内核，承载叙界文字冒险与 Module4 的状态与玩法。"

如果字段级映射显示无法表达的语义超过可接受阈值，或历史评估不支持"Game 内核值得作为重建基础"，则 B 的前提被削弱/证伪，应转向 A+ / 双运行时族等候选。

## 调查范围（只读）

1. Game Actor State ↔ 叙界状态协议（[STATE]/[CHANGES] 等 8 标签 + adventure 序列化字段）逐字段映射；
2. Game Actor State ↔ Module4 facts/world schema（world v1.5、facts、knowledge、events、clock、npcRelations…）逐字段映射；
3. 列出无法统一/无法在 Actor State 内表达的字段与语义，并判断哪些可经"内核扩展"解决、哪些属于根本范式差异；
4. 现有三类存档格式调查（叙界 IndexedDB/localStorage、Module4 localStorage、Game jsonl/sidecar），为后续迁移评估提供格式基线；
5. 非浅克隆历史分析：模块变更频率、bug 修复情况、耦合情况（注意：Narraverse2.0 GitHub 历史仅有 8 commits/5 天，需如实报告该限制）。

## 唯一事实源

- GitHub 私有仓库 `llys-p/Narraverse2.0`，branch `main`，commit `b37acfc76ebddd02760ab1794c3dce6e92485dbc`
- 本地克隆：`C:/Users/11/WorkBuddy/2026-09-07-14-00-01/Narraverse2.0-remote`（历史已补全：8 commits，2026-09-03 起）
- 禁止以 `E:\Narraverse`、`D:\Narraverse2.0` 作为 Narraverse2.0 当前架构事实来源。

## 产出

- `docs/research/STATE_MAPPING_ANALYSIS.md`（字段映射证明 + 存档格式 + 历史评估 + 对 B 的验证结论）
- `docs/handoffs/HANDOFF-002A.md`

## 验收标准

- [ ] 所有结论标出真实文件路径 + 类型/函数/API 键名（基于 b37acfc）；
- [ ] 字段映射分"直映 / 需内核扩展 / 无法表达"三档，且给出无法表达清单及原因；
- [ ] 存档格式按真实序列化键名描述；
- [ ] 历史分析如实报告 GitHub 历史深度限制，不夸大结论；
- [ ] 对路线 B 给出可证伪的验证结论（支持 / 削弱 / 证伪 + 阈值说明）；
- [ ] 未修改 Narraverse2.0 架构代码；
- [ ] HANDOFF 明确下一步最适合交给谁及原因。
