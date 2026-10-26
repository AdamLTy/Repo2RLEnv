---
name: maintainer-growth-plan
description: Build an evidence-based personal roadmap and long-term learning journal for becoming a core maintainer of any repository (any host, any language), then keep it up to date with Feynman-style spaced reviews. Use when asked for a "maintainer growth plan for this repo", "how do I become a core maintainer / committer of X", "contributor-to-maintainer roadmap", or to log a study session or run a review for an existing journey. 中文触发：帮我设计成为核心维护者的路线、怎么成为这个仓库的 maintainer / committer、维护者成长计划、开源贡献路线图、记录学习进度、费曼复习。
---

# Maintainer Growth Plan

为一个人和一个仓库生成成为 core maintainer 的路线图，再配一套长期的学习记录文件（journey），之后持续维护。

**输出语言**：跟随用户提问时使用的语言；技术术语、文件名、命令和 label 保留原文。本文件和 `templates/` 是中文写的，写入用户文件时按需翻译。

**核心原则**：
- 每个结论都要能追溯到文件、commit、PR 或 issue。
- 没核实的事写明"未核实"。
- 不猜用户的账号。
- 仓库里没有成文的 maintainer 路径时，直接写"没有正式路径"。

## 1. Intake（一条消息问完；用户没回答的项用默认值）

| 项 | 默认值 |
|---|---|
| 目标仓库（URL 或本地路径） | 当前工作目录 |
| 起点水平（零基础 / 用过 / 贡献过） | 零基础 |
| 每周可投入的小时数 | 6 h，并在 learning plan 顶部注明这是假设 |
| 复习方式 | Feynman 输出式复习：每次复习都要写一段解释，只重读不算 |
| 输出语言 | 用户提问用的语言 |
| 存放位置 | `docs/maintainer-journey/`（不要放进目标仓库本身，除非用户要求） |
| 时区和开始日期 | 先问；没给就用当天，并写明时区 |
| 用户在代码托管平台上的 handle | 可选；没给就不查也不猜 |

## 2. 研究仓库（证据优先）

先从最新的默认分支开始（`git fetch origin <default>`）。如果网络不稳定（API 返回 5xx），重试并改用小分页，不要直接放弃。

### 2.1 治理文件

```bash
git ls-files | grep -iE '(^|/)(CODEOWNERS|OWNERS|MAINTAINERS|COMMITTERS|GOVERNANCE|CONTRIBUTING|CODE_OF_CONDUCT|SECURITY)(\.[a-z]+)?$'
git ls-files | grep -iE '(^|/)(rfcs?|adrs?|decisions|proposals|enhancements|keps?|peps?|design)/' | head
git ls-files | grep -iE '(^|/)(AGENTS|CLAUDE)\.md$|copilot-instructions|\.cursor/rules'
```

除此之外还要读：README 里的 Community / Governance 段落、PR 和 issue 模板、agent 规则文件（里面常写着仓库的硬规则）。

要回答的问题：
- 谁有决定权（BDFL、TSC、steering committee、公司）？
- merge 或 commit 权限怎么授予？
- 有没有成文的 contributor ladder？
- 什么改动需要 RFC 或 ADR？

如果找不到任何关于"如何获得 merge 权限"的文字，就明确写出"没有正式路径"，以及唯一能找到的相关表述。

### 2.2 谁在提交、各自负责哪个区域

```bash
git shortlog -sne --since="6 months ago" --no-merges          # 注意同一个人可能有多个邮箱
git log --since="6 months ago" --no-merges --format='@@%an' --name-only \
 | awk '/^@@/{a=substr($0,3);next} NF{split($0,p,"/"); print a"\t"p[1]"/"p[2]}' \
 | sort | uniq -c | sort -rn | head -60                        # 每个人 × 每个目录
for a in "<name1>" "<name2>"; do git log --author="$a" --reverse --format='%cs' | sed -n '1p;$p'; done   # 第一次和最后一次提交
```

- 把明显是同一个人的多个名字合并；把 bot（dependabot、renovate、各种 `[bot]`）单独列出。
- 用"第一次提交 → 开始 review 或 merge → 写设计文档"的时间线，回答"他们怎么走到今天"。设计文档头部的 Authors 字段也是证据。

### 2.3 谁在 merge 和 review

squash merge 的 commit 显示的提交者常常是平台本身（比如 "GitHub"），所以 merge 人要从平台的 API 查。

- **GitHub**：

  ```bash
  gh pr list -R O/R --state merged --limit 300 --json number,author,mergedBy,mergedAt
  ```

  如果结果为空、报 502 或数据量太大，改用 GraphQL 分页（每页 25–40 条，带重试），并按 `createdAt` 排序：

  ```bash
  gh api graphql -f query='query($c:String){repository(owner:"O",name:"R"){pullRequests(states:MERGED,first:30,after:$c,orderBy:{field:CREATED_AT,direction:DESC}){pageInfo{hasNextPage endCursor} nodes{number createdAt mergedAt author{login} mergedBy{login} reviews(first:10){nodes{author{login} state}}}}}}' -f c=<cursor>
  ```

  注意：app token 可能不支持 `--search "merged:>=…"`，那就拉全量后在本地按日期过滤。
