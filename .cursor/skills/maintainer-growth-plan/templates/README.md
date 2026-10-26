# Maintainer Journey：{{name}} → {{repo}} core maintainer

这是 {{name}} 成为 {{repo}} core maintainer 的长期学习记录。日期一律用 {{timezone}} 时间。

agent 怎么更新这些文件，见 {{playbook_link}}。

## 文件

| 文件 | 用途 | 多久更新一次 |
|---|---|---|
| [roadmap.md](roadmap.md) | 分阶段路线、证据、每个阶段的信号和要避免的事 | 每月；进入新阶段时 |
| [learning-plan.md](learning-plan.md) | 按周排的计划（每周 {{hours}} h，可以按比例调整） | 每周 |
| [review-schedule.md](review-schedule.md) | 间隔复习队列（间隔 1/3/7/14/30 天） | 每次学习时 |
| [feynman/](feynman/) | 一个概念一篇笔记，每次复习都写一段带日期的解释 | 每次学习或复习时 |
| [knowledge-map.md](knowledge-map.md) | 按区域列出概念和掌握度（0–3），并链接到对应笔记 | 每周 |
| [journal.md](journal.md) | 带日期的日志：做了什么、学到什么、卡在哪、下一步 | 每次学习时 |
| [contributions.md](contributions.md) | issue、PR、review 和设计提案评论，附链接和结果 | 每次有贡献时 |

## 日常流程（每次学习）

1. **先复习。** 打开 `review-schedule.md`，找出"下次复习"日期 ≤ 今天的条目，每条都要不看资料、在 Feynman 笔记里写一段新的解释，再对照来源核对。只重读不算复习。
2. **学习。** 做 `learning-plan.md` 里当天的任务。
3. **记录新概念。** 用 `feynman/_template.md` 建一篇笔记，在 `review-schedule.md` 里加一行（下次复习 = 明天），在 `knowledge-map.md` 里加一行（掌握度 1）。
4. **写日志。** 在 `journal.md` 写 4 行：做了什么、学到什么、卡在哪、下一步。
5. **记贡献。** 有 issue、PR 或 review 时，记到 `contributions.md`。

## 每周（每周最后一次学习时，20 min）

勾掉 `learning-plan.md` 里完成的任务，处理逾期的复习，更新掌握度，在 `journal.md` 写周总结，对照 `roadmap.md` 当前阶段的信号看看进度。

## 每月（月末，30 min）

统计 `contributions.md`；对照 roadmap 判断要不要进入下一阶段；重新估计时间；如果仓库治理有变化，重新核实 roadmap 里的证据。
