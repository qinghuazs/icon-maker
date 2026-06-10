# icon-maker Skill 设计文档

**日期**：2026-05-22
**作者**：xingleiwang（与 Claude Code 协作）
**状态**：Draft（待审阅）

---

## 1. 目标与动机

将"从 Iconify 拿图标 → 合成圆形/方形/渐变背景 → 导出 PNG"这个高频流程封装成一个可重用的 skill，让 Claude 在用户描述"颜色 + 形状 + 图标语义"组合需求时，自动按统一流程生成图标资源。

**核心价值**：
- 流程透明（用户能看到每一步在做什么）
- 跨平台（不绑定特定脚本）
- 易扩展（改 markdown 指引即可，无需改代码）

## 2. 触发条件

只要满足下列任一条件，Claude 即应主动加载 `icon-maker` skill：

| 类型 | 示例 |
|---|---|
| 中文动词 | "做图标"、"生成图标"、"图标合成"、"帮我做个图标" |
| 英文关键词 | iconify、icon-maker、make icon、generate icon |
| 图标库名 | lucide、mdi、tabler、heroicons、phosphor、simple-icons |
| 样式描述组合 | "紫色圆形购物车图标"、"蓝色圆角方形搜索图标"、"渐变背景的设置图标" |

**不触发**：用户要 AI 生成（绘画式）图标 / 处理位图 / 设计字体图标，这些不在本 skill 范围。

## 3. 实现形式

**方案 C：知识流程化** — skill 不封装可执行脚本，而是给 Claude 一份操作指引 + 模板片段 + 配色速查。Claude 按指引调用 `curl`/`rsvg-convert`/`Write` 等基础工具完成任务。

## 4. 目录结构

```
~/.claude/skills/icon-maker/
├── SKILL.md                    # 入口指引（含 YAML frontmatter）
├── SPEC.md                     # 本设计文档
├── references/
│   ├── iconify-libs.md         # 主流图标库速查
│   ├── svg-templates.md        # 4+ 种背景模板（含渐变变体）
│   └── color-palettes.md       # 中文色名→hex、设计系统、渐变 palette
└── assets/
    └── examples/               # 示例 PNG（紫圆购物车、蓝方搜索、渐变 Logo）
```

## 5. SKILL.md frontmatter

```yaml
---
name: icon-maker
description: 用于从 Iconify 等公开 CDN 拉取图标并合成"背景+图标"复合资源的 skill。触发词：做图标 / 生成图标 / 图标合成 / iconify / icon-maker / make icon / lucide / mdi / tabler / heroicons / simple-icons，或描述"颜色+形状+图标语义"组合（如"紫色圆形购物车图标"、"蓝紫渐变方形搜索图标")。输出 SVG 和 PNG。
---
```

## 6. 核心流程

```
1. 解析需求 → 提取：图标语义 / 背景色 / 形状 / 尺寸 / 数量 / 输出路径 / 图标颜色
2. 查图标   → curl https://api.iconify.design/{lib}/{name}.svg
              默认 lucide；失败 fallback 到 mdi → tabler → heroicons
              品牌 Logo 优先 simple-icons
3. 合成 SVG → 按 references/svg-templates.md 套对应背景模板
              替换 {BG} / {ICON_PATHS} / {ICON_STYLE} 占位符
4. 导出 PNG → rsvg-convert -w {size} in.svg -o out.png
              若 rsvg-convert 不可用：尝试 Python+cairosvg，再不行只出 SVG 并提示
5. 预览     → Read 工具读 PNG 让用户/Claude 看效果
```

## 7. 输入参数（自然语言提取，全部有默认值）

| 参数 | 默认 | 接受形式 |
|---|---|---|
| **图标语义** | （必填） | 中文（购物车/用户/心）、英文（cart/user/heart）、库前缀（`mdi:cart`） |
| **背景色** | `#5B50E8` indigo | 中文色名 / hex / 渐变描述（"蓝紫渐变"/"线性 #A→#B"/"径向"）/ "无背景" |
| **图标颜色** | 白色 `#FFFFFF` | 同背景色规则 |
| **形状** | 圆形 | 圆形 / 圆角方形 / squircle（iOS 风）/ 透明 |
| **尺寸** | 200×200 | "200"、"64x64"、"小/中/大" |
| **数量** | 1 | 自然语言批量（"4 个"、"一套"） |
| **输出路径** | `./icons/` | 相对或绝对路径 |
| **图标库** | lucide | 用户可指定 `mdi:cart` 等强制覆盖 |

