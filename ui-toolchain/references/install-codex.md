# 在 Codex 中使用本工具链（无插件壳，手动四步）

Codex 没有本插件的市场/清单体系，但内容全部可用：skills 与 AGENTS.md 是 Codex 原生认识的约定，命令与评审员各有一行式的替代用法。

## 1. skills（自动触发层）

把三个 skill 目录拷进 Codex 识别的共享 skills 路径：

```bash
mkdir -p ~/.agents/skills
cp -R <插件根>/skills/design-spec ~/.agents/skills/
cp -R <插件根>/skills/frontend-design ~/.agents/skills/
cp -R <插件根>/skills/checklist-design ~/.agents/skills/
```

此后 Codex 会话中提及 UI 任务时，design-spec 规约可被检索与注入（与在 ZCode 中同等的软触发层）。

## 2. 可选：全局硬层（每会话必在的规约指针）

默认不需要——软触发层已覆盖绝大多数场景。若希望规约**每会话必在**（不依赖语义匹配），把下面三行追加到 Codex 原生读取的全局指令文件 `~/.codex/AGENTS.md`：

```markdown
## UI / 前端 / 桌面 GUI 任务
- 任何界面相关工作，先读取并遵循 design-spec skill（~/.agents/skills/design-spec/SKILL.md）与 frontend-design skill。
- UI 任务完成的定义：状态矩阵截图/快照观测 + 七维 rubric 自评 + 确定性检查（对照规约）。
- 评审与实现分离：先出评分报告再改码；评审结论回写负面清单。
```

## 3. 命令（/ui-review 的 Codex 等价物）

Codex 的自定义提示词目录是 `~/.codex/prompts/`：

```bash
cp <插件根>/commands/ui-review.md ~/.codex/prompts/ui-review.md
```

之后在 Codex CLI 中输入 `/ui-review <截图路径或URL>` 即调用。注意：Codex 提示词的参数占位与 Claude 系不同（位置参数 `$1`），若 `$ARGUMENTS` 未被替换，把目标路径直接附在 `/ui-review` 之后即可。

## 4. ui-reviewer（实现/评审分离的替代用法）

Codex 无子代理定义文件。等价做法：**开一个全新会话**，把 `<插件根>/agents/ui-reviewer.md` 全文贴为提示词，再附上待评审截图/URL——新会话无实现上下文，天然满足"评审与实现分离"。

## 5. references（按需 Read）

`references/` 下的 gui-recipes.md（各 GUI 栈截图/对齐/字体配方）、sop-new-project.md、sop-legacy.md、web-interface-guidelines.md 放进项目或任意目录，让 Codex 在对应环节 Read——它们是被动手册，无注册动作。

## 可选：MCP

Codex 通过 `~/.codex/config.toml` 的 `[mcp_servers]` 段配置 MCP。chrome-devtools-mcp 等可选项按主文档的"何时值得装"判断，默认不需要。
