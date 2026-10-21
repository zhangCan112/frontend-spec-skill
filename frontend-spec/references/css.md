# CSS 规范（原生 CSS）

> 原生 CSS 没有局部作用域，类名就是边界。命名清楚，比任何工具都重要。

> 级别：**红线**=顶部「本域红线」+ 正文里的「禁止/必须/不得」；其余未标注的祈使句=**默认**（破例说明理由）；标「约定」=按项目/团队。

## 本域红线

- 原生 CSS：不用 Sass/Less/Stylus、CSS Modules、Tailwind、CSS-in-JS。
- 类名用「组件前缀 + kebab-case」；单词只用连字符分隔；不用 `.header`、`.title` 这类泛名；变体/状态用组合类。
- `!important` 只允许出现在 `vendor-overrides.css`，且必须注释原因；不用 id 选择器、`@import`、`composes`、`:global`、`:local`。
- 颜色、间距、尺寸、圆角、字体、阴影、层级来自 token，不在组件样式里硬编码。
- 不删除 `:focus-visible` 轮廓而不给替代方案。
- 不用类型选择器限定 id/class（`ul#nav`、`div.error`）。
- 优先扁平类名；原生嵌套只用于伪类/伪元素，最多 3 层。

## 1. 范围与默认选择

- **原生 CSS**：不用 Sass / Less / Stylus，不用 CSS Modules，不用 Tailwind / CSS-in-JS。
- 所有 class 都是全局的；用「组件前缀 + kebab-case」防冲突。
- 不用 BEM 的 `__` / `--`；用扁平的组件前缀类名。
- 全局样式只放在 `src/styles/`，组件样式和组件放在一起。
- 目标浏览器为最近两个版本；原生嵌套、`@layer`、`@container` 可用，但不要为了「新」而滥用。

因为 class 是全局的：

- block 类名必须全局唯一；新增组件前先搜索是否已存在。
- 不要用 `.header`、`.title`、`.content`、`.active`、`.disabled`、`.container` 这类泛名。
- 不要依赖「文件路径」或「加载顺序」来避免冲突；命名本身要能说明归属。

## 2. 文件组织

```text
features/user/
  components/
    user-card.ts
    user-card.css
  styles/
    user-page.css
```

- 一个组件/页面一个 CSS 文件，和实现放在一起。
- 全局文件只放 `src/styles/`：
  - `reset.css`
  - `tokens.css`
  - `base.css`
  - `utilities.css`
  - `vendor-overrides.css`
- 加载方式：`<link rel="stylesheet">` 或 bundler 的 `import './user-card.css'`。
- 禁止 `@import`；它会增加关键渲染路径的往返。
- 小项目可以在入口按顺序引入全局文件；大项目用 `@layer` 固定级联顺序（见第 9 节）。
- 文件较长时用分段注释分区，段与段之间空一行：

```css
/* Header */

.app-header { ... }

/* Footer */

.app-footer { ... }
```

## 3. 类名约定

### 3.1 基本结构

| 类型 | 规则 | 例 |
| --- | --- | --- |
| block | 组件名 kebab-case，全局唯一 | `.user-card` |
| element | block + `-` + 元素名 | `.user-card-header` |
| variant | block/element + `-` + 变体名 | `.user-card-primary` |
| state | block/element + `.is-*` / `.has-*` | `.user-card.is-disabled` |
| utility | `u-` + 名字 | `.u-sr-only` |
| shared | `app-` + 名字 | `.app-button` |

补充：

- 类名中的单词只用连字符分隔：不用下划线（`.error_status`）、不用驼峰（`.demoImage`）、不粘连（`.demoimage`）。
- 名称要反映元素的**目的**，不要表象命名：`.app-button-primary` 优于 `.button-green`——哪天按钮改成蓝色，类名就成了谎言。
- 通用名称（`.aux`、`.alt`）只用于非特殊、与兄弟无区别的辅助元素。
- 命名在能表意的前提下尽量简短：常用缩写（`nav`、`url`）可用，不要删字母造缩写（`hdr`、`atr`）。
- block 名建议带 feature 名：`.user-card`、`.order-card`，不要用 `.card`。
- 同一个 feature 内，block 名只出现一次；不要在多个文件里重复定义 `.user-card`。
- 大型项目或会被嵌入其它站点的代码，可再加一层短的命名空间前缀（如 `.adw-help`），防止与宿主样式冲突。

