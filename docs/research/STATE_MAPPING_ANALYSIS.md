# STATE MAPPING ANALYSIS：路线 B 状态字段映射证明与验证

> TASK-002A 产出 · 只读调查 + 分析 · 2026-09-07
> 任务文件：`docs/tasks/TASK-002A-system-state-mapping.md`
> 关联文档：`docs/research/SYSTEM_MAP.md`（TASK-001）、`docs/research/ARCHITECTURE_REVIEW.md`（豆包独立复核）
> 立场声明：**本报告验证路线 B 是否成立，不默认支持 B。** 对 B 不利的证据同样如实列出。

---

## 0. 调查事实来源（Provenance）

| 项 | 值 |
|---|---|
| 调查仓库 | GitHub 私有仓库 `llys-p/Narraverse2.0` |
| 分支 | `main` |
| commit SHA | `b37acfc76ebddd02760ab1794c3dce6e92485dbc` |
| 本地调查路径 | `C:/Users/11/WorkBuddy/2026-09-07-14-00-01/Narraverse2.0-remote` |
| 历史深度 | 已补全为全历史克隆（8 commits，2026-09-03 发布快照起；见 §7） |
| 事实来源声明 | 未使用 `E:\Narraverse`、`D:\Narraverse2.0` 等旧目录作为当前架构事实来源。 |

文中路径相对仓库根；行号基于 `b37acfc`。判定均分三档：✅ 直映（可直接落到 Actor State 表示）／🔶 需内核扩展（语义真实存在，但当前 Actor State 缺承载，需加组件）／❌ 无法表达或根本范式差异（即便扩内核也牵强或改变玩法本质）。

---

## 1. 三套状态模型的代码级事实摘要

### 1.1 Game Actor State（统一候选内核的现状）

- **存储形态**：单一全局 `map[string]any`，根键 `actors`（`denova-src/internal/interactive/actor_state.go:18`），其下每个 actor 是子树 `{id,name,template_id,role,description,state:{fieldID:value…},traits:[…]}`（`story_state.go:306-315`）；另有 `actor_archives` 归档（`actor_archive.go:11`）。
- **内置 actor**：`protagonist`、`story`、`world`（`actor_state.go:16`、`actor_state_presets.go:13-14`）。字段值类型仅 6 种：`number/string/bool/enum/object/list`（`actor_state.go:90`）。
- **字段全集（内置默认模板）**：identity.*、panel.level/strength…charisma、state.health/state.mana/effects/cooldowns、current.situation/goal_situation/presence_location、protagonist_relation.favorability/summary、knowledge.about_protagonist、scene.current_time/location/current_event/present_actors/continuation_hook、world.situation、tasks.current、world.locations/factions、abilities.records/relations.records/assets.important_items（证据 `actor_state_preset_fields.go:25-194`；最大 64 字段 `actor_state.go:19`）。
- **变更协议**：`submit_interactive_turn` 的 `state_changes` = oneOf `replace/delta/create/archive/restore`（`agent/interactive_story_tools.go:177-213`）；delta 仅 number、带 ≤16 段 subpath；底层 reducer op：`set/merge/push/pull/inc/unset`（`story_state.go:249-327`）。
- **序列化**：`story-<id>-actor-state.json` = `ActorStateSchemaSnapshot{version,revision,system{Templates,InitialActors,TraitPools},trpg_system,adaptation,…}`；实时 state 就是 `map`；历史在 `story-<id>.jsonl`（meta/turn/state_delta/branch/hot_choices/context_compaction…）。
- **缺什么（代码级）**：无时钟/日程/日结组件（`scene.current_time` 只是字符串）；无独立世界事实层（世界是 story/world 两个 actor 的文本字段）；无"谁知晓 knownTo/认知状态"概念（全仓无 knownTo 命中）；事件链只存在于 jsonl 历史，不在 state。

### 1.2 叙界 Narraverse（app/，浏览器）

