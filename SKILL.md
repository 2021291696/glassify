---
name: glassify
description: 将已有前端项目（手写 CSS / Tailwind / Flutter）改造为玻璃质感 glassmorphism 风格（毛玻璃/磨砂效果）。触发词：玻璃化、glassify、上玻璃、毛玻璃、磨砂。流程：先问颜色→出方案确认→动手改→视口截图验收。暗色玻璃 fill 必须高透（alpha .12–.35），禁止实色块。
---

# 玻璃化改造 — glassify

将现有前端项目中的主容器（卡片、面板、导航栏、弹窗等）改造为玻璃质感（glassmorphism）风格。每次调用时先问颜色，出方案确认后动手，改完截图验证。

## 触发

```
/glassify <项目路径> [颜色说明]
```

- 项目路径：必填，绝对路径或当前工作目录下的相对路径
- 颜色说明：可选，跳过颜色提问直接指定色板描述（如"深邃蓝紫风"、"暖陶土色"）

## 流程

### 阶段 1：技术栈探测

自动识别项目前端技术栈：

| 探测依据 | 判定为 |
|---------|-------|
| 存在 `src/` + `.css` 文件且无 `tailwind` 依赖 | **手写 CSS** |
| `tailwind.config` 或 `@tailwind` 或 `@import "tailwindcss"` | **Tailwind** |
| 存在 `pubspec.yaml` + `lib/` + `.dart` 文件 | **Flutter** |
| 同时匹配多个 | 取最外层/最活跃的栈 |

如探测失败或存疑，输出猜测结果让用户确认。

### 阶段 2：颜色提问

**当用户没有在触发词中给出颜色说明时**，询问：

> 你想用哪种颜色风格？
> 1. **destiny 暖纸系** — 暖白底+黄蓝红彩色光斑，亮色调玻璃
> 2. **soshow 赛博暗** — 深夜蓝底+紫青霓虹光晕，暗色调玻璃
> 3. **跟随目标项目现有主色** — 提取项目样式中的主色和辅色，衍生光斑
> 4. **自定义** — 描述你想要的色板（如"薄荷绿+天蓝，清新冷调"）

根据用户选择，确定：
- 玻璃面底色明暗（亮色/暗色）
- 光斑颜色（1~4 个径向渐变）
- 文字色（浅色背景用深色、深色背景用浅色）

### 阶段 3：产出方案，等待确认

向用户展示：

```
📋 玻璃化方案
──────────────
技术栈：手写 CSS
底色：亮色暖白
光斑：黄(#ffe81a) + 蓝(#0758f7) + 红(#ff3044)
要动：
  1. src/components.css → .chart-result, .domain-console 等 → 加玻璃配方
  2. src/styles.css → .birth-form, .site-header → 加玻璃配方
  3. src/styles.css → 背景补光斑（当前为纯色平底）
维持实色：按钮、输入框、标签
────────────────
确认开始改造？(y/n)
```

**用户确认前，不写任何文件。**

### 阶段 4：执行改造

**动手前硬闸口：`git status` 确认项目工作区干净。** 若有未提交改动，先请用户提交或 stash——否则本次改造的回滚会连带吞掉无关改动。闸口未过，不写任何文件。

按技术栈执行对应配方。

#### 暗色配方铁律（每次必检）

暗色玻璃最容易做成「糊一层深色块」。下列任一违反 = 配方失败，重做，不要交付：

- **fill alpha `.12–.35`**，面板永远不许 ≥ `.60`（那是实色，不是玻璃）
- **blur ≥ 24px + saturate ≥ 1.6**
- **蚀刻边缘必齐**：内嵌高光 + 内嵌暗边 + 双层外投影。缺一项就没有厚度
- **光泽折进 `background`**，禁止 `::before` 叠在文字上（会洗对比度）
- **背景光斑必须又大又亮**，玻璃后面没东西可虚化 = 看起来只是半透明色块
- 顶栏可比面板略实（alpha `.30–.55`），面板保持高透

颜色跟项目走；以上是结构，不是色值。

#### 三套共用配方核心