```html
<article class="user-card">
  <header class="user-card-header">
    <img class="user-card-avatar" src="/avatar.webp" alt="张三的头像" width="40" height="40">
    <h2 class="user-card-name">张三</h2>
  </header>

  <button class="app-button app-button-primary" type="button">保存</button>
</article>
```

```css
.user-card {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  padding: var(--space-4);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  background: var(--color-surface);
}

.user-card-header {
  display: flex;
  align-items: center;
  gap: var(--space-2);
}

.user-card-avatar {
  inline-size: var(--size-avatar);
  block-size: var(--size-avatar);
  border-radius: 50%;
}

.user-card-name {
  font-size: var(--font-size-md);
  font-weight: 600;
  color: var(--color-text);
}
```

### 3.2 变体与状态

变体和状态只写差异；HTML 必须同时带 base 类和变体/状态类：

```html
<!-- variant：base + modifier -->
<button class="app-button app-button-primary" type="button">保存</button>

<!-- state：block + state -->
<article class="user-card is-disabled">...</article>
```

```css
.app-button {
  padding: var(--space-2) var(--space-4);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-sm);
  background: var(--color-surface);
  color: var(--color-text);
}

.app-button-primary {
  border-color: var(--color-primary);
  background: var(--color-primary);
  color: var(--color-on-primary);
}

.user-card.is-disabled {
  opacity: 0.5;
  pointer-events: none;
}
```

规则：

- 变体类只写差异，不重复 base 的完整样式。
- 状态类只写差异，并且必须和 block/element 组合使用。
- 不要单独定义 `.is-disabled`、`.active` 这种全局状态类；它们会污染所有组件。
- 变体/状态类名要能看出归属：`.user-card-primary` 比 `.primary` 好。
- 多个类在 HTML 里按「block → variant → state」排列：

```html
<article class="user-card user-card-compact is-disabled">...</article>
```

### 3.3 共享组件和工具类

- 共享组件的 block 用 `app-` 前缀：`.app-button`、`.app-button-primary`、`.app-link`。
- 工具类用 `u-` 前缀：`.u-sr-only`、`.u-truncate`、`.u-visually-hidden`。
- 工具类只放 `src/styles/utilities.css`，数量要少；不要发展成一套自建框架。
- 共享组件的样式和组件放在一起，例如 `src/shared/components/` 下的 `button.ts` + `button.css`；不要藏在某个 feature 里。

## 4. 设计 token

颜色、间距、圆角、字体、阴影、层级统一用 CSS 变量，定义在全局 `tokens.css`：

```css
:root {
  --color-text: #1f2329;
  --color-text-muted: #646a73;
  --color-border: #dee0e3;
  --color-surface: #fff;
  --color-primary: #3370ff;
  --color-on-primary: #fff;
  --color-danger: #f54a45;

  --space-1: 4px;
  --space-2: 8px;
  --space-4: 16px;
  --space-6: 24px;

  --radius-sm: 4px;
  --radius-md: 6px;

  --size-avatar: 40px;

  --font-size-sm: 12px;
  --font-size-md: 14px;
  --font-size-lg: 16px;

  --z-header: 100;
  --z-modal: 1000;
}
```

规则：

- 组件样式里不写死颜色、间距、尺寸：用 `var(--color-*)`、`var(--space-*)`、`var(--size-*)`。
- 允许直接写的值：`0`、`1px`（细线）、`100%`、`auto`、`1fr`、`50%`（圆形）、`1.5`（行高）。
- 新值先加 token，再使用；不要在组件里临时发明一个颜色。
- 命名用 kebab-case：`--color-primary`，不用 `--colorPrimary`。
- 层级用 `--z-*`，不要在组件里写 `z-index: 9999`。
- 主题覆盖用属性选择器，不用 JS 改一堆内联样式：