- **持久化**：IndexedDB `adventureAI_db` store `kv`，键 `state`（备份 `state_bak`），localStorage 镜像 `adventureAI_state`(+`_bak`)；实际只序列化 `{adventures, apiConfig, currentId, customThemes}`（`app/app.js:563-570`）。对话永久档案分块 `conversationArchive:v1:<id>:chunk:N`（`app.js:402-403`）。
- **adventure 对象字段**：`createAdventure`（`app.js:903-953`）含 character{hp/mp/attributes(5)/skills/items/…}、conversationHistory、contextSummary/Compressed、plotNodes/eventNodes、plotLines/currentLineId、characterCards、backgroundBooks、mandalaCards、snapshots/branches、quests/combat、novelPlan、versionLedger、syncMeta、stats、affections/relations。
- **协议可写字段**：`[CHANGES]`→hp/mp/items/mood/skills/attributes/exp/level/skillPoints/attributePoints/affections/relations；`[STATE]`→hp/maxHp/mp/maxMp/location/chapter/profession/mood/affections；`[QUESTS]`→quests；`[COMBAT]`→combat；`[CHARACTERS]`→mandalaCards；`[PLOT]`→plotNodes（`applyParsedResult` `app.js:2127-2257`、`applyCharacterUpdates` 4159、`applyPlotUpdates` 4599）。
- **范式**：自由对话 + 模型驱动状态 diff；无导演、无世界时钟、无 NPC 日程、无日结；右栏状态台纯展示。

### 1.3 Module4（app/module4/，浏览器）

- **持久化**：localStorage 单键整存 `narraverse:module4:state`（store.js:9）+ recovery；`{version:2,currentWorldId,worlds[]}`（store.js:11）；world schemaVersion=2 / rulesVersion='v1.5'，revision 乐观锁。
- **world 字段**：player{energy,maxEnergy}、clock{day,period,tick}、npcs、locations、sourceBindings、scheduleRewrites、sceneObjects、events、npcRelations、knowledge{viewerId→[认知]}、facts[]、dailyLogs、actionLogs、narrativeEntries、currentDay{day,previewGenerated,previewStatus,schedules,events,interactedNpcIds}（world.js:204-220）。
- **fact schema**（facts.js:71-87）：`id,day,period,type,actors[],summary,visibility(public/restricted/private),knownTo[],observerId,locationId,sourceFactId,epistemicStatus(observation/heard/inference/belief),persistent,sourceActionId,createdAt`——**全项目唯一的"事实+谁知晓+认知状态"模型**。
- **npc 运行时**：id/sourceRef/name/mood/relation{stage,value}/goals[]/temporaryState[]/schedule{morning,afternoon,evening}（world.js:172-186）。
- **范式**：时间驱动模拟（时钟自主推进、NPC 日程、evening 结算），确定性规则优先（action.js parseFreeText 纯规则解析），AI 仅叙事续写与每日预演。

---

## 2. 映射一：叙界 → Game Actor State

