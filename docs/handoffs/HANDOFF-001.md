# HANDOFF：TASK-001（Narraverse2.0 全系统地图）

## 执行者

Workbuddy（2026-09-07）

## 已完成

- 读取协同仓库规则：`README.md`、`AGENTS.md`、`docs/MASTER_PLAN.md`、`docs/CURRENT_STATE.md`、`docs/AI_COLLAB.md`、`docs/templates/*`、`prompts/WORKBUDDY_START.md`、`docs/tasks/TASK-001-system-map.md`。
- 调查前身份验证与记录（纠正流程要求）：
  - 调查对象远程：`https://github.com/llys-p/Narraverse2.0.git`（私有）
  - 分支：`main`；HEAD：`b37acfc76ebddd02760ab1794c3dce6e92485dbc`（"feat: 完善总资料库管理与本地入口"）
  - 本地调查路径：`C:/Users/11/WorkBuddy/2026-09-07-14-00-01/Narraverse2.0-remote`
- 对 Narraverse2.0 做 8 区域只读调查（A 资料导入与整理 / B 总资料库 / C 具体世界与冒险资料 / D 写作模式 / E 游戏模式预设体系 / F 叙界文字冒险与桥接 / G Module4 开放沙盒 / H 技术边界），结论全部基于克隆内真实代码，主要证据与函数/API 行号见 `docs/research/SYSTEM_MAP.md`。
- 产出：`docs/research/SYSTEM_MAP.md`（总览/数据流/三套对照表/重复清单/可复用清单/废弃清单/A-B-C 初步判断/待调查问题 + 10 个核心问题回答）。

## 修改内容

- 仓库：`llys-p/Narraverse-Orchestrator`（协同仓库）
- 分支：`main`
- 文件（新增）：
  - `docs/research/SYSTEM_MAP.md`
  - `docs/handoffs/HANDOFF-001.md`
- Commit：见本 HANDOFF 提交记录。
- 未修改 `llys-p/Narraverse2.0` 任何代码或文档（只读）。

## 验证

- 本次为纯只读调查，未执行构建/测试/运行，未调用任何模型 API。
- git 身份记录：remote=github.com/llys-p/Narraverse2.0.git，branch=main，HEAD=b37acfc76ebddd02760ab1794c3dce6e92485dbc（与远端一致）。
- 结论交叉验证方式：A–H 每区由独立调查产出证据（文件路径+符号/行号），关键论断（如四模块模型网关归一、预设组合开关、总库 sha256 去重、Module4 纯规则检定、复制投影）均有多处代码引用；无法在克隆验证的生成物/运行时状态已在 SYSTEM_MAP §11 与各节标注"待确认"，未当作事实。
- 事实来源声明：本任务未使用 `E:\Narraverse`、`D:\Narraverse2.0`、`E:\Workspace` 等旧目录作为当前架构依据（SYSTEM_MAP 开头已声明）。

## 尚未完成

- SYSTEM_MAP §11 列出的 8 项待确认/待验证（`.denova` 预设样例抽查、变更协议无损表达对照、计分 vs d20 统一可行性、嵌入态模型路径实测、双进程写穿现状、字段漂移实测、Module4↔叙界互通入口、存量存档规模）。
- 路线 A/B/C 的正式锁定（本报告只给初步判断，未实施任何重构）。
- 未更新协同仓库 `docs/CURRENT_STATE.md`（如需可在后续 TASK 一起做，避免多人同时改同一批文件）。

## 风险与疑点

- 克隆为 depth 1，历史演进不可见；`local_library.js`、`.denova/`、`config.toml`、`output/` 等 gitignore 生成物未在克隆内，相关结论标注为"生成/运行时才有"。
- 两套模型调用路径（服务端网关 vs 浏览器直连）并存，嵌入态是否仍会触发本地直连兜底未经运行态实测，可能影响实际割裂程度。
- Go 与 Python(8097) 直接读写同一 `.denova/projects` 文件树，克隆内未见锁协调证据，属于一致性与冲突风险点。
- 本 HANDOFF 与 SYSTEM_MAP 的代码行号基于 commit `b37acfc`；若 Narraverse2.0 main 前进，行号可能漂移（结论语义一般不受影响）。

## 关键结论

