# HANDOFF：TASK-002A（状态字段映射证明与历史评估）

## 执行者

Workbuddy（2026-09-07）

## 已完成

- 阅读协同仓库 `docs/research/SYSTEM_MAP.md`、`docs/research/ARCHITECTURE_REVIEW.md`、`docs/CURRENT_STATE.md`（已更新至 9c091ae 线），并同步确认 AGENTS.md/AI_COLLAB.md v3 分工（豆包升为主力执行者之一）。
- 调查身份记录：`llys-p/Narraverse2.0` @ main @ `b37acfc76ebddd02760ab1794c3dce6e92485dbc`，本地克隆 `C:/Users/11/WorkBuddy/2026-09-07-14-00-01/Narraverse2.0-remote`；未使用 E:\Narraverse / D:\Narraverse2.0 作为当前架构事实源。
- 完成三套状态模型字段级盘点（Game Actor State / 叙界协议与存档 / Module4 world+facts），产出逐字段映射（✅直映 / 🔶需内核扩展 / ❌无法表达）与无法统一清单（U1–U10，其中 U1–U6 为核心语义域）。
- 完成存档格式调查（叙界 IndexedDB/localStorage 键与 adventure 结构、Module4 localStorage world、Game jsonl+sidecar；规模属运行态数据待 002B）。
- 将 Narraverse2.0-remote **补全为全历史克隆**并做历史分析：**GitHub 历史仅 8 commits / 5 天（2026-09-03 发布快照起）**，如实报告其对"变更频率/bug 密度/耦合演化"评估的限制；补充协作日志快照内的修复活动统计。
- 正式任务文件落盘：`docs/tasks/TASK-002A-system-state-mapping.md`。

## 修改内容

- 仓库：`llys-p/Narraverse-Orchestrator`
- 分支：`main`（基线已快进到 9c091ae）
- 文件（新增）：
  - `docs/tasks/TASK-002A-system-state-mapping.md`
  - `docs/research/STATE_MAPPING_ANALYSIS.md`
  - `docs/handoffs/HANDOFF-002A.md`
- 未修改 `llys-p/Narraverse2.0` 任何代码。

## 验证

- 纯只读调查 + 分析；未执行构建/测试/模型调用，未改正式代码。
- 字段结论由两个独立盘点代理交叉产出，证据为 file:line + 键名（克隆内复核可行）；映射判定（✅/🔶/❌）均给出代码级理由。
- 历史：`git rev-list --count HEAD` = 8，`git log` 全 8 提交人工核对；协作日志统计为 grep 计数（粗粒度）。
- 无法在克隆内验证的运行态事实均标注"待 TASK-002B / 运行环境"。

## 尚未完成

- TASK-002B（A+/B' 路线成本对比 + 存档迁移评估 + 1 vs 2 运行时 + 运行态实测：embedded 模型路径、Go/Python 双写锁、Module4↔叙界互通、存档规模）。
- 豆包第二次独立复核（按 CURRENT_STATE 计划）。
- 路线锁定。

## 风险与疑点

- 行号基于 b37acfc；main 前进会漂移。
- ".denova 预设 JSON 样例"与 local_library.js 不在克隆内，字段全集以内置模板+测试内联为准（标注于报告 §9）。
- 历史维度证据不足是**仓库发布形态的固有约束**，不是本次执行的疏漏；真实演化史需要用户授权的本地历史对比任务。
- 若用户后续把 Narraverse2.0 main 推进到新 commit，映射报告需在锁定前基于新 HEAD 复核一遍。

## 关键结论（对路线 B 的验证结果）