| 叙界字段（证据） | 目标表示 | 判定 | 备注 |
|---|---|---|---|
| character.hp/mp/level/exp/attributes(力量…)/skills/items/skillPoints | actors.protagonist.state.{health,mana,panel.*,abilities.records,assets.important_items} | ✅ | 数值系统同构；`[CHANGES]` 的 inc/set/merge 与 StateOp 语义对齐；需做字段映射表（hp→state.health 等，一次性） |
| character.mood / location / chapter / profession | actor.state 自定义字段或 story actor scene.location | ✅ | 字符串字段直映（注意叙界 location 是自由文本，Module4 是 id 引用，见 §4） |
| affections（名→0-100） | protagonist_relation.favorability（仅对主角） | 🔶 | 叙界是**任意 NPC↔主角**多维表，Actor State 只给重要角色预设 favorability；多目标关系需 object 字段或 relation 组件 |
| relations（名→描述文本） | relations.records（object） | 🔶 | 对象可承载，但叙界文本关系无结构（无 stage/value），语义靠提示词 |
| quests/combat/plotNodes（剧情节点树） | actor.state 自定义 + story jsonl turn 历史 | 🔶 | 单当前任务 tasks.current 可映；**多任务队列/多 plot 节点/节点树**（plotLines/currentLineId/分支 IF 线）超出"当前任务"，需扩展或留在 jsonl |
| conversationHistory + conversationArchive（永久档案） | 无对应槽 | ❌ | Actor State 与 story jsonl 都**不存对话正文归档**（jsonl turn 有 User/Narrative 但叙界 archive 用于回溯/压缩/小说化，是独立全文历史）。叙界核心资产，需 transcript/archive 组件 |
| contextSummary/contextCompressed（前情压缩） | jsonl 的 context_compaction 事件 | ✅/🔶 | Game 有 compaction 事件但语义是"移除旧事件"，叙界是"生成摘要文本继续引导"，需对齐为同一摘要语义 |
| snapshots/branches（时间机器/分支叙事） | jsonl branch/reparent/rewind | 🔶 | Game 分支是事件流层面的；叙界 snapshot 是"状态全文快照+恢复点"，含 UI 层 restore，需语义对齐（可迁移） |
| mandalaCards/characterCards（角色卡实例） | actor 的 source 快照（类比 Module4 sourceBindings） | 🔶 | 角色卡应转为 actor 来源绑定，但叙界卡含 V3 字段（alternate_greetings/first_mes 等对话素材），不属于 state，需"卡→actor"映射规则 |
| versionLedger / novelPlan / syncMeta / stats | 不入运行时内核 | —— | 创作/同步元数据，属外围层，不需 Actor State 承载 |
| [STATE] 整段覆盖式写回 vs [CHANGES] 增量 | replace vs delta | ✅ | 协议原语一一对应，是最有力的可统一证据 |
| 无世界时钟/日程/日结/自由行动解析 | — | ✅(缺) | 叙界本就没有这些能力，**不需要**为它新增；问题只在 Module4（见 §3） |

**叙界侧小结**：回合式交互 + 模型驱动 diff + 状态快照/分支，与 Game 的回合协议是同族范式；**最核心缺口是"对话/叙事全文历史（transcript）"**——叙界把它当一等公民（压缩、回溯、小说化都基于它），Game 模型只把 turn 当事件不保留可回放全文（jsonl 里有 Narrative，但无 archive/导出语义）。若统一，需明确 transcript 归属。

---

## 3. 映射二：Module4 → Game Actor State

| Module4 字段（证据） | 目标表示 | 判定 | 备注 |
|---|---|---|---|
| player.energy/maxEnergy + clock.applyAction 消耗 | actor.state 数值 + 规则消耗 | ✅ | 属性数值直映；energy 语义=行动点 |
| npc.mood / goals[] / temporaryState[] | actor 自定义字段 | ✅ | 直映（goals 数组=object/list 字段） |
| npc.relation{stage,value} + npcRelations（NPC↔NPC 有向图，value+stage） | 无对应 | 🔶❌ | Actor State 只支持"对主角好感/关系记录"，**NPC↔NPC 有向关系图+stage 语义**需要专门组件；这是"预设字段"承载不了的数据结构 |
| facts[]{visibility,knownTo[],epistemicStatus,…} | 无对应 | ❌ | Actor State 无"独立世界事实+谁知晓+认知状态"层（全仓无 knownTo）。Module4 这套是路线 A+/Canon 层的核心资产，塞进 actor.state 会退化为散字段 |
| knowledge{viewerId→[{claim,epistemicStatus,…}]} | 无对应 | ❌ | 同上：逐 NPC 认知视图需要"认知组件"，不是字段 |
| clock{day,period,tick} + schedule{morning,afternoon,evening} + scheduleRewrites | 无对应（scene.current_time 是字符串） | ❌ | **时间驱动推进是运行时调度机制，不是状态字段**。ARCHITECTURE_REVIEW 已指出，代码级再次确认：需要"世界时钟+日程执行器+结算触发器"核心扩展，不是预设 |
| events[]{status,stage,transitions,…} 事件链跨天发展 | story jsonl 事件历史 | 🔶 | Game 分支/回合历史是"玩家触发的流"，Module4 事件是**自主演化的世界事件对象**（candidate/active/resolved/expired、deadline 触发器）；需"世界事件"对象组件，语义不同 |
| settlement（evening 日结→day+1） | 无 | ❌ | 时间驱动的日界结算 = 调度器能力，非状态 |
| actionLogs/dailyLogs/narrativeEntries（规则化轨迹+AI 叙事） | jsonl turn 历史 | 🔶 | 结构不同：Module4 有 day/period/type/before/after 结构化日志，Game 是 TurnEvent；可转换但需 schema 迁移 |
| locations[]（id 引用）+ 移动/在场判定 | actor.state 自定义 + rule binding | 🔶 | 叙界/Module4 的 location 语义（引用+在场）与 Game（字符串）不同，需位置模型统一 |
| sourceBindings.snapshot（角色卡来源快照） | actor.source 绑定 | ✅ | 与"卡→actor"思路同构，可复用 |
| parseFreeText 自由输入→动作规则解析 | prepare_turn 内意图解析 | 🔶 | 思路可平移为"自由文本→结构化动作"，但 Game 目前由模型/导演驱动而非确定性正则——玩法侧差异，见 §5 |

