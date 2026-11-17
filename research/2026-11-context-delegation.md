# 主代理上下文与子代理分工：时间线，以及和大厂做法的对照

**副题：从 2026 年 5 月的过夜任务 skill 到 11 月的一轮删减**

## 一句话

保护主代理的上下文干净、全局思路清晰，不光效果和体验更好，也省下大量返工。一个月的一手数据里，返工是首次交付的 6～52 倍，子代理的总消耗还不到主会话的六成；数据证明，比起子代理的 token 消耗，主代理上下文污染、反复返工消耗的 token 才最严重。

## 时间线

| 时间 | 做了什么 | 证据 |
|---|---|---|
| 2026-05-19 | 第一版过夜任务 skill（overnight-task）：主 Agent 只和用户沟通、写清目标；子 Agent 异步执行，输出压缩后回收；状态写文件，断点续跑 | 下方摘录（当时未公开） |
| 2026-05 下旬 | 日常任务里开始让子代理做图片转写、批量出图、多来源分头调研，出发点就是不让过程输出占满主会话 | 本机会话记录 |
| 2026-06-02 | 每日升级审计 skill 第一版：调度器先走，多个扫描子代理并行读对话，主代理只负责路由和汇总 | 私有仓首个提交 |
| 2026-06-08 | 抽成独立的长任务编排规范，也就是本仓的前身 | 私有仓 |
| 2026-07-08 | 随首批 skill 开源 | [本仓首发版](https://github.com/ruodou233/agent-orchestration/blob/6933c37/SKILL.md)、[upgrade-audit 首发版](https://github.com/ruodou233/upgrade-audit/blob/5432694/SKILL.md) |
| 2026-09 | 拆一个月 Codex 会话做一手数据研究 | [委派不省钱，返工才烧钱](2026-09-delegation-study.md) |
| 2026-11 | 对照三家公开做法和本机 30 天数据，吸收一部分，删掉大半规则 | 本文 |

2026-05-19 那一版的开头（原文摘录）：

```text
用户 ←→ 主Agent（与用户沟通、创建 task_goal.md、启动任务）
              ↓
         指挥官Agent（拆解任务、发 JSON 指令、决策与交接）
              ↓
         编排器 Python（状态机、日志、队列、执行、回收、熔断）
              ↓
         子Agent / shell / tool / 插件脚本
```

> 基础层负责：任务目标、计划、日志、交接、熔断；子 Agent 异步执行与结果回收；超时、连续失败、输出压缩、崩溃恢复。

那时为了弥补模型能力，加了指挥官、Python 编排器和一整套状态文件。后来模型变强，这些大多删掉了，留下来的是同一个核心：主代理管目标和判断，执行和原文交出去，只收结论。

## 三家分享了什么

| 来源 | 日期 | 要点 |
|---|---|---|
| DeepSeek Harness：[等子代理时结束本轮](https://github.com/deepseek-ai/deepseek-harness/blob/master/.agents/notes/implemented/bug-fix/2026-10-21-subagent-end-turn-guidance.md) | 2026-10-21 | 受控 A/B：写明「只剩等结果时结束本轮」后，空等的 sleep 从 29/300 降到 0/300 |
| Anthropic：[When to fold subagents into your main agent loop](https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop) | 2026-10-22 | 按上下文依赖切分，不按角色切分；切出去的活应几乎不需要其余信息、能交回紧凑结果 |
| DeepSeek Harness：Super-agent 设计（[原文](https://mp.weixin.qq.com/s/zoS7GlReWPhfCjENGxXCdg)、[原型仓](https://github.com/deepseek-ai/dsh-superagent-prototype)） | 2026-10-31 | 主会话只路由、工人隔离上下文执行；转发用户原话，概括只作导读 |
| Anthropic：[Which Claude model should you build with?](https://claude.dev/blog/which-claude-model-should-you-build-with/) | 2026-11-09 | 强模型领队、便宜模型并行读取，在一个约 2,200 万 token 的检索基准上得分相当、成本少约三分之一 |
| Google：[Agent Distraction](https://developers.googleblog.com/agent-distraction/) | 2026-11-06 | Google 内部数百万次会话：代理读到过时的 TODO 会被带偏，相关会话步数多约 70%、token 多约 77% |
| Google：[Gemini 4 Hydrogen in Antigravity](https://antigravity.google/blog/gemini-4-hydrogen-in-google-antigravity) | 2026-11-12 | 强模型规划、快模型执行、卡住交回 |
| Google Research：[Antigravity 的 /teamwork](https://research.google/blog/advancing-science-engineering-with-antigravitys-teamwork-feature/) | 2026-11-13 | 多日任务里注意力会随上下文窗口变长而分散，所以每个专门代理只拿它需要的文件和工具；协调者在里程碑之间写交接文件，换全新继任者接着做 |
| Anthropic：[Claude Code best practices](https://code.claude.com/docs/en/best-practices)「What you can stop doing」 | 2026-11 当前版 | 「用子代理保护上下文窗口」已不必要；子代理的价值在并行、挡住冗长输出、要一个没看过推理过程的审查者 |

## 对照

**不谋而合的**：主代理只管口径、裁决和验收；子代理读原文，只回结论和证据路径；状态写进任务自己的交付物；任务书自包含；独立验收用没参与执行的全新上下文。这些在 7 月首发版里都已经有了。

**从三家学到、这次吸收的**：

- 委派的理由从「保护上下文窗口」改成「主会话用最贵的模型，留在里面的过程输出每轮都要重发，也分散判断」。窗口变成 1M 之后，前一个理由站不住，后一个才是实情。
- 只读的检索、抓取、统计交给便宜一档，要对照判断或出改稿的仍用强模型。
- 长任务优先用平台自带的 dynamic workflow、`/goal` 和定时任务，不再手搓队列。
- Codex 原生工人默认只给任务书，不整段继承父线程历史。

**没采纳的**：主代理用轻模型、不许给工人写验收标准（DeepSeek 原型的前提是主代理不看材料、不验收，和我们相反）；压缩后再自检一轮（属于输出前自检，我们一直在删这类规则）。

**这次删掉的**：三层指挥官、模型钉版、缓存保温、失败熔断、新会话恢复包等，正文从 22KB 减到 11KB。模型越强，需要的束缚越少。

## 数据

9 月研究归因的 39 条返工里，近半（49%）是「设计与执行未拆开」：规划和执行堆在同一个主会话里互相干扰，这正是主代理上下文被污染的表现。

本机 30 天（2026-10-18～11-17，Claude Code 63 个会话）：

- 自动压缩 0 次，压缩后也没有「你忘了」一类的纠正：主会话跑在约 1M 窗口上，窗口本身不是瓶颈。
- 子代理回传中位 1.4k 字，没有一条把大段原文灌回主会话：「只回结论」已经落地。
- 子代理的 token 是主会话的 2.2 倍，84% 跑在最强模型上：「便宜档执行」没兑现，所以补了只读采集降档。
- 20 个会话要我开口才派子代理：旧理由被平台内置的「拿不准就别派」压过，所以换了理由。
- 每个会话开局约 85k token 是工具定义，我们自己写的常驻文档只占约 1%：最大的杠杆在打开工具按需加载，而不是继续精简文档。

## 收获

方向上，我们 5 月摸出来的做法和三家公开的思路不谋而合；细节上，从他们那里学到不少，比如理由该怎么说、哪些活可以降档、哪些机制平台已经替你做好了。最大的收获是一次大删减：模型变强之后，早期为弥补能力加的规则大多成了负担。