```css
[data-theme='dark'] {
  --color-text: #f5f6f7;
  --color-surface: #1f2329;
  --color-border: #3a3f4a;
}
```

## 5. 复用方式

原生 CSS 没有 `composes`、mixin、`@extend`，复用靠：

1. **多个 class**：base + variant + state，见第 3 节。
2. **共享 class**：把真正通用的样式抽成 `.app-button`、`.app-link` 这类有明确归属的 block。
3. **CSS 变量**：值层面的复用。
4. **工具类**：只用于 `sr-only`、`truncate` 这种跨组件的小能力。

禁止：

- `composes`（CSS Modules 语法）
- `:global` / `:local`（CSS Modules 语法）
- `@extend`、mixin、function、loop（预处理器语法）
- `@apply`（Tailwind）
- 为了「少写几行」复制一整段样式；要么抽共享 class，要么接受少量重复。

## 6. 属性顺序与数值写法

### 6.1 声明按字母序排列

声明按属性名字母序排列：容易记、容易维护、风格统一。浏览器前缀在排序时忽略（`-moz-` 排在 `-webkit-` 之前）；基础格式（缩进、分号、空行）交给 Prettier，不要手调。

```css
.user-card-header {
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  color: var(--color-text);
  display: flex;
  font-size: var(--font-size-md);
  line-height: 1.5;
  padding: var(--space-4);
  position: relative;
  z-index: var(--z-header);
}
```

### 6.2 数值简写

- 尽可能用简写属性，即使只显式设置其中一部分：

```css
/* bad */
border-top-style: none;
padding-bottom: 2em;
padding-left: 1em;
padding-right: 1em;
padding-top: 0;

/* good */
border-top: 0;
padding: 0 1em 2em;
```

- `0` 后面不写单位：`margin: 0;`，不是 `margin: 0px;`。
- -1 到 1 之间的值省略前导 `0`：`font-size: .8em;`。
- 能用 3 位十六进制的颜色就用 3 位：`color: #ebc;`，不是 `#eebbcc;`。

### 6.3 引号与 URL

- 属性选择器和属性值里的字符串用**单引号**：`font-family: 'open sans', arial, sans-serif;`、`[data-theme='dark']`。
- `url()` 里不加引号：`url(//cdn.example.com/logo.svg)`。
- 特例：`@charset` 只允许双引号（语法规定）。
- 跨域的完整 URL 省略协议头（`http:` / `https:`），写成 `//cdn.example.com/...`，避免协议混合内容和不必要的重复下载；同源资源照常用根相对路径 `/styles/...`。

## 7. 选择器与特异性

- 一个选择器只用一个 block 类：`.user-card`，不要 `.page .user-card .header`。
- 不用类型选择器限定 id/class：`ul#example`、`div.error` 都是坏味道；id 本身唯一、class 加前缀后语义自足，类型限定只添噪音还拖性能。
- 状态/变体用组合类：`.user-card.is-disabled`，不要写成深层后代。
- 禁止 id 选择器：优先级太高，后续只能靠 `!important` 覆盖。
- 禁止 `!important`。唯一例外：第三方覆盖，集中放在 `src/styles/vendor-overrides.css` 并注释原因：

```css
/* 第三方 modal 的内联 z-index 过高，集中在此覆盖 */
.third-party-modal {
  z-index: var(--z-modal) !important;
}
```

- 选择器和声明各自独立成行；规则块之间空一行（交给格式化工具维护）。
- 优先扁平类名。原生嵌套只用于同一元素的伪类/伪元素（`&:hover`、`&::before`），不要嵌套后代选择器。
- 如果确实需要嵌套，最多 3 层；不要用它模拟 BEM：