**Module4 侧小结**：Module4 是**时间驱动模拟族**，其"事实+认知+时钟+日程+自主事件"五件套中，前两件（事实/认知）是宝贵的领域模型（应上浮到统一事实/Canon 层），后三件（时钟/日程/结算）是需要**核心运行时扩展**的调度机制——它们都不是 Game 现有"预设"能覆盖的配置项。

---

## 4. 无法统一 / 需要单独处理的字段与语义清单

| # | 语义 | 在哪套 | 为什么不能简单统一 | 可解途径 | 是否属于"内核扩展"而非"预设" |
|---|---|---|---|---|---|
| U1 | 对话/叙事全文历史（transcript + 永久归档 + 压缩 + 小说化输入） | 叙界 | Actor State 与 story jsonl 无 archive/回放全文语义 | 新增 transcript 组件（内核）或外置归档服务 | 内核扩展 |
| U2 | 时间驱动推进：clock day/period/tick、NPC 日程、跨日 settlement | Module4 | 自主时间循环 ≠ 回合制事件流；scene.current_time 只是字符串 | 新增世界时钟/日程/结算内核组件 | **是**（否决"纯预设可承载"） |
| U3 | 世界事实 + 谁知晓 + 认知状态（facts/knowledge，knownTo/epistemicStatus） | Module4 | Actor State 无事实层/认知图 | 上浮为**统一 Canon/事实层**（A+ 方向），运行时经服务读写 | 独立层，非 Actor State 内部件 |
| U4 | NPC↔NPC 有向关系（value+stage）+ 叙界多维好感 | Module4/叙界 | Actor State 仅"对主角"关系 | 关系图组件（内核）或事实层 | 内核扩展 |
| U5 | 分支叙事（plotLines/IF 线/snapshot restore） | 叙界 | Game jsonl 分支是事件流分支；叙界是"内容分支+恢复点"双语义 | 语义对齐协议（transcript 组件内） | 需设计 |
| U6 | 世界事件对象（自主演化 candidate→active→resolved、deadline） | Module4 | Game 无自主世界事件对象 | 事件引擎组件（内核） | 内核扩展 |
| U7 | 多 plot 节点树 / 任务队列 | 叙界 | Actor State 单 tasks.current | object 字段可承载（≤64 字段约束需评估） | 可预设/字段 |
| U8 | 角色卡 V3 对话素材（alternate_greetings/first_mes/…） | 叙界 | 属"卡"而非"state" | 卡→actor 来源绑定 + 素材库 | 数据映射 |
| U9 | 关系/事实中的自由文本 vs 结构化 | 全 | 叙界文本关系、Module4 结构化 | 允许 actor.state 文本字段 + 事实层结构化并存 | 设计选择 |
| U10 | location 自由文本 vs id 引用+在场 | 叙界 vs Module4 | 语义不一致 | 位置模型统一（引用 or 文本） | 需决策 |

