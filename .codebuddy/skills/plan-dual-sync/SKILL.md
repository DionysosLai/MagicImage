---
name: plan-dual-sync
description: Use when the user asks to create, update, modify, or refine a plan/计划/规划/方案 document, or when CodeBuddy plan_create/plan_update tools are invoked, or when the user mentions ".artifact" plan files. Triggers on words like "plan/计划/规划/方案/todo list/路线图" combined with create/update/change/sync/同步/更新/改/加.
---

# Plan Dual-Sync (Plan 文档双向同步)

## 概述

CodeBuddy 的 plan 系统会把规划文档写在 AppData 的隐藏路径里，工程目录看不到，无法 git 跟踪。本技能要求 AI 在维护 plan 时，**始终把工程目录下的 `.artifact/<plan_name>.md` 当作"用户面前的真源"**，而把 AppData 路径下的 plan.md 仅作为"系统状态机后端"，每次 plan 变动都要立刻把两者同步。

**核心原则**：用户只看工程内的 `.artifact/<plan_name>.md`，AI 负责保证它和系统真源内容一致。

## 何时使用

**必须触发的场景**：

- 用户说"创建/新建一个 plan / 计划 / 规划 / 方案"
- 用户说"更新/修改/调整/改一下 plan / 计划 / 规划"
- 用户说"plan 加点东西 / plan 加个 todo / plan 改 X"
- 用户说"重新生成 plan / 重做规划"
- 用户对话里提到 `.artifact/*plan*.md` 这类路径
- 调用了 `plan_create` 或 `plan_update` 工具
- 用户说"以工程版为准 / 同步一下 plan"

**自查场景（中度自查）**：

- 用户消息里出现关键词 `plan / 计划 / 规划 / 方案 / 路线图 / todo list`
- 此时进行一次轻量校验：检查工程内 `.artifact/<plan_name>.md` 是否存在、顶部时间戳是否最新；不一致时提醒用户并自动同步

**不要触发**：

- 用户只是聊技术问题，没提 plan / 计划
- 用户在执行已有 todo（plan 状态 building 中），且没要求改 plan

## 关键约束

1. **禁止只改一边**：任何对系统真源（plan_create / plan_update / replace_in_file 改 AppData plan.md）的修改，都必须立刻把整份内容覆盖同步到工程内副本，反之亦然
2. **工程内副本路径固定**：`<workspace_root>/.artifact/<plan_name>.md`（其中 `<plan_name>` 与 plan_create 时的 name 字段一致）
3. **副本顶部必须带同步说明区块**（见下方模板）
4. **副本底部必须带 Todo 镜像表格**（与系统 todolist 完全一致）
5. **同步时一律是"全文覆盖"**，不做局部 diff，避免遗漏

## 同步流程（每次 plan 变动后）

### 步骤 1：定位文件

- 系统真源：调用 plan_create/plan_update 后，从工具返回中读取 plan id，路径为：
  `C:\Users\<user>\AppData\Roaming\CodeBuddy CN\User\globalStorage\tencent-cloud.coding-copilot\plans\<plan_id>\plan.md`
  （macOS / Linux 路径不同，需按实际系统调整；首次不确定时让用户确认）
- 工程内副本：`<workspace_root>/.artifact/<plan_name>.md`

### 步骤 2：读取真源全文

用 read_file 读取系统真源完整内容。

### 步骤 3：组装工程内副本

按下方模板组装：顶部同步说明 → 真源完整正文 → 底部 Todo 表格。

### 步骤 4：写入

用 write_to_file **整文件覆盖**写到工程内副本路径。如果 `.artifact/` 目录不存在会自动创建。

### 步骤 5：简短回报

只汇报"同步完成 + 文件路径 + 关键变更点"，不展开 plan 全文（用户可自己打开看）。

## 工程内副本模板

