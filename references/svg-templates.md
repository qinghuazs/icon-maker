# SVG 背景模板

8 个模板覆盖 4 种形状 × 纯色/线性渐变/径向渐变。所有模板使用 `viewBox="0 0 200 200"`，输出尺寸由 `width`/`height` 控制。

## 占位符

| 占位符 | 含义 | 示例 |
|---|---|---|
| `{SIZE}` | 输出宽高（像素） | `200` |
| `{BG}` | 背景填充（纯色用 hex，渐变用 `url(#bg)`） | `#5B50E8` / `url(#bg)` |
| `{ICON_STYLE}` | 图标的 stroke/fill 属性 | `fill="none" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"` |
| `{ICON_PATHS}` | Iconify SVG 内部的 path/circle 元素 | `<path d="..."/><circle ...>` |
| `{COLOR_1}` | 渐变起点色 | `#6366F1` |
| `{COLOR_2}` | 渐变终点色 | `#8B5CF6` |

## 图标定位

所有非透明模板内嵌图标用：

```
transform="translate(50 50) scale(4.166)"
```

这把 Iconify 默认 `viewBox="0 0 24 24"` 的图标缩放到 100×100，居中放在 200×200 画布的中央。如果原图标 viewBox 不是 24×24，需调整 scale。

---

## 模板 1：纯色 - 圆形

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="{SIZE}" height="{SIZE}" viewBox="0 0 200 200">
  <circle cx="100" cy="100" r="90" fill="{BG}"/>
  <g transform="translate(50 50) scale(4.166)" {ICON_STYLE}>
    {ICON_PATHS}
  </g>
</svg>
```

**用例**：默认形状。截图同款"紫圆白购物车"。

---

## 模板 2：纯色 - 圆角方形

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="{SIZE}" height="{SIZE}" viewBox="0 0 200 200">
  <rect x="10" y="10" width="180" height="180" rx="40" fill="{BG}"/>
  <g transform="translate(50 50) scale(4.166)" {ICON_STYLE}>
    {ICON_PATHS}
  </g>
</svg>
```

**用例**：Material 风、Android App 图标。
**变体**：圆角更小 `rx="20"`、更大 `rx="60"`。

---

## 模板 3：纯色 - Squircle（iOS App 风）

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="{SIZE}" height="{SIZE}" viewBox="0 0 200 200">
  <path d="M100,10 C160,10 190,40 190,100 C190,160 160,190 100,190 C40,190 10,160 10,100 C10,40 40,10 100,10 Z" fill="{BG}"/>
  <g transform="translate(50 50) scale(4.166)" {ICON_STYLE}>
    {ICON_PATHS}
  </g>
</svg>
```

**用例**：iOS / macOS App 风格。介于圆和圆角方形之间。

---

## 模板 4：透明背景

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="{SIZE}" height="{SIZE}" viewBox="0 0 24 24" {ICON_STYLE}>
  {ICON_PATHS}
</svg>
```

**用例**：只要图标本身，无背景。注意 viewBox 是 24×24，不需要 translate/scale。
**ICON_STYLE 示例**：
- stroke 风：`fill="none" stroke="#FFFFFF" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"`
- fill 风：`fill="#FFFFFF"`

---

## 模板 5：线性渐变 - 圆形

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="{SIZE}" height="{SIZE}" viewBox="0 0 200 200">
  <defs>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="{COLOR_1}"/>
      <stop offset="100%" stop-color="{COLOR_2}"/>
    </linearGradient>
  </defs>
  <circle cx="100" cy="100" r="90" fill="url(#bg)"/>
  <g transform="translate(50 50) scale(4.166)" {ICON_STYLE}>
    {ICON_PATHS}
  </g>
</svg>
```

**渐变方向变体**（修改 `<linearGradient>` 坐标）：

| 方向 | x1 | y1 | x2 | y2 |
|---|---|---|---|---|
| 对角（默认）↘ | 0% | 0% | 100% | 100% |
| 水平 → | 0% | 0% | 100% | 0% |
| 垂直 ↓ | 0% | 0% | 0% | 100% |
| 反对角 ↙ | 100% | 0% | 0% | 100% |

**多色渐变（3+ stops）**：

```svg
<linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
  <stop offset="0%" stop-color="{COLOR_1}"/>
  <stop offset="50%" stop-color="{COLOR_MID}"/>
  <stop offset="100%" stop-color="{COLOR_2}"/>
