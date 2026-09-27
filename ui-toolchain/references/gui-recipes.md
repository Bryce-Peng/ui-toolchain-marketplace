# GUI 观测通道与对齐配方（即时模式 / canvas UI 实测）

## 观测通道决策表

| 栈 | 通道 | 质量 |
|---|---|---|
| Web (DOM) | 浏览器截图（MCP/内置浏览器能力） | ★★★★★ |
| egui 0.36 (wasm/glow) | **截图结构性失效**（WebGL2 上下文未传属性，preserveDrawingBuffer:false 写死无开关）→ 用人眼 oracle + accesskit 树 | ☆ |
| Slint 1.18 | 软件渲染器：MinimalSoftwareWindow + `renderer.render(&mut buf, stride)` → PNG | ★★★★ |
| Fyne v2.8 | `test.NewApp` + `test.NewWindow` + `Canvas().Capture()` 无头渲染 → PNG（1x 分辨率） | ★★★★☆ |
| 任意原生窗口 | macOS `screencapture -x`（全屏） | ★★ |

**规则：动手写 UI 之前，先写 5 行冒烟程序验证截图通道能落盘非空白 PNG，否则评审环节失明。**

## egui：墨迹对齐（修"文字相对图标偏高"）

文本 galley 包围盒含基线下伸部预留，`Align2::LEFT_CENTER` 会让可见字身高于几何中心。用 `Galley.mesh_bounds`（真实墨迹紧包围盒）把字身墨迹中心对到目标中心：

```rust
fn paint_text_ink_centered(painter: &egui::Painter, left_center: egui::Pos2,
                           text: &str, font: egui::FontId, color: Color32) {
    let galley = painter.layout_no_wrap(text.to_owned(), font, color);
    let ink_dy = if galley.mesh_bounds.is_finite() && galley.mesh_bounds.height() > 0.0 {
        galley.mesh_bounds.center().y
    } else { galley.rect.center().y };
    painter.galley(egui::pos2(left_center.x, left_center.y - ink_dy), galley, color);
}
```

同型问题：原生 `ui.label` / `Button` / TextEdit 居中的都是包围盒 → 需要精确对齐时全部换墨迹居中绘制（参考 `ink_label`：allocate galley 尺寸参与布局，绘制时墨迹居中）。

## egui：回退字体垂直校准（修"中文与数字混排一高一低"）

中文字符走回退字体、数字走默认字体，两字体墨迹相对共享基线位置不同：

```rust
let mut data = egui::FontData::from_owned(bytes);
data.tweak.y_offset_factor = 0.06; // 正值=按字号比例下移；单点常量可调
```

## egui 0.36 API 迁移速查

- App trait：`fn ui(&mut self, ui: &mut egui::Ui, frame: &mut Frame)`（无 update，无 CentralPanel）
- web 启动：`eframe::WebRunner` + `runner.start(canvas, WebOptions, creator)`（canvas 由 index.html 传入）
- `SidePanel`/`TopPanel` 已合并为 `egui::Panel::left/top(...)`
- `RichText::size()` 只收 f32；`Context::all_styles_mut` 替代 `style_mut`
- wasm 截图失效为结构性（preserveDrawingBuffer 写死），勿浪费时间尝试截图

## Slint 1.18：无头快照配方

feature 集合：`default-features=false, features=["std","renderer-software","compat-1-18"]`（compat-1-18 是强制门禁）+ build.rs `slint_build::compile_with_config(path, cfg.with_style("fluent"))`。快照模式：`MinimalSoftwareWindow` + `set_platform` + `draw_if_needed(|r| r.render(&mut buf, stride))` → RGB 缓冲编码 PNG。注意：`Grid` 不存在（用 GridLayout）；自由放置元素默认撑满父宽（胶囊需 `width: ly.preferred-width`）；默认字体无 CJK（font-family 指向系统族或嵌入字体）。

## Fyne v2.8：无头 Capture 配方

```go
app := test.NewApp(); app.Settings().SetTheme(th)
w := test.NewWindow(content); w.Resize(fyne.NewSize(960, 600))
img := w.Canvas().Capture() // image.Image → png.Encode 落盘
```

- 自定义 Theme：嵌入 `theme.LightTheme()` 后 switch 覆写 Color/Size/Font（唯一能对抗"默认深色跟随系统"的确定性方案）
- Font 覆盖：`Theme.Font(TextStyle) fyne.Resource` 返回单文件 ttf 静态资源（sfnt 不支持 ttc 集合，PingFang/冬青不可用；找单文件 ttf/otf）
- 自定义 Layout 必须垂直居中子部件（`o.Move(pos{x, (h-childMin)/2})`），顶对齐会让徽章与文本错位
- `go build ./...` 多包丢二进制 → `go build -o bin/x ./x`
