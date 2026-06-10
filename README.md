# icon-maker

一个 [Claude Code](https://claude.com/claude-code) Skill：从 [Iconify](https://iconify.design/) 公开 CDN 拉取开源图标，合成「背景 + 图标」复合资源，一句话输出成品 SVG 和 PNG。

> "做一个紫色圆形购物车图标"
> "蓝紫渐变方形搜索图标，64x64"
> "GitHub 黑色圆形 Logo"

## 效果示例

| 紫色圆形购物车 | 蓝紫渐变方形搜索 | GitHub Logo |
|:---:|:---:|:---:|
| <img src="assets/examples/example-cart-purple.png" width="120"> | <img src="assets/examples/example-search-gradient.png" width="120"> | <img src="assets/examples/example-github-logo.png" width="120"> |
| `紫色圆形购物车图标` | `蓝紫渐变方形搜索图标` | `GitHub 黑色圆形 Logo` |

## 设计理念：知识流程化

本 skill **不包含任何可执行脚本**——只有一份操作指引、8 个 SVG 模板和配色速查表。Claude 按指引调用 `curl` / `rsvg-convert` / 文件写入等基础工具完成全流程。

- **流程透明**：每一步在做什么，用户都看得到
- **跨平台**：不绑定特定脚本运行环境
- **易扩展**：改 Markdown 指引即可，无需改代码

## 安装

**方式一：直接 clone 到 skills 目录**

```bash
git clone git@github.com:qinghuazs/icon-maker.git ~/.claude/skills/icon-maker
```

**方式二：clone 到代码目录 + 软链接**（便于和其他仓库统一管理）

```bash
git clone git@github.com:qinghuazs/icon-maker.git ~/code/skills/icon-maker
ln -s ~/code/skills/icon-maker ~/.claude/skills/icon-maker
```

重启 Claude Code 后生效。

## 使用

直接用自然语言描述「颜色 + 形状 + 图标语义」的组合需求即可触发：

```
做一个紫色圆形购物车图标
做一套蓝色圆角方形图标：home / search / user / settings
日落渐变 squircle，64x64，4 个：cart / user / heart / star
用 mdi 的 account，红色背景
只要白色的 lucide heart SVG
```

触发词：`做图标` / `生成图标` / `图标合成` / `iconify` / `make icon`，或直接提及图标库名（`lucide` / `mdi` / `tabler` / `heroicons` / `simple-icons`）。

### 参数一览（全部可用自然语言表达，未指定走默认值）

| 参数 | 默认值 | 接受形式 |
|---|---|---|
| 图标语义 | （必填） | 中文（购物车）、英文（cart）、库前缀（`mdi:cart`） |
| 背景色 | `#5B50E8` indigo | 中文色名 / hex / 渐变描述 / "无背景" |
| 图标颜色 | 白色 `#FFFFFF` | 同背景色规则 |
| 形状 | 圆形 | 圆形 / 圆角方形 / squircle（iOS 风）/ 透明 |
| 尺寸 | 200×200 | "200"、"64x64"、"小/中/大" |
| 数量 | 1 | 自然语言批量（"4 个"、"一套"） |
| 输出路径 | `./icons/` | 相对或绝对路径 |
| 图标库 | lucide | `mdi:cart` 等前缀强制覆盖 |

### 渐变支持

- `蓝紫渐变` / `X到Y渐变` → 线性渐变（默认对角方向）
- `径向渐变` / `中心X边缘Y` → 径向渐变
- 只说 `渐变背景` → 默认品牌紫 `#6366F1 → #8B5CF6`

内置 8 组精选渐变 palette（日落 / 晨曦 / 海洋 / 天空 / 紫罗兰 / 极光 / 品牌紫 / 夜空），详见 [references/color-palettes.md](references/color-palettes.md)。

## 工作原理（5 步）

```
1. 解析需求 → 提取图标语义 / 背景色 / 形状 / 尺寸 / 数量 / 输出路径
2. 查图标   → curl https://api.iconify.design/{lib}/{name}.svg
              失败自动 fallback：lucide → mdi → tabler → heroicons
              图标名查不到时自动尝试同义词（cart ↔ shopping-cart ↔ basket）
3. 合成 SVG → 套用模板（3 种形状 × 纯色/线性渐变/径向渐变），替换占位符
4. 导出 PNG → rsvg-convert 渲染（含 cairosvg / sharp fallback）
5. 预览     → Claude 读取 PNG 确认效果
```

## 支持的图标库

| 前缀 | 风格 | 何时用 |
|---|---|---|
| `lucide` | 细线条圆角 stroke | **默认首选**，现代 UI |
| `mdi` | 实心 fill，~7000 个 | Material 风格，覆盖最全 |
| `tabler` | 细线条带轮廓 | 后台/管理系统 |
| `heroicons` | 线条 + solid 双版本 | Tailwind 项目 |
| `phosphor` | 6 种粗细变体 | 需要风格统一时 |
| `simple-icons` | 品牌 Logo | GitHub / Twitter 等品牌图标 |

完整对比见 [references/iconify-libs.md](references/iconify-libs.md)。

## PNG 导出依赖

按以下顺序自动探测，任装一个即可（都没有时仅输出 SVG）：

1. `rsvg-convert`（首选）：`brew install librsvg`
2. Python + cairosvg：`pip install cairosvg`
3. Node + sharp（仅当项目已安装）

## 仓库结构

```
icon-maker/
├── SKILL.md                    # 入口指引（含 YAML frontmatter）
├── SPEC.md                     # 完整设计文档（范围边界、验收场景）
├── references/
│   ├── iconify-libs.md         # 主流图标库速查
│   ├── svg-templates.md        # 8 个 SVG 背景模板
│   └── color-palettes.md       # 中文色名→hex、渐变 palette
└── assets/
    └── examples/               # 示例 SVG + PNG
```

## 范围边界

本 skill 只做「开源图标 + 几何背景」合成，以下场景不在范围内：

- AI 绘画式图标生成（DALL·E / Midjourney 领域）
- 付费图标库（Flaticon 付费、Icons8 Premium）、iconfont 授权图标
- 位图图标处理、矢量 path 编辑
- 动效图标（Lottie、SVG SMIL）
