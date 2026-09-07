# SYSTEM MAP：Narraverse2.0 全系统地图

> TASK-001 产出 · 只读调查 · 2026-09-07
> 依据 AGENTS.md 规则："先查清楚，再改"。本文不实施任何重构，仅提供基于当前真实代码的证据地图。

---

## 0. 调查事实来源（Provenance）

| 项 | 值 |
|---|---|
| 调查仓库 | GitHub 私有仓库 `llys-p/Narraverse2.0` |
| 分支 | `main` |
| commit SHA | `b37acfc76ebddd02760ab1794c3dce6e92485dbc`（"feat: 完善总资料库管理与本地入口"） |
| 本地调查路径 | `C:/Users/11/WorkBuddy/2026-09-07-14-00-01/Narraverse2.0-remote`（浅克隆，depth 1） |
| 事实来源声明 | **本次调查未使用 `E:\Narraverse`、`D:\Narraverse2.0`、`E:\Workspace` 等任何旧版本目录作为当前架构事实来源。** 全部结论来自上述克隆内的代码与文档；生成物（`.gitignore` 排除的 `app/local_library.js`、`denova-src/web/dist`、`denova-src/output`、`.denova/`、`config.toml` 等）在克隆内不存在时已明确标注"生成/运行时才有"。 |

文中路径均相对仓库根（如 `app/app.js:1872` = 克隆内 `app/app.js` 第 1872 行）。

---

## 1. 当前系统总览

Narraverse2.0 是 **Denova 托管**的创作平台，四用户模块 + 三层支持设施：

| 层 | 组成 | 代码位置（证据） |
|---|---|---|
| 用户模块 1 | Writing 长篇写作 | Denova：`denova-src/web/src/features/chapters`（仅 diff 视图）+ IDE/Agent（`denova-src/internal/agent`、`internal/prompts/system.go`）；叙界内另有"小说化"单向导出（`app/app.js` buildNovelChapters 等，见 D2） |
| 用户模块 2 | Game 结构化互动（本文"游戏模式"） | Denova：`denova-src/internal/interactive`（Go）+ `denova-src/web/src/features/interactive`（React）+ 预设 Skills（`denova-src/skills/story-director-config` 等） |
| 用户模块 3 | Narraverse 叙界文字冒险 | `app/` 原生 JS SPA：`app.js`（主）、`ai-client.js`（嵌入态 AI 客户端）、`game_engine.js`（离线小说线）、`bridge.js`（宿主协议）、`index.html`（加载顺序 L1266-1295） |
| 用户模块 4 | Open Sandbox 开放沙盒（Module4） | `app/module4/`（core/ai/state/ui 四层，23 文件，约 275KB） |
| 宿主/运行时 | Denova Go 后端 + React 前端 | `denova-src/internal/*`、`denova-src/web/src`；正式 exe 由 `tools/setup_narraverse2.ps1`→`scripts/build.sh` 产出到 `denova-src/output/denova.exe`（仓库内无 output/） |
| 桥接设施 | Python 8097 数据桥 + 翻译 + Pixiv 8098 | `tools/denova_bridge.py`、`tools/denova_full_export.py`、`tools/pixiv_bridge.py` |
| 资料设施 | 源知识库 + 总资料库(Master Library) | `knowledge-base/`（源文件）；Go 独立工作区 `projects/narraverse-master-library`（`denova-src/internal/book/master_library.go`） |

模块模式路由：`denova-src/web/src/components/workbench/ModeRouter.tsx`（`?mode=writing|game|narraverse|library…`）；叙界作为第三顶层模式 iframe 嵌入（`web/src/features/narraverse/NarraverseWorkspace.tsx`）。

**一句话结构判断**：数据/资料资产与"网关"已经向 Go 服务收口（Master Library、预设库、模型网关），但**三套互动玩法（Game/Narraverse/Module4）的应用层状态机与规则内核仍是三套独立实现**；叙界与 Module4 作为同源 iframe 活在浏览器侧，经 postMessage v1 + 8097 文件桥与 Go 世界弱耦合。

---

## 2. 八区域调查结果（A–H，均附证据）

### A. 资料导入与整理

**A1 角色卡导入（叙界 app 侧，真实入口均在 `app/app.js`）**
- 入口链：文件选择 `uploadCardToPending`(L6258) / `parseFileToCards(file,cb)`(L6464) → PNG 走 `extractCharaFromPNG`(L6391，手写 PNG chunk 解析 tEXt key="chara")→`parseCharaPayload`(L6416)；文本走 `parseCharacterCardsFromFile`(L6426)。字段归一 `normalizeCardObject`(L6343) 兼容 chub/tavern spec_v2 与 V3 字段；`autoFillCardFields`(L6300) 从 notes 拆外貌/性格/关系。
- 落点：`state.uploadedLoadCards`（仅备选）或 `adv.characterCards`（`importCardToAdventure` L6506，按 name 去重）→ 整 state 持久化（IndexedDB + localStorage）。**不直接进总资料库**。
- 卡库聚合：`getCardLibrary`(L6523) + `gatherLoadItems`(L6621)：本冒险卡 + uploadedLoadCards + `window.LOCAL_LIBRARY.cards`（生成物）+ Denova lore（`loadDenovaBooks` L6582 → 8097 `GET /api/denova/projects` + `/api/denova/lore?book=`）。

