---
name: icon-maker
description: 用于从 Iconify 等公开 CDN 拉取图标并合成"背景+图标"复合资源。触发词：做图标 / 生成图标 / 图标合成 / iconify / icon-maker / make icon / lucide / mdi / tabler / heroicons / simple-icons，或用户描述"颜色+形状+图标语义"组合（如"紫色圆形购物车图标"、"蓝紫渐变方形搜索图标"）。输出 SVG 和 PNG。
---

# icon-maker

## 何时使用

用户描述出"图标语义 + 视觉样式（颜色/形状/渐变）"的组合需求，或明确说要"做/生成/合成图标"时。

**不适用**：AI 绘画式图标、付费图标库授权、iconfont 授权图标、动效图标（lottie / SVG SMIL）、位图图标处理。

## 流程（5 步）

### Step 1：解析需求

从用户输入提取以下参数（未指定的用默认值）：

| 参数 | 默认值 | 接受形式 |
|---|---|---|
| 图标语义 | （必填） | 中文（购物车/用户/心）、英文（cart/user/heart）、库前缀（mdi:cart） |
| 背景色 | `#5B50E8` indigo | 中文色名 / hex / 渐变描述 / "无背景" |
| 图标颜色 | 白色 `#FFFFFF` | 同背景色规则 |
| 形状 | 圆形 | 圆形 / 圆角方形 / squircle / 透明 |
| 尺寸 | 200×200 | "200"、"64x64"、"小/中/大" |
| 数量 | 1 | 自然语言批量（"4 个"、"一套"） |
| 输出路径 | `./icons/` | 相对或绝对路径 |
| 图标库 | lucide | 用户可强制覆盖（`mdi:cart`） |

### Step 2：查图标

```bash
curl -s "https://api.iconify.design/{lib}/{name}.svg" -o /tmp/icon-raw.svg
```

- 默认 `lib=lucide`
- 品牌 Logo 用 `simple-icons`
- 失败 fallback 顺序：lucide → mdi → tabler → heroicons
- 图标名查不到时，自动尝试同义词（cart ↔ shopping-cart ↔ basket）

选库前查阅 [references/iconify-libs.md](references/iconify-libs.md)。

### Step 3：合成 SVG

按 [references/svg-templates.md](references/svg-templates.md) 选模板：

- **纯色** × {圆形 / 圆角方形 / squircle / 透明}
- **线性渐变** × {圆形 / 圆角方形 / squircle}
- **径向渐变** × {圆形}

替换占位符：
- `{SIZE}` → 尺寸（默认 200）
- `{BG}` → 纯色 hex 或 `url(#bg)`
- `{ICON_STYLE}` → 图标 stroke/fill 属性
- `{ICON_PATHS}` → Iconify SVG 内部的 path 元素
- `{COLOR_1}` / `{COLOR_2}` → 渐变两端色

**渐变识别规则**：
- "蓝紫渐变" / "X到Y渐变" / "X→Y" → 线性渐变（默认对角，左上→右下）
- "径向渐变" / "中心X边缘Y" → 径向渐变
- "渐变背景"（未指定具体色）→ 默认品牌紫 `#6366F1 → #8B5CF6`

颜色查询：[references/color-palettes.md](references/color-palettes.md)。

### Step 4：导出 PNG

```bash
rsvg-convert -w {SIZE} input.svg -o output.png
```

**Fallback 链**：
1. `rsvg-convert` 可用 → 直接渲染（首选）
2. Python + cairosvg 可用 → `python3 -c "import cairosvg; cairosvg.svg2png(url='in.svg', write_to='out.png', output_width={SIZE})"`
3. Node + sharp 可用 → 用 sharp 渲染
4. 都没有 → 仅出 SVG，提示用户：`brew install librsvg`

### Step 5：预览

用 `Read` 工具读 PNG 给 Claude / 用户看效果。批量场景可同时 Read 多个文件。

**注意**：透明背景 + 浅色图标（白/淡色）在白底预览里会"看不见"——这是正常的（PNG 透明像素 + 白底显示）。提示用户：把 SVG 嵌入深色页面或换个图标色再预览；或直接看 SVG 源码验证。

## 参考资料

- [references/iconify-libs.md](references/iconify-libs.md) — 主流图标库速查
- [references/svg-templates.md](references/svg-templates.md) — 8 个 SVG 模板
- [references/color-palettes.md](references/color-palettes.md) — 配色 + 渐变 palette
- [SPEC.md](SPEC.md) — 完整设计文档（含范围边界）
- [assets/examples/](assets/examples/) — 3 个示例资源