## 8. SVG 模板（references/svg-templates.md）

### 8.1 纯色 - 圆形

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="{SIZE}" height="{SIZE}" viewBox="0 0 200 200">
  <circle cx="100" cy="100" r="90" fill="{BG}"/>
  <g transform="translate(50 50) scale(4.166)" {ICON_STYLE}>
    {ICON_PATHS}
  </g>
</svg>
```

### 8.2 纯色 - 圆角方形

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="{SIZE}" height="{SIZE}" viewBox="0 0 200 200">
  <rect x="10" y="10" width="180" height="180" rx="40" fill="{BG}"/>
  <g transform="translate(50 50) scale(4.166)" {ICON_STYLE}>
    {ICON_PATHS}
  </g>
</svg>
```

### 8.3 纯色 - Squircle（iOS App 风）

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="{SIZE}" height="{SIZE}" viewBox="0 0 200 200">
  <path d="M100,10 C160,10 190,40 190,100 C190,160 160,190 100,190 C40,190 10,160 10,100 C10,40 40,10 100,10 Z" fill="{BG}"/>
  <g transform="translate(50 50) scale(4.166)" {ICON_STYLE}>
    {ICON_PATHS}
  </g>
</svg>
```

### 8.4 线性渐变 - 圆形（新增）

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

**渐变方向参数化**：
- 对角（默认）：`x1=0% y1=0% x2=100% y2=100%`
- 水平：`x1=0% y1=0% x2=100% y2=0%`
- 垂直：`x1=0% y1=0% x2=0% y2=100%`
- 反向对角：`x1=100% y1=0% x2=0% y2=100%`

### 8.5 径向渐变 - 圆形（新增）

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

### 8.6 透明背景

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="{SIZE}" height="{SIZE}" viewBox="0 0 24 24" {ICON_STYLE}>
  {ICON_PATHS}
</svg>
```

### 8.7 渐变 + 圆角方形 / Squircle

同样的 `<defs><linearGradient>` / `<radialGradient>` 模式套到 rect / path 上，模板里给 4 个完整变体。

## 9. 配色策略（references/color-palettes.md）

### 9.1 中文色名 → hex 速查

```
红 #EF4444    粉 #EC4899    紫 #8B5CF6    蓝 #3B82F6
青 #06B6D4    绿 #10B981    黄 #F59E0B    橙 #F97316
灰 #6B7280    黑 #111827    白 #FFFFFF
深紫 #5B50E8（用户图截默认色）
靛蓝 #6366F1
```

### 9.2 渐变 palette（精选 8 组）

```
日落    #F97316 → #EF4444
晨曦    #FCD34D → #F97316
海洋    #06B6D4 → #3B82F6
天空    #93C5FD → #3B82F6
紫罗兰  #8B5CF6 → #EC4899
极光    #10B981 → #06B6D4
品牌紫  #6366F1 → #8B5CF6（推荐默认渐变）
夜空    #1E293B → #6366F1
```

### 9.3 设计系统配色（链接到 Tailwind / Material / Apple HIG 文档）

提供查找规则，不全量复制。

## 10. Fallback 策略

| 情境 | 处理 |
|---|---|
| `rsvg-convert` 可用 | 直接转 PNG（首选） |
| 仅 Python + cairosvg | `python3 -c "import cairosvg; cairosvg.svg2png(...)"` |
| 仅 Node + sharp | `node -e "..."`（仅当用户项目已装 sharp） |
| 都没有 | 只出 SVG 并打印安装提示：`brew install librsvg` |
| Iconify 网络不通 | 提示并请求用户给出 SVG / 改用本地 npm 包 |
| 图标名查不到 | 自动尝试同义词（cart ↔ shopping-cart ↔ basket），仍失败则列出候选 |

## 11. 不在范围内（YAGNI）

- 不做 AI 生成图标（DALL·E/Midjourney 那是另一个领域）
- 不做付费图标库授权（Flaticon 付费、Icons8 Premium）
- 不做 iconfont（需登录授权）
- 不做位图图标处理（PNG → PNG 不需要 skill）
- 不做动效（lottie、SVG SMIL 动画）
- 不做矢量图标编辑器（修改 path）

## 12. 测试场景（验收清单）