**A2 设定书/世界书**
- 叙界无独立"文件导入设定书"函数；设定书以 `{title,content}` 挂载/深拷贝进 `adv.backgroundBooks`（L3490/L5535）。
- chub.ai：**克隆内无任何现役下载代码**（仅 `代码指南.md` 历史记录曾下载 18 本，knowledge-base 留有 `chub-*.json`）。
- `knowledge-base/`：`世界观设定/` 50 json+1 txt、`角色卡/` 39 json+23 png、另有 草稿/、修复前备份/、根 `设定集元数据.json`、`说明文档.md`。

**A3 清洗/结构化（无内容级去重）**
- `tools/gen_local_library_v2.py`：截断（BOOK_CAP 10000/NOTES_CAP 3000/MES_CAP 1500）+ `source_ref{version,kind,relative_path,source_id(sha256),source_hash}` 溯源 + 输出 `window.LOCAL_LIBRARY` → `app/local_library.js`（gitignore，克隆内不存在）。
- `tools/rebuild_meta_categories.py`：仅关键词正则补 `category/summary/tags/nsfw` 到 `设定集元数据.json`，**无重复内容检测**。
- `tools/kb_sync.py`：简介三处同步（元数据/说明文档/Obsidian md）。

**A4 翻译链路（8097 服务 + 全量导出）**
- `tools/denova_full_export.py` `ExportJobManager`(L861)：工作目录 `.denova/narraverse-export-jobs|cache|translation-jobs`；`_run`(L1779) 源材料+剧本原子提交到 `.denova/projects/<name>/.narraverse/`（含 `import-manifest.json`、`.narraverse/backups/<ts>` 回滚）；`translate_lore_fields`(L891) 用 `LocalTranslator`(L284，HY-MT via Ollama) 按术语表 `tools/translation_glossary.json` 分块翻译 lore 的 name/brief_description/content；既有手工改动经 `_merge_lore`(L1727) hash 比对，冲突写 `.narraverse/conflicts/<stamp>/<id>.json`(L1832)。**注意：这条导出线译文直接覆盖 lore 活动内容，与 Master Library 的"原文/译文分版本存"语义不同**。
- 8097 路由（`tools/denova_bridge.py` do_GET L600 / do_POST L731）：`/api/denova/health`、`/translator/status`、`/materials`、`/translator/jobs`、`/sync`、`/projects`、`/lore`、`/director-presets`；POST `/export/jobs`、`/translator/install|jobs|translate`、`/sync`、`/import-kb`（叙界整理回写知识库，app.js L8059 调用）。

**A5 去重/版本/来源**
- Go 总资料库按 **sha256 内容去重**：`upsertSourceUnlocked`(`book/master_library.go:903`) 同 digest 复用 source/revision（revision id=`sha256:<digest>`），新版本记 `parent_revision`，原件归档 `.narraverse/source/originals/<sourceID>/<digest>/<file>`；Ingest 幂等（idempotencyKey=hash(adventureKey+sourceID+revision)，L314-323）。
- 叙界侧无内容级去重、无 updated_at（`local_library.js` 生成时间戳历史上硬编码过 "2026-08-10"，需运行态复核）。

### B. 总资料库（Master Library）——Go 侧已成型，是最完整的资料数据架构

- **工作区**：独立 `projects/narraverse-master-library`（`ResolveMasterLibraryWorkspace` `book/master_library.go:264`），与用户冒险工作区平级。
- **存储布局**：manifest `.narraverse/master-library-manifest.json`（schema_version=3，`sources/items/translations/imports/instances` + `legacy_*`，L38-52）；条目体 `.narraverse/master/items/<id>.json`；翻译版本 `.narraverse/translations/<id>.json`；历史 `.narraverse/master/history/<itemID>/<rev>.json`；事务 `.narraverse/transactions/<id>.json`；删除墓碑 `.narraverse/master/tombstones/<id>.json`。
- **核心类型**：`MasterItem`(L116) 按 `RecordKind=character_template|lorebook_template` + 嵌套条目稳定 id；`MasterField`(L135) 关键在 **`SourceText/SourceSHA256`（原文）与 `ActiveText/ActiveKind/ActiveTranslationVersionID`（当前活动版=用户界面所见，通常是中文译文）分离**；`MasterTranslationVersion`(L146) 原文/译文分存、带 model/confirmed/revision。
- **关键方法**：`Ingest`(L292)、`ApplyTranslation`(L366，CAS 校验 source sha256+revision+base version)、`ArchiveMasterAsset`(L515，停用→legacy+墓碑，源文件与既有冒险实例保留)、`GetAssetAvatar`(L605)、`UpdateMasterAssetDescription`(L649)、`PrepareAssetInstantiation`(L767)、`MarkImportInstantiated`(L720)、`RemoveMasterAsset`(`master_runtime.go:133`，先取消 8097 翻译队列)；查询 `book/master_library_query.go`（ListAssets/GetAsset/GetAssetPipeline）；Service 层 `book/material_import.go`（`ImportMaterial` L176、`InstantiateMasterAsset` L203、`FinalizeMasterImport` L418、`buildMasterRuntimeOperations` L741）。
- **API**（`denova-src/internal/api/routes.go`）：`/api/library/*` 全组（assets 列表/详情/avatar/pipeline/translations/usages/**instances(POST 加入冒险)**/description/fields/entries/character-entries/runtime/stop/DELETE/sync-adventure/proposals/agent）；`/api/workspace/import-material*`、`/api/master/finalize`。
- **前端**：`web/src/features/library`（LibraryView L110、五页签详情 LibraryDetail L337、LorebookReader L1064、CharacterReader L895、MasterImportDialog L9）；api-client `master-library.ts` / `master-library-runtime.ts`。
- **总库 ↔ 冒险实例化**：`POST /library/assets/:id/instances` → `InstantiateMasterAsset` → `PrepareAssetInstantiation`（幂等去重）→ `FinalizeMasterImport`：读总库活动工作版本（ActiveText=译文）经 `buildMasterRuntimeOperations` 生成 LoreOperation **写入当前冒险 LoreStore（复制投影）**，原件与 source 不动；`ReprojectMasterAsset`(`book/master_sync.go:121`) 供 `/sync-adventure` 按新活动修订重投影既有实例。
- **能力判断**：原件/译文/历史/CAS 冲突/来源 sha256/实例化/重投影 均已具备——这是全项目最接近"统一资料母版"的架构，建议作为后续统一世界资料层的骨架候选。

