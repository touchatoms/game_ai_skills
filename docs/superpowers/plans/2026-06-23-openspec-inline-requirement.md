# OpenSpec Inline Requirement Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Extend `gameai-openspec-from-doc` so direct requirement text follows the same Phase 2–4 workflow as an extracted requirement document.

**Architecture:** Add input normalization at the start of Phase 1. Both file extraction and inline text produce `$req_content`; all downstream analysis and OpenSpec generation continue through the existing shared workflow.

**Tech Stack:** Markdown Skill instructions, OpenSpec CLI, Python 3 for existing `.docx` extraction, Codex Skill validator.

## Global Constraints

- Preserve existing `.docx`, `.md`, `.txt`, `-r`, `-d`, and `-i` document workflows.
- Only requirement input supports inline text; design and interface inputs remain file-only.
- Do not silently interpret a missing file path as requirement content.
- Keep Phase 2–4 behavior and generated OpenSpec artifacts unchanged.

---

### Task 1: Normalize file and inline requirement inputs

**Files:**
- Modify: `gameai-openspec-from-doc/SKILL.md`

**Interfaces:**
- Consumes: the bare argument or `-r` / `--requirement` value supplied to `/opsx:from-doc`
- Produces: `$req_content` plus an input-source label of `需求文档` or `需求文字`

- [ ] **Step 1: Record baseline validation**

Run:

```bash
python3 /Users/pimingzhen/.codex/skills/.system/skill-creator/scripts/quick_validate.py gameai-openspec-from-doc
```

Expected: either `Skill is valid!` or a pre-existing frontmatter compatibility error recorded before editing. No new validation error may be introduced.

- [ ] **Step 2: Expand the trigger description and input contract**

Update the frontmatter description to:

```yaml
description: 输入一段需求文字，或输入一个或多个需求/设计/接口文档，自动归一化输入→分类分析→融合生成 OpenSpec change（proposal/design/tasks/specs），最后进入 explore 讨论。
```

Replace the opening sentence with:

```markdown
输入一段需求文字，或输入一个或多个文档，自动完成：
```

Add these input examples:

```markdown
| `/opsx:from-doc 做一个连续签到活动，连续7天，每天奖励递增` | 直接输入需求文字 |
| `/opsx:from-doc -r "做一个连续签到活动，连续7天，每天奖励递增"` | 使用 `-r` 输入需求文字 |
```

State that `-r` accepts a supported file path or non-empty requirement text, while `-d` and `-i` remain file-only.

- [ ] **Step 3: Replace the no-input prompt and minimum-input guardrail**

Document this interaction:

```markdown
如果未提供任何输入，先询问“请粘贴需求文字，或提供需求文档路径”。用户提供需求后，再按需询问可选的设计文档和接口文档路径。至少需要一段非空需求文字或一个可成功提取的需求文档，否则终止。
```

- [ ] **Step 4: Add Phase 1 requirement input normalization**

Insert this decision flow before document extraction:

```markdown
1. 获取裸参数或 `-r` / `--requirement` 的完整值，不按空格拆分需求正文。
2. 如果该值是存在且扩展名受支持的文件，按文档规则提取并设置 `$req_content`。
3. 如果该值像文件路径但文件不存在，提示路径不存在，并让用户选择重新提供路径或明确改为需求文字；不得静默降级。
4. 其他非空值直接作为 `$req_content`，来源标记为“需求文字”。
5. 纯空白输入视为未提供需求。
```

将“像文件路径”定义为包含路径分隔符，或以 `.docx`、`.md`、`.txt` 结尾。添加以下成功标识：

```text
📝 需求文字: 已接收 <字符数> 字
```

- [ ] **Step 5: Align naming, examples, and guardrails**

Change Phase 3 naming guidance to prefer a requirement title, then use the first sentence or core feature for untitled inline text. Add an inline-text Phase 1 output example. Update guardrails to require requirement text or a requirement document, reject whitespace-only input, preserve Markdown/newlines, avoid guessing incomplete requirements, and require clarification when both a requirement file and extra bare text are supplied.

- [ ] **Step 6: Verify the edited Skill**

Run:

```bash
python3 /Users/pimingzhen/.codex/skills/.system/skill-creator/scripts/quick_validate.py gameai-openspec-from-doc
git diff --check
git diff -- gameai-openspec-from-doc/SKILL.md
```

Expected: validation is no worse than baseline, `git diff --check` prints nothing, and the diff changes only the intended input contract, Phase 1 normalization, naming guidance, examples, and guardrails.

- [ ] **Step 7: Commit the Skill update**

```bash
git add gameai-openspec-from-doc/SKILL.md docs/superpowers/plans/2026-06-23-openspec-inline-requirement.md
git commit -m "优化 OpenSpec 文字需求输入"
```