```css
/* 玻璃卡核心配方（亮色底）——光泽已折进 background，无 ::before */
.glass-card {
  position: relative;
  border: 1px solid rgba(255,255,255,.75);
  border-radius: 18px;
  background:
    linear-gradient(180deg, rgba(255,255,255,.50), transparent 52%),
    linear-gradient(165deg, rgba(255,255,255,.62) 0%, rgba(255,255,255,.34) 100%);
  backdrop-filter: blur(16px) saturate(1.5);
  -webkit-backdrop-filter: blur(16px) saturate(1.5);
  box-shadow:
    inset 0 1px 0 rgba(255,255,255,.95),
    inset 0 0 0 1px rgba(255,255,255,.28),
    inset 0 -1px 0 rgba(17,17,17,.05),
    0 2px 5px rgba(17,17,17,.07),
    0 20px 44px -14px rgba(17,17,17,.28);
  overflow: hidden;
  transform: translateZ(0);
}
```

```css
/* 玻璃卡核心配方（暗色底）——高透 + 蚀刻，alpha 峰值 .32 */
.glass-card-dark {
  position: relative;
  border: 1px solid rgba(255,255,255,.16);
  border-radius: 20px;
  background:
    linear-gradient(180deg, rgba(255,255,255,.10), rgba(255,255,255,.03) 52%, transparent),
    linear-gradient(160deg, rgba(255,255,255,.14), rgba(20,20,28,.20) 46%, rgba(12,12,18,.32));
  backdrop-filter: blur(24px) saturate(1.8);
  -webkit-backdrop-filter: blur(24px) saturate(1.8);
  box-shadow:
    inset 0 1px 0 rgba(255,255,255,.30),
    inset 0 -1px 0 rgba(0,0,0,.30),
    inset 0 0 0 1px rgba(255,255,255,.04),
    inset 0 -24px 44px -26px rgba(0,0,0,.55),
    0 10px 26px rgba(0,0,0,.45),
    0 30px 64px -24px rgba(0,0,0,.60);
  overflow: hidden;
  transform: translateZ(0);
}

/* 顶栏可比面板略实，仍必须能透光斑 */
.glass-bar-dark {
  backdrop-filter: blur(24px) saturate(1.7);
  -webkit-backdrop-filter: blur(24px) saturate(1.7);
  background: linear-gradient(160deg, rgba(40,40,52,.55), rgba(16,16,22,.30));
  border-bottom: 1px solid rgba(255,255,255,.12);
  box-shadow:
    inset 0 1px 0 rgba(255,255,255,.16),
    0 6px 22px rgba(0,0,0,.40);
}
```

渐变里的中性暗色可换成项目主色（如深咖/深夜蓝），**只换色相，不把 alpha 加回去**。

**Safari 前缀**：手写 CSS 必须同时带 `backdrop-filter` 和 `-webkit-backdrop-filter`（Safari 至今要求前缀，缺一个玻璃全失效）。Tailwind 的 `backdrop-blur-*` 工具类已内置双前缀，无需手补。

#### 手写 CSS 项目改造

1. 在全局 CSS 或组件 CSS 中插入玻璃配方
2. 给目标元素追加 `.glass-card`/`.glass-card-dark` 类
3. 背景补光斑：在 `body` 或 `#root` 或固定层（`position:fixed; inset:0; z-index:-1`）上加多层 `radial-gradient`

```css
/* 背景光斑模板（亮色），颜色和位置按用户色板调整 */
.app-backdrop {
  position: fixed;
  inset: 0;
  z-index: -1;
  pointer-events: none;
  background:
    radial-gradient(720px 480px at 10% -8%,
      color-mix(in srgb, #ffe81a 78%, transparent), transparent 68%),
    radial-gradient(820px 560px at 98% 14%,
      color-mix(in srgb, #0758f7 26%, transparent), transparent 66%),
    radial-gradient(900px 620px at 46% 112%,
      color-mix(in srgb, #ffe81a 56%, transparent), transparent 70%),
    radial-gradient(520px 420px at 82% 88%,
      color-mix(in srgb, #ff3044 14%, transparent), transparent 72%),
    var(--paper);
}

/* 背景光斑模板（暗色）——光斑必须够大够亮，否则高透玻璃后面是一片黑 */
.app-backdrop-dark {
  position: fixed;
  inset: 0;
  z-index: -1;
  pointer-events: none;
  background:
    radial-gradient(840px 560px at 10% -8%,
      color-mix(in srgb, var(--accent, #7c5cff) 60%, transparent), transparent 70%),
    radial-gradient(940px 660px at 104% 10%,
      color-mix(in srgb, var(--accent-2, #3de0ff) 36%, transparent), transparent 68%),
    radial-gradient(840px 620px at 78% 104%,
      color-mix(in srgb, var(--accent, #7c5cff) 26%, transparent), transparent 70%),
    radial-gradient(700px 560px at 90% 68%,
      color-mix(in srgb, var(--accent-2, #3de0ff) 16%, transparent), transparent 74%),
    var(--paper, #12121a);
}
```

