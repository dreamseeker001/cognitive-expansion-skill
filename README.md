# 认知拓展 · cognitive-expansion-zh

面向日常中文对话的 Agent Skill，帮助识别隐含假设、替代解释和重要盲点，减少迎合、锚定与过早收敛，并回到可验证的下一步。

同时支持 **DeepSeek Harness（DSH）** 与 **Codex**。`SKILL.md` 中的〈在 DeepSeek Harness（DSH）中的落地〉一节把技能的四个动作（外部证据、独立复核、澄清、收敛落地）映射到 DSH 的宿主工具，并写明哪些不能当独立证据用。

## 使用

在支持 Agent Skills 的工具中安装后，可显式调用：

```text
$cognitive-expansion-zh
帮我检查这个计划里可能改变判断的假设和遗漏，并提出一个低成本的验证动作。
```

支持四种对话模式：

| 模式 | 用途 |
| --- | --- |
| 轻扫 | 默认模式，补充最有价值的一两点；没有实质发现则直接回答 |
| 深挖 | 比较不同框架、解释和路径，再收敛到行动 |
| 反证 | 公平检验当前方案的失败条件及替代解释 |
| 收敛 | 停止发散，形成建议和下一步 |

例如：“用反证模式检查我的计划”或“现在收敛，给我下一步”。模式词无需独立安装。

## 安装到 DeepSeek Harness（DSH）

DSH 的技能提供者扫描固定的技能根目录，技能是目录包 `<name>/SKILL.md`（或根目录下的 `<name>.md`），只解析 `SKILL.md` 的 frontmatter，正文按需加载。技能根目录会被监听，新增、改名、删除技能**不需要重启**。

| 层级 | 路径 |
| --- | --- |
| 用户级 | `<dshHome>/skills/cognitive-expansion-zh/SKILL.md`（默认 `~/.dsh/skills/`） |
| 项目级 | `<项目根>/.dsh/skills/cognitive-expansion-zh/SKILL.md`（项目根 = 最近的包含 `.git` 的祖先目录，没有则用当前目录） |

Linux / macOS：

```sh
skill_root="${DSH_HOME:-$HOME/.dsh}/skills/cognitive-expansion-zh"
mkdir -p "$skill_root" && cp SKILL.md "$skill_root/"
```

Windows PowerShell：

```powershell
$dshHome = if ($env:DSH_HOME) { $env:DSH_HOME } else { Join-Path $env:USERPROFILE '.dsh' }
$skillRoot = Join-Path $dshHome 'skills\cognitive-expansion-zh'
New-Item -ItemType Directory -Force $skillRoot | Out-Null
Copy-Item .\SKILL.md $skillRoot
```

安装后在下一轮对话中确认技能已出现在会话技能目录里，随后用 `$cognitive-expansion-zh` 或自然语言调用；DSH 的技能目录会监听磁盘变化，无需重启。

DSH 只读 `SKILL.md` 的 frontmatter（`name`、`description`、`whenToUse`）；`agents/openai.yaml` 仅供 Codex 使用，DSH 会忽略它。

> 注意：frontmatter 缺少 `name` 或 `description`、名字不合法，或把调用开关写成 `disableModelInvocation` / `userInvocable` 这类旧式驼峰名，技能都会被**静默跳过**（只留一条宿主警告，模型侧看到的就是「这个技能不存在」）。所以安装后要确认它真的出现在技能目录里。

## 安装到 Codex

将本仓库中的 `SKILL.md` 和 `agents` 文件夹放入：

```text
$CODEX_HOME/skills/cognitive-expansion-zh/
```

未设置 `CODEX_HOME` 时，使用用户目录下的 `.codex/skills/cognitive-expansion-zh/`。已有同名技能时先比较内容，避免覆盖个人修改。安装后在下一轮对话中确认技能已被发现。

界面配置允许自动选择，但是否触发由当前请求和宿主判断；显式写出技能名更明确。简单查询、翻译、格式修改和目标已确定的直接执行不主动触发。

## 设计边界

- 尊重用户目标和已确定的约束，不为反对而反对。
- 区分事实、推测与建议，不把类比或模拟角色当成独立证据。
- 探索未知的重点是外部反馈、失败样本和小实验，不保证发现所有盲点。
- 用户要求收敛或开始执行后停止无收益的发散。
- 不需要 API 密钥、脚本或第三方服务；外部查证取决于宿主可用工具（DSH 上的具体映射见 `SKILL.md`）。
- DSH 上的子代理复核只解决「同一模型的独立上下文」，不算外部证据；缺少查证工具时明确标注未查证。

这里的“防止过拟合”指对话中减少对措辞、偏好与少量案例的过度依赖，不是机器学习训练方法。

## 来源与验证状态

本技能是面向日常对话重新编写的中文指导，设计参考了 [K-Dense Scientific Brainstorming](https://github.com/K-Dense-AI/claude-scientific-skills/tree/main/skills/scientific-brainstorming) 中的假设检查、反证与证据区分思路，并非其逐字翻译或官方版本。

已人工检查内容和基础文件结构，尚未完成独立行为评测；实际效果应通过真实使用持续检验。技能正文见 [SKILL.md](SKILL.md)。

DSH 适配层（frontmatter 的 `whenToUse`、〈在 DeepSeek Harness（DSH）中的落地〉一节、本文件的 DSH 安装章节）基于 DSH 的技能格式与宿主工具编写，同样以实际使用为准。
