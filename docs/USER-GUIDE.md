---
doc_id: SKILLS-USER-GUIDE
type: user-guide
status: active
owner: qwenwork
created: 2026-11-23
updated: 2026-11-25
verified_at: 2026-11-23
verified_against: matt-pocock-skills-zh-cn@7326522
---

# Matt Pocock Skills 中文定制发行版 · 使用指南

> **锚点注记（2026-11-26）**：frontmatter 的 `verified_against: 7326522` 是作者重写前的 HEAD。2026-11-26 作者重写（公司邮箱 → GitHub noreply）后该 SHA 仅经 refs/original 可达；重写内容保真已验证（tree 集合相等、逐文件一致），当前 HEAD 为 `4e01df4`。本指南结论不受影响。

> 面向使用这个 Skills 库的人。所有技能数量、名称、安装命令均从当前仓库实测（2026-11-23，HEAD `7326522`）。
> 一句话定位：**Matt Pocock 的 Agent Skills 中文适配 + 定制发行副本**，含翻译、4 处上游回填、兼容性说明与上游同步流程。

---

## 1. 仓库定位

- 英文上游：[`mattpocock/skills`](https://github.com/mattpocock/skills)
- 中文基线：[`vinvcn/mattpocock-skills-zh-CN`](https://github.com/vinvcn/mattpocock-skills-zh-CN)
- 本仓库：[`Vin535LFJ/matt-pocock-skills-zh-cn`](https://github.com/Vin535LFJ/matt-pocock-skills-zh-cn)
- 性质：**中文适配的定制发行副本**，不是上游官方中文版，也不是独立创作。
- 内容：**37 个技能**（`skills/**/SKILL.md` 实测），另有 1 个项目级翻译技能在 `.skills/translate-skill/`（不属于对外发行的 37 个）。
- 它不是知识库，也不是 MCP/Tools 注册中心；它只提供「Agent 工作方法」。

## 2. Skills 分类

仓库用**目录 bucket** 分类（写在 `AGENTS.md:3-9`）：

| bucket | 数量 | 含义 |
|---|---|---|
| `engineering/` | 20 | 工程流程：需求、设计、实现、测试、审查、发布 |
| `productivity/` | 7 | 通用协作：追问、交接、写作、教学 |
| `misc/` | 4 | 配置与迁移类小工具 |
| `in-progress/` | 6 | 未完成/未发布，**不随 plugin 发布**，不在 README 主清单 |
| `deprecated/` | 0 | 已废弃 |

另有一个正交维度：**调用方式**（frontmatter `disable-model-invocation`）：

- **user-invoked（22 个）**：只能由人显式触发，模型不得自动调用。
- **model-invoked（15 个）**：模型可按 description 自动选择。

## 3. 每类 Skills 的用途

- **engineering**：把「想法 → spec → tickets → 实现 → 测试 → 审查 → PR → 复盘」拆成可执行的技能。适合真实代码仓库。
- **productivity**：不写代码也需要的工作方法：追问式设计（grill）、会话交接（handoff）、教学（teach）、为 Agent 写文档。
- **misc**：一次性配置：git 防护、pre-commit、练习脚手架、测试断言迁移。
- **in-progress**：作者仍在打磨，**不建议生产使用**；可读、可试用，但接口可能变。

## 4. 如何查找 Skill

```powershell
# 按关键词搜 description / 正文
rg -n "关键词" skills -g "SKILL.md"

# 列出全部技能目录
Get-ChildItem skills -Recurse -Filter SKILL.md | ForEach-Object { $_.Directory.Name }

# 仓库自带脚本（需要 bash，Git Bash/WSL）
bash scripts/list-skills.sh
```

**入口规则**：先看 `skills/<bucket>/README.md`，再进具体 `SKILL.md`。

## 5. 如何选择 Skill

1. 不知道用哪个：先用人触发 `ask-matt`（它是本仓库的路由器）。
2. 要写代码：`implement` / `implement-spec`（user-invoked）→ 内部会调 `tdd` → `code-review`。
3. 要审代码：`code-review`（model-invoked，说「review this branch」即可）。
4. 要调研：`research`（要求一手来源、落盘带引用）。
5. 要澄清需求：`grill-me` / `grilling` / `grill-with-docs`（后者会产出 ADR 与词汇表）。
6. 要交接：`handoff`（压缩对话）或 `claude-handoff`（交给后台 Agent，in-progress）。
7. 要教学：`teach`（把当前目录当有状态教学工作区）。

**选择原则**：user-invoked 的技能必须由你明确触发；不要让模型「直接读 SKILL.md 代替触发」。

## 6. 如何安装

### 方式 A：skills 安装器（推荐）

```powershell
npx skills@latest add Vin535LFJ/matt-pocock-skills-zh-cn
```

按提示选择 skills 与目标 Agent。首次务必勾选 `setup-matt-pocock-skills`，安装后在 Agent 里运行一次。

> 环境要求：`skills` CLI 的 `engines.node` 为 `>=22.20.0`。本机当前 Node **v20.19.5**，因此这条路径在本机**未实测通过**（见 §15）。

### 方式 B：Claude Code plugin

```
/plugin marketplace add Vin535LFJ/matt-pocock-skills-zh-cn
/plugin install mattpocock-skills@mattpocock
```

`plugin.json` 当前发布 **31 个**技能（engineering 20 + productivity 7 + misc 4），in-progress 6 个不发布。

### 方式 C：手动投放

把 `skills/<bucket>/<skill>/` 整个目录复制到目标宿主的技能目录（例如 `.claude/skills/`、`.agents/skills/`）。这种方法最通用，但需要你自己确认宿主的发现规则。

## 7. 如何验证 Agent 是否发现 Skill

按宿主分别验证，**不要跨宿主推断**：

| 宿主 | 验证方式 | 当前证据 |
|---|---|---|
| Claude Code | `/plugin` 列表出现 `mattpocock-skills`；或看 `.claude/skills/` 是否有目录 | 仓库内无安装产物，**未验证** |
| Codex | 技能目录投放后，让 Agent 列出可用 skills | **未验证** |
| 其他（Cursor/OpenCode/ZCode/VS Code Copilot） | 按各自 skills 目录约定 | **未验证** |
| QwenWork（千问办公桌面版 2.1.0，Windows） | 手动投放到 `%USERPROFILE%\.qwenworkcn\skills\<技能名>\`；查设置里的技能清单；会话内用技能检索与加载工具 | **部分验证**（2026-11-25，`handoff`、`writing-for-agents`）：安装成功，设置清单即时可见；安装后的会话里起初检索与加载失败，会话重建后发现与加载均成功（记录 §8）；全新会话未验证。记录见 [HOST-VERIFICATION-2026-11-25.md](HOST-VERIFICATION-2026-11-25.md) |

安装器层面可以做一次「识别」验证（需 Node ≥22）：

```powershell
npx skills@latest add . --list
```

它会列出可安装技能。**「安装器识别」≠「宿主发现」≠「模型调用」。**

## 8. 如何验证 Agent 是否真正调用 Skill

三个层次，逐层验证：

1. **发现**：让 Agent 说出它看到的技能名（应包含你安装的技能）。
2. **触发**：用 description 里的触发词提问，例如「review this branch」→ 应命中 `code-review`。
3. **执行**：检查 Agent 是否按 SKILL.md 的步骤产出（例如是否先找固定点、是否双轴并行）。

**记录证据**：把会话截图/日志/产出文件存到你自己的验证记录里。仓库内目前没有这些记录，所以所有「已验证调用」的说法都应视为未验证。

## 9. 不同 Agent 环境的兼容性

| 环境 | 文档声称 | 审计判定 |
|---|---|---|
| Claude Code | 官方 plugin + 文件投放 | **未验证**（仓库无 `.claude/` 产物） |
| Codex | skills.sh 安装器支持 | **未验证**（无 `.agents/` 产物） |
| OpenCode / Cursor / VS Code Copilot | 安装器列表点名 | **未验证** |
| ZCode | `.zcode/skills/` | **未验证** |
| WorkBuddy | 需手动投放，历史注册失败 | **未验证**（文档自认未测全流程） |
| QwenWork | 手动投放到 `.qwenworkcn\skills\` | **部分验证**（2026-11-25）：安装与设置清单可见；会话重建后发现与加载成功，执行测试通过（记录 §8）；全新会话未验证。`disable-model-invocation` 与 `agents/openai.yaml` 在该宿主上的效果未验证 |

**硬依赖**（跨技能调用，来自 `AGENT-COMPATIBILITY.md:3.1`）：
`grill-with-docs → grilling + domain-modeling`；`implement → tdd + code-review`；`tdd → codebase-design`；`retro → writing-for-agents`。
如果宿主没有原生 Skill 工具，model-invoked 技能可退化为「读取 SKILL.md 执行」；**user-invoked 的 7 个禁止这样绕过**。

## 10. 如何更新

```powershell
cd C:\Users\Giant\Desktop\GITCODE\AI-Knowledge\matt-pocock-skills-zh-cn
git pull origin main
npx skills@latest add Vin535LFJ/matt-pocock-skills-zh-cn   # 覆盖安装
```

更新后跑 §11 的三条校验，确认没有翻译/结构破坏。

## 11. 如何同步上游

本仓库**不做 Git fork-sync**，采用「内容本地化」流程（`UPSTREAM.md` / `SYNC.md` / `.skills/translate-skill/SKILL.md`）：

```powershell
git fetch upstream-zh upstream-en
git log --oneline <基线SHA>..upstream-zh/main
git log --oneline <基线SHA>..upstream-en/main
```

1. 逐条比对差异，只翻译自然语言，**保留** identifiers、命令、路径、代码块。
2. 重新应用 4 处回填（见 §12）。
3. 校验：

```powershell
node scripts/check-translation.mjs     # 结构/名称/许可证/README 关键串
node scripts/audit-english.mjs         # 英文残留启发式
node -e "const p=require('./.claude-plugin/plugin.json'); console.log(p.skills.length)"
```

4. 可选：`npx skills@latest add . --list`、`claude plugin validate . --strict`（需相应 CLI）。

**当前基线**：英文 `24fe0ef`（v1.3.1）、中文 `bf98e53`；上游英文 HEAD 已前进（未整体同步）。

## 12. 如何维护 Backports

记录文件：`BACKPORTS.md`（当前 **4 处**）。

| # | 技能 | 内容 |
|---|---|---|
| 1 | `engineering/implement` | `/tdd`、`/code-review` 斜杠式改为「调用 Skill 工具」；ticket 标题前置 |
| 2 | `engineering/code-review` | standards 来源扩大；两个 sub-agent 前台并行；tracker doc 措辞泛化 |
| 3 | `engineering/diagnosing-bugs` | 强制造红须用 `diff` 未改动副本自证 |
| 4 | `productivity/grilling` | 问题措辞须让「是」等于接受推荐答案 |

维护规则：
- 回填用 HTML 注释标记最小 diff，便于上游刷新后重新推导。
- 每次同步后跑 `git grep -l "本地回填" -- skills`，应仍为 4 个文件。
- 新增/删除回填必须同步 `BACKPORTS.md`。

## 13. 如何处理翻译问题

1. 术语先查 `TRANSLATION-GLOSSARY.md` / `GLOSSARY.md`，不要自创译名。
2. 只译说明性文字；**不要译**：技能名、目录名、命令、路径、代码、frontmatter `name`。
3. 改完跑：

```powershell
node scripts/check-translation.mjs
node scripts/audit-english.mjs
```

4. `audit-english` 是启发式（单行英文单词 ≥8 个报警），不是硬校验；报警需人工判断。
5. 已知残留：`triage`、`wizard` 仍是英文 H1；`writing-fragments`、`to-questionnaire` 是占位标题。

## 14. 如何贡献新的 Skills

1. 放进正确的 bucket（未完成放 `in-progress/`）。
2. 写 `SKILL.md`：frontmatter 必须有 `name`（ASCII slug）与 `description`（≤1024 字符）。
3. 必须有 `agents/openai.yaml`（与现有 37 个一致）。
4. 更新 bucket README 与（若要发布）`.claude-plugin/plugin.json` 的 skills 数组。
5. 跑全量校验（§11）。
6. 不要在 `in-progress` 未完成时加进 README 主清单或 plugin。

## 15. 已知限制（审计实测）

1. **除 QwenWork 外，宿主兼容性均无仓库内证据**：没有 `.claude/`、`.agents/`、`.zcode/`、`skills-lock.json`、安装日志。QwenWork 有一份部分验证记录（[HOST-VERIFICATION-2026-11-25.md](HOST-VERIFICATION-2026-11-25.md)）：安装成功、设置清单可见，会话重建后发现与加载成功，全新会话未验证。
2. **安装器路径本机未验证**：Node v20.19.5 < `skills` CLI 要求 v22.20。
3. **文档陈旧**：`AGENT-COMPATIBILITY.md` 多处仍写「14 技能」「7 个 user-invoked」，与 37 技能/22 user-invoked 冲突。
4. **错误安装命令**：`skills/in-progress/README.md:12` 仍写 `npx skills@latest add vinvcn/mattpocock-skills-zh-CN`（指向上游，不是本仓库）。
5. **悬空文档**：`docs/engineering/resolving-merge-conflicts.md` 对应技能已删除。
6. **上游未整体同步**：仅 4 处回填；上游新增 `in-progress/chief-of-staff` 未收录。
7. **风格规则违反**：自定「不用 `—`」，实测 17 个 md、109 处。
8. **翻译覆盖不完整**：20 个 SKILL.md 无 H1（正文直入指令，与上游形态一致）；2 个英文标题、2 个占位标题。

---

## 附：完整 Skills 索引（37 个，磁盘实测）

> 兼容状态：`handoff` 与 `writing-for-agents` 在 QwenWork 上为「部分验证」（见 [HOST-VERIFICATION-2026-11-25.md](HOST-VERIFICATION-2026-11-25.md)），其余为「未验证」：仓库内无其他宿主或其他技能的安装/发现/调用证据。安装器可识别是另一层，不等于宿主可用。
> 入口均为仓库相对路径；`(user)` = 只能人工触发，`(model)` = 模型可自动触发。

| # | Skill | 分类 | 用途 | 适用任务 | 兼容状态 | 使用入口 |
|---|---|---|---|---|---|---|
| 1 | ask-matt | engineering / 路由 | 判断当前情境该用哪个技能 | 不确定选哪个技能 | 未验证 | `skills/engineering/ask-matt/SKILL.md` (user) |
| 2 | code-review | engineering / 审查 | 从固定点按 Standards/Spec 双轴审查 | 审 branch/PR/commit | 未验证 | `skills/engineering/code-review/SKILL.md` (model) |
| 3 | codebase-design | engineering / 架构 | 深模块设计的共享词汇 | 设计接口、找 seam | 未验证 | `skills/engineering/codebase-design/SKILL.md` (model) |
| 4 | diagnosing-bugs | engineering / 调试 | 棘手缺陷/性能回退的诊断循环 | debug、broken、slow | 未验证 | `skills/engineering/diagnosing-bugs/SKILL.md` (model) |
| 5 | domain-modeling | engineering / 领域 | 打磨领域模型、写 GLOSSARY/ADR | 统一术语、记录决策 | 未验证 | `skills/engineering/domain-modeling/SKILL.md` (model) |
| 6 | grill-with-docs | engineering / 需求 | 追问式访谈并产出 ADR/词汇表 | 打磨计划且要留文档 | 未验证 | `skills/engineering/grill-with-docs/SKILL.md` (user) |
| 7 | implement | engineering / 实施 | 按 spec/ticket 实施，驱动 tdd+review | 开始写功能 | 未验证 | `skills/engineering/implement/SKILL.md` (user) |
| 8 | implement-spec | engineering / 实施 | 单集成分支实现整份 spec（并行子代理） | 大 spec 一次落地 | 未验证 | `skills/engineering/implement-spec/SKILL.md` (user) |
| 9 | improve-codebase-architecture | engineering / 架构 | 扫描深化机会并生成 HTML 报告 | 架构体检 | 未验证 | `skills/engineering/improve-codebase-architecture/SKILL.md` (user) |
| 10 | pr | engineering / 写作 | 写 PR 正文：摘要+前后证据+风险 | 起草/改写 PR | 未验证 | `skills/engineering/pr/SKILL.md` (model) |
| 11 | prototype | engineering / 原型 | 一次性原型回答设计问题 | 验证 UI/状态模型 | 未验证 | `skills/engineering/prototype/SKILL.md` (model) |
| 12 | research | engineering / 研究 | 对照一手来源调研并落盘带引用 | 收集 API 事实/文档 | 未验证 | `skills/engineering/research/SKILL.md` (model) |
| 13 | retro | engineering / 复盘 | 对一次 coding session 复盘 | 改进环境/流程 | 未验证 | `skills/engineering/retro/SKILL.md` (user) |
| 14 | setup-matt-pocock-skills | engineering / 配置 | 首次配置 tracker/labels/docs 布局 | 新仓库初始化 | 未验证 | `skills/engineering/setup-matt-pocock-skills/SKILL.md` (user) |
| 15 | tdd | engineering / 测试 | red-green-refactor 垂直切片 | 先写测试 | 未验证 | `skills/engineering/tdd/SKILL.md` (model) |
| 16 | to-spec | engineering / 文档 | 把对话综合成 spec 并发到 tracker | 冻结需求 | 未验证 | `skills/engineering/to-spec/SKILL.md` (user) |
| 17 | to-tickets | engineering / 项目 | 拆 tracer-bullet tickets + blocking edges | 大工作拆分 | 未验证 | `skills/engineering/to-tickets/SKILL.md` (user) |
| 18 | triage | engineering / 项目 | issues/PR 分类、验证、写 brief | 处理 incoming | 未验证 | `skills/engineering/triage/SKILL.md` (user) |
| 19 | wayfinder | engineering / 规划 | 把超大工作规划成 decision tickets 地图 | 方向不清的大项目 | 未验证 | `skills/engineering/wayfinder/SKILL.md` (user) |
| 20 | wizard | engineering / 运维 | 生成交互式 bash wizard 引导人工步骤 | 配 secrets/迁移 | 未验证 | `skills/engineering/wizard/SKILL.md` (model) |
| 21 | claude-handoff | in-progress / 协作 | 把对话交给后台 Agent 接手 | 长任务转交 | 未验证（未发布） | `skills/in-progress/claude-handoff/SKILL.md` (user) |
| 22 | loop-me | in-progress / 需求 | 就待建工作流规格反复访谈 | 规格打磨 | 未验证（未发布） | `skills/in-progress/loop-me/SKILL.md` (user) |
| 23 | setup-ts-deep-modules | in-progress / 架构 | 接入 dependency-cruiser 做深模块 | TS 仓库架构 | 未验证（未发布） | `skills/in-progress/setup-ts-deep-modules/SKILL.md` (user) |
| 24 | writing-beats | in-progress / 写作 | 把素材组装成节拍式文章 | 长文写作 | 未验证（未发布） | `skills/in-progress/writing-beats/SKILL.md` (user) |
| 25 | writing-fragments | in-progress / 写作 | 先挖原始片段，不加结构 | 素材收集 | 未验证（未发布） | `skills/in-progress/writing-fragments/SKILL.md` (user) |
| 26 | writing-shape | in-progress / 写作 | 逐段把素材塑造成文章 | 成稿 | 未验证（未发布） | `skills/in-progress/writing-shape/SKILL.md` (user) |
| 27 | git-guardrails-claude-code | misc / 安全 | Claude Code hooks 拦截危险 git | 防 push/reset | 未验证 | `skills/misc/git-guardrails-claude-code/SKILL.md` (model) |
| 28 | migrate-to-shoehorn | misc / 测试 | `as` 断言迁移到 shoehorn | 测试类型清理 | 未验证 | `skills/misc/migrate-to-shoehorn/SKILL.md` (model) |
| 29 | scaffold-exercises | misc / 教学 | 生成练习目录结构并过 lint | 课程练习 | 未验证 | `skills/misc/scaffold-exercises/SKILL.md` (model) |
| 30 | setup-pre-commit | misc / 配置 | Husky + lint-staged + 类型/测试 | 提交前检查 | 未验证 | `skills/misc/setup-pre-commit/SKILL.md` (model) |
| 31 | grill-me | productivity / 需求 | 持续追问打磨计划/设计 | 方案压力测试 | 未验证 | `skills/productivity/grill-me/SKILL.md` (user) |
| 32 | grilling | productivity / 需求 | 追问 primitive，逐分支收敛 | 设计树探索 | 未验证 | `skills/productivity/grilling/SKILL.md` (model) |
| 33 | handoff | productivity / 协作 | 把对话压缩成交接文档 | 换会话/换 Agent | QwenWork 部分验证：安装成功，会话重建后发现与加载成功，全新会话未验证 | `skills/productivity/handoff/SKILL.md` (user) |
| 34 | teach | productivity / 教学 | 把当前目录当有状态教学工作区 | 跨 session 学习 | 未验证 | `skills/productivity/teach/SKILL.md` (user) |
| 35 | to-questionnaire | productivity / 协作 | 把答不了的决策转成问卷 | 需要他人输入 | 未验证 | `skills/productivity/to-questionnaire/SKILL.md` (user) |
| 36 | wait-what | productivity / 协作 | 「刚才没讲清，重述一遍」 | 让 Agent 复述 | 未验证 | `skills/productivity/wait-what/SKILL.md` (user) |
| 37 | writing-for-agents | productivity / 文档 | 为 Agent 写文档（skills/AGENTS.md） | 写技能/规则 | QwenWork 部分验证：安装成功，会话重建后发现与加载成功，全新会话未验证 | `skills/productivity/writing-for-agents/SKILL.md` (model) |

> 计数校验：37 = engineering 20 + productivity 7 + misc 4 + in-progress 6；deprecated 0。
> `.skills/translate-skill/` 是维护者自用翻译技能，不在上表。

---

**最后验证**：2026-11-23，Skills `7326522`（工作树干净；`check-translation` / `audit-english` 均 exit 0）。
**最近更新**：2026-11-23。