#### Tailwind 项目改造

Tailwind 的 `backdrop-blur` 只有预设档位，需要精确值时在 `tailwind.config` 或 `globals.css` 补自定义值：

```css
@theme inline {
  --blur-glass: 16px;     /* 亮色面板 */
  --blur-glass-lg: 24px;  /* 暗色面板 / 顶栏 */
}
```

`@theme` 是 Tailwind v4 语法；v3 项目改在 `tailwind.config` 的 `theme.extend.backdropBlur` 里加同名档位，否则档位静默无效。

玻璃化时用这些工具类组合：

```jsx
// 亮色玻璃卡（光泽折进 bg，无 ::before）
<div className="relative border border-white/75 rounded-[18px] bg-[linear-gradient(180deg,rgba(255,255,255,.50),transparent_52%),linear-gradient(165deg,rgba(255,255,255,.62),rgba(255,255,255,.34))] backdrop-blur-glass saturate-150 shadow-[inset_0_1px_0_rgba(255,255,255,.95),inset_0_0_0_1px_rgba(255,255,255,.28),inset_0_-1px_0_rgba(17,17,17,.05),0_2px_5px_rgba(17,17,17,.07),0_20px_44px_-14px_rgba(17,17,17,.28)] overflow-hidden [transform:translateZ(0)]">

// 暗色玻璃卡（高透，禁止 bg-[rgba(30,30,50,.85)] 这种实色）
<div className="relative border border-white/16 rounded-[20px] bg-[linear-gradient(180deg,rgba(255,255,255,.10),rgba(255,255,255,.03)_52%,transparent),linear-gradient(160deg,rgba(255,255,255,.14),rgba(20,20,28,.20)_46%,rgba(12,12,18,.32))] backdrop-blur-glass-lg saturate-180 shadow-[inset_0_1px_0_rgba(255,255,255,.30),inset_0_-1px_0_rgba(0,0,0,.30),inset_0_0_0_1px_rgba(255,255,255,.04),inset_0_-24px_44px_-26px_rgba(0,0,0,.55),0_10px_26px_rgba(0,0,0,.45),0_30px_64px_-24px_rgba(0,0,0,.60)] overflow-hidden [transform:translateZ(0)]">
```

**背景光斑**：加在布局外壳层，用 `bg-[radial-gradient(...)]`。暗色项目套用上面 `.app-backdrop-dark` 的多层径向，不要只铺纯色。

```jsx
<div className="bg-[radial-gradient(720px_480px_at_10%_-8%,color-mix(in_srgb,var(--wash)_78%,transparent)_0%,transparent_68%),radial-gradient(820px_560px_at_98%_14%,color-mix(in_srgb,var(--accent)_26%,transparent)_0%,transparent_66%),var(--paper)]">
```

#### Flutter 项目改造

```dart
// 亮色玻璃卡
Container(
  decoration: BoxDecoration(
    borderRadius: BorderRadius.circular(18),
    border: Border.all(
      color: Colors.white.withValues(alpha: 0.75),
    ),
  ),
  child: ClipRRect(
    borderRadius: BorderRadius.circular(18),
    child: BackdropFilter(
      filter: ImageFilter.blur(sigmaX: 16, sigmaY: 16),
      child: Container(
        decoration: BoxDecoration(
          gradient: LinearGradient(
            begin: Alignment.topLeft,
            end: Alignment.bottomRight,
            colors: [
              Colors.white.withValues(alpha: 0.62),
              Colors.white.withValues(alpha: 0.34),
            ],
          ),
        ),
        // 内容
      ),
    ),
  ),
);

// 暗色玻璃卡 —— surface alpha 用 0.22，禁止 0.85（实色块）
// 背后必须有亮光斑/图片，否则 BackdropFilter 虚化空气
Container(
  decoration: BoxDecoration(
    borderRadius: BorderRadius.circular(20),
    border: Border.all(
      color: Colors.white.withValues(alpha: 0.16),
    ),
    boxShadow: [
      BoxShadow(
        color: Colors.black.withValues(alpha: 0.45),
        blurRadius: 26,
        offset: const Offset(0, 10),
      ),
    ],
  ),
  child: ClipRRect(
    borderRadius: BorderRadius.circular(20),
    child: BackdropFilter(
      filter: ImageFilter.blur(sigmaX: 24, sigmaY: 24),
      child: Container(
        decoration: BoxDecoration(
          gradient: LinearGradient(
            begin: Alignment.topLeft,
            end: Alignment.bottomRight,
            colors: [
              Colors.white.withValues(alpha: 0.14),
              cs.surface.withValues(alpha: 0.22),
            ],
          ),
        ),
        // 内容
      ),
    ),
  ),
);
```

