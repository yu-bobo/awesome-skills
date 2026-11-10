---  
name: git-commit  
description: 按仓库 Git 提交规范分析改动并创建 commit。通过 /git-commit 显式触发；用户要求提交代码、写 commit message、创建 git commit 时使用。  
disable-model-invocation: true  
---

# Git 提交

分析当前改动，按规范撰写 message 并创建 commit。仅在用户 `/git-commit` 或明确要求提交时执行。

## 提交信息格式

```
type(scope): [TICKET-ID] 简述

- 详细说明 1
- 详细说明 2
- 影响范围：...
```

### type

| type       | 用途             |
| ---------- | -------------- |
| `feat`     | 新功能            |
| `fix`      | Bug 修复         |
| `refactor` | 重构（无功能变化）      |
| `perf`     | 性能优化           |
| `style`    | 样式 / UI（不影响逻辑） |
| `docs`     | 文档             |
| `chore`    | 构建、依赖、配置等杂项    |
| `test`     | 测试相关           |
| `revert`   | 回滚             |

禁止模糊 type（如 `update`、`add`、`pref`）；性能类用 `perf`。

### scope

取改动所属的子系统、模块或包名。从 `git log --oneline` 对齐项目既有的 scope 命名风格。跨模块改动取主要模块或 `common`。

### TICKET-ID

有工单号时放在冒号后、描述前，如 `fix(dcontrol): EDGE-119 风险模块表单输入框添加前后空格trim`。无工单号可省略，直接写描述。

### 简述

中文，一句话概括「做了什么」，约 50 字内。

### 详细说明（Body）

**必填**，用 `-` 列表，至少覆盖：

1. **做了什么**：具体功能点 / 修复点（可多条）
2. **为什么**（可选）：背景或问题现象
3. **影响范围**：以 `- 影响 ...` 收尾，写明模块、页面或关键路径

一条 commit 只做一类事；无关改动拆多次提交。

### 示例

```
fix(dcontrol): EDGE-119 风险模块表单输入框添加前后空格trim

- 风险对象管理：objectName、majorHazardCode 添加 normalize trim
- 风险单元管理：unitNumber 添加 normalize trim
- 风险点管理：pointNumber、pointName 添加 normalize trim
- 影响 dcontrol 风险模块的新增/编辑表单
```

```
feat(smart-patrol): H5 今日任务对接分页并触底无限加载

- 列表接口改为分页请求，首次加载 + 触底加载更多
- 加载完毕展示无更多数据提示
- 影响智能巡检 H5 今日任务列表页
```

```
style(ledger): 统一台账筛选 FILTER_SPAN 与查询重置按钮

- 各台账页筛选栏 span 统一为 FILTER_SPAN 常量
- 查询/重置按钮对齐到筛选行末尾
- 影响所有台账列表页筛选区域
```

```
# ❌ BAD
fix: 修复bug
feat: 优化页面
EDGE-119: 修复表单空格问题
```

## 执行步骤

按顺序执行；不修改 git config；不 push（除非用户明确要求）。

### 1. 并行摸清状态

```bash
git status
git diff
git diff --staged
git log -8 --oneline
```

用 `git log` 对齐本仓库简述语气与 scope 命名。

### 2. 起草 message

- 根据 **staged + 将要纳入的 unstaged/untracked** 如实概括，不编造未做工作
- 排除密钥与敏感文件（`.env`、`credentials.json` 等）；若用户要求纳入则先警告并跳过
- 若改动混杂多类（feat+fix 无关），先询问是否拆分，或只提交用户指定部分

### 3. 暂存并提交

```bash
git add <相关文件>
```

Windows / PowerShell 下用多行字符串传 message（不要用 Linux HEREDOC）：

```powershell
git commit -m @"
type(scope): [TICKET-ID] 简述

- 详细说明 1
- 详细说明 2
- 影响范围：...
"@
```

提交后：`git status` 确认成功。

### 4. Hook 失败

- 按提示修复后 **新建一次 commit**，不要 `--amend`（除非用户明确要求且满足下方 amend 条件）
- 禁止 `--no-verify` 等跳过 hook（除非用户明确要求）

## 安全约束

- 不更新 git config
- 不 `push --force`、不 hard reset；破坏性操作仅当用户明确要求
- 不 `commit --amend`，除非同时满足：用户要求或 hook 自动改文件需纳入；且 HEAD 由本次对话创建；且未推送到远程
- 无改动可提交时不要空 commit
- 不使用 `git add -i` / `rebase -i` 等需交互的命令

## 注意

- 用户未要求 push 就不要 push
- message 必须与本次 staged 内容一致
- 若存在用户规则 / 项目 rule 与本 skill 冲突，以更严格且与仓库近期 commit 一致的一方为准
