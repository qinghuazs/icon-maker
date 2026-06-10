# Iconify 图标库速查

## API 调用规则

```
https://api.iconify.design/{prefix}/{name}.svg
```

支持参数：
- `?color=red` — 改 fill/stroke 颜色（仅对单色图标）
- `?width=32&height=32` — 改尺寸
- `?rotate=90` — 旋转

## 推荐图标库

| 前缀 | 名称 | 风格 | 数量 | 何时用 |
|---|---|---|---|---|
| `lucide` | Lucide | 细线条圆角 stroke | ~1500 | **首选**。现代 UI、Feather 风、最干净 |
| `mdi` | Material Design Icons | 实心 fill | ~7000 | Material UI 项目，覆盖最全 |
| `tabler` | Tabler Icons | 细线条带轮廓 | ~5000 | 后台/管理系统，工程感强 |
| `heroicons` | Heroicons | 细线条 + solid 双版本 | ~300 | Tailwind 项目首选 |
| `phosphor` | Phosphor | 6 种粗细变体 | ~7000 | 需要风格统一（thin/light/regular/bold/duotone/fill） |
| `simple-icons` | Simple Icons | 品牌彩色 Logo | ~3000 | **品牌 Logo**（GitHub/Twitter/Discord 等） |
| `solar` | Solar Icons | 现代圆润 | ~7000 | 设计感强，有 broken/bold/duotone 变体 |
| `ph` | Phosphor regular | Phosphor 别名 | 同上 | 同上 |

## Fallback 顺序（用户不指定库时）

```
lucide → mdi → tabler → heroicons
```

品牌 Logo 强制走 `simple-icons`。

## 同义词映射

| 用户输入 | Iconify 候选 |
|---|---|
| 购物车 / cart | shopping-cart, cart, basket |
| 用户 / user | user, account, person, profile |
| 主页 / home | home, house |
| 搜索 / search | search, magnifying-glass, find |
| 设置 / settings | settings, cog, gear, preferences |
| 心 / heart | heart, favorite, like |
| 星 / star | star, favorite-star |
| 删除 / trash | trash, delete, bin, remove |
| 编辑 / edit | edit, pencil, pen |
| 下载 / download | download, arrow-down-tray |
| 上传 / upload | upload, arrow-up-tray |
| 关闭 / close | close, x, x-mark |
| 菜单 / menu | menu, bars, hamburger |
| 加 / plus | plus, add |
| 减 / minus | minus, subtract |
| 通知 / bell | bell, notification |
| 锁 / lock | lock, key |
| 邮件 / mail | mail, envelope |
| 日历 / calendar | calendar, calendar-days |
| 时钟 / clock | clock, time |

## 浏览图标

找图标名时访问：
- https://icon-sets.iconify.design/ — Iconify 官方搜索
- https://lucide.dev/icons/ — Lucide 浏览
- https://pictogrammers.com/library/mdi/ — MDI 浏览
- https://heroicons.com/ — Heroicons 浏览
- https://simpleicons.org/ — Simple Icons 品牌库

## 单色 vs 多色

- **单色图标**（lucide/mdi/tabler/heroicons/phosphor）：默认 `currentColor`，可通过 `stroke`/`fill` 属性覆盖
- **多色 Logo**（simple-icons 部分）：自带配色，慎用 `?color` 参数

## 拉取示例

```bash
# Lucide 购物车
curl -s "https://api.iconify.design/lucide/shopping-cart.svg" -o cart.svg

# MDI account，红色
curl -s "https://api.iconify.design/mdi/account.svg?color=red" -o user.svg

# Simple Icons GitHub Logo
curl -s "https://api.iconify.design/simple-icons/github.svg" -o github.svg

# Heroicons 实心心形
curl -s "https://api.iconify.design/heroicons/heart-solid.svg" -o heart.svg

# Phosphor bold 版齿轮
curl -s "https://api.iconify.design/ph/gear-bold.svg" -o gear.svg
```

## 路径解析

Iconify 返回的 SVG 结构：

```svg
<svg xmlns="..." viewBox="0 0 24 24">
  <!-- 单色 stroke 风（lucide/tabler/heroicons outline） -->
  <g fill="none" stroke="currentColor" stroke-width="2" ...>
    <path d="..."/>
    <circle cx="..." cy="..." r="..."/>
  </g>

  <!-- 单色 fill 风（mdi/heroicons solid） -->
  <path fill="currentColor" d="..."/>

  <!-- 多色（simple-icons 品牌） -->
  <path fill="#181717" d="..."/>
</svg>
```

合成时把 `<g>`（或裸 `<path>`）的内容提取出来嵌入背景模板的内圈。