**阈值判断**：无法直接表达的不是"零星字段"，而是 **U1–U6 六个核心语义域**（transcript、时间驱动、事实认知、关系图、分支叙事、自主事件）。其中 U3/U4 的一部分应上浮到"统一事实层"，其余（U1/U2/U5/U6）**每一项都要求对现有 Game 内核做结构性扩展**。据此，ARCHITECTURE_REVIEW 的担心得到代码级证实：**"以现有 Game 代码为内核直接承载另两套"不成立；成立的是"以 Game 的设计模式（事件溯源+actor schema+预设）+ 上述内核扩展 + 独立事实层"重建一个统一内核——这实际上接近 B'（参考 Game 设计的新内核）或 A+（保留多运行时 + 统一事实层）**。

---

## 5. 对路线 B 的验证结论（可证伪）

1. **部分支持**：回合制家族（Game + 叙界）在状态协议原语层面高度同构（[STATE] 整段写回≈replace、[CHANGES] 增量≈delta；事件流≈jsonl）。若目标是"Game+叙界共用一核"，B 的证据较强。
2. **明确削弱**：Module4 的时间驱动 + 事实/认知 + 自主事件三块，代码级证明**不是预设配置**而是运行时/数据层能力。把 Module4 塞进 Game 回合内核，需要新增时钟/日程/结算/事件对象/事实层等**与内核等量级**的扩展——届时"以 Game 为内核"名存实亡。
3. **因此**：**"单运行时统一三套"（B 的强版本）在本轮字段映射下不支持**；建议把验证目标改为：
   - **两个运行时族**（回合制：Game+叙界；时间驱动：Module4）+ **统一数据/事实层（Canon 服务）**——即 ARCHITECTURE_REVIEW 的 M3/A+ 方向，且 U3/U4 的事实/关系资产恰好是 Canon 层的现成领域模型；
   - 或 **B'**：参考 Game 事件溯源/actor schema/预设**设计**一个能同时承载回合制与时间驱动的内核（工程量接近 C 的一层，需 TASK-002B 成本对比）。
4. **预设抽象边界**（回应复核）：可作纯配置预设的 = 叙事风格/图像方案/事件包内容/TRPG 规则集（d20 与 Module4 计分同为"检定规则集"，可预设化切换）；**不可作预设**的 = Actor State schema 本身、时钟/日程/结算、事实/认知层、transcript——它们必须进内核或独立层。
5. **对叙界"世界管理/档案中心"定位**：叙界作为前端视图/档案中心仍成立，但它的 transcript/archive/小说化资产是回合制内核与 Canon 层都要接的源，不能简单降级为"薄壳"。

---

## 6. 现有存档格式调查（为迁移评估提供格式基线）

> 数据规模（冒险数量/world 数量/story 数量）属于运行态数据，克隆内不可得，需 TASK-002B 在运行环境统计。此处只给**格式与键名**事实。

### 6.1 叙界（浏览器）
- 位置：IndexedDB `adventureAI_db` / store `kv`（主键 `state`、备份 `state_bak`）；localStorage `adventureAI_state`（>1MB 仅主存、>4MB 删除镜像）。
- 内容：单 JSON 根 `{adventures[], apiConfig, currentId, customThemes}`（apiConfig 含 endpoint/apiKey/model/扩展配置——**密钥随档存浏览器**）。
- adventure 元素字段见 §1.2；`conversationHistory` 消息 `{role,content,id?,createdAt?}`，首条为 system。
- 永久档案：IndexedDB 键 `conversationArchive:v1:<id>:meta|chunk:N`（meta 含 version/adventure_id/next_seq/chunk_count/active_ids/recovery_status）。
- 快照/分支在 adventure 内嵌（snapshots[]/branches[]）；novelPlan/versionLedger 内嵌。
- 迁移含义：无 schema 版本号（adventure 对象无 version 字段）；apiConfig 与创作档案耦合在同一键，迁移需拆分。