- **GitLab**：`glab mr list --merged`，或调 API `GET /projects/:id/merge_requests?state=merged&updated_after=…`（看 `merged_by` 字段），审批人用 `/approvals` 接口。
- **Gitea / Forgejo / Bitbucket**：用对应的 REST API 查 merged PR 和 reviewer。
- **邮件列表型项目或没有平台访问权限时**：

  ```bash
  git log --since="6 months ago" --format=%b | grep -E '^(Reviewed|Acked|Tested)-by:' | sort | uniq -c | sort -rn | head
  git log --merges --since="6 months ago" --format='%cn' | sort | uniq -c | sort -rn
  git for-each-ref --sort=-creatordate refs/tags --format='%(creatordate:short) %(taggername) %(refname:short)' | head   # 谁在发版
  ```

输出：每个 merge 人合并了多少个 PR；外部贡献者的 PR 是谁合的；人工 review 次数（单独列出 bot reviewer）；按月的变化。

### 2.4 Backlog 和需要帮助的地方

```bash
gh label list -R O/R --limit 100
gh issue list -R O/R --state open --limit 300 --json number,title,labels,assignees,createdAt,updatedAt,comments
gh pr list -R O/R --state open --limit 200 --json number,title,author,createdAt,updatedAt,isDraft,reviewDecision
```

- 统计有多少 issue 打了 label、有没有人真正在用 `good first issue` / `help wanted` 类 label。
- **推荐 issue 之前，先查它有没有关联的 PR 或认领人**（GitHub 用 GraphQL `timelineItems(itemTypes:[CROSS_REFERENCED_EVENT])` 查关联 PR，再看评论里有没有人认领）。已经被认领的 issue 不要推荐，除非对应 PR 已经停滞，并且要写明"先问作者"。
- 找停滞的外部 PR（几个月没动、CHANGES_REQUESTED 后没有更新），找没有 owner 的目录（只有批量改动或 bot 改动、但有 open bug），找 maintainer 的 triage 风格（他们追问什么信息）。

### 2.5 CI、测试和 release

- 列出所有 CI 配置：`.github/workflows/`、`.gitlab-ci.yml`、`.circleci/`、`Jenkinsfile`、`azure-pipelines.yml`。查默认分支最近的运行结果（`gh run list --branch <default> --json workflowName,conclusion`），看哪些 workflow 经常失败。
- 读 CI 的测试命令，找出它**排除**了什么（marker、ignore、skip），然后在本地跑被排除的测试。CI 不跑的测试最容易悄悄过时，这是很好的入门贡献。
- 按 CONTRIBUTING 的步骤**亲自**装环境、跑测试、启动应用。哪条命令失败了、哪个文档过时了，都是可以写进路线图的证据。
- release：看 release 模板、发布脚本、版本号文件，判断流程里有没有写着过时的版本，以及哪些步骤需要特殊权限。

### 2.6 社区渠道和用户账号

```bash
git grep -niE 'discord|slack|zulip|matrix|gitter|mailing list|discourse|forum|office hours|community call|meeting' -- '*.md' '*.rst' '*.txt' | head -30
gh api repos/O/R --jq .has_discussions
gh api user        # 返回 403 或是 bot/app 账号时，说明无法识别用户，不要猜
```

拿到用户的 handle 后，查他已有的贡献：`gh search prs --repo O/R --author <h>`，以及 `--reviewed-by <h>`。

## 3. 写路线图（`roadmap.md`）

结构（每节都要短，链接要完整）：

1. **结论先行**：有没有正式路径；谁在 merge（附数字）；推荐的 3 个起步方向。
2. **治理**：成文规则表（文件 → 要点，包括"缺失"一行）；谁在 review 和 merge（人 → 区域 → 证据）；他们怎么走到今天（按类型归纳）。如果权限和雇主或成员公司强相关，如实写出来。
3. **哪里需要帮助**：已有 PR、等人 review 的 bug；需要先讨论的 bug；没有 owner 的区域；亲自验证过的文档、测试、CI 问题。
4. **社区渠道**和**用户账号**情况。
5. **分阶段路线**（工时按用户的每周小时数换算成周数）：

| 阶段 | 默认总工时 | 内容 |
|---|---|---|
| Phase 0 入门 | 约 12 h | 阅读顺序（每项标重点和时长）、已验证的环境命令、5–7 个动手练习（结果先由 agent 实测）、自检清单（能讲清楚才算） |
| Phase 1 首批贡献 | 约 60 h | 文档或测试修正 → 复现后 review → 一个小的代码修复 → 第一条设计讨论评论 |
| Phase 2 稳定贡献者 | 约 80 h | 选一个缺 owner 的区域；每周固定 review 和 triage |
| Phase 3 area owner / reviewer | 约 150 h | (co)own 一个设计提案；向 maintainer 要 triage 或 reviewer 角色 |
| Phase 4 core maintainer | 不定 | 直接问"授予 merge 权限需要看到什么"，把答案记进 roadmap |

   每个阶段都要写：具体行动（链接到真实的 issue 或 PR）、进入下一阶段的信号、时间范围、要避免的事。
