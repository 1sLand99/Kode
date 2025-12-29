# Kode CLI Refactor — 执行手册（todo_tasks_detail.md）

本文件用于把 `todo_tasks.json` 的任务拆解与“工程硬门禁/结构规范”落到可执行的步骤，并作为每一步重构的更新日志（保证窗口压缩/上下文丢失后也能快速恢复进度）。

## 0) 不可破坏的对外契约（Hard Contract）

- 参考基线：通过 `KODE_REFERENCE_REPO` 指向 legacy CLI 仓库根目录（用于 `bun run parity:reference`）
- 最低要求：**外部行为/流程/体验/参数/协议 100% 一致**
  - CLI：命令名、flags、默认值、help 文案与顺序、exit code、stdout/stderr、structured stdio、`--print` 行为
  - 协议：MCP/ACP/stdio 协议字段、tool schema/描述、工具列表与顺序（除非旧实现本身不稳定且有测试解释）
  - 路径/持久化：配置目录、session/messages、Windows/mac/Linux 差异

## 1) 本仓库硬门禁（必须长期保持全绿）

- `bun run parity:reference`：与参考仓库离线 parity（stdout/stderr/exit code + help matrix + tool manifest）
- `bun test`：单测/集成/E2E 全量离线通过
- `bun run typecheck`
- `bun run lint`
- `bun run build`

> 约定：任何结构调整都必须在门禁全绿后才允许进入下一任务。

## 2) 结构与边界规范（目标状态）

### 2.1 `src/` 目录规则

- `src/` 一级目录：只允许目录，不允许散落单文件（已满足）。
- 分层依赖方向（强约束）：
  - `src/core/**`：不得依赖 `src/ui/**`
  - `src/services/**`：不得依赖 `src/ui/**`
  - `src/ui/**`：可依赖 `core/services/utils/types`
  - `src/entrypoints/**`：允许依赖所有层（作为编排层）

### 2.2 命名与一致性

- 目录名优先使用 `lowercase`（必要的兼容入口除外）。
- 禁止大小写不一致的 import（会导致 Linux/CI 崩溃）；计划启用 TS `forceConsistentCasingInFileNames` 作为门禁的一部分。

> 注：legacy 基线存在大小写历史不一致；parity 脚本已做兼容探测，本仓库统一使用一致的大小写命名。

## 3) 当前任务状态（同步自 todo_tasks.json）

- T020 `success`：新增并维护 `todo_tasks_detail.md`
- T021 `success`：整理 `src/tools/` 一级结构（分类目录化）
- T022 `success`：移动 MCP 审批 UI 到 `src/ui/`，消除 service→ui 依赖
- T023 `success`：Tool 类型/描述函数下沉到 core，并清理 core/tools 碎片目录
- T024 `success`：cost tracker 扁平化 + React hook 移出 core
- T025 `success`：启用并修复大小写一致性（跨平台）
- T026 `success`：二次清理无用代码/冗余结构与文档归档
- T027 `success`：统一 commands 模块命名风格（移除下划线/对齐 kebab-case）
- T028 `success`：拆分并收敛 `@utils/messages`（core 与 UI 输入处理分离）
- T029 `success`：加固结构与边界的自动化检查（core→ui 禁止、src 结构规则）
- T030 `success`：命名一致性终清（utils/types/screens 大小写混乱修复）
- T031 `success`：命名规范门禁化（防回归的自动化检查）
- T032 `success`：UI 组件目录命名一致性（消除 components 目录命名风格混乱）
- T033 `success`：permissions 子目录命名一致性（全部 kebab-case 小写）
- T034 `success`：`src/utils` 领域分组收敛（减少散落单文件，结构更清晰）
- T035 `success`：utils 结构规范门禁化（防止 `src/utils` 根目录回归散乱）
- T036 `success`：Node.js 可运行基础（ripgrep 改为 `@vscode/ripgrep`，移除 vendor）
- T037 `success`：`BunShell` 迁移为 Node.js spawn/which 实现
- T038 `success`：`BunFile`/`BunSearcher` 迁移为 Node.js fs/glob 实现
- T039 `success`：清除 `src/` 运行时代码中的 Bun 专有 API
- T040 `success`：npm 分发 Node 可直接运行（build:npm + wrapper 逻辑）
- T041 `success`：GitHub Actions 发布：npm + 多平台 Bun 单文件二进制
- T042 `success`：更新 README/发布文档/AGENTS.md
- T043 `success`：重构 services 分组（plugins 域）
- T044 `success`：重构 services 分组（ai 域收敛）
- T045 `success`：重构 services 分组（system/auth/telemetry/context/ui 域）
- T046 `success`：services 结构门禁化 + 文档对齐
- T047 `success`：文档深度清理（对外版本）
- T048 `success`：清理源码注释（保持语义 100% 不变）
- T049 `success`：全仓最终扫尾（去外部引用 + 门禁复验）
- T050 `success`：从系统中干净移除 ThinkTool