**Flutter 补充约束：**

- `withValues(alpha:)` 需要 Flutter ≥ 3.27；更旧的项目改用 `withOpacity()`（已废弃但可编译），全文件保持同一套 API，不要混用
- `BackdropFilter` 底层走 saveLayer，开销大：**长列表不要给每个 item 套玻璃**（滚动掉帧），列表项用实色半透明近似，玻璃只给悬浮卡片/顶栏/弹窗

### 阶段 5：验证

**Web 项目**：用 Playwright 打开 `localhost` 截图。没有 dev server 的静态页面，先在项目目录起 `python -m http.server` 再访问（Playwright MCP 拒绝 `file://` 协议，直接打开会报错）。

**铁律：只用视口截图，禁止 `fullPage: true`。** `backdrop-filter` 在 Playwright 拼接长图时会钉死在第一屏，看起来像「玻璃面板拦腰截断」——那是伪影，不是真实渲染。用户会按伪影误判配方失败。

**Flutter 项目**：执行 `flutter analyze` 检查编译，尝试 `flutter run` 截图。

检查清单：
- 玻璃卡片背后有可模糊的内容（光斑/图片/渐变）；暗色底光斑肉眼可见，不是淡得像没加
- 暗色面板透过玻璃能看到光斑色相；若只看到实色块 → alpha 超标，重做
- 玻璃卡片相互堆叠时透光正常（不出现全白全黑）
- 文字对比度不因玻璃背景降低（WCAG AA 水准）——光泽不得盖住文字
- 按钮、输入框、标签没有误玻璃化
- 44px 触摸目标未被装饰破坏
- 兼容 `prefers-reduced-motion`（玻璃效果本身不涉及动画，不动）
- Web 截图是视口模式，不是 fullPage

### 阶段 6：回滚（如有需要）

回滚只针对本次玻璃化的改动，前提是阶段 4 硬闸口已确认工作区干净：

- 工作区当时干净 → 可整体恢复：`git checkout -- .`
- 当时留有无关未提交改动 → 只能逐文件恢复：`git checkout -- <本次改动的文件列表>`

禁止在对工作区状态不明时执行无差别整体回滚。

## 内置判定表

### 哪些元素应该玻璃化

| 元素类型 | 是否玻璃化 | 理由 |
|---------|-----------|------|
| 主卡片/面板 | ✅ | 玻璃感最集中体现 |
| 侧边栏/抽屉 | ✅ | 悬浮在背景上，透光效果佳 |
| 导航栏/底部栏 | ✅ | 半透明悬浮，视觉层次清晰 |
| 弹窗/Modal | ✅ | 浮层效果与玻璃质感天然契合 |
| Toast/通知条 | ✅ | 轻微半透明，不干扰正文 |
| 搜索框/输入框 | ❌ 保持实色 | 内容可读性优先 |
| 按钮 | ❌ 保持实色 | 可点击辨识度 |
| 标签/badge | ❌ 保持实色 | 小元素不需透光 |
| 表格/列表行 | ❌ 保持实色 | 大量文字需要清晰对比度 |

### 背景处理规则

| 现有背景 | 处理方式 |
|---------|---------|
| 已有渐变/图片/纹理 | 保留不动，直接上玻璃 |
| 纯色平底 | 补光斑层（2~4 个径向渐变，用用户色板） |
| 深色纯色 | 补霓虹光晕（暗色底配方，光斑够大够亮） |
| 已有光斑/光晕 | 保留，玻璃卡自然透出 |

### 暗色/亮色选择规则

| 用户选色 | 明暗流派 |
|---------|---------|
| destiny 暖纸系 | 亮色白玻璃 |
| 自定义浅色系 | 亮色白玻璃 |
| soshow 暗系 | 暗色烟熏玻璃 |
| 自定义深色系 | 暗色烟熏玻璃 |
| 跟随项目主色 | 提取页面底色明暗判断 |

## 不支持的场景

- 没有 `backdrop-filter` 支持的浏览器（IE11 等）——降级策略：卡面纯色背景，保留边框和阴影
- 纯文本/终端项目
- 无 CSS 的纯后端项目
- 非 Web/Flutter 的桌面端应用（Electron 算 Web 项目）