### C. 具体世界 / 冒险资料

- **叙界冒险对象**：`createAdventure`(`app/app.js:897-960`) 返回对象含 `title/theme/setting/character(内含 hp/mp/attributes/skills/items…)/conversationHistory/characterCards[]/backgroundBooks[]/mandalaCards[]/snapshots[]/branches[]/quests[]/combat/plotNodes/plotLines/novelPlan/versionLedger/syncMeta…`；素材**深拷贝**入冒险（L5530/L5535 `JSON.parse(JSON.stringify(...))`），此后与来源脱钩。
- **存档**：IndexedDB `adventureAI_db` store `kv` 键 `state`(L351-395) 为主，localStorage `adventureAI_state`(+_bak) 镜像（L666-695），loadState 回退链(L596)；永久对话档案 IndexedDB `conversationArchive:v1:<id>:chunk:N`(L402)。
- **Denova 侧（Game 世界）**：事件溯源 JSONL——`<ws>/interactive/story/story-<id>.jsonl`（首行 StoryMeta，后续 StoryEventRecord；`book/interactive/story_storage.go:13-35`），actor schema 冻结侧车 `story-schema/story-<id>-actor-state.json`；会话另有 `session/store.go` jsonl。**世界状态 = Actor State 经 state_delta 累积的 map**，未见独立 Facts/WorldState 类型。
- **关系**：叙界/Module4/Game 三套世界数据**互不相通**（Module4 独用 localStorage `narraverse:module4:state`，见 G）；叙界经 8097 把冒险 push/pull 成 Denova 工程 lore（`pushAdventureSync` app.js:811 / `pullAdventureSync`:877）——属工程级投影，非同一状态。

### D. 写作模式

- **Denova Writing**：正文工作区 `.denova/chapters/`，长设 `setting/*.md`；主 Agent 工具 `list/read/write_lore_items`（`internal/prompts/system.go:148-217`），lore=长期设定，常驻正文+64KiB 目录+渐进式召回；写作子 Agent `context-planner/writer/reviewer/fixer/final-gate/memory-patcher`（配置样例 `config/testdata/writing-subagents.toml`，父 ide）；`skills/novel-standard/SKILL.md`：初稿→委派 reviewer→修订并同步 `progress.md/character-states.md`，仅长期设定变化才 `write_lore_items`。
- **Master 变更纪律**：`skills/master-library/SKILL.md` 规定只经 `create_master_proposal` 生成候选、**UI 确认才应用**（`book/master_agent.go` proposals 目录 `.narraverse/master-agent/proposals`）。
- **叙界"写作"入口是单向小说化**：`novelExportModal`（index.html L1076-1102）；`buildNovelChapters`(app.js:7906，按卷切永久档案)、`generateNovelChapter`(L8428)、`generateNovelOutline`(L8467)、`saveNovelPlan`、`downloadNovel`——产物存 `adv.novelChapters[]`，**不改写角色/世界**。
- **Canon 边界**：叙界无 adopt/canon 机制；弱回写仅经 Denova sync（`applyDenovaSyncResult` 可把 world/loreBook/chapters 拉回冒险，app.js:846-866）。Denova 侧"稳定设定才写 lore、总库须 Proposal+确认"——**写作草稿与世界运行实验不会自动成为正式事实**，此边界目前靠流程纪律而非机制强制。

### E. Game Mode（Denova 游戏模式）——运行时内核最完整的一套

**预设/方案体系（重点，证据来自 `denova-src/internal/interactive/`）**：

| 预设 | 类型 | 落盘目录 |
|---|---|---|
| 故事导演 | `StoryDirector`（story_directors.go:41），`StoryDirectorStrategy` 含 enabled/mainline_strength/failure_policy/pacing/event_frequency/director_agent_mode/rule_state_consumption_mode/branch_planning | `novaDir/story-directors/` |
| 叙事风格 Teller | `Teller`/`TellerContextPolicy`/`TellerPromptSlot`（tellers.go:33/51/57） | `novaDir/story-tellers/` |
| 图像方案 | `imagepreset.Preset`（internal/imagepreset/library.go:30） | `novaDir/image-presets/` |
| 事件包 | `EventPackageModule`/`TellerEventCard`（director_modules.go:72） | `novaDir/story-director-modules/event-packages/` |
| TRPG 检定 | `RuleSystemModule`/`StoryDirectorTRPGSystem`（director_modules.go:87；`RuleCheck` orchestration.go:52，1d20、五档难度、四档后果、state bindings、reroll） | `.../rule-systems/` |
| 状态系统 | `ActorStateModule`/`StoryDirectorActorStateSystem`（actor_state.go:22；schema 冻结 `FreezeActorStateSchemaWithRules`） | `.../actor-states/` |

