# 需求转 OpenSpec

输入一段需求文字，或输入一个或多个文档，自动完成：

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  需求输入     │    │  设计文档     │    │  接口文档     │
│  文字/文档    │    │  .docx/.md   │    │  .docx/.md   │
└──────┬───────┘    └──────┬───────┘    └──────┬───────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           ▼
               ┌─────────────────────┐
               │   融合需求分析        │
               └─────────┬───────────┘
                         ▼
               ┌─────────────────────┐
               │  OpenSpec Propose   │
               │  proposal / design  │
               │  specs   / tasks    │
               └─────────┬───────────┘
                         ▼
               ┌─────────────────────┐
               │  Explore 讨论        │
               └─────────────────────┘
```

最终产出：
- `openspec/changes/<name>/proposal.md` — 做什么 & 为什么
- `openspec/changes/<name>/design.md` — 怎么做（融合设计文档内容）
- `openspec/changes/<name>/specs/*/spec.md` — 接口 & 功能规格（融合接口文档内容）
- `openspec/changes/<name>/tasks.md` — 实施步骤
- 一份融合需求分析报告
- 待确认问题清单

**环境要求：** 使用 OpenSpec CLI；提取 `.docx` 时需要 Python 3。

---

## 输入

统一入口为 `$game_ai_openspec_apply`。以下 `/opsx:from-doc` 写法仅供已有项目命令映射使用；安装 Skill 不会自动注册该命令。支持以下格式：

| 格式 | 说明 |
|---|---|
| `/opsx:from-doc 做一个连续签到活动，连续7天，每天奖励递增` | 直接输入需求文字 |
| `/opsx:from-doc -r "做一个连续签到活动，连续7天，每天奖励递增"` | 使用 `-r` 输入需求文字 |
| `/opsx:from-doc <需求.docx>` | 单个需求文档 |
| `/opsx:from-doc -r <需求.docx> -d <设计.docx>` | 需求 + 设计文档 |
| `/opsx:from-doc -r <需求.docx> -d <设计.docx> -i <接口.docx>` | 全量三文档 |
| `/opsx:from-doc -r <需求.docx> -i <接口.docx>` | 需求 + 接口文档 |

**参数说明：**

| 参数 | 含义 | 影响 |
|---|---|---|
| `-r` / `--requirement` | 需求文字或需求文档 | 输入到 Phase 2 全维度分析 |
| `-d` / `--design` | 设计文档（技术方案/架构设计） | 融合到 design.md artifact |
| `-i` / `--interface` | 接口文档（API 协议/数据结构） | 融合到 specs + design.md |

**支持的文件格式：** `.docx`、`.md`、`.txt`。`-d` / `--design` 和 `-i` / `--interface` 仍只接受文档路径。

如果未提供任何输入，先使用 当前环境可用的提问工具（无工具时直接提问） 询问：
1. "请粘贴需求文字，或提供需求文档路径"
2. 获取有效需求后，再按需询问："请提供设计文档路径（可跳过）"
3. "请提供接口文档路径（可跳过）"

至少需要一段非空需求文字或一个可成功提取的需求文档，否则终止。

---

## 执行流程

### Phase 1: 输入归一化与文档提取

先将需求输入归一化为 `$req_content`：

1. 获取裸参数或 `-r` / `--requirement` 的完整值，不按空格拆分需求正文。
2. 如果该值是存在且扩展名受支持的文件，按下方文档规则提取并设置 `$req_content`。
3. 如果该值像文件路径但文件不存在，提示路径不存在，并让用户选择重新提供路径或明确改为需求文字；不得静默降级。
4. 其他非空值直接作为 `$req_content`，来源标记为“需求文字”。
5. 纯空白输入视为未提供需求。

“像文件路径”是指输入包含 `/`、`\\` 等路径分隔符，或以 `.docx`、`.md`、`.txt` 结尾。

需求文字接收成功后显示：

```text
📝 需求文字: 已接收 <字符数> 字
```

对每个提供的文档：

```bash
ls "<doc_path>"
```

根据文件扩展名选择提取方式：

**`.docx` — 使用 python3 解压 XML：**

```python
import zipfile
import xml.etree.ElementTree as ET

with zipfile.ZipFile('<doc_path>', 'r') as z:
    xml_content = z.read('word/document.xml')

tree = ET.fromstring(xml_content)
for p in tree.iter('{http://schemas.openxmlformats.org/wordprocessingml/2006/main}p'):
    texts = [t.text for t in p.iter('{http://schemas.openxmlformats.org/wordprocessingml/2006/main}t') if t.text]
    if texts:
        print(''.join(texts))
```

**`.md` / `.txt` — 直接 Read：**

使用 Read 工具直接读取文件内容。

**提取完成后标记每个文档：**
```
📄 需求文档: 签到-新版.docx — 75 段
📄 设计文档: 签到技术方案.md — 120 行
📄 接口文档: signin-api.docx — 45 段
```

将每个文档的内容保存为独立变量：
- `$req_content` — 需求文字或需求文档内容
- `$design_content` — 设计文档内容
- `$interface_content` — 接口文档内容

---

### Phase 2: 融合需求分析

1. **对需求输入** → 分析功能概述、玩法流程、状态流转、客户端任务、服务端任务、配置表、UI/资源、接口协议、数据存储、埋点、异常情况、风险、开发顺序、自测重点和待确认问题这 15 个维度；不适用项注明原因，无需安装归档分析 Skill。

2. **对设计文档** → 提取并整理：
   - 技术架构/系统分层
   - 关键设计决策
   - 模块划分
   - 数据流
   - 依赖关系

3. **对接口文档** → 提取并整理：
   - 所有 API 端点（method + path）
   - 请求参数（字段名、类型、必填、说明）
   - 响应字段（字段名、类型、说明）
   - 错误码定义
   - 数据结构/枚举定义
   - 调用时机和频率

4. **融合输出**：
   将三份分析合并为一份完整的分析报告，明确标注信息来源：

```markdown
## 接口协议（来源：接口文档）
| 接口 | Method | 请求参数 | 响应字段 | 说明 |
|---|---|---|---|---|
| get_signin_info | GET | activity_id | consecutive_days, ... | 拉取签到状态 |
```

将融合分析写入本次 change 的 `analysis.md`（创建 change 前可使用本轮独立临时文件），避免多个变更共用临时报告。

---

### Phase 3: 创建 OpenSpec Change

1. **确定 change name**（优先复用用户已指定或上下文明确的名称；否则从需求标题或核心功能提取 kebab-case，仅存在实质歧义时询问）

2. **创建或续用 change 目录**
   已有同名变更时，核对范围与已有 artifacts；明确续做则复用，不覆盖现有内容，不另建重复变更。范围冲突且无法判断时询问。新变更执行：
   ```bash
   openspec new change "<name>"
   ```

3. **获取 artifact 依赖**
   ```bash
   openspec status --change "<name>" --json
   ```

4. **按依赖顺序生成 artifacts，融合多文档内容：**

   | Artifact | 内容来源 |
   |---|---|
   | **proposal.md** | 需求输入分析（15维度概括） |
   | **design.md** | 需求分析的技术部分 + **设计文档提取的架构/决策** + 接口文档的数据结构 |
   | **specs/\*.md** | 需求功能点 + **接口文档的 API 定义**（每个接口的 request/response 写入对应 spec） |
   | **tasks.md** | 需求分析任务 + 设计文档的模块拆分 + 接口文档的联调任务 |

   对每个 ready artifact：
   ```bash
   openspec instructions <artifact-id> --change "<name>" --json
   ```
   - 用 `template` 作为文件结构
   - 将对应来源内容融合填充（不要只是拼接，要整合）
   - `context`、`rules` 是约束，不写入文件

   **特别处理 — 接口文档 → specs：**
   如果提供了接口文档，在 `specs/` 目录下额外创建 `api-contract/spec.md`：
   ```
   specs/api-contract/spec.md
   ```
   内容包含：
   - 每个 API 的 Requirement + Scenario
   - Request/Response schema
   - 错误码表

涉及 Unity UGUI、Cocos Creator 或 Cocos2d-lua 界面时，按统一入口的“方案到 UI 的交接”同步页面状态、绑定契约和 UIApply 任务；只做方案时不执行界面制作。

5. **验证完成**
   ```bash
   openspec status --change "<name>"
   ```

---

### Phase 4: Explore 讨论

1. 展示融合方案摘要
2. 逐条"待确认问题"（标注每个问题来自需求文字、需求文档、设计文档或接口文档）
3. 如果提供了设计文档和接口文档，重点讨论：
   - 设计方案是否有缺口？
   - 接口定义是否完整？
   - 与需求是否一致？
4. 仅方案请求在此交付；用户已要求接着制作 UI 时，继续统一入口的 UIApply 流程，不重复询问是否执行。

---

## 输出示例

```
## 📥 Phase 1: 输入归一化与文档提取

📄 需求文档: 签到-新版.docx — 75 段
📄 设计文档: 签到技术设计.md — 120 行
📄 接口文档: signin-api.docx — 45 段

直接输入需求文字时，需求输入行改为：

📝 需求文字: 已接收 35 字

## 🔍 Phase 2: 融合分析

✅ 需求分析: 15/15 维度
✅ 设计提取: 系统分层 + 5 个关键决策
✅ 接口提取: 4 个 API + 3 个数据结构 + 8 个错误码
融合报告: openspec/changes/daily-signin-v2/analysis.md

## 📋 Phase 3: OpenSpec 方案

**Change:** daily-signin-v2

✅ proposal.md — 融合需求概述
✅ design.md — 融合技术设计决策 + 接口架构
✅ specs/daily-signin/spec.md
✅ specs/cumulative-signin/spec.md
✅ specs/signin-coin-limit/spec.md
✅ specs/api-contract/spec.md — 接口协议规格
✅ tasks.md — 42 个实施任务（含联调任务）

## 💬 Phase 4: 待讨论

### 来自需求文档 (5)
1. 连续签到最大天数？
...

### 来自设计文档 (2)
6. 缓存策略是 Redis 单层还是 Redis+本地？
...

### 来自接口文档 (3)
8. get_signin_info 是否需要分页？
...

### 跨文档一致性问题 (2)
11. 需求说"支持最多2个道具"，接口设计支持3个 — 以哪个为准？
...
```

---

## Guardrails

- **至少需要一段非空需求文字或一个需求文档**，设计文档和接口文档可选
- **文字输入仅用于需求**，设计和接口输入仍必须是受支持的文档路径
- **纯空白输入无效**，提示用户重新提供需求
- **疑似文件路径不存在时不得静默转为需求文字**，必须让用户重新提供路径或明确确认按文字处理
- **需求文件与额外裸文字同时出现时必须确认主需求**，不得静默合并
- **保留需求文字中的换行和 Markdown 语义**，不要按空格拆分或当作额外参数
- **超短或语义不完整的文字不得自行补全**，将缺失信息列入待确认问题
- **方案模式执行 Phase 1→4**；已有变更的 UI 制作模式直接读取现有 artifacts，不重跑需求方案生成
- **跨文档一致性检查**：如果需求、设计、接口之间存在矛盾，必须在"待确认问题"中高亮标记
- **信息来源标注**：artifact 中融合多文档内容时，不要混淆来源
- **文档提取失败**：提示文件格式问题，不影响其他已成功提取的文档
- **change name 已存在**：核对并复用明确的已有变更；不得静默覆盖，范围冲突时询问
- **openspec CLI 不可用**：按目标项目已有 schema/模板准备方案草稿并标记未做 CLI 验证；没有可核实模板时只交付分析与缺口，不伪造已通过的 OpenSpec change
- **分析不明确处列为待确认**，不猜测
- 使用 当前环境可用的提问工具（无工具时直接提问） 让用户做关键决策