### 6.2 Module4（浏览器）
- 位置：localStorage `narraverse:module4:state`；恢复键 `…:recovery`。
- 内容：`{version:2, currentWorldId, worlds[]}`；world 字段见 §1.3；**world 有 schemaVersion=2/rulesVersion='v1.5'/revision 乐观锁**——是三套中唯一自带版本与并发控制的结构。
- 迁移含义：可版本化迁移（migrations.js apply 已示范 v1.5 归一路径）；但 rulesVersion legacy 分支仍存在（旧世界兼容），迁移需处理 legacy 世界。

### 6.3 Game（Denova 工作区，运行时目录 .denova/）
- 位置：`<workspace>/interactive/story/story-<id>.jsonl`（首行 StoryMeta + StoryEventRecord 逐行）+ `story-schema/story-<id>-actor-state.json`（冻结 schema）+ `usage` jsonl + 分支私有目录 `…/director/main/`（DirectorPlan）。
- 事件类型：meta/turn/state_delta/branch/hot_choices/context_compaction(+removed)；envelope `{v,type,id,parent_id,branch_id,ts}`。
- 恢复：`snapshotFromLines` 全量重放。
- 迁移含义：**无显式存档版本号在 story 文件内**（schema 冻结在 sidecar，版本=该快照 revision）；格式纯 JSONL 可解析，但"重放即当前态"的设计使字段演进需迁移器或双读兼容。

### 6.4 共性观察
- 三套存档彼此隔离、格式互异、键名自成体系；**没有任何一套有"导出给外部/迁移用"的稳定 schema 文档**（叙界 export schema_v4 只覆盖导出剧情素材，非状态迁移格式）。
- 迁移到统一 schema 的最小公共子集：actor 级数值/文本状态（✅）、关系（需图）、事实（Module4 独有）、时间（Module4 独有）、transcript（叙界独有）——即 §4 映射结论的存储侧翻版。

---

## 7. 非浅克隆历史分析

### 7.1 事实：GitHub 历史只有 8 commits / 5 天
补全历史后（fetch --unshallow，2026-09-03→09-07，全为作者 llys-p）：

| commit | 日期 | 类型 | 规模 |
|---|---|---|---|
| 6ba260b | 09-03 | chore: 发布 Narraverse 2.0 交付快照 | 1551 文件 / +459K（**全量导入**） |
| 85b9640 | 09-03 | docs: 跨机部署交接 | 6 文件 |
| 6ff98f8 | 09-04 | docs: AI 协作决策原则 | 5 文件 |
| 02154d3 | 09-04 | docs: 渐进上下文阅读 | 3 文件 |
| 1ad0ee5 | 09-06 | feat: 集成 Narraverse 2.0 更新 | 132 文件 / +75K（**成批功能**） |
| 6da1b5c | 09-06 | merge origin/main | 2 文件 |
| a4568d7 | 09-06 | docs | 1 文件 |
| b37acfc | 09-07 | feat: 完善总资料库管理与本地入口 | 30 文件 / +1.8K |