- **可组合/独立关闭**：`StoryDirectorModuleRefs`（director_modules.go:37）含 `narrative_style_id/event_package_ids/rule_system_id/actor_state_id/image_preset_id` 及各自 `*_disabled`；保存时 `ResolveStoryDirectorModules` 展开成 `ResolvedSnapshot`，模块缺失可回退最近可用展开图。Skills 约束协议：`story-director-config`/`teller-config`/`image-preset-config`（仅 config_manager write 工具可写）。
- **运行时主循环**：`Store.AppendTurnWithState`（story.go:618）把 `TurnEvent{User,Narrative,RuleResolution,TurnResult}` 编译成 StateOp/ActorStateOp（`CompileTurnStateUpdates`）→ `state_delta` 追加 JSONL；分支 `BranchMeta{Head,From,FromEvent}` 支持 rewind/切版本；Hot Choices=`TurnResult.Choices`。
- **模型调用**：不走裸 provider，而是 **internal/agent 基于 ADK**：`BuildInteractiveStory`（agent/builder.go:44）+ `buildInteractiveStoryRunner`（app/runtime_builder.go:129）→ `chatService.RunWithOptions(..., Mode:"interactive")`；回合工具 `prepare_interactive_turn`（固定 d20 检定）/`submit_interactive_turn`（replace/delta/create/archive/restore 五类 oneOf，**可承载任意状态变更协议**）。模型配置按 `AgentKindInteractiveStory` 从共享 `/api/settings` 层叠读取。
- **导演后台**：`DirectorPlan`（director_plan.go:108）分文件存 plan/agent_brief/lore_context（`.denova/interactive/stories/<id>/director/main/`）；回合后 `DecideDirectorRunAfterTurn` → 后台维护任务（app/interactive_director.go:875）。
- **存档/恢复**：jsonl 事件溯源重启重放（`snapshotFromLines` story.go:1306）。
- **前端**：StoryStage.tsx + InteractiveLayout；Actor State 面板 `StoryStateLedger` + `StateLayoutEditor`；导演台 `director-backstage/DirectorBackstage`、`director-console/DirectorConsole`（StatePanelTabs/StateSchemaCard/RuleAuditCard/EventRuntimeCard/DirectorGate）；回合 SSE `POST /api/interactive/chat`（api.ts:380）。
- **运行时能力清单（可复用内核候选）**：① 事件溯源故事内核（jsonl/branch/rewind/compaction/token ledger）；② Actor State 系统（schema 冻结、typed ops、字段迁移、trait 抽取、archive/restore）；③ d20 检定引擎（RuleCheck/TurnCheckRequest→RuleResolution）；④ 导演规划器（DirectorPlan/后台维护）；⑤ **预设加载器体系（版本化 JSON Library + ensureBuiltins + 修订冲突 + resolved snapshot + ownership）——天然的"统一运行时配置层"**；⑥ Lore 拼装（resident+计划引用+命中集）；⑦ 模型网关（ADK Runner+SSE+回合协议中间件+原子落盘）。

### F. 叙界 Narraverse 文字冒险（app/）

- **回合主链**：`sendMessage`(app.js:5309) → `updateSystemPrompt`(L1826) → 压缩 `manageContext`(L2276) → `callLLM`(L1872) → 流式 `readStream`(L1935) → `parseGameResponse`(L1973) → `applyParsedResult`(L2127，hp/mp/物品/属性/技能/EXP) → `applyCharacterUpdates`(L4159)/`applyPlotUpdates`(L4599) → `saveState`(L5401) → 渲染。选择 `handleChoice`(L5437)。
- **协议标签全集（仅 8 种）**：`[NARRATIVE][STATE][CHANGES][CHOICES][QUESTS][COMBAT][CHARACTERS][PLOT]`（L1975）；CHANGES 子命令 HP±/MP±/ITEM±/MOOD/SKILL_NEW/SKILL_UP/ATTR_UP/EXP+/LEVEL_UP/SKILLPT/ATTRPT/AFF/RELATION；QUESTS/COMBAT/PLOT 各含 NEW/UPDATE/…；CHARACTERS=`角色名|身份=…`。模板含"22 条规则"(adventure, L1796-1818) / 酒馆 8 条（`buildTavernSystemPrompt` L1527）。
- **提示词构建链**：`buildSystemPrompt`(L1639)→公共块 `buildCharacterCardsBlock`(L1094)/`buildActiveSceneBlock`(L1118)/`buildLorebookBlock`(L1472，World Info 按关键词注入：解析 `parseBookEntries` L1426，扫最近 14 条+loreBudgetPct)/`buildPlotLineBlock`(L1411)/`buildPostHistoryTail`(L1180)；宏 `resolvePromptMacros`(L1226) `{{char/location/hp/mood/date/time/random}}`。
- **AI 客户端双轨**：嵌入态 `ai-client.js`（`NarraverseSharedAI.refresh/test/chat` → 同源 `/api/model/status|chat`，密钥留服务端）；standalone 态 `callLLM` 直连 OpenAI 兼容 `fetch(endpoint/chat/completions)`（密钥在浏览器 `state.apiConfig`，默认 model `deepseek-chat`）。**两套路径与两处配置并存（重复候选，见 H）**。
- **生命周期**：三步创建向导（`generateStoryProposals` L5998 要求 AI 产出 3 套方案 JSON → 确认开局）；V3 角色卡字段（normalizeCardObject L6343）；快照 `createSnapshot`(L4376)；自动存档 `maybeAutoSave`(L4246)。
- **桥接（与 Denova）**：`bridge.js` postMessage v1：出站 `ready/switch-mode/module4-closed`（宿主 `NarraverseWorkspace.tsx:24`），入站 `theme-changed/locale-changed/visibility-changed/module4-open`(L28)；iframe 同源 `/narraverse/index.html?embedded=denova`(L18)。同步：`pushAdventureSync`(L811，防抖 1.2s)/`pullAdventureSync`(L877)；全量导出 `exportToDenova`(L8296，schema_v4 分片)↔ 8097 `denova_bridge.py` + `denova_full_export.py ExportJobManager`。
- **离线小说线**：`window.GameEngine`（game_engine.js:6），入口 `open()`(L708)；事件图指令 `@next/@stage/end:/@random:`、检定 `rollCheck`(L214, d20+attrMod≥DC)、战斗 `startCombat`(L478)、结局 `showEnding`(L393)；AI 增强 `aiCall`(L102) **复用全局 callLLM**，`<action:xxx>` 驱动火柴人，失败自动回退，结果按 `pack+事件id` 缓存 localStorage。与在线线共享 callLLM/apiConfig/Stickman；**不写 IndexedDB、无存档**。
- **运行时能力清单（供对照）**：统一 LLM 入口+SSE；8 标签结构化协议+命令式 diff 状态机；双模板（战斗/酒馆）；角色卡 V2/V3 导入（JSON+PNG）；World Info 按需注入+预算；宏替换；多状态右栏（属性/技能/背包/任务/好感/战斗/剧情节点/曼陀罗）；三步 AI 向导；三层存档+快照；双模式；iframe v1 协议；工程同步/全量导出；离线事件图引擎+可选 AI 增强；d20 检定指令注入。