6. **前两周清单**：按日期列出，起点为零基础。
7. **任何阶段都不要做的事**：从仓库的规则文件里提炼，并注明出处。
8. **附录：数据来源和局限**：用了哪些命令、样本窗口、哪些数字可能偏低。

## 4. 建立 journey 文件

1. 在存放位置下建 `feynman/`。复制 `templates/` 里的文件：`feynman-note.md` 复制为 `feynman/_template.md`，其余文件同名复制。填好占位符，删掉用不上的占位行。
2. 在 `README.md` 里写清楚 agent playbook 在哪里：用户要求单独的 playbook 文件时，把本 skill 第 5 节写成那个文件并链接过去；否则直接链接本 skill。
3. 在 `learning-plan.md` 顶部写明每周工时的假设和换算公式。Phase 0 按 session 排（session 数 ≈ 每周小时数 ÷ 2），Phase 1 按周排，Phase 2–4 按月排。如果计划跨越用户当地的节假日，标成减量周。
4. 从 Phase 0 选出 5–7 个核心概念，给前 4–6 个建 Feynman 笔记：写好提示问题和来源文件，**讲解部分留空**（学习者本人写）。其余概念在 `knowledge-map.md` 里标"待建"。
5. 用脚本计算复习日期，填进 `review-schedule.md`（附上前两周的复习日历）：

   ```bash
   python3 -c "
   from datetime import date, timedelta as t
   for n, d in [('slug-a', date(2026,1,6)), ('slug-b', date(2026,1,8))]:
       print(n, [str(d + t(i)) for i in (1, 3, 7, 14, 30)])"
   ```

6. `contributions.md` 的候选表从路线图第 3 节摘录；`journal.md` 写一条"系统建立"记录，列出还没确定的事项。

## 5. 后续维护（agent playbook）

**通用规则**：
- 追加记录，不覆盖历史。
- Feynman 解释必须是学习者本人写的，agent 只提问、核对、记录。
- 贡献记录必须来自用户的描述或平台查询。
- 统一用 intake 时确定的时区。

**A. 用户报告一次学习时**
1. 在 `journal.md` 写一条：做了 / 学到 / 卡点 / 下一步。
2. 在 `learning-plan.md` 勾掉完成的任务，把没完成的挪到下一次。
3. 每个新概念：用模板建笔记，在 `review-schedule.md` 加一行（下次复习 = 学习日期 + 1 天），在 `knowledge-map.md` 加一行（掌握度 1）。
4. 有贡献就记到 `contributions.md`。

**B. Feynman 复习（在对话里做）**
1. 从队列里取出到期的条目，按日期从早到晚，一次最多 3 个。
2. 每个概念：
   1. 请学习者不看资料、给新人讲一遍；
   2. 针对讲解里的空白追问 2–3 个问题（为什么、反例、边界情况、代码在哪个文件）；
   3. 打开来源文件核对，指出错漏，并给出文件路径；
   4. 请他用更简单的话或类比再讲一遍。
3. 在笔记末尾追加一节带日期的复习记录，四个小节都要写。
4. 评分并排下一次：
   - 通过：进入下一个间隔（1/3/7/14/30 天，从学习日期算起）。
   - 部分：重复当前间隔，从今天算起。
   - 未通过：回到 1 天，状态改为 `需重学`。
5. 更新掌握度：第一次通过 → 2；通过 30 天复习，或在真实的 PR/review 里用上 → 3；未通过 → 降一级（最低 1）。

**C. 每周复盘**
- 统计本周完成的任务、复习结果和贡献。
- 处理逾期的复习，把没完成的任务挪到下周。
- 连续两周完成率低于 70% 时，问用户要不要调整每周时间，并按公式重新排周次。
- 在 `journal.md` 写周总结，用一句话对照当前阶段的信号。

**D. 每月复盘和阶段切换**
- 填好贡献的每月统计（PR 状态用平台查询核实）。
- 当前阶段的信号都满足（或用户明确决定）时，切换阶段：更新 roadmap 顶部的"当前阶段"，把新阶段拆成按周的计划，把新概念排进计划。
- 每季度或治理变化时，重新核实 roadmap 的证据部分，并更新"最近核实"日期。

## 6. 验证和汇报

```bash
ls -R <root>
cd <root> && python3 -c "
import re, pathlib
for p in pathlib.Path('.').rglob('*.md'):
    for l in re.findall(r'\]\(([^)#\s]+\.md)\)', p.read_text()):
        if not l.startswith('http') and not (p.parent / l).exists(): print('BROKEN', p, '->', l)"
```

最后向用户汇报：
- 列出所有创建或移动的路径。
- 用 5–8 行总结：谁是 maintainer、有没有正式路径、推荐的 3 个起步方向。
- 列出待用户确认的事项（handle、每周时间等）。
