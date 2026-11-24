# game-ai-skills

面向游戏研发团队的 AI 工作流集合，覆盖需求转 OpenSpec、Unity UI 制作、版本与分支审查、H5 项目迁移。

每个 Skill 以 `SKILL.md` 定义输入、执行步骤、证据要求和交付边界，可按需安装到 AI 编程工具中。当前入口见下表；早期需求分析、测试、配置、引擎专项审查等 Skill 已移入 `archived/`。

## 选择 Skill

| 任务 | 入口与说明 |
|---|---|
| 将需求文字或需求／设计／接口文档转为 OpenSpec change | [game_ai_function](game_ai_function/SKILL.md)：融合分析，生成 proposal、design、specs、tasks，最后进入 explore 讨论 |
| 根据设计图和资源制作 Unity UI Prefab | [openspec-ui-apply](openspec-ui-apply/SKILL.md)：制作或更新界面，完成结构、运行与视觉核对，交接 OpenSpec 任务 |
| 审核 OpenSpec 版本的客户端 C# 实现 | [openspec-ai-review](openspec-ai-review/SKILL.md)：对比 C# 差异并核查有效需求；按模式执行内部审核或发布单文件 HTML 到 SVN、飞书 |
| 综合审查固定版本，兼查新增与历史问题 | [game_ai_review_version_evidence](game_ai_review_version_evidence/SKILL.md)：独立发现、核证，逐项反馈并最终统一交付 |
| 仅查源分支或 MR 合入目标分支引起的问题 | [game_ai_merge_review](game_ai_merge_review/SKILL.md)：核证源变更与合并交互的因果，历史已有问题不计新增 |
| 查某个固定分支现存的问题 | [game_ai_branch_history_review](game_ai_branch_history_review/SKILL.md)：单分支审查现存缺陷、历史遗留与需求遗漏，不要求对照基线 |
| Cocos Creator H5 项目迁移或迁移审查 | [H5 项目迁移](game_ai_h5_project_migration_check/SKILL.md)：覆盖客户端与 `tools_h5`，区分迁移实施和只读审查 |

审查入口按目标选择：综合审查用 `game_ai_review_version_evidence`，合并归因用 `game_ai_merge_review`，单分支现状用 `game_ai_branch_history_review`。逐项反馈规则目前仅在综合入口明确配置，不能默认认为拆分入口已同步；三者的对照测试程度也不能视为相同。

`game_ai_review_shared` 是两个拆分审查入口的共享依赖，不是独立 Skill。综合入口包含自己的工具与参考文件，可单独安装。

## 安装与开始使用

### 1. 获取仓库

```bash
git clone https://github.com/touchatoms/game_ai_skills.git
cd game_ai_skills
```

只复制需要的 Skill 完整目录，保留其中的 `references/`、`scripts/`、`agents/` 等文件。不要只复制 `SKILL.md`，也不要把整个仓库嵌套成单个 Skill。

### 2. 选择安装位置

| 工具 | 项目内安装 | 用户级安装 |
|---|---|---|
| Codex | `<项目>/.agents/skills/<skill>/` | `~/.agents/skills/<skill>/` |
| Claude Code | `<项目>/.claude/skills/<skill>/` | `~/.claude/skills/<skill>/` |