### G. Module4（开放沙盒，app/module4/）

- **生命周期**：`window.Module4`（module4.js:6，版本串 `v1.5-context-interaction`）；宿主 bridge `module4-open/closed` 开合（bridge.js:52-56、module4.js:47）；世界列表/新建 UI 在 `ui/shell.js`+`world-create.js`（rulesVersion:'v1.5'）。
- **存储**：仅 localStorage：`narraverse:module4:state` + `:state:recovery`（store.js:8-10）；world schema v1.5(schemaVersion=2) 见 store.js:70-98（id/rulesVersion/revision/player{energy,maxEnergy}/clock{day,period,tick}/npcs/locations/sourceBindings/scheduleRewrites/sceneObjects/events/npcRelations/knowledge/facts/dailyLogs/actionLogs/narrativeEntries/currentDay…）；`revision` 乐观锁防并发（L220）；解析失败进 recovery 键（L27-47）。
- **内核（core/，确定性、纯函数）**：`clock.js`（3 时段×3 tick，能量表 L11-20；`advanceTicks` 撞 evening 末返回 requiresSettlement）；`schedule.js`（NPC 日程 slot + `scheduleRewrites` 按 `{npcId,day,period}` 覆盖，只允许改未来 slot 且须带溯源）；`location.js`；`encounter.js`（`checkNatural`=NPC 当段日程地点==玩家所在地；`seekNpc`）；`facts.js`（事实 schema L71-87：visibility public/restricted/private + knownTo + epistemicStatus + sourceActionId——**已为"谁知晓"预留结构**）；`events.js`（事件 status candidate/active/resolved/expired、触发器 action/enter_location/judgment/time/deadline、活跃上限 3、advance 效果白名单、每步转 fact）；`action.js`（`Action.execute` L547-761 单一流水线：解析→守卫→场景对象→移动→NPC→人际判定→judgment→日程改写→钟推进→相遇→事件→facts→跨日 settlement→日志；`parseFreeText` L93-139 **纯正则规则**把"我去酒吧找林雨"拆 move+seek，未命中→custom 按闲聊）；`judgment.js`（**非 d20**：确定性计分 base60−难度+关系/心情/日程/精力/属性，≥70 success/≥55 costly/≥40 failure/else critical_failure，L145-150；custom 命中风险词 强行|潜入|偷|威胁…才触发）；`settlement.js`（仅 evening 结算：当日可见 facts+未互动 NPC 摘要→dailyLog→day+1）。
- **AI 层（ai/）**：`preview.js`（Daily Preview：`run` L242-290 经 `root.callLLM`，失败重试≤2 次后**本地 fallbackPreview**；`startDay` 按 worldId:day 去重）；`interaction.js`（自由叙事回合：先 `Action.execute` 得规则结果再 callLLM 续写；`parseResponse` 强制 JSON、narrative≤1600；`safeKnowledge` 只允许引用上下文事实 id；**模型失败不落库**且提示检查 Denova Settings）；`context.js`（forPlayer/forNpc/forPreview 三段，NPC 只见自己认知、隐藏 key 正则过滤）；`prompts.js`（严格 JSON，与叙界 8 标签协议**无关**）。
- **角色来源与绑定**：候选 = `root.getCardLibrary()`（遍历叙界 `state.adventures[].characterCards`）+ `root.LOCAL_LIBRARY.cards`（生成物）；`addNpcFromSource` → `world.sourceBindings[sourceRef]={sourceRef,sourceVersion,snapshot(9 字段),?migrated}`（world.js:104-113）；Runtime NPC id 由 worldId+sourceRef 派生，**永不回写原卡**。
- **能力分层**：运行时内核=schedule/move-action/Encounter/Judgment/Facts/Settlement/事件推进（确定性）；AI 服务层=Daily Preview（含本地 fallback）、interaction（紧耦合规则结果）；UI=play-view/shell/world-list 等；调试=`module4-debug-panel`"试玩检查"（play-view.js:500-520，不参与规则）。

### H. 技术边界（重复/割裂候选集中在 H5-H6）