| # | 输入 | 期望输出 |
|---|---|---|
| 1 | "做一个紫色圆形购物车图标" | `./icons/cart-purple.{svg,png}`，#5B50E8 圆 + 白购物车 |
| 2 | "做一套蓝色圆角方形：home/search/user/settings" | 4 个 PNG + 4 个 SVG |
| 3 | "用 mdi 的 account，红色背景" | 强制 lib=mdi，#EF4444 圆 |
| 4 | "GitHub 黑色圆形 Logo" | 用 simple-icons 库，#181717 圆 |
| 5 | "只要白色的 lucide heart SVG" | transparent 模板，只出 SVG |
| 6 | **"蓝紫渐变圆形设置图标"** | 线性渐变 #6366F1 → #8B5CF6 + 白齿轮 |
| 7 | **"日落渐变 squircle，64x64，4 个：cart/user/heart/star"** | 4 个 64×64，#F97316 → #EF4444 |
| 8 | 图标名错（"购物筐"） | 自动 fallback 到 shopping-cart / basket / cart |

## 13. 与其他 skill 的关系

- **`superpowers:writing-skills`** — 本 skill 的诞生流程依赖它
- **`Viral_Writer_Skill`** — 配图阶段可能调用本 skill 给文章生成插图
- **`pencil` MCP** — 如果用户在 .pen 设计文件里用，应优先调 pencil 工具而非本 skill

## 14. 后续可能的扩展（不在 v1）

- 支持本地 `@iconify-json/*` npm 包（离线模式）
- 支持 PNG 转 ICO（Windows favicon）
- 支持 1x/2x/3x 多倍图批量
- 支持 dark/light 双主题成对生成
- 支持 React/Vue 组件代码片段直接输出（而不是 SVG 文件）

---

## 附录 A：SKILL.md 最终内容草稿

```markdown
---
name: icon-maker
description: 从 Iconify 等公开 CDN 拉取图标并合成"背景+图标"复合资源。触发词：做图标 / 生成图标 / 图标合成 / iconify / icon-maker / make icon / lucide / mdi / tabler / heroicons / simple-icons，或描述"颜色+形状+图标语义"组合（如"紫色圆形购物车图标"、"蓝紫渐变方形搜索图标")。输出 SVG 和 PNG。
---

# icon-maker

## 何时使用

用户描述出"图标语义 + 视觉样式（颜色/形状/渐变）"的组合需求，或明确说要"做/生成/合成图标"时。
不适用：AI 绘画式图标、付费图标库、iconfont 授权图标、动效图标。

## 流程

按以下 5 步执行：

### 1. 解析需求
从用户输入提取参数（图标语义、背景色、形状、尺寸、数量、输出路径）。
未指定的用默认值（见 SPEC.md §7）。

### 2. 查图标
- 优先 `curl -s https://api.iconify.design/{lib}/{name}.svg`
- 默认 lib=lucide，品牌 Logo 用 simple-icons
- 失败 fallback：lucide → mdi → tabler → heroicons
- 图标名查不到时，尝试同义词

### 3. 合成 SVG
按 [references/svg-templates.md](references/svg-templates.md) 选模板：
- 纯色圆形 / 圆角方形 / squircle / 透明
- 线性渐变 / 径向渐变（各形状变体）

替换占位符：`{SIZE}`、`{BG}`（纯色或 `url(#bg)`）、`{ICON_STYLE}`、`{ICON_PATHS}`、`{COLOR_1}`/`{COLOR_2}`（渐变）。

### 4. 导出 PNG
- 首选 `rsvg-convert -w {size} in.svg -o out.png`
- Fallback：Python+cairosvg / Node+sharp / 仅 SVG

### 5. 预览
用 `Read` 工具读 PNG 给用户/Claude 看效果，确认无误。

## 参考资料

- [references/iconify-libs.md](references/iconify-libs.md) — 主流图标库特点对比
- [references/svg-templates.md](references/svg-templates.md) — 完整 SVG 模板集
- [references/color-palettes.md](references/color-palettes.md) — 配色 + 渐变 palette
```

---

## 附录 B：实施清单（交给 writing-plans skill）

1. 创建 `~/.claude/skills/icon-maker/SKILL.md`（按附录 A）
2. 创建 `references/iconify-libs.md`（主流库速查表）
3. 创建 `references/svg-templates.md`（含 8 个完整模板：3 形状 × 纯色/渐变/径向）
4. 创建 `references/color-palettes.md`（中文色名表 + 渐变 palette + 设计系统）
5. 生成 3 个 `assets/examples/` 示例 PNG 作为参考
6. 端到端测试 §12 的 8 个场景
