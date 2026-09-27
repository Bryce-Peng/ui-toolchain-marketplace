---
name: design-spec
description: "Unified design spec & review standard for UI/web/desktop-GUI work (token discipline, negative list, state completeness, 7-dimension rubric, immediate-mode alignment recipes, per-stack observation channels). UI/前端/桌面 GUI 任务必读：令牌纪律、负面清单、四态、七维 rubric 与各栈观测通道；构建或重塑界面前先读，交付前按 rubric 自评。"
---

# UI 设计规约与评审标准（design-spec）

适用于所有 UI/前端/GUI 任务。核心原则：**能用确定性工具判定的（颜色值、对比度、间距、组件来源）不用眼睛；眼睛只负责确定性工具管不了的审美与整体感。**

## 一、令牌纪律（生成前约束）

- 间距只用 8pt 刻度：4 / 8 / 12 / 16 / 24 / 32 / 48 / 64 / 96，禁止 magic number。
- 字号阶梯：12 / 14 / 16 / 20 / 24 / 32 / 48，正文 13-16，层级靠字重与字号双重区分。
- 配色：中性灰阶 + **至多一个**强调色；hover/active 只调明度不改色相；语义色（成功/警告/危险/中性信息）除外。
- 圆角同族（嵌套时子 ≤ 父）；阴影至多两档（环境光+直射光，色相偏向 ink）。
- 数字对比场景用 tabular-nums（web 用 `font-variant-numeric`；GUI 工具包用等宽数字/右对齐替代）。

## 二、负面清单（出现即不合格）

- 紫蓝渐变横幅/按钮；emoji 当图标；无意义的 ALL-CAPS 眉题。
- 同一圆角+同一灰阴影切所有卡片（"SaaS 卡片套餐"）。
- 裸 `#000`/`#FFF`（用带色相的近黑近白）。
- 无 hover/focus/disabled 态的可交互元素；`outline: none` 无替代焦点样式。
- `transition: all`；动画布局属性（top/left/width/height）；`transition` 必须显式列属性。
- 颜色作为唯一状态标识（必须冗余：圆点/图标/文字）。

## 三、状态完整性

每个数据视图必须有 **loading / empty / error / success** 四态；操作必须有即时反馈（optimistic UI 或明确 pending）；破坏性操作需确认**或**提供撤销窗口（二选一，硬删除禁止）。

## 四、交互细则

MUST 级条目以 Checklist-Design 与 vercel web-interface-guidelines 为准（要点：键盘完整支持、focus 环可见、命中目标 ≥24px、URL 反映状态、Enter 提交单行输入、错误内联展示并聚焦首错、`prefers-reduced-motion`、表格需排序/空态/导出）。

## 五、七维评审 Rubric（交付前自评，每维 0-5 分）

| # | 维度 | 0 分特征 | 5 分特征 |
|---|---|---|---|
| 1 | 视觉层次 | 无主次 | 3 秒可读出页面用途与下一步 |
| 2 | 间距对齐 | magic number | 严格 8pt 刻度 + 光学对齐 |
| 3 | 配色克制 | 多强调色/渐变 | 一强调色 + 灰阶，对比度达标 |
| 4 | 字体阶梯 | 同字号靠加粗 | 阶梯清晰，字重有意 |
| 5 | 状态完整 | 只有 happy path | 四态齐全，空态引导行动 |
| 6 | 交互反馈 | 无 hover/focus | 全状态反馈，focus 环清晰 |
| 7 | 一致性 | 组件各自为政 | 同类元素规范统一 |

每条 <3 分必须给具体修改指令（区域 + 期望）。**评审与实现分离**：先独立完成评分报告，再据报告修改。

## 六、即时模式 / canvas GUI 专项（egui、Slint、Fyne 等自绘 UI）

这类栈没有浏览器 DOM 的兜底样式，规约靠代码自建，且有三类高频事故：

1. **几何对齐必须对"墨迹"做，不能对"包围盒"做**。文本 galley/包围盒含基线下伸部预留，`Align2::LEFT_CENTER`（egui）或包围盒居中会让字身偏高。修法：用 galley 的 `mesh_bounds`（真实墨迹紧包围盒）计算字身中心，反推锚点（egui 参考 `paint_text_ink_centered` 模式）。
2. **注入回退字体必须校准垂直度量**。默认字体（拉丁）与回退字体（CJK）墨迹相对基线位置不同，混排会一高一低。egui 用 `FontData.tweak.y_offset_factor`（正值下移，按字号比例）；起点 0.06，做成常量可调。
3. **同一行的自绘部件必须显式统一高度**（如统一 30px），自定义布局务必垂直居中子部件，禁止 y=0 顶对齐。
4. **图标必须矢量自绘或纹理化**，文本按钮达不到图形化底线；封装成 Icon/icon_button 组件一次投入。
5. **CJK 字体**：各栈默认字体均无中文字形。egui：HTTP fetch 字节 → FontDefinitions 注入（ttc 需 skrifa，epaint 0.36+ 可用）；Slint：font-family 指向系统族或嵌入字体；Fyne：Theme.Font() 返回单文件 ttf 资源（sfnt 不支持 ttc 集合）。加载失败必须优雅回退。

## 七、观测通道决策表（先建眼睛，再做界面）

| 栈 | 通道 | 质量 |
|---|---|---|
| Web (DOM) | 浏览器截图 MCP / 内置浏览器能力 | ★★★★★ |
| egui / femtovg wasm | **截图结构性失效**（preserveDrawingBuffer 写死）→ 人眼 oracle + accesskit 树 | ☆ |
| Slint 原生/wasm | 软件渲染器 render_to_buffer → PNG（MinimalSoftwareWindow 模式） | ★★★★ |
| Fyne | `test.NewApp` 无头 + `Canvas().Capture()` → PNG（1x 分辨率） | ★★★★☆ |
| 任意原生窗口 | macOS `screencapture`（需窗口定位配合） | ★★ |

规则：**动手写 UI 代码之前，先确认本栈的截图通道能落盘 PNG 且非空白**（写 5 行冒烟程序验证），否则评审环节失明。

## 八、工作模式

- 版本留档：v0（无规约基线）→ v1（规约+skill）→ v2（评审修复），截图与评审报告按版本存目录。
- 布局/主题等多方案决策时，做**变体选择页**（同 DOM/数据，切令牌或切布局），把开放题变选择题。
- 评审结论中反复出现的问题，回写进「负面清单」沉淀。

## 九、文案语言

界面文案语言遵循用户/项目的明示要求；未明示时与用户对话语言一致。**不设"设计语言模式"开关。**

**硬约束**：若文案含 CJK 且技术栈为自绘 GUI（egui/Slint/Fyne 等默认字体无中文字形），对应 CJK 字体注入管线为**验收项**（配方见 references/gui-recipes.md）——字体缺失即乱码，属不合格而非观感问题。

未来若需同一产品出多语言版本，正确形态是文案外置（strings/资源文件分离）+ locale 格式（日期/数字走 Intl），而非切换设计模式。