</linearGradient>
```

---

## 模板 6：线性渐变 - 圆角方形

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="{SIZE}" height="{SIZE}" viewBox="0 0 200 200">
  <defs>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="{COLOR_1}"/>
      <stop offset="100%" stop-color="{COLOR_2}"/>
    </linearGradient>
  </defs>
  <rect x="10" y="10" width="180" height="180" rx="40" fill="url(#bg)"/>
  <g transform="translate(50 50) scale(4.166)" {ICON_STYLE}>
    {ICON_PATHS}
  </g>
</svg>
```

---

## 模板 7：线性渐变 - Squircle

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="{SIZE}" height="{SIZE}" viewBox="0 0 200 200">
  <defs>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="{COLOR_1}"/>
      <stop offset="100%" stop-color="{COLOR_2}"/>
    </linearGradient>
  </defs>
  <path d="M100,10 C160,10 190,40 190,100 C190,160 160,190 100,190 C40,190 10,160 10,100 C10,40 40,10 100,10 Z" fill="url(#bg)"/>
  <g transform="translate(50 50) scale(4.166)" {ICON_STYLE}>
    {ICON_PATHS}
  </g>
</svg>
```

---

## 模板 8：径向渐变 - 圆形

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="{SIZE}" height="{SIZE}" viewBox="0 0 200 200">
  <defs>
    <radialGradient id="bg" cx="50%" cy="50%" r="50%">
      <stop offset="0%" stop-color="{COLOR_CENTER}"/>
      <stop offset="100%" stop-color="{COLOR_EDGE}"/>
    </radialGradient>
  </defs>
  <circle cx="100" cy="100" r="90" fill="url(#bg)"/>
  <g transform="translate(50 50) scale(4.166)" {ICON_STYLE}>
    {ICON_PATHS}
  </g>
</svg>
```

**径向变体**（移动中心点）：

```svg
<!-- 偏左上的高光效果 -->
<radialGradient id="bg" cx="30%" cy="30%" r="70%">
  ...
</radialGradient>
```

---

## ICON_STYLE 写法速查

按图标库类型选 `ICON_STYLE`：

| 来源库 | ICON_STYLE 模板 |
|---|---|
| lucide / tabler / heroicons outline | `fill="none" stroke="{COLOR}" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"` |
| mdi / heroicons solid | `fill="{COLOR}"` |
| phosphor regular / light / thin | `fill="none" stroke="{COLOR}" stroke-width="{16/12/8}"`（viewBox 256） |
| phosphor bold / fill / duotone | `fill="{COLOR}"` |
| simple-icons（品牌彩色） | 不要覆盖，保留原色（去掉 ICON_STYLE） |

## 完整示例（替换后）

紫圆白购物车（lucide：shopping-cart）：

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200" viewBox="0 0 200 200">
  <circle cx="100" cy="100" r="90" fill="#5B50E8"/>
  <g transform="translate(50 50) scale(4.166)" fill="none" stroke="#FFFFFF" stroke-linecap="round" stroke-linejoin="round" stroke-width="2">
    <circle cx="8" cy="21" r="1"/>
    <circle cx="19" cy="21" r="1"/>
    <path d="M2.05 2.05h2l2.66 12.42a2 2 0 0 0 2 1.58h9.78a2 2 0 0 0 1.95-1.57l1.65-7.43H5.12"/>
  </g>
</svg>
```

蓝紫渐变 squircle 设置图标（lucide：settings）：

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200" viewBox="0 0 200 200">
  <defs>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="#6366F1"/>
      <stop offset="100%" stop-color="#8B5CF6"/>
    </linearGradient>
  </defs>
  <path d="M100,10 C160,10 190,40 190,100 C190,160 160,190 100,190 C40,190 10,160 10,100 C10,40 40,10 100,10 Z" fill="url(#bg)"/>
  <g transform="translate(50 50) scale(4.166)" fill="none" stroke="#FFFFFF" stroke-linecap="round" stroke-linejoin="round" stroke-width="2">
    <!-- 这里替换成 settings 的真实 path -->
  </g>
</svg>
```

## 渲染验证

写完 SVG 后用 rsvg-convert 渲染：

```bash
rsvg-convert -w 200 input.svg -o output.png
```

如 PNG 出来背景缺失 → 检查 `{BG}` 是否替换；图标缺失 → 检查 `{ICON_PATHS}` 缩进；图标偏位 → 调整 `translate` / `scale`。