1. **部分支持**：叙界↔Game 状态协议原语同构（[STATE]≈replace、[CHANGES]≈delta、事件流≈jsonl），回合制家族共用内核证据较强。
2. **证伪一项**：Module4 不能被 Game 现有"预设"承载——时钟/日程/跨日结算/自主事件对象是内核级运行时能力（代码级证据），且事实/认知（facts/knowledge/knownTo/epistemicStatus）在 Actor State 中无承载。
3. **B 强版本（单运行时统一三套）不支持**：需等量级内核扩展，"以 Game 为内核"名存实亡；建议收敛为 **A+（保留运行时 + 统一 Canon/事实层 + Master 唯一真源）** 与 **B'（参考 Game 设计重建双范式内核）** 两案在 TASK-002B 对比。
4. Module4 的 facts/knowledge 是全项目唯一成熟的"事实+谁知晓+认知"模型，是 Canon 层的现成领域资产，应上浮而非下塞。
5. 历史：8 commits/5 天无法为"Game 内核经演化考验"提供支持论据，也不否定；资产判断仍以快照质量为准。
6. 与 ARCHITECTURE_REVIEW 的关系：本报告用字段级证据证实了其"preset 过度拉伸 / 单运行时前提未审查 / 需映射证明"三条异议，并对"以 Game 代码为内核"给出明确削弱结论。

## 下一步建议

下一步最适合交给：**豆包**（TASK-002B 主执行者），或按分工原则在豆包与 Workbuddy 间选择；本报告建议 002B 主体给豆包（研究/方案密集，适合其 5 小时窗口），Workbuddy 可承接其中的运行态实测部分（需本地环境）。

原因：状态映射与历史评估已完成并落盘，下一步是 A+/B' 路线成本对比与迁移评估（研究型）→ 豆包为当前主力且已掌握复核语境；运行态实测（embedded 路径/双写锁/存档规模）需要操作本地运行环境 → 更适合 Workbuddy。建议拆分执行避免单任务跨窗口。

## 给用户的直接操作

方式一（TASK-002B 主体交豆包，运行态实测并行给 Workbuddy）：

> 把下面这段发给豆包：
>
> 请先读取协同仓库 `llys-p/Narraverse-Orchestrator` 的 `docs/research/STATE_MAPPING_ANALYSIS.md`、`docs/research/ARCHITECTURE_REVIEW.md`、`docs/research/SYSTEM_MAP.md` 与 `docs/handoffs/HANDOFF-002A.md`。执行 TASK-002B：对 A+（保留运行时+统一 Canon/事实层+Master 唯一真源）与 B'（参考 Game 设计重建双范式统一内核）做同等深度的成本-风险-收益对比，加入存档迁移方案评估（叙界/Module4/Game 三格式→目标 schema 的迁移脚本方案与回滚）、1 vs 2 运行时结论、以及 Canon/事实采纳层的服务设计草案（可参考 Master Library proposal 机制与 Module4 facts 模型）。若涉及只读调查请基于 `llys-p/Narraverse2.0` 克隆（main @ b37acfc），禁止使用 E:\Narraverse 旧目录。产出 `docs/research/ROUTE_COMPARISON.md` 与 `docs/handoffs/HANDOFF-002B.md`；完成后把 HANDOFF-002B 带回来。
>
> 它完成后，请把 **HANDOFF-002B** 带回来。

方式二（若希望运行态实测先走，把下面发给 Workbuddy）：

> 请基于协同仓库 `docs/research/STATE_MAPPING_ANALYSIS.md` 的"后续还需验证"清单做运行态实测（不修改正式代码）：①叙界 embedded 首轮是否强制本地 apiConfig（NarraverseSharedAI.refresh 调用链）；②Module4↔叙界冒险是否存在用户可见数据互通；③Go(8080) 与 Python(8097) 同写 .denova/projects 的锁/冲突现状；④本地三类存档（叙界 adventure 数量、Module4 world 数量、Game story 数量）与格式抽样。产出 `docs/research/RUNTIME_VERIFICATION.md` 与 `docs/handoffs/HANDOFF-002A-RUNTIME.md`。
>
> 它完成后，请把 **HANDOFF-002A-RUNTIME** 带回来。