```markdown
# <Plan 名称（中文）> — 规划文档（工程内主参考版）

> **📌 文档同步说明**
>
> - **本文档为工程内主参考版**：今后查阅、git 跟踪、团队/AI 协作以此为准
> - **同步策略**：plan 任何变更，AI 会先改系统真源、立刻把整份覆盖同步到本文件
> - **系统真源**（CodeBuddy plan 状态机用，不要手动改）：
>   `<完整真源路径>`
> - **Plan ID**: `<plan_id>`
> - **当前状态**: `<ready / building / finished>`
> - **最后同步时间**: `<YYYY-MM-DD HH:MM>`
> - **本次变更**: `<一句话说明本次同步改了什么；首次创建写"首次创建"；纯 todo 状态变更写"todo 状态：xxx → yyy">`

---

<系统真源 plan.md 的完整正文，原样保留>

---

## Todo List（与系统真源同步）

| # | ID | 内容 | 依赖 | 状态 |
|---|----|------|------|------|
| 1 | <todo_id> | <content 摘要> | <依赖列表> | <pending/in_progress/completed/cancelled> |
| ... |
```

## 自查机制（用户提到 plan 关键词时执行）

```
触发条件：用户消息含 plan/计划/规划/方案/路线图/todo list

执行：
  1. 检查 <workspace_root>/.artifact/ 下是否存在 *plan*.md
  2. 不存在 → 提醒用户「工程内还没有 plan 副本，是否同步？」
  3. 存在 → 读取顶部"最后同步时间"
     - 若超过 30 分钟前 或 当前会话有 plan_create/plan_update 操作未同步
       → 提醒用户「工程版可能过期，正在自动同步」并执行同步流程
     - 若新鲜 → 不打扰用户，正常回应
```

## 跨电脑迁移（Skill 已跟工程 git）

新电脑 / 新 IDE 拿到工程后：

1. `git clone` 工程 → `.codebuddy/skills/plan-dual-sync/` 自动带过来
2. CodeBuddy 启动时会自动加载工程内的 skills
3. 第一次让 AI 处理 plan 时，AI 会主动加载本 Skill 并按规则执行

如果换的是**不同操作系统**（如从 Windows 换到 macOS），需要让 AI 重新确认系统真源的 plan.md 路径前缀（macOS 是 `~/Library/Application Support/CodeBuddy CN/...`）。第一次同步时让用户确认即可，AI 会记住。

## 常见错误

| 错误 | 后果 | 正确做法 |
|------|------|----------|
| 只改了 plan_create / plan_update，没同步工程内副本 | 用户看到的版本过期 | 改完真源立刻覆盖工程内副本 |
| 工程内副本用 replace_in_file 局部改 | 容易和真源结构错位 | 一律 write_to_file 全文覆盖 |
| Todo 状态变化没同步表格 | 表格信息陈旧 | 每次 todo_write 后也要重写底部 Todo 表格 |
| 副本路径写错（如放到 docs/ 而不是 .artifact/） | 用户找不到 | 严格遵守 `.artifact/<plan_name>.md` |
| 顶部缺少同步说明区块 | 用户分不清主从 | 模板固定，必带 |
| 缺少"本次变更"字段或写得太笼统（如只写"更新"） | 用户看不出改了啥，要逐字对比新旧版 | 必须用一句话写清核心改动；纯 todo 状态变更写"todo 状态：xxx → yyy"；首次创建写"首次创建" |
| 在 plan mode 的 ready 状态下擅自改 plan | 违反 plan mode 规则 | 只在用户明确要求"更新/修改 plan"时动手 |

## 红线信号 - 停止并修正

- 改了真源没改副本
- 改了副本没改真源
- 副本顶部没有"最后同步时间"
- 副本顶部没有"本次变更"字段，或写得空泛（如只写"更新"/"修改"）
- 副本路径不是 `.artifact/<plan_name>.md`
- 用户问"以哪份为准"，AI 还在解释而不是直接同步

**所有这些都意味着：立刻同步到一致状态，再继续对话。**
