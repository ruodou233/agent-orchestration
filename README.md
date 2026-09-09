# agent-orchestration

复杂任务跑到半夜，你不可能一直盯着。这套编排方案让 Agent 分工跑长任务和过夜任务：批量调研、读长文、做产物，都可以安排好接着干，你只管第二天早上收结果。主代理负责指挥和判断，进度写进任务自己的交付物，换个会话还能接着干。

## 哪些活可以交给它

- **一大批资料要读**：“这是一个过夜任务，帮我读完这些资料，明早给结论。”先清点规模，再分批阅读，把值得你看的结论和证据汇总到一起。
- **一批东西要做**：“按这份要求批量生成，每批做完检查一下，最后统一验收。”把工作拆成可对照的小任务，工人分头做，主代理处理差异和共性问题。
- **复杂调研需要多人分工**：“让子代理分别调查这几条路线，你负责比较和最后判断。”根据任务规模选择两层或三层编排，把上下文分给真正需要它的角色。
- **任务还没完，上下文快满了**：“整理当前任务并续跑。”把已经决定的事、未决问题和下一步写进交付物，留下接续时能读到的关键材料。

你负责说清想收什么结果，Agent 负责安排怎么把这批活做完，不用一直盯着。

## 核心功能/亮点

- 适配过夜、无人值守、批量扫描、批量生成、长对话分析等高上下文任务。
- 目标、进度和取舍写进这份任务自己的交付物；批量对象用稳定 ID 对照。
- 明确主代理、指挥官、工人、收敛位的职责，降低上下文污染。
- 支持两层或三层编排，并提供 fan-out、失败熔断和阶段恢复规则。
- 高风险块与最终交付做独立验收，汇报压缩为“结论 + 输出路径”。

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
- [cross-review](https://github.com/ruodou233/cross-review)：AI 的活总差一点，总要你擦屁股，总打丑补丁？让另一家 AI 挑刺复查，自己把活干完整，不用你一直兜底。
- [upgrade-audit](https://github.com/ruodou233/upgrade-audit)：把你教过 AI 的东西留下来：沉淀偏好、复盘踩坑、更新 skill 和工作流程
- [claude-cache-keepalive](https://github.com/ruodou233/claude-cache-keepalive)：缓存保温：实测命中、算清收益，让长会话少花冤枉 token
- [connect-computers](https://github.com/ruodou233/connect-computers)：把电脑连起来：轻薄本调家中工作机，闲置电脑跑任务，出门也能接着干

完整目录见 [GitHub 主页](https://github.com/ruodou233)。
