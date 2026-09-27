# UI 工具链

面向 AI coding agent 的 UI 质量工具链（ZCode / Claude Code 兼容插件）：设计规约与七维评审 rubric、Anthropic 官方审美指导、129 份权威 UX 审计清单、截图评审命令与独立评审子代理。

[English (README.md)](README.md)

## 安装

**ZCode**：Settings → Plugin Marketplace → Add → Add Plugin Marketplace → 粘贴本仓库地址（`https://github.com/Bryce-Peng/ui-toolchain-marketplace`）→ 在市场中 Install **UI 工具链**。

**Claude Code**：`/plugin marketplace add <你>/<仓库>` → `/plugin install ui-toolchain`。


## 组成

| 组件 | 类型 | 说明 |
|---|---|---|
| `design-spec` | skill | 设计规约：令牌纪律、负面清单、状态完整性、七维 rubric、即时模式 GUI 对齐教训、五栈观测通道决策表 |
| `frontend-design` | skill | Anthropic 官方审美方向指导（anti-slop），vendored |
| `checklist-design` | skill | Checklist Design 129 份权威 UX 清单（audit/critique 两模式，离线可用），vendored，MIT |
| `/ui-review` | command | 对截图/URL/本地页面跑七维 rubric 评审，输出评分表+结构化修改指令 |
| `ui-reviewer` | agent | 独立评审子代理（实现/评审分离用） |
| `references/` | 资源 | vercel 交互细则、两套 SOP（新项目/存量改造）、GUI 观测配方与避坑清单 |

## 使用

- 任何 UI 任务自动触发 design-spec 规约；交付前说"按 rubric 自评"。
- 评审某页面：`/ui-review <截图路径或 URL>`，或派工 ui-reviewer 子代理。
- 存量改造：让 agent 按 `references/sop-legacy.md` 的三步渐进流程走。

## 生成界面文案的语言

跟随你的对话/项目要求——**无模式开关**。硬约束：文案含 CJK 且技术栈为自绘 GUI（egui/Slint/Fyne）时，CJK 字体注入是验收项（配方见 `references/gui-recipes.md`）。

## 许可

- 插件结构：MIT
- `frontend-design`：Apache-2.0（Anthropic），vendored
- `checklist-design`：MIT（Checklist Design）