1. 三套互动玩法（Game Mode / 叙界文字冒险 / Module4）**具备共享一个运行时核心的条件**：模型网关已把三者统一映射到 `interactive_story` Agent kind；Game Mode 已具备事件溯源内核 + Actor State schema + 预设加载器（可组合、可独立关闭），是最可能的统一内核骨架。
2. Game Mode 预设体系对另两套的覆盖度：状态系统/检定/事件包可覆盖大部分；**未覆盖** Module4 的时钟-日程-跨日结算与叙界 World Info 关键词注入（需新增"世界时钟/上下文预设"类组件）。
3. 最严重三个结构性问题：① 同一角色/条目 3–5 库并存且冒险与 Module4 均为"复制投影"，事实真源不唯一；② 三套各自状态机与规则引擎，无共用 actor/facts schema；③ Go 服务与 8097 文件桥+浏览器直连双轨并存（双模型配置、双 lore 入口、4 条同步线）。
4. 初步路线判断：**更支持 B（保留资产，重构核心）**——以 Master Library 为统一资料层 + Game 事件溯源/预设/Actor State 为统一运行时内核 + 模型网关为统一调用层；叙界收敛为回合协议预设、Module4 收敛为世界模拟预设。未锁定，需 TASK-002 原型验证。
5. 高价值资产清单与废弃候选见 SYSTEM_MAP §7/§8（Master Library 数据架构、Game 预设体系、事件溯源内核、Module4 确定性内核、叙界角色卡解析与提示词管线值得保留）。

## 下一步建议

下一步最适合交给：**Workbuddy**。

原因：路线定案前仍有 8 项"待确认/运行态验证"属于大规模只读补查与预研（SYSTEM_MAP §11），这正是 Workbuddy 的默认职责；Codex 额度稀缺，应在 WORKBUDDY 完成验证、明确"统一内核取舍清单"之后再进入跨模块核心重构；豆包额度稀缺，适合在正式锁定 B/C 路线时做一次反方复核（本报告已给初步判断，但属重大架构决策，建议复核后落地）。

建议任务顺序：
1. **Workbuddy 执行 TASK-002**（验证 §11 待确认项 + 出具"统一运行时取舍清单"）；此阶段无需用户中转新 AI。
2. 若 §11 验证后要**锁定路线**：先由豆包复核（第二意见），再交 Codex 实施核心重构。
3. 全程继续遵守：不修改 Narraverse2.0 正式代码以外的无关范围；每次执行有新 TASK；结束必出 HANDOFF。

## 给用户的直接操作

方式一（推荐，继续由 Workbuddy 深化验证）：

> 把下面这段发给 Workbuddy：
>
> 请先读取协同仓库 `llys-p/Narraverse-Orchestrator` 的 `docs/research/SYSTEM_MAP.md` 与 `docs/handoffs/HANDOFF-001.md`，然后创建并执行 TASK-002：对 SYSTEM_MAP 第 11 节的 8 项待确认清单做只读验证（`.denova` 预设样例、submit_interactive_turn 与 8 标签协议/Module4 facts 的对照、计分与 d20 统一可行性、嵌入态模型路径实测、8097/8080 双写现状、四库字段漂移、Module4↔叙界互通、存量存档规模），产出一份"统一运行时取舍清单"（哪些进内核/哪些做预设/哪些迁移/哪些废弃），仍不修改 Narraverse2.0 正式代码，完成后更新 `docs/research/SYSTEM_MAP.md` 与 `docs/handoffs/HANDOFF-002.md`。
>
> 它完成后，请把 **HANDOFF-002** 带回来。

方式二（若想先让豆包复核路线判断，再决定是否进入核心重构）：

> 把下面这段发给豆包：
>
> 请阅读协同仓库 `llys-p/Narraverse-Orchestrator` 的 `docs/research/SYSTEM_MAP.md`，对其中第 9、10 节做反方复核：三套互动系统（Denova Game Mode / 叙界文字冒险 / Module4）是否真的应该收敛为"一个事件溯源+预设化的统一运行时"？请重点挑战：预设体系是否足以承载 Module4 的时钟/日程/跨日结算与叙界的回合协议；"以 Master Library 为统一资料层"是否会把运行时世界资料与母版资料过度耦合；以及 A/B/C 三路线各自的真实风险。给出明确结论与最需要先验证的 1–3 件事。
>
> 它完成后，请把 **复核意见** 带回来（用于决定下一步交给 Workbuddy 还是 Codex）。
