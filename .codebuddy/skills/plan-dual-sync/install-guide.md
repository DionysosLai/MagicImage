# plan-dual-sync 安装与使用指南

## 一、当前工程已经装好（无需操作）

文件位置：

```
H:\AI\MagicImage\
└── .codebuddy\
    └── skills\
        └── plan-dual-sync\
            ├── SKILL.md         # 主规则文件
            └── install-guide.md # 本文档
```

CodeBuddy 启动时会自动扫描工作区下的 `.codebuddy/skills/` 目录加载 Skill，无需手动激活。

## 二、换电脑使用（推荐路径：跟 git 走）

### 2.1 前置：把当前工程提交到 git

```powershell
cd H:\AI\MagicImage
git init
git add .codebuddy/skills/plan-dual-sync/
git add .artifact/
git add .gitignore   # 已经规划好排除 workspace/ / .env 等
git commit -m "chore: 添加 plan-dual-sync skill 与 plan 工程内副本"
git remote add origin <你的远程仓库地址>
git push -u origin main
```

### 2.2 新电脑上拉取

```powershell
git clone <仓库地址> H:\AI\MagicImage   # 或别的路径
cd H:\AI\MagicImage
```

### 2.3 启动 CodeBuddy 验证

新电脑上打开工程，第一次让 AI 处理 plan 时观察：

- AI 应该自动遵循 SKILL.md 的规则（顶部带同步说明、底部带 Todo 表格）
- 第一次同步时 AI 会询问系统真源路径（因为不同电脑的用户名/路径不同），按 IDE 提示提供即可

## 三、移植到其他工程

如果想在别的工程也用同一套规则：

```powershell
xcopy /E /I H:\AI\MagicImage\.codebuddy\skills\plan-dual-sync <目标工程>\.codebuddy\skills\plan-dual-sync
```

或者更简单：把 `.codebuddy/skills/` 整个目录复制过去。

## 四、升级到用户级（可选，所有工程通用）

如果以后觉得每个工程都拷贝太麻烦，想让所有工程默认都生效，把 SKILL.md 复制到用户级 skills 目录：

**Windows**：
```
C:\Users\<用户名>\.codebuddy\skills\plan-dual-sync\SKILL.md
```

**macOS / Linux**：
```
~/.codebuddy/skills/plan-dual-sync/SKILL.md
```

注意：用户级 Skill 对所有 CodeBuddy 工程都生效。如果某个工程不想用，在工程根放一个 `.codebuddy/disabled-skills.json` 排除即可（具体语法以 CodeBuddy 文档为准）。

## 五、卸载

不想用本 Skill 时：

- 工程级：删除 `<工程>/.codebuddy/skills/plan-dual-sync/` 目录
- 用户级：删除 `~/.codebuddy/skills/plan-dual-sync/` 目录

删除后 AI 不会再自动按规则同步，但已生成的 `.artifact/*plan*.md` 文件保留，不会被清理。

## 六、Skill 工作机制速查

| 触发动作 | AI 自动行为 |
|---------|-------------|
| 用户说"创建 plan" | 调 plan_create → 同步到 `.artifact/<plan_name>.md` |
| 用户说"更新 plan / 加个 todo" | 改系统真源 → 整文件覆盖工程内副本 |
| 用户消息含"plan/计划/规划"等词 | 自查工程内副本时间戳，过期则提醒并自动同步 |
| 用户问"以哪份为准" | 答："工程内 .artifact/ 下那份" |
| Todo 状态变化（todo_write） | 重写副本底部 Todo 表格 |

## 七、常见问题

**Q: 我手动改了工程内副本，AI 会发现吗？**
A: 不会自动发现。下次 AI 同步时会以系统真源为准覆盖你的手动改动。如果想让 AI 把你手动改的内容反向同步回真源，明确说"把工程版的改动同步回真源"。

**Q: 工程内副本可以多份吗（多个 plan）？**
A: 可以。每个 plan 一份独立的 `.artifact/<plan_name>.md`，文件名由 plan_create 时的 name 字段决定。

**Q: 工程没用 git 怎么办？**
A: 那 Skill 就只在当前电脑当前工程生效，跟 git 无关。换电脑要手动拷贝 `.codebuddy/skills/` 目录过去。

**Q: 想让 AI 不要自查（关掉中度自查）？**
A: 改 `SKILL.md` 里的"自查机制"章节，把触发条件留空或改为"用户明确说'同步'时才执行"。