**启动与进程**：`Launch-Narraverse.cmd`→`tools/setup_narraverse2.ps1`（kill 旧 exe→gen_local_library_v2.py→sync 到 `denova-src/web/public/narraverse/`→pnpm install→`bash scripts/build.sh` 产出 output/denova.exe）→`start_narraverse.ps1`（起 8097 bridge→可选 8098 pixiv→启动 exe→开 `127.0.0.1:<port>/?mode=narraverse`）。端口：8080 Go 后端（config/settings.go DefaultSettings）、8097 Python、8098 Pixiv、5173 Vite、5174 standalone http.server、Ollama 11434 外部依赖。
**模型配置**：Go `config.LoadLayeredWithStartupConfig`（config.toml+workspace 配置→Settings.Merge→`ResolveAgentModel(cfg, agentKind)` model_profiles.go:70）；`model_gateway.go` 四模块归一映射：narraverse/module3、game/interactive/module2、module4/sandbox 全 → `interactive_story`（interactive_story），writing→ide（normalizeModelModule L269-282）；端点 `/api/model/status|test|chat`。

**关键判定（正常分层 vs 割裂）**：

| 项 | 判定 | 证据 |
|---|---|---|
| Go/React/Python 分工 | 正常分层 | 8097 桥只做文件/翻译/工程，不托管模型 |
| localStorage/IndexedDB 双镜像 | 刻意降级策略 | app.js saveState 注释 |
| 同源 iframe + postMessage v1 | 正常封装 | bridge.js / NarraverseWorkspace |
| config.toml 全局+工作区叠层 | 正常分层 | config/settings.go |
| 同一角色/条目 3–5 库表示 | **割裂候选** | KB 源 ↔ 生成 local_library ↔ `.denova/lore/items.json` ↔ master-library ↔ 运行时（冒险深拷贝 / Module4 world.npcs） |
| 两套模型调用路径+两处默认模型/密钥 | **重复候选** | embedded `/api/model/chat` vs standalone 直连（app.js:1876/1887） |
| 8097/8080 lore 双读入口 + 4 条同步线 | **重复/写穿候选** | `/api/lore/*` vs `/api/denova/lore`；sync/export/lore_sync.js/kb_sync |
| 三套玩法状态机无共用 schema | **割裂候选** | Actor State vs 8 标签 diff vs Module4 world schema |
| 双上下文压缩 | **割裂候选** | 浏览器 conversationArchive vs Go compaction |
| 两套检定/日程/日结 | **割裂候选** | Module4 judgment.js(计分) vs Game rule-system(d20) |

---

## 3. 数据流总图（文字版）

```
[knowledge-base 源资料(角色卡/世界观)]
   │  gen_local_library_v2.py（截断+source_ref）
   ▼
[app/local_library.js (生成快照, gitignore)] ──→ 叙界素材选择 / Module4 角色候选
   │
   ├── 叙界冒险(浏览器) createAdventure: 深拷贝挂载 characterCards/backgroundBooks
   │         │  pushAdventureSync / exportToDenova(8097)
   │         ▼
   │  [Denova 工程 .denova/projects/<name>/: lore/items.json + .narraverse/ 源/清单/backups/conflicts]
   │         │  (HY-MT 翻译队列→译文覆盖 lore；冲突落 .narraverse/conflicts)
   │         ▼
   │  import-material / import-material/master/translation
   │         ▼
[Master Library 工作区 projects/narraverse-master-library: manifest + master/items + translations + originals]
   │         │  POST /api/library/assets/:id/instances（复制 ActiveText 活动版→冒险 LoreStore，幂等）
   │         ▼
   │  冒险/写作的 lore 上下文（.denova/lore/items.json 同工作区共享：writing/game 常驻、interactive 计划引用）
   │
[Game Mode 运行时: interactive story jsonl 事件溯源 + Actor State + 预设(story-directors/tellers/module 库)]
[Module4 运行时: localStorage narraverse:module4:state（world 自含 npcs/facts/clock…，与上二者隔离）]
```

---

## 4. 三套互动系统逐项对照表

| 维度 | E. Game Mode（Denova 结构化游戏） | F. 叙界 Narraverse 文字冒险 | G. Module4 开放沙盒 |
|---|---|---|---|
| 代码位置 | `denova-src/internal/interactive` + `web/src/features/interactive` | `app/app.js` + `game_engine.js`（浏览器） | `app/module4/`（浏览器） |
| 状态模型 | Actor State（schema 冻结+typed ops+state_delta 累积），jsonl 事件溯源 | 冒险 JS 对象 + `[STATE]/[CHANGES]` 命令式 diff；存档 IndexedDB/localStorage | world 自含 schema v1.5（npcs/locations/facts/knowledge/clock/events…）localStorage 整存 |
| 玩家回合 | 玩家输入→ADK Agent（prepare/submit_interactive_turn）→TurnEvent 原子落盘 | sendMessage→callLLM→8 标签协议解析→applyParsedResult | 自由输入→Action.execute 纯规则流水线→（结果后再 AI 续写 narrative） |
| 检定 | TRPG d20 RuleCheck（1d20+modifier+五档难度+四档后果+reroll+state binding） | 无内置检定器（战斗按钮注入"掷骰 d20"文本给模型）；离线线 rollCheck d20 | judgment.js 确定性计分（非骰子；base60−难度+关系/心情/精力/属性） |
| 导演/推进 | StoryDirector：DirectorPlan、后台维护、mainline/pacing/event_frequency、director_update | 无导演；靠 22 条规则+系统提示词与 PLOT/事件节点 | 无导演；事件引擎（events.js 候选/升级/过期）+ Daily Preview（AI） |
| NPC/日程 | 预设/资料库角色入戏；Actor 抽取规则 | 角色卡 V3+NPC 在场群像+话痨度；无日程 | NPC schedule 3 时段+scheduleRewrites；关系/心情/目标/已知信息 |
| 事实系统 | 无独立 Facts 类型（历史=jsonl 已提交 turn；当前=Actor State；稳定=资料库） | 无事实系统（剧情节点/好感等字段近似） | facts.js（visibility/knownTo/epistemicStatus，最接近"世界事实+认知"模型） |
| 预设/可组合 | **有**：叙事风格/事件包/TRPG/状态系统/图像方案 + 各自 disabled | 无预设系统（仅场景预设 presets.js 管 UI 主题与右栏 Tab） | 无预设（rulesVersion 版本化规则固定） |
| 模型调用 | ADK Runner（服务端，共享 /api/settings） | embedded 走 `/api/model/chat`；standalone 直连（双轨） | 复用叙界 `root.callLLM`（embedded 同样走网关 module4/sandbox） |
| 存档 | jsonl 事件溯源（重启重放） | IndexedDB 主+localStorage 镜像+对话归档+快照 | localStorage 单键整存+recovery+revision 锁 |
| 世界推进 | 导演/分支/重开线 | 无"世界自走"（完全回合制） | 事件链跨天自走、Settlement 日结、可跳过时间 |
| UI | StoryStage/导演台/状态账本 | 冒险台/右栏状态/Tab 化 | play-view 场景书页/世界档案抽屉/试玩检查 |