### 7.2 由此得出的结论（含对 ARCHITECTURE_REVIEW M6 的回应）
1. **M6 的前提不成立**：本仓库是"发布形态"仓库——93%+ 的插入量来自两次整库快照（6ba260b、1ad0ee5），没有每功能粒度提交。**"最近 6 个月变更频率/bug 密度"无法从 GitHub 历史得出**。真实开发历史在本地工作副本（按协作规则须另开历史对比任务，本任务未使用 D:\）。
2. **模块变更频率**：仅能观察 b37acfc（总资料库纵向切片：Go `internal/book` + `web/src/features/library` + api-client + i18n + 配套 tools 测试）与 1ad0ee5（横跨 app/module4、denova 前端/后端、tools、docs/architecture 的成批更新）。前者显示清晰的"Go 数据层+前端+测试"一起改的切片习惯；后者显示**单次功能批量横跨全部技术栈**的开发风格。
3. **bug 修复情况**：GitHub 提交信息中无独立"fix:"提交（0 个显式修复类提交）。**修复实况只能从仓库内 `项目协作日志.md` 快照间接看到**：统计（2026-09-04~09-07 条目为主）——"修复"出现 112 次、"回归"111 次、"失败"118 次、"重建"43 次；按模块提及：Module4 69 + 模块四 31、叙界 53、Master/总资料库 ~99、翻译 79、网关 10、写作 19、游戏模式 4。近期 bug/修复活动集中在 **Module4 UI/交互、总资料库(Master)、翻译、设置/网关**；Game 模式本体近期改动少（与"Game 是相对稳定内核"的假设一致，但历史样本仅 5 天，不能过度解读）。
4. **耦合观察**：两次成批提交都同时改 app/、denova-src/、tools/、docs/——仓库层面模块间强耦合是**发布操作的特征**，无法据此判定代码级模块耦合；代码级耦合判断仍以 SYSTEM_MAP §H（同一 lore/工程文件被 Go+Python 双写、共享 .denova 工作区等）为准。
5. **对"资产保留 vs 重写"的影响**：由于没有真实演化史，"该模块高频变更/反复修同一类 bug"这类**否定性历史证据全部缺失**——因此本报告**无法为路线 B 增加"Game 内核经过演化考验"的支持论据**，同时也无法以历史缺陷否定它。资产价值判断仍以快照质量为准（见 SYSTEM_MAP §7）。这是一个对 B **既不支持也不否定、但显著降低证据强度**的结论。

---

## 8. 汇总：B 验证结论与建议

| 验证点 | 结论 |
|---|---|
| 状态协议原语可统一（叙界↔Game） | ✅ 支持（replace/delta 同构） |
| Module4 可被 Game 现有预设承载 | ❌ 证伪（时钟/日程/结算/事件对象是内核级能力，非预设） |
| 单运行时统一三套（B 强版本） | ❌ 本轮不支持（需等量级内核扩展，"以 Game 为内核"名存实亡） |
| 事实/认知/关系模型（Module4 facts/knowledge） | 是宝贵领域资产，**应上浮到统一 Canon/事实层（A+）**，而非塞进 actor.state |
| 双运行时族 + 统一数据/事实层 | ✅ 与映射结果自洽（回合制 Game+叙界 / 时间驱动 Module4） |
| 历史维度 | 证据不足（8 commits/5 天），无法为 B 增加演化支持，也不否定 |
| 存档迁移 | 三套格式互异、叙界无版本号、Game 重放式——迁移成本非零，需 002B 量化 |

**对路线建议（TASK-002B 应做的对比）**：
- 将路线收敛为两套可比方案：**A+（保留三运行时 + Master 资料层 + 新建统一 Canon/事实层，逐步收敛重复桥与同步线）** vs **B'（参考 Game 设计重建一个可承载回合制+时间驱动的统一内核 + Canon 层）**；
- 判据：能否在不推翻 Module4 时间驱动范式与叙界 transcript 资产的前提下达成"真源唯一+事实可采纳"；若 A+ 能达成且成本更低，B/B' 无必要；
- C 路线的对比基础（重写成本 vs 长期收益）需要 Codex 的工程估算，建议在 002B 由 Codex 或豆包补量。

**后续还需验证（本任务因只读约束未做）**：运行态实测（embedded 首轮模型路径、`NarraverseSharedAI.refresh` 调用方、Module4↔叙界是否真无互通、Go/Python 同写锁冲突现状）、存档数据规模分布、`submit_interactive_turn` 变体对叙界 [STATE] 覆盖式写回的端到端表达试验（建议以 1 个真实冒险做映射试跑）。

---

## 9. 待确认 / 局限

- 行号基于 b37acfc；Narraverse2.0 main 若前进会漂移。
- `.denova/` 预设 JSON（actor-state 模块样例）与生成物（local_library.js）不在克隆内，字段全集以内置默认模板+测试内联样例为准。
- 运行态事实（存档规模、双写锁、嵌入态模型路径）未验证——需运行环境或 TASK-002B。
- 历史分析仅覆盖 GitHub 8 commits；真实演化史需用户授权的本地历史对比任务。
