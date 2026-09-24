# Issue 跟踪器：GitHub

本仓库的 issue 与 spec 均存放在 GitHub Issues 中。所有操作使用 `gh` CLI。

## 约定

- **创建 issue**：`gh issue create --title "..." --body "..."`。多行正文使用 heredoc。
- **读取 issue**：`gh issue view <number> --comments`，用 `jq` 过滤评论，并同时获取标签。
- **列出 issue**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，按需附加 `--label` 与 `--state` 过滤条件。
- **评论**：`gh issue comment <number> --body "..."`
- **添加 / 移除标签**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **关闭**：`gh issue close <number> --comment "..."`

仓库信息从 `git remote -v` 推断；在 clone 内运行时 `gh` 会自动识别。

## PR 是否作为请求入口

**PR 作为请求入口：否。** _（若本仓库将外部 PR 视为功能请求，把此项改为 `yes`；`/triage` 会读取此标志。）_

设为 `yes` 时，PR 与 issue 走同一套标签与状态，使用 `gh pr` 的对应命令：

- **读取 PR**：`gh pr view <number> --comments`，diff 用 `gh pr diff <number>`。
- **列出待 triage 的外部 PR**：`gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments`，随后仅保留 `authorAssociation` 为 `CONTRIBUTOR`、`FIRST_TIME_CONTRIBUTOR` 或 `NONE` 的条目（剔除 `OWNER`/`MEMBER`/`COLLABORATOR`）。
- **评论 / 打标 / 关闭**：`gh pr comment`、`gh pr edit --add-label`/`--remove-label`、`gh pr close`。

GitHub 的 issue 与 PR 共用同一编号空间，裸写的 `#42` 可能是两者之一：先用 `gh pr view 42`，失败再回退 `gh issue view 42`。

## 当技能说"发布到 issue tracker"

创建一个 GitHub issue。

## 当技能说"取回相关工单"

执行 `gh issue view <number> --comments`。

## Wayfinding 操作

由 `/wayfinder` 使用。**地图（map）**是一条 issue，**子工单**是挂在它下面的 issue。

- **地图**：单条打 `wayfinder:map` 标签的 issue，承载 Notes / Decisions-so-far / Fog 正文。`gh issue create --label wayfinder:map`。
- **子工单**：通过 GitHub sub-issue 机制（对 sub-issues 端点调用 `gh api`）挂到地图下。若未启用 sub-issues，则把子项加入地图正文的任务清单，并在子工单正文开头写 `Part of #<map>`。标签：`wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。一旦被认领，工单指派给主导开发。
- **阻塞**：使用 GitHub **原生 issue 依赖**（规范、UI 可见的表示）。用 `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>` 添加一条边，其中 `<blocker-db-id>` 是阻塞方的数字**数据库 id**（`gh api repos/<owner>/<repo>/issues/<n> --jq .id`，**不是** `#number` 或 `node_id`）。GitHub 报告 `issue_dependencies_summary.blocked_by`（仅未关闭的阻塞方，即实时闸门）。若依赖功能不可用，回退为在子工单正文开头写一行 `Blocked by: #<n>, #<n>`。所有阻塞方关闭后，工单解除阻塞。
- **前沿查询**：列出地图的未关闭子项（`gh issue list --state open`，限定到地图的 sub-issues / 任务清单），剔除存在未关闭阻塞方（`issue_dependencies_summary.blocked_by > 0`，或 `Blocked by` 行中有未关闭 issue）或已有指派人的条目；按地图顺序取第一个。
- **认领**：`gh issue edit <n> --add-assignee @me`——本会话的首次写操作。
- **解决**：`gh issue comment <n> --body "<answer>"`，然后 `gh issue close <n>`，最后把上下文指针（gist + 链接）追加到地图的 Decisions-so-far。
