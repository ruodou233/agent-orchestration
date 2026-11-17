# 多智能体任务编排｜Multi-Agent Orchestration

Coordinate AI agents for long-running tasks, parallel work, and overnight workflows.

**为强模型做减法**：模型越强，需要的束缚和 skill 越少，规则写多了反而挤掉它自己的判断。2026-11 这一版删掉了大半条文和配套机制，只留方向和判断依据。

复杂任务跑到半夜，你不可能一直盯着。这套编排方案让 Agent 分工跑长任务和过夜任务：批量调研、读长文、做产物，都可以安排好接着干，你只管第二天早上收结果。主代理负责指挥和判断，进度写进任务自己的交付物，换个会话还能接着干。

## 哪些活可以交给它

- **一大批资料要读**：“这是一个过夜任务，帮我读完这些资料，明早给结论。”先清点规模，再分批阅读，把值得你看的结论和证据汇总到一起。
- **一批东西要做**：“按这份要求批量生成，每批做完检查一下，最后统一验收。”把工作拆成可对照的小任务，工人分头做，主代理处理差异和共性问题。
- **复杂调研需要多人分工**：“让子代理分别调查这几条路线，你负责比较和最后判断。”子代理只回结论和证据路径，主代理的上下文留给比较和判断。
- **任务要换个会话接着做**：“整理当前任务并续跑。”把已经决定的事、未决问题和下一步写进交付物，留下接续时能读到的关键材料。

你负责说清想收什么结果，Agent 负责安排怎么把这批活做完，不用一直盯着。

## 为什么默认两层

我们把一个月的 Codex 会话（392 个日志文件、15 个本机案例）拆开看了看：本机没有一例真三层成功，成败分界是分批回收与产物落盘；翻了 15 个外部来源，没有任何一个给出三层优于两层的数字。[完整研究与数据](research/2026-09-delegation-study.md)

## 核心功能/亮点

- 适配过夜、无人值守、批量扫描、批量生成、长对话分析等高上下文任务。
- 目标、进度和取舍写进这份任务自己的交付物；批量对象用稳定 ID 对照。
- 主代理负责口径、裁决和验收，工人只回“结论 + 输出路径”；只读的检索抓取交便宜档，要判断的留强模型。
- 默认两层；Claude Code 与 Codex 的长任务优先用平台自带的 dynamic workflow、`/goal` 和定时任务。
- 错误影响大、不可逆时，让没参与执行的全新上下文独立验收。

## 安装

Claude Code：

```bash
git clone https://github.com/ruodou233/agent-orchestration.git ~/.claude/skills/agent-orchestration
```

Codex：

```bash
git clone https://github.com/ruodou233/agent-orchestration.git ~/.agents/skills/agent-orchestration
```

其他支持 SKILL.md 的平台：放入其 skills 目录即可。

## 首次使用：环境自适应

首次触发时，Agent 默认只读探测已安装的 Agent CLI、支持并行的能力、使用者声明的订阅或额度约束。写入任何本地配置前，必须先说明写到哪里、写什么，并获得明确同意。配置读取顺序为 `~/.config/agentops-skills/agent-orchestration/local-config.md` → skill 目录内 `local-config.md` → 无配置则使用本次只读探测结果。格式参考 `local-config.example.md`。

## Changelog

| 时间 | 变更 |
|---|---|
| 2026-11-17 | 按作者现行版本重写：去掉三层指挥官、跨平台默认派工、模型钉版与缓存保温等已退役内容；补平台自带长任务机制、只读采集降档、Codex 工人默认不继承整段历史 |
| 2026-09-14 | 默认两层、委派按条件判断、状态落盘为硬要求；附[一手数据研究](research/2026-09-delegation-study.md) |
| 2026-08-26 | 进度写进任务交付物；中间块按风险验收 |
| 2026-07 | 首次开源发布 |

## 反馈与作者

这个 skill 我长期维护。如果你有修改方案、发现问题、或者改出了更好的版本，欢迎通过以下任一渠道找到我：

- GitHub：本仓库提 issue 或 PR
- 小红书：错误乱码
- 微信公众号：能工智人错误乱码
- B站：若逗道人

## 相关 Skill 推荐

<!-- 本表由维护脚本生成，勿手工编辑 -->
- [cross-review](https://github.com/ruodou233/cross-review)：AI 的活总差一点，总要你擦屁股，总打丑补丁？让另一家 AI 挑刺复查，自己把活干完整，不用你一直兜底。<br>An agent skill for independent code and design reviews across AI providers, checking correctness, complexity, and better approaches.
- [upgrade-audit](https://github.com/ruodou233/upgrade-audit)：把你教过 AI 的东西留下来：沉淀偏好、复盘踩坑、更新 skill 和工作流程<br>Review conversation history, agent memory, and skills to identify reusable lessons and propose updates to outdated instructions.
- [claude-cache-keepalive](https://github.com/ruodou233/claude-cache-keepalive)：缓存保温：实测命中、算清收益，让长会话少花冤枉 token<br>Measure prompt cache hits and costs, then configure automatic keepalive when the savings justify it.
- [connect-computers](https://github.com/ruodou233/connect-computers)：把电脑连起来：轻薄本调家中工作机，闲置电脑跑任务，出门也能接着干<br>Set up SSH, VPN, remote desktop access, and remote AI agents across your computers for work at home and on the go.

完整目录见 [GitHub 主页](https://github.com/ruodou233)。