---

## 5. 数据模型重复清单（同一信息多处表示）

1. **角色卡/设定条目**：`knowledge-base/`源 JSON ↔ 生成物 `local_library.js` ↔ Denova 工程 `.denova/lore/items.json` ↔ Master Library 资产（`.narraverse/master/items/`）↔ 运行时冒险深拷贝（app.js:5530）↔ Module4 `world.npcs`（sourceBindings.snapshot 快照）。至少 4–6 处表示，同步靠单向/多向脚本，无统一引用。
2. **世界状态**：Game = Actor State map；叙界 = 冒险对象字段；Module4 = world 字段。三份 schema 不兼容。
3. **事实/日志**：Game = jsonl turn（隐含）；叙界 = conversationArchive（文本归档）；Module4 = facts/dailyLogs/actionLogs。仅 Module4 有显式事实结构。
4. **"加入冒险的同一角色"**：叙界按 name 字符串去重；Master 实例化按 import 幂等去重；Module4 addNpc 按 sourceRef 幂等。三种去重键。
5. **模型配置默认值**：Go `settings.go:116` DeepSeek v4-pro 默认；app.js:1907 `deepseek-chat` 兜底；qwen-image-3.0-pro；翻译 hy-mt（Ollama）。四处默认，未见同一常量源。

## 6. 状态 / API / Agent / 模型调用重复清单

- **重复 API**：lore 读取双入口（Go `/api/lore/items` vs 8097 `/api/denova/projects|lore`，读同一 `items.json`）；"素材→lore/世界观文件"两入口（Go `/api/library/import-material`/`/api/workspace/import-material` vs 8097 `import-kb`/`export/sync`）；Go 与 Python 直接读写同一 `.denova/projects` 文件树（无锁协调证据）。
- **重复模型调用路径**：嵌入态走服务端网关（ai-client.js→`/api/model/chat`）与 standalone 直连 OpenAI 兼容端（callLLM）并存；服务端 Settings vs 浏览器 `state.apiConfig` 两处密钥/默认值。
- **重复 Agent 概念**：服务端 ADK Agent（interactive_story/ide 等 10 类，agent_registry.go）vs 浏览器"AI 写手/管家"（叙界直接 prompt 无 agent 框架）；Module4 单模型多角色视角（无多 Agent）。
- **重复上下文压缩**：Go IDE compaction / Go interactive compaction / 浏览器 conversationArchive 三套。
- **重复规则引擎**：Module4 计分检定+日程+日结 vs Game d20+director+event package——两套命名与 schema（见 §4）。
- **模型网关已归一**：writing/game/narraverse/module4 全部映射到 agent kind（interactive_story/ide），是现有最有力的"统一运行时"收敛证据。

## 7. 可复用资产清单（代码级别，值得保留）