## 4) 更新日志（每完成一项任务追加一条）

### 2025-12-27

- 初始化本文件，建立门禁/边界/命名约束与任务看板（T020）。
- 工具目录收敛：`src/tools/` 一级仅保留分类目录（`agent/ai/interaction/mcp/network/search` + `filesystem/system`）与 `index.ts`，并保持 tool manifest/顺序不变（T021）；门禁：typecheck/lint/test/build/parity 全绿。
- UI/Service 解耦：将 MCP server approval 交互迁到 `src/ui/screens/mcpServerApproval.tsx`，删除 `src/services/mcpServerApproval.tsx`，并新增边界单测 `tests/unit/layer-boundaries.test.ts` 防止 services→ui 回归；门禁：parity 全绿（T022）。
- Tool 核心收敛：将 Tool interface 与 `getToolDescription` 下沉到 `src/core/tools/tool.ts`，`@tool` 指向 core；并把 `src/core/tools/*/index.ts` 单文件目录合并为 `src/core/tools/{tool,registry,executor,defineTool}.ts`（T023）；门禁：test/build/parity 全绿。
- Cost 解耦：将 cost tracker 收敛到 `src/core/costTracker.ts`（core 不再依赖 React），并把 `useCostSummary()` 迁移到 `src/ui/hooks/useCostSummary.ts`；门禁：test/build/parity 全绿（T024）。
- 大小写一致性：启用 `tsconfig.json` 的 `forceConsistentCasingInFileNames`，并移除 `src/Tool/` 目录（`src/` 一级目录全小写）；同时更新 `scripts/reference-parity-check.mjs` 的 tool manifest 探测逻辑以兼容参考仓库的 `src/Tool.ts`（T025）。
- 测试临时目录清理：将若干单测/集成测里写入固定 `.tmp*` 目录的逻辑改为使用 `mkdtempSync`（并在用例结束时清理），避免污染仓库；并清除历史遗留的 `.tmp/`、`.tmp-kode-config/`（T026）。
- Commands 命名统一：`src/commands` 移除下划线文件名并对齐 kebab-case（含 `approved-tools.ts`、`refresh-commands.ts`、`ctx-viz.ts`、`messages-debug.ts`、`pr-comments.ts`），同时更新导入与变量命名；门禁：typecheck/lint/test/build/parity 全绿（T027）。

### 2025-12-28

