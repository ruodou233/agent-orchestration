# agent-orchestration

长任务治理：主代理指挥、子代理干活、状态落盘、断点续跑。

## 这是什么 / 解决什么问题

这个 skill 用来处理过夜任务、无人值守任务、批量调研、批量生成、长文本分析和多子代理并行。它把“让 Agent 一直做下去”拆成可执行的流程：先澄清目标，再清点估算，建立 manifest 和 queue，分派工人，回收压缩结果，最后独立验收并交付。核心目标是避免主代理吞全文、上下文爆炸、状态丢失、失败无限重试和跨会话无法接续。它不绑定某个具体平台，首次使用时由你的 Agent 只读探测本地可用平台，再经你同意写入本地映射。

## 核心功能/亮点

- 适配过夜、无人值守、批量扫描、批量生成、长对话分析等高上下文任务。
- 用 `workspace/manifest.jsonl` 和 `workspace/queue.jsonl` 建立稳定 ID 与 append-only 状态流。
- 明确主代理、指挥官、工人、收敛位的职责，降低上下文污染。
- 支持两层或三层编排，并提供 fan-out、失败熔断和 handoff 规则。
- 交付前要求独立验收，汇报压缩为“结论 + 输出路径 + 自检结果”。

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

## 使用示例

- “这是一个过夜任务，帮我批量读完这些资料并明天给结论。”  
  预期行为：先只读清点输入规模，建立工作区状态文件，分派批量阅读任务，回收摘要并生成 handoff。

- “上下文窗口快满了，请整理当前任务并续跑。”  
  预期行为：生成 handoff，列出已完成、进行中、失败方法、状态文件位置和下一步。

- “让主代理负责指挥，子代理去做批量调研，最后统一验收。”  
  预期行为：判断两层或三层编排，建立 manifest/queue，分派工人，交付前做独立验收。

## 首次使用：环境自适应

首次触发时，Agent 默认只读探测已安装的 Agent CLI、支持并行的能力、使用者声明的订阅或额度约束。写入任何本地配置前，必须先说明写到哪里、写什么，并获得明确同意。配置读取顺序为 `~/.config/agentops-skills/agent-orchestration/local-config.md` → skill 目录内 `local-config.md` → 无配置则使用本次只读探测结果。格式参考 `local-config.example.md`。

## Changelog

| 时间 | 变更 |
|---|---|
| 2026-07 | 首次开源发布 |

## 反馈与作者

这个 skill 我长期维护。如果你有修改方案、发现问题、或者改出了更好的版本，欢迎通过以下任一渠道找到我：

- GitHub：本仓库提 issue 或 PR
- 小红书：错误乱码
- 微信公众号：能工智人错误乱码
- B站：若逗道人

## 相关 Skill 推荐

<!-- 本表由维护脚本生成，勿手工编辑 -->
- [cross-review](https://github.com/ruodou233/cross-review)：跨厂商双审：让另一家公司的最强模型独立审你的方案
- [upgrade-audit](https://github.com/ruodou233/upgrade-audit)：升级审计：让 Agent 定期把对话里的知识沉淀进文档体系
- [claude-cache-keepalive](https://github.com/ruodou233/claude-cache-keepalive)：Claude 缓存保温：实测 TTL、按环境设计保温节拍，控制冷读成本
- [connect-computers](https://github.com/ruodou233/connect-computers)：多电脑互联：VPN+SSH+远控+远端 Agent 的分层配置方案

完整目录见 [GitHub 主页](https://github.com/ruodou233)。
