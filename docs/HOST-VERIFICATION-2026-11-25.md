---
doc_id: SKILLS-HOST-VERIFICATION-20261125
type: review-report
status: active
owner: qwenwork
created: 2026-11-25
updated: 2026-11-25
---

# 宿主实机验证记录：QwenWork（2026-11-25）

> 目的：证明本仓库的技能不只是文件存在，而是在真实 Agent 环境里能安装、能被发现、能被调用。
> 结论先说：**安装成功，宿主设置清单能立即识别；但在安装它的那个会话里，技能检索和技能加载都失败；新会话里的表现尚未验证。** 本记录不把"读取 SKILL.md 照做"算作宿主调用成功。

## 1. 测试环境

| 项 | 值 |
|---|---|
| 宿主 | QwenWork 桌面版（千问办公），版本 2.1.0，Windows x64 |
| 模型 | 未记录：宿主未向会话提供可记录的模型标识 |
| 仓库版本 | `matt-pocock-skills-zh-cn` 提交 `4604bd6`；`.claude-plugin/plugin.json` 版本 1.3.1 |
| 技能安装位置 | `C:\Users\Giant\.qwenworkcn\skills\<技能名>\SKILL.md`（用户级技能目录，按文件夹名识别） |
| 技能格式 | 一个文件夹加 `SKILL.md`（frontmatter 含 `name`、`description`）；`.skill-metadata.yaml` 附属文件是可选的（同目录下多数已装技能没有） |
| 安装方式 | USER-GUIDE §6 方式 C：手动投放整个技能目录 |

本仓库提供的另外两种安装方式（skills 安装器、Claude Code plugin）面向其他 Agent，不适用于该宿主。

## 2. 选取的技能

| 技能 | 类型 | 选择理由 |
|---|---|---|
| `handoff` | user-invoked（frontmatter `disable-model-invocation: true`），只能由人点名调用 | 把当前对话压缩成交接文档，是使用者真实的日常工作 |
| `writing-for-agents` | model-invoked，Agent 可按任务调用 | 修改 `AGENTS.md` 时适用，有真实任务可做 |

## 3. 结果

| 测试 | 结果 | 证据 |
|---|---|---|
| A 安装 | **成功** | `handoff` 2 个文件、`writing-for-agents` 3 个文件；安装后逐文件 SHA256 与仓库源文件一致 |
| B 发现（宿主设置清单） | **成功** | 安装后立即查询宿主技能清单，两个技能均出现，状态为启用，用户技能数 18 变 20；无需重启 |
| B 发现（会话内技能检索） | **失败** | 按名称检索 `handoff`：只返回另一个技能，`hasMore:false`，不含目标。按名称检索 `writing-for-agents`：翻完 3 页共 15 个候选，不含目标。按任务关键词检索"写交接文档让另一个代理接手"：翻完 3 页共 15 个候选，不含 `handoff` |
| C 调用（宿主技能加载） | **失败** | 两次加载均返回原文：`Skill not found. Use skill_search to discover an exact permitted name.` |
| D 执行结果 | **部分** | 见下 |

**子代理复核**：安装之后才启动的一个子代理重复了 B、C 两步，结果相同（检索不到、加载失败），并确认两个目录与 `SKILL.md` 存在。说明不是单个会话的偶发问题：技能目录在会话开始时固定。

**D 的处理方式（如实区分）**：

- `writing-for-agents` 属于 model-invoked。USER-GUIDE §9 允许这类技能在宿主没有原生技能加载时退化为"读取 SKILL.md 执行"。按此做法读取已安装位置的正文，并据其"context pointer"写法完成一处真实修改（Engine 仓库根 `AGENTS.md` 补一行技能指针，提交 `4951d99`）。**这只证明技能内容可用，不证明宿主调用成功。**
- `handoff` 属于 user-invoked。USER-GUIDE §9 明确禁止对这类技能用读取方式绕过验证，因此判为 **NOT VERIFIED**。

## 4. 兼容性限制（本次观察到的）

1. **技能目录在会话开始时加载**：安装后，当前会话（以及其后启动的子代理）检索与加载都找不到新技能；宿主设置清单却是实时的。两者不一致，容易让人以为"设置里看到了就能用"。
2. **`disable-model-invocation` 标记在本宿主上的效果未验证**：因为技能根本发现不了，无从验证宿主是否尊重这个标记。
3. **`agents/openai.yaml` 是否被读取未知**：文件随目录一并装入，宿主未报告任何相关行为。
4. **检索排序**：两次任务关键词检索的前几页都被描述很长的已装技能占满（样本只有 2 次查询，不下普遍结论）。

## 5. 没有验证的

- 新会话里能否发现并调用（本会话无法自行重开）。
- 其他 35 个技能（未安装、未测）。
- 其他宿主（Claude Code、Codex 等）仍未验证。
- `handoff` 的执行结果。

## 6. 下一步（需要使用者在 QwenWork 里做）

新开一个会话，说："用 handoff 技能给当前对话写一份交接文档。" 观察：技能检索是否返回 `handoff`；技能加载是否成功；产出是否是一份交接文档且含"suggested skills"一节、存放在系统临时目录而不是工作区。把结果补到本文件或 USER-GUIDE §7 的证据列。

移除这两个技能：在 QwenWork 设置的技能列表里停用或移除，或删除 `C:\Users\Giant\.qwenworkcn\skills\handoff` 与 `...\writing-for-agents` 两个目录。

## 7. 证据来源

- 安装与完整性：脚本输出（源文件数、安装后文件数、逐文件 SHA256 一致）。
- 宿主技能清单：`qwenwork.settings.skills` 查询结果（含两个新技能，`source: user`，`disabled: false`）。
- 检索与加载：技能检索工具与技能加载工具的原始返回，主会话与子代理各一次。
- `writing-for-agents` 的真实修改：Engine 仓库提交 `4951d99`。