- `@utils/messages` 收敛拆分：将原 `src/utils/messages.tsx` 移入 `src/utils/messages/core.ts`（移除 Ink/React/UI 依赖，仅保留核心消息/规范化逻辑），并把 `processUserInput()`（含 bash/koding/SlashCommand 处理）下沉到 `src/utils/messages/userInput.tsx`；`src/utils/messages/index.ts` 通过 lazy import 暴露 `processUserInput`，确保非 UI 模块引用 `@utils/messages` 不会引入 Ink/React；门禁：typecheck/lint/test/build/parity 全绿（T028）。
- 边界门禁加固：扩展 `tests/unit/layer-boundaries.test.ts`，新增 `src/core/**` 禁止依赖 `src/ui/**`（同时避免误伤 `ui-helpers` 命名），并增加 `src/` 一级目录仅目录/全小写与 case-insensitive 路径冲突检测；门禁：typecheck/lint/test/build/parity 全绿（T029）。
- 命名一致性终清：将 `src/utils/{BunShell,BunFile,BunSearcher,Cursor}.ts` 统一为 `src/utils/{bunShell,bunFile,bunSearcher,cursor}.ts`，将 `src/types/{PermissionMode,RequestContext}.ts` 统一为 `src/types/{permissionMode,requestContext}.ts`，并将 `src/ui/screens/mcpServerApproval.tsx` 统一为 `src/ui/screens/MCPServerApproval.tsx`；同步更新所有 import 路径并保持导出/行为不变；门禁：typecheck/lint/test/build/parity 全绿（T030）。
- 命名规范门禁化：新增命名单测 `tests/unit/naming-conventions.test.ts`，将 `src/utils`/`src/types` “禁止大写开头文件名”、`src/ui/screens` “要求大写开头文件名”固化为离线可跑的回归门禁；门禁：typecheck/lint/test/build/parity 全绿（T031）。
- UI 组件目录命名一致性：将 `src/ui/components/CustomSelect/` 重命名为 `src/ui/components/custom-select/`，将 `src/ui/components/messages/UserToolResultMessage/` 重命名为 `src/ui/components/messages/user-tool-result-message/` 并同步更新所有 import；同时在 `tests/unit/naming-conventions.test.ts` 增加对 `src/ui/components` 顶层目录与 `messages` 子目录“必须全小写”的回归门禁；门禁：typecheck/lint/test/build/parity 全绿（T032）。
- permissions 目录命名一致性：将 `src/ui/components/permissions/*PermissionRequest/` 全部重命名为 kebab-case 小写目录（例如 `ask-user-question-permission-request/`、`plan-mode-permission-request/` 等），同步更新所有 import；并在 `tests/unit/naming-conventions.test.ts` 增加 `src/ui/components/permissions` 子目录必须全小写的回归门禁；门禁：typecheck/lint/test/build/parity 全绿（T033）。

### 2025-12-29

