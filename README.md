# game-ai-skills

面向游戏研发团队的 AI 工作流集合，覆盖需求转 OpenSpec、Unity、Cocos Creator 和 Cocos2d-lua 界面制作、版本与分支审查、H5 项目迁移。

每个 Skill 以 `SKILL.md` 定义输入、执行步骤、证据要求和交付边界，可按需安装到 AI 编程工具中。当前工作树有 4 个主入口、1 个共享依赖包和 9 个归档 Skill。下表使用各入口在 `SKILL.md` 中声明的调用名。

[选择 Skill](#选择-skill) · [安装与调用](#安装与调用) · [调用示例](#调用示例) · [仓库结构](#仓库结构) · [归档 Skill](#归档-skill) · [贡献与维护](#贡献与维护)

## 选择 Skill

| 任务 | Skill 入口 | 适用范围 |
| --- | --- | --- |
| 需求转 OpenSpec，或按请求继续制作游戏 UI | [game_ai_openspec_apply](game_ai_openspec_apply/SKILL.md) | 统一需求方案、UI 契约、界面制作与验收；也可直接续做已有变更的 UI |
| 综合审查固定版本，兼查新增与历史问题 | [game_ai_review_version_evidence](game_ai_review_version_evidence/SKILL.md) | 独立发现、核证，逐项反馈并最终统一交付 |
| 查某个固定分支现存的问题 | [game_ai_branch_history_review](game_ai_review_branch_history/SKILL.md) | 单分支审查现存缺陷、历史遗留与需求遗漏，不要求对照基线 |
| Cocos Creator H5 项目迁移或迁移审查 | [gameai_h5_project_migration_check](game_ai_h5_project_migration_check/SKILL.md) | 覆盖客户端与 `tools_h5`，区分迁移实施和只读审查 |

审查带 MR 的固定版本可用 `game_ai_review_version_evidence`，默认同时覆盖新增与仍存在的历史问题，可按请求限定仅新增。只想检查一个固定分支的现状、没有对照基线时，用 `game_ai_branch_history_review`。

`game_ai_review_shared` 提供单分支入口使用的审查流程、证据判定、报告与打包工具。固定版本综合入口自带这些工具，可独立安装。两套流程均可核对本版有效的 OpenSpec 条款及其实现。

## 安装与调用

### 1. 获取仓库

```bash
git clone https://github.com/touchatoms/game_ai_skills.git
cd game_ai_skills
```

只复制需要的 Skill 完整目录，保留其中的 `references/`、`scripts/`、`agents/` 等文件。不要只复制 `SKILL.md`，也不要把整个仓库嵌套成单个 Skill。

### 2. 选择安装位置

| 工具 | 项目内安装 | 用户级安装 |
| --- | --- | --- |
| Codex | `<项目>/.agents/skills/<skill>/` | `~/.agents/skills/<skill>/` |
| Claude Code | `<项目>/.claude/skills/<skill>/` | `~/.claude/skills/<skill>/` |

以上为通用发现目录，分别见 [Codex 官方说明](https://developers.openai.com/codex/skills/) 和 [Claude Code 官方说明](https://code.claude.com/docs/en/skills)。本仓库部分入口另有固定路径要求，见下文。

在仓库根目录执行以下命令，将示例路径替换为目标游戏项目路径。

**Codex 项目内安装**

```bash
mkdir -p /path/to/game-project/.agents/skills
cp -R game_ai_openspec_apply /path/to/game-project/.agents/skills/
```

**Claude Code 项目内安装**

```bash
mkdir -p /path/to/game-project/.claude/skills
cp -R game_ai_openspec_apply /path/to/game-project/.claude/skills/
```

已有同名安装时，先比较本地定制内容和版本，再更新。

### 3. 配置入口名称与依赖

- **功能与 UI 统一入口**：完整安装 `game_ai_openspec_apply` 即可使用方案、UIApply 和联合流程。已有安装升级时更新完整目录，检查本地定制。
- **单分支历史审查**：仓库目录是 `game_ai_review_branch_history`，调用名为 `game_ai_branch_history_review`。其 `SKILL.md` 明确从 `${CODEX_HOME:-$HOME/.codex}/skills/game_ai_review_shared/WORKFLOW.md` 读取依赖。即使入口放在 `.agents/skills/`，也须把完整的 `game_ai_review_shared` 放到上述位置；不能只放在入口旁边便认为可用。
- **H5 项目迁移**：仓库目录已改为 `game_ai_h5_project_migration_check`，但 `SKILL.md` 的 `name` 及 `agents/openai.yaml` 的提示仍是 `gameai_h5_project_migration_check`。当前调用沿用旧名；安装到发现目录时也使用旧名以保持一致。

**单分支历史审查**（首次安装，目标入口目录尚不存在）

```bash
mkdir -p ~/.agents/skills
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R game_ai_review_branch_history ~/.agents/skills/game_ai_branch_history_review
cp -R game_ai_review_shared "${CODEX_HOME:-$HOME/.codex}/skills/"
```

**H5 项目迁移**（首次安装，目标目录尚不存在）

```bash
mkdir -p ~/.agents/skills
cp -R game_ai_h5_project_migration_check ~/.agents/skills/gameai_h5_project_migration_check
```

### 4. 确认加载并调用

在目标项目启动 AI 工具，确认可用 Skill 列表中出现对应入口，再提供文档、仓库、分支和输出路径等实际输入。以下示例采用 Codex 的 `$skill_name` 写法；Claude Code 使用 `/skill_name`。

若未加载，检查发现目录、`SKILL.md`、入口名称和完整依赖。`AGENTS.md` 可补充团队选择规则，但不能代替安装；按项目需要合并规则，保留项目已有内容。本仓库的 [AGENTS.md](AGENTS.md) 仍含归档 Skill 的路由，使用前按实际安装情况调整。

## 调用示例

示例中的路径、分支、提交和版本号均须替换为实际值；具体执行以对应 `SKILL.md` 为准。

### OpenSpec 方案与 UI 制作

#### 需求转 OpenSpec

```text
使用 $game_ai_openspec_apply，把以下资料融合为 OpenSpec change：
需求：/path/to/docs/活动需求.docx
设计：/path/to/docs/客户端设计.md
接口：/path/to/docs/接口文档.md
生成 proposal、design、specs、tasks，并列出待确认问题。
```

也可直接输入需求文字。该入口需要 OpenSpec CLI；提取 `.docx` 时需要 Python 3。

#### Unity UI 制作

```text
使用 $game_ai_openspec_apply，以 UIApply 模式为当前 OpenSpec change 制作活动界面。
设计效果图路径：/path/to/design/活动主界面.png
UI 资源路径：/path/to/ui-assets/活动主界面
Prefab 保存路径：Assets/Res/Prefab/Activity/UIActivityWin.prefab
Prefab 资源命名：UIActivityWin
```

四项输入必须明确：效果图和资源须可访问，Prefab 目标路径须合法、可写且与命名一致；制作和验收还需可用的 Unity Editor 或已配置的 Unity 工具。UIApply 是自定义流程，不会自动注册 `/opsx:ui-apply` 命令。

#### Cocos Creator 和 Cocos2d-lua UI 制作

使用 `$game_ai_openspec_apply` 指定已有 change，提供设计图路径、UI 资源路径，再按模块填写主产物路径和名称。以下是三个独立调用的输出字段示例。

| 目标模块 | 主产物保存路径示例 | 产物名称 |
| --- | --- | --- |
| Cocos Creator 预制体 | `assets/activity/prefabs/ActivityPanel.prefab` | `ActivityPanel` |
| Cocos2d-lua 代码布局 | `src/app/views/ActivityPanel.lua` | `ActivityPanel` |
| Cocos2d-lua 使用 Studio 布局 | `cocosstudio/activity/ActivityPanel.csd` | `ActivityPanel` |

Studio 项目还需从工程导出规则核对运行时布局及其加载路径。联合流程同样适用于这两个引擎。

#### 从需求到 UI 制作

```text
使用 $game_ai_openspec_apply，把需求转为 OpenSpec change 后继续制作 Unity UI。
需求：/path/to/docs/活动需求.docx
设计：/path/to/docs/客户端设计.md
接口：/path/to/docs/接口文档.md
设计效果图路径：/path/to/design/活动主界面.png
UI 资源路径：/path/to/ui-assets/活动主界面
Prefab 保存路径：Assets/Res/Prefab/Activity/UIActivityWin.prefab
Prefab 资源命名：UIActivityWin
```

两个阶段复用同一 change、UI 契约与任务。只要求生成方案时不会执行 UI 制作；缺少制作输入时可先完成方案，UI 任务保留未完成。

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
├── game_ai_openspec_apply/              # OpenSpec 方案与 Unity、Cocos Creator、Cocos2d-lua 的 UI 制作及验收
├── game_ai_review_version_evidence/     # 固定版本综合审查，覆盖新增与历史问题
├── game_ai_review_branch_history/       # 固定分支历史审查，调用名为 game_ai_branch_history_review
├── game_ai_review_shared/               # 单分支历史审查的共享依赖
├── game_ai_h5_project_migration_check/  # H5 项目迁移与审查，调用名为 gameai_h5_project_migration_check
├── archived/                            # 9 个归档 Skill
├── AGENTS.md                            # 团队路由规则
├── CONTRIBUTING.md                      # 贡献与发布流程
└── README.md                            # Skill 索引、安装说明与调用示例
```

## 归档 Skill

以下目录保留在 `archived/`，不属于当前主入口。需要复用时，先阅读对应说明并核对依赖及适用范围，再单独安装。

| Skill | 原用途 |
| --- | --- |
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