```css
/* good */
.app-button {
  color: var(--color-text);

  &:hover {
    color: var(--color-primary);
  }

  &:focus-visible {
    outline: 2px solid var(--color-primary);
    outline-offset: 2px;
  }
}

/* bad：靠 DOM 结构选择 */
.user-card > div > span {
  color: var(--color-text);
}

/* bad：后代嵌套让选择器难以预测 */
.user-card {
  .header {
    color: var(--color-text);
  }
}
```

- 需要降低特异性时用 `:where()`，不要用 `!important`：

```css
:where(.app-reset) a {
  color: inherit;
}
```

- 避免 CSS hacks（针对特定浏览器的选择器/属性技巧）和用户代理（UA）检测：它们是样式差异的捷径，但会让代码库失控；作为最后手段，用 `@supports` 按能力检测。
- 大型项目可以用 `@layer` 统一级联顺序；小项目不要为了时髦引入：

```css
@layer reset, base, components, utilities;
```

## 8. 布局与响应式

- 用 Flexbox / Grid；不用 float 做布局。
- 用 `gap` 控制间距，不要给子元素加 margin 再处理最后一个。
- 全局设置 `box-sizing: border-box`。
- 优先逻辑属性：`margin-inline`、`padding-block`、`inset-inline-start`，方便 RTL。
- 布局容器不写固定高度；用 `min-height`、`aspect-ratio`。媒体/图标需要固定尺寸时，用 size token（如 `--size-avatar`）。
- 移动优先：默认样式是小屏，用 `min-width` 向上覆盖。

```css
.user-card {
  padding: var(--space-2);
}

@media (min-width: 768px) {
  .user-card {
    padding: var(--space-4);
  }
}
```

- 原生 CSS 的媒体查询不能使用 `var()`，断点用字面量，并在文件顶部或团队文档写清楚刻度：

```css
/*
 * Breakpoints: sm 480px / md 768px / lg 1024px
 * 媒体查询里不能使用 CSS 变量，这里保持字面量与团队文档一致。
 */
```

- 组件级响应式优先容器查询（`@container`）；页面级布局再用媒体查询：

```css
.user-card-list {
  container-type: inline-size;
}

@container (min-width: 480px) {
  .user-card {
    grid-template-columns: 1fr 1fr;
  }
}
```

- 不靠固定宽度解决移动端问题。

## 9. 全局 CSS 组织

推荐加载顺序：

1. `reset.css`：清零浏览器差异
2. `tokens.css`：设计变量
3. `base.css`：全局元素默认样式
4. `utilities.css`：少量工具类
5. 组件/页面 CSS
6. `vendor-overrides.css`：第三方覆盖

规则：

- `base.css` 可以写元素选择器（`body`、`a`、`button`），组件 CSS 不写元素选择器。
- 组件 CSS 只写自己的 block 前缀类，不写别的组件的类。
- 第三方覆盖集中在一个文件。
- 需要固定级联顺序时用 `@layer`；不要靠加载顺序和 `!important` 打架。

## 10. 可访问性

对比度、非颜色状态提示、点击目标尺寸的标准见 `html.md` §6；这里只写 CSS 侧的实现。

- 不删除焦点轮廓；用 `:focus-visible` 给键盘用户清晰的焦点：

```css
.app-button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}
```

- 尊重减少动效：

```css
@media (prefers-reduced-motion: reduce) {
  .user-card-animated {
    animation: none;
    transition: none;
  }
}
```

- 不要用 `display: none` 隐藏需要被屏幕阅读器读到的内容；用 `.u-sr-only` 模式，内部用 `clip-path: inset(50%)`，不要用已废弃的 `clip`。
- 不要让 hover 成为唯一交互方式；触屏没有 hover。
- 固定头部/浮层不要遮挡键盘焦点。

## 11. 资源与性能

- 图标优先 SVG；不要用雪碧图解决所有问题。
- 不在 CSS 里写巨大的 base64；用资源文件。
- 动画优先 `transform` / `opacity`；不要动画 `width` / `height` / `top` / `left`。
- 大面积阴影/模糊谨慎使用；必要时用 `will-change`，但不要滥用。

> 与阿里规约、Google HTML/CSS 指南的取舍记录见 `provenance.md`；写代码时不需要读。