1. **Master Library 数据架构**（`denova-src/internal/book/master_library*.go`）：原件/活动版分离、sha256 去重、翻译版本树、CAS、历史、tombstone、实例化/重投影——统一资料母版的现成骨架。
2. **Game Mode 预设加载器体系**（story-directors/tellers/actor-states/rule-systems/event-packages/image-presets 版本化 JSON 库 + `StoryDirectorModuleRefs` 组合 + `*_disabled` + resolved snapshot）：天然可作为统一运行时"配置层"。
3. **Game Mode 事件溯源内核**（interactive story jsonl、branch/rewind/compaction、Actor State typed ops、`submit_interactive_turn` 五类变更协议）。
4. **模型网关 + ADK Runner**（/api/model/*、`internal/agent`、interactiveTurnProtocolMiddleware）——四模块已共用。
5. **Module4 确定性内核**（schedule/location/encounter/action/facts/events/settlement 纯函数 + 规则化自由输入解析 + Daily Preview 本地 fallback）——可作为"世界模拟预设"直接迁移。
6. **叙界 8 标签协议与提示词管线**（成熟、验证充分），及本地"World Info 关键词注入+预算"模式。
7. **角色卡 V2/V3 导入与 PNG tEXt 解析**（叙界前端实现，语义与格式可在任意模块复用）。
8. **知识库生成管线**（gen/rebuild/kb_sync）与 translation 术语表/缓存/原子写模式。
9. 测试资产：Module4 16 项 tools/test_module4_*.js、前端 Vitest 950 项、Go 定向测试——重构的安全网。

## 8. 建议废弃 / 只迁数据清单（初步，仅列出不实施）

- **废弃候选（功能重复或被取代）**：8097 与 8080 的 lore/工程写入口二选一收敛；叙界 standalone 直连路径在嵌入态为主时可降级为调试用；`kb_sync` 与 `denova_lore_sync.js` 等多条同步线可向 Master/sync-adventure 收敛（需逐一确认调用方）。
- **只迁数据/规则、代码不值得保留的候选**：Module4 若并入统一内核，其 core/* 逻辑应以"预设+规则数据"迁移而非保留浏览器全局单例；叙界 22 条规则/8 标签协议若并入，应迁为"提示词预设"。
- **明确不废弃**：Master Library、Game 预设体系、事件溯源内核、叙界角色卡解析、知识库源资料。

## 9. 核心问题回答（10 问，初步证据结论）

1. **三套能否共享一个运行时核心？** 基本可以，且收敛证据强：模型网关已把 game/narraverse/module4 全部映射到 `interactive_story` Agent kind；Game Mode 已具备事件溯源+状态 schema+预设组合能力；差异主要在应用层协议与状态 schema（8 标签 vs submit_interactive_turn vs 纯规则）。
2. **Game 预设能覆盖多少另外两套？** 覆盖度评估：状态系统（Actor State）可表达叙界属性/好感/任务 与 Module4 npc/facts 的大部分；TRPG d20 可承载 Module4 judgment（规则从计分换成 d20 属预设差异）；事件包可承载 Module4 事件引擎与叙界剧情节点；**未覆盖**：Module4 的时钟/日程/跨日结算（需新增"世界时钟"类预设或内核组件）、叙界 World Info 关键词注入（可做成 lore 上下文预设）、游戏引擎式离线确定性数值（保留给 offline 场景）。
3. **哪些应成为统一运行时内核？** 事件溯源故事存储、Actor/世界状态 schema（含字段迁移）、回合变更协议（submit 类 oneOf）、确定性规则引擎（检定+时间+事实落库）、导演/推进调度、模型网关、预设加载器。
4. **哪些只是预设开关？** 叙事风格、检定规则集（d20/计分）、事件包内容、状态系统模板、图像方案、玩法文案/提示词模板、剧情类型参数。
5. **哪些只是不同 UI？** 叙界右栏状态台、Game 导演台/状态账本、Module4 场景书页/世界档案——底层数据若统一，三者只是视图/预设差异。
6. **哪些现有代码值得直接复用？** 见 §7 清单。
7. **哪些只能迁移数据/规则？** Module4 规则内核、叙界 22 条规则提示词、现有 world/冒险数据（若并入统一 schema，需迁移+校验）。
8. **哪些应该废弃？** 见 §8（重复桥入口、standalone 直连路径降级、冗余同步线）。
9. **从零设计视角，最严重三个结构性问题？** ① 同一角色/条目 3–5 库并存且"复制投影"无订阅——事实真源不唯一；② 三套玩法各自状态机与规则引擎，无共用 actor/facts schema——规则能力重复实现、无法组合；③ 双轨架构（Go 服务 vs 8097 文件桥+浏览器直连）造成双模型配置/双 lore 入口/4 条同步线——状态一致性与维护成本高。
10. **A/B/C 初步判断**：见下节。

## 10. A/B/C 路线初步判断（仅判断，不实施）

**初步更支持 B（保留资产，重构核心）**，理由：
- A（渐进整合）只能缓解、不能根治三大结构性问题（复制投影、三状态机、双轨）——补丁成本会持续累积；
- C（近乎重写）会丢弃高价值资产：Master Library 数据架构、Game 预设体系、事件溯源内核、叙界提示词管线与测试网——这些正是统一内核的现成骨架；
- B 的现实路径：**以 Master Library 为统一资料层 + Game Mode 事件溯源/预设/Actor State 为统一运行时内核 + 模型网关为统一调用层**，把叙界冒险收敛为"8 标签/回合"预设接入、Module4 收敛为"世界模拟/日程时钟"预设接入；叙界 UI 与 Module4 玩法资产迁入预设，测试 950+16 项作为安全网。
- 保留的不确定点：Module4 的时钟/日程/跨日结算属于内核还是预设、Actor State 与 facts 的认知模型（knownTo/epistemicStatus）是否并入内核——需 TASK-002 用原型验证，**本报告不锁定路线**。

## 11. 仍需进一步调查的问题（建议 TASK-002）

1. `.denova/` 与预设目录的实际样例 JSON（克隆内不存在；需在运行工作区抽查）——校验 §E 预设 schema 推断。
2. Game 的 `submit_interactive_turn` 变更协议能否无损表达叙界 `[STATE]/[CHANGES]` 与 Module4 facts 语义（对照试验）。
3. Module4 judgment(计分) ↔ Game RuleCheck(d20)：能否以"规则集预设"统一且不损失玩法手感。
4. 叙界 embedded 首轮是否强制要求本地 apiConfig（F 报告疑点：`NarraverseSharedAI.refresh` 无调用方）——影响"两套模型路径"实际割裂程度。
5. 8097 Go/Python 同写 `.denova/projects` 的锁/一致性现状与冲突频率。
6. 角色卡在 4 处存储的字段漂移实测（KB→lore→master→冒险/Module4 各缺哪些字段）。
7. Module4 与叙界冒险之间是否存在用户可见的数据互通入口（代码未见，需运行态验证）。
8. 现有本地存档/世界数据的规模与格式分布——决定 B 路线迁移成本。