以上为通用发现目录，分别见 [Codex 官方说明](https://developers.openai.com/codex/skills/)和 [Claude Code 官方说明](https://code.claude.com/docs/en/skills)。本仓库部分入口另有固定路径要求，见下文。

例如，在目标游戏项目中安装需求转 OpenSpec 入口（以下命令在本仓库根目录执行，将示例路径换成实际路径）：

```bash
mkdir -p /path/to/game-project/.agents/skills
cp -R game_ai_function /path/to/game-project/.agents/skills/
```

Claude Code 对应示例：

```bash
mkdir -p /path/to/game-project/.claude/skills
cp -R game_ai_function /path/to/game-project/.claude/skills/
```

已有同名安装时，先比较本地定制内容和版本，再更新。

### 3. 处理入口的特殊要求

- **两个拆分审查入口**：其 `SKILL.md` 明确从 `${CODEX_HOME:-$HOME/.codex}/skills/game_ai_review_shared/WORKFLOW.md` 读取依赖。即使入口放在 `.agents/skills/`，也须把完整的 `game_ai_review_shared` 放到上述位置；不能只放在入口旁边便认为可用。
- **OpenSpec AI Review**：按该入口约定，仅安装在当前工程 `.codex/skills/openspec-ai-review/`，并使用其本机配置。先确认当前客户端能发现该目录；不要仅套用通用安装示例。配置方式见 [配置与运行](openspec-ai-review/references/configuration.md)。
- **H5 项目迁移**：仓库目录已改为 `game_ai_h5_project_migration_check`，但 `SKILL.md` 的 `name` 及 `agents/openai.yaml` 的提示仍是 `gameai_h5_project_migration_check`。当前调用沿用旧名；安装到发现目录时也使用旧名以保持一致。

拆分审查安装示例：

```bash
mkdir -p ~/.agents/skills
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R game_ai_merge_review game_ai_branch_history_review ~/.agents/skills/
cp -R game_ai_review_shared "${CODEX_HOME:-$HOME/.codex}/skills/"
```

H5 迁移首次安装示例（目标目录尚不存在时执行）：

```bash
mkdir -p ~/.agents/skills
cp -R game_ai_h5_project_migration_check ~/.agents/skills/gameai_h5_project_migration_check
```

### 4. 确认加载并调用

在目标项目启动 AI 工具，确认可用 Skill 列表中出现对应入口，再提供文档、仓库、分支和输出路径等实际输入。以下示例采用 Codex 的 `$skill_name` 写法；Claude Code 使用 `/skill_name`。

若未加载，检查发现目录、`SKILL.md`、入口名称和完整依赖。`AGENTS.md` 可补充团队选择规则，但不能代替安装；按项目需要合并规则，保留项目已有内容。本仓库的 [AGENTS.md](AGENTS.md) 仍含归档 Skill 的路由，使用前按实际安装情况调整。

## 调用示例

示例中的路径、分支、提交和版本号均须替换为实际值；具体执行以对应 `SKILL.md` 为准。

### 需求转 OpenSpec

```text
使用 $game_ai_function，把以下资料融合为 OpenSpec change：
需求：/path/to/docs/活动需求.docx
设计：/path/to/docs/客户端设计.md
接口：/path/to/docs/接口文档.md
生成 proposal、design、specs、tasks，并列出待确认问题。
```

也可直接输入需求文字。该入口需要 OpenSpec CLI；提取 `.docx` 时需要 Python 3。

### Unity UI 制作

```text
使用 $openspec-ui-apply，为当前 OpenSpec change 制作活动界面。
设计效果图路径：/path/to/design/活动主界面.png
UI 资源路径：/path/to/ui-assets/活动主界面
Prefab 保存路径：Assets/Res/Prefab/Activity/UIActivityWin.prefab
Prefab 资源命名：UIActivityWin
```

四项输入必须明确且可访问；制作和验收还需可用的 Unity Editor 或已配置的 Unity 工具。UIApply 是自定义 Skill，不会自动注册 `/opsx:ui-apply` 命令。

### OpenSpec C# 审核

```text
$openspec-ai-review target_branch=release/1.2.0 base_branch=master mode=review
```

`review` 仅执行内部取证与审核，不输出报告文件或发送消息。发布模式 `review-publish` 会将 HTML 提交到 SVN 并同步飞书，也是入口的默认模式；只需内部审核时应显式指定 `mode=review`。发布前须准备好本机配置。仅安装或讨论该 Skill 不执行发布。审核仅覆盖 C# 代码及其有效需求实现，不做资源差异或视觉布局审核。

### 固定版本综合审查

```text
使用 $game_ai_review_version_evidence，完成独立审查、核证及交付。
仓库：/path/to/client
目标：develop_1.9.0
目标提交：<完整 SHA>
基线：master
基线提交：<完整 SHA>
完整输入：/path/to/评审文档
输出根目录：/path/to/综合审查输出

以指定提交为准，冻结实际 Skill 版本及全部输入文件 SHA-256。
新建本轮独立输出目录，交付一个 HTML 阅读入口、一个证据 ZIP 及 SHA-256 校验文件。
```

该入口替代此前的 `review-version-evidence` 名称。续审或维护旧报告时使用原冻结技能与工具，详见入口说明。

### 分支或 MR 合入审查

```text
使用 $game_ai_merge_review，审查源分支合入目标分支引起的问题。
仓库：/path/to/client
源分支：develop_1.9.0
源提交：<完整 SHA>
合入目标分支：master
目标基线提交：<完整 SHA>
完整输入：/path/to/评审文档
输出根目录：/path/to/合并审查输出
历史已有问题不计新增；冻结入口、共享工作流及全部输入 SHA-256。
```

也可提供 MR 链接、允许使用的资料与输出根目录，由入口解析并冻结实际版本。

### 固定分支历史审查

```text
使用 $game_ai_branch_history_review，审查指定固定分支现存的问题。
仓库：/path/to/client
被审分支：master
被审提交：<完整 SHA>
完整输入：/path/to/评审文档
输出根目录：/path/to/历史审查输出
本轮为单分支审查，无对照基线；冻结入口、共享工作流及全部输入 SHA-256。
```

### H5 项目迁移审查

```text
使用 $gameai_h5_project_migration_check，只读审查以下项目的迁移情况。
目标仓库：/path/to/h5newgame_client_wheel
审查范围：客户端及 tools_h5 的迁移相关变更
```

若要求实际迁移，需提供源仓库、源分支、目标仓库和版本号；只读审查可复用已有仓库与提交上下文。默认范围不包含 Jenkins、打包发布及真机验收；迁移实施仅生成本地提交，不提供远端推送。

## 仓库结构

```text
game_ai_skills/
├── game_ai_function/                  # 需求转 OpenSpec
├── openspec-ui-apply/                 # Unity UI 制作
├── openspec-ai-review/                # OpenSpec C# 审核
├── game_ai_review_version_evidence/   # 固定版本综合审查
├── game_ai_merge_review/              # 合入问题审查
├── game_ai_branch_history_review/     # 固定分支历史审查
├── game_ai_review_shared/             # 两个拆分审查入口的共享依赖
├── game_ai_h5_project_migration_check/ # H5 迁移，调用名暂沿用 gameai_ 前缀
├── archived/                         # 早期 Skill 归档
├── AGENTS.md                         # 团队路由规则
├── CONTRIBUTING.md                   # 贡献与发布流程
└── README.md
```

## 已归档 Skills

以下目录保留在 `archived/`，不属于当前主入口。需要复用时，先阅读对应说明并核对依赖及适用范围，再单独安装。

| Skill | 原用途 |
|---|---|
| [gameai_analyze_requirement](archived/gameai_analyze_requirement/SKILL.md) | 游戏需求分析与任务拆解 |
| [gameai_testcase_generator](archived/gameai_testcase_generator/SKILL.md) | 测试用例和开发自测清单 |
| [gameai_analyze_activity](archived/gameai_analyze_activity/SKILL.md) | 捕鱼活动需求分析 |
| [gameai_check_config](archived/gameai_check_config/SKILL.md) | 捕鱼活动及奖励配置检查 |
| [gameai_code_review_cocos2d_lua](archived/gameai_code_review_cocos2d_lua/SKILL.md) | Lua / Cocos2d-lua 代码审查 |
| [gameai_code_review_unity](archived/gameai_code_review_unity/SKILL.md) | Unity C# 代码审查 |
| [gameai_code_review_cocos_creator](archived/gameai_code_review_cocos_creator/SKILL.md) | Cocos Creator / Cocos2d 代码审查 |
| [gameai_check_release](archived/gameai_check_release/SKILL.md) | 游戏发版前检查 |
| [gameai_check_performance](archived/gameai_check_performance/SKILL.md) | 游戏性能检查 |

## 贡献与维护

按 [贡献固定流程](CONTRIBUTING.md) 执行：确定范围 → 修改内容与示例 → 验证 → Fork 功能分支提交 PR → 维护者审核合并。同一任务复用已有分支、PR 和仍然有效的验证。

- 新增入口使用 `game_ai_` 前缀；已有名称及替代关系先核对，目录、`name`、调用示例和引用保持一致。
- 每个入口包含 `SKILL.md`，顶部 YAML 声明 `name` 和明确触发场景的 `description`；依赖、配置及交付边界需写清楚。
- 纯文档修改核对内容、路径、引用并运行 `git diff --check`；脚本或工作流行为变化运行相关验证。
- Git 提交、推送须有当前会话对操作和范围的明确授权；合并、变基另需授权。安装与实际调用验证分别记录，不把 PR 创建视为已部署。