- `src/utils` 领域分组收敛：将 `src/utils` 根目录散落 util 文件按领域迁移到 `agent/ bun/ commands/ config/ fs/ identity/ log/ model/ plan/ session/ state/ system/ terminal/ text/ theme/ tooling/` 等目录，并将 `ask` 迁移到 `src/app/ask.ts`；同步更新全仓库 import 路径与 utils 内部相对导入，确保 `src/utils` 根目录不再有单文件；门禁：typecheck/lint/test/build/parity 全绿（T034）。
- utils 结构规范门禁化：在 `tests/unit/naming-conventions.test.ts` 增加 `src/utils` 根目录不得出现散落 `.ts/.tsx` 文件的回归检查；门禁：typecheck/lint/test/build/parity 全绿（T035）。
- ripgrep 迁移：使用 `@vscode/ripgrep` 替代 vendor ripgrep（优先系统 `rg`，fallback 到 `rgPath`），并移除 `vendor/` 与 ensure-ripgrep 脚本/workflow；更新单测；门禁：typecheck/lint/test/build/parity 全绿（T036）。
- `BunShell` Node 化：`src/utils/bun/shell.ts` 用 `child_process.spawn` + `which` 替代 `Bun.spawn/Bun.which`，并保持 sandbox/timeout/background/输出增量行为与参考一致；门禁：typecheck/lint/test/build/parity 全绿（T037）。
- `BunFile`/`BunSearcher` Node 化：`src/utils/bun/file.ts` 改为 `fs/promises` 实现读写/append/partial read；`src/utils/bun/searcher.ts` 改为 `glob` 实现并支持 `AbortSignal`；同步简化 `src/utils/fs/file.ts` 的 glob 路径；门禁：typecheck/lint/test/build/parity 全绿（T038）。
- 清除剩余 Bun API：移除 `src/utils/session/kodeHooks.ts` 的 `Bun.spawn` 与 `src/services/pluginRuntime.ts` 的 `Bun.Glob`（改为 Node `spawn`/`glob`/`which`），并新增回归测试 `tests/unit/no-bun-runtime-api.test.ts` 防止 `src/**` 回归引入 `Bun.` 访问；门禁：typecheck/lint/test/build/parity 全绿（T039）。
- npm Node 运行闭环：新增 `build:npm`（esbuild，Node target），`cli.js/cli-acp.js` wrapper fallback 改为 `node dist/index.js`（不再依赖 Bun runtime），并更新 `scripts/build.mjs`/`scripts/prepublish-check.js`；新增 `tests/integration/node-runtime-smoke.test.ts` 验证 Node 可运行；门禁：typecheck/lint/test/build/parity 全绿（T040）。
- CI/Release 更新：tag/dev 发布 workflow 使用 `build:npm`，并按 matrix 产出并上传多平台 Bun `--compile` 单文件二进制资产（命名与 `scripts/binary-utils.cjs` 一致），确保发布流程无需人工介入（T041）。
- 文档对齐新分发策略：更新 `README.md`（npm 安装 + 二进制运行）、`docs/release_checklist.md`/`docs/baseline_verification.md`（build:npm + Node smoke + parity）、`docs/upgrade_design.md`（路径/策略修正）与 `AGENTS.md`（binary-first + Node fallback + 关键路径索引）；门禁：typecheck/lint/test/build/parity 全绿（T042）。
- services/plugins 域下沉：将 `src/services/{customCommands,pluginRuntime,pluginValidation,skillMarketplace}.ts` 移入 `src/services/plugins/`，并通过 `tsconfig.json` 精确 paths 映射保持 `@services/*` 导入 specifier 不变；门禁：typecheck/lint/test/build/parity 全绿（T043）。
- services/ai 域收敛：将 `src/services/{openai,gpt5ConnectionTest,llmLazy,llmConstants,responseStateManager}.ts` 下沉到 `src/services/ai/`，并用 `tsconfig.json` 精确 paths 映射保持 `@services/*` 导入 specifier 不变；同时移除 `src/services/{llm,modelAdapterFactory}.ts` stub，改为直接指向 `src/services/ai/*`；门禁：typecheck/lint/test/build/parity 全绿（T044）。
- services 域拆分：新增 `src/services/{system,auth,telemetry,context,ui}/`，并将 system/auth/telemetry/context/ui 相关服务下沉；通过 `tsconfig.json` 精确 paths 映射保持 `@services/*` 导入 specifier 不变，同时修复少量跨域相对导入为 `@services/*`；门禁：typecheck/lint/test/build/parity 全绿（T045）。
- services 结构门禁化：移除 `src/services/*.ts` 根目录 stub（含 `mcpClient/mcpCliUtils`），改用 `tsconfig.json` 精确 paths 指向 `src/services/mcp/index.ts` 等真实模块；新增单测 `tests/unit/naming-conventions.test.ts` 约束 `src/services` 根目录仅允许目录；同步更新 `docs/upgrade_design.md`；门禁：typecheck/lint/test/build/parity 全绿（T046）。
- 文档精简（对外版本）：去除对外部项目/个人路径的依赖描述，精简并校准 `AGENTS.md`、`docs/upgrade_design.md`、`docs/baseline_verification.md` 与 `todo_tasks_detail.md`；门禁：typecheck/lint 全绿（T047）。
- 清理源码/测试注释：移除非指令型注释并修复误删导致的正则/URL/template literal 截断（从 dist sourcemap 与参考仓库还原关键片段），补齐 `tests/helpers/canListen.ts` 以在禁止 listen 的环境下跳过相关网络监听用例；门禁：typecheck/lint/test/build/parity 全绿（T048）。
- 全仓最终扫尾：移除 `scripts/reference-parity-check.mjs` 的本机绝对路径 fallback，统一要求 `--reference` 或 `KODE_REFERENCE_REPO`；同步更新 `docs/release_checklist.md` 与 `docs/baseline_verification.md` 示例；门禁：typecheck/lint/test/build/parity 全绿（T049）。
- 移除 ThinkTool：删除 `src/tools/agent/ThinkTool/` 及其 prompt，实现层与 UI 层移除所有引用（`src/utils/model/thinking.ts`、`src/ui/components/messages/AssistantToolUseMessage.tsx`）；门禁：typecheck/lint/test/build/parity 全绿（T050）。
