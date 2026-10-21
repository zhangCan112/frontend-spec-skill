# HTML / 模板规范

> HTML 不是「div 拼图」。先选对元素，再谈样式和脚本；语义和可访问性是读代码的人（以及屏幕阅读器）最直接的信息。

> 级别：**红线**=顶部「本域红线」+ 正文里的「禁止/必须/不得」；其余未标注的祈使句=**默认**（破例说明理由）；标「约定」=按项目/团队。

## 本域红线

- 必须有 `<!doctype html>`、`<html lang>`、`<meta charset="utf-8">`、`<meta name="viewport">`、`<title>`。
- 图片有 `alt`；表单控件有 `<label>`；按钮用 `<button>`，导航用 `<a>`。
- 模板中的不可信输入必须转义；`| safe` 只在已 sanitize 后使用。
- 不用内联事件处理器（`onclick=`）；不用 `javascript:` URL。
- 不通过 `user-scalable=no` 禁用缩放。
- 不用内联 `style` 属性写表现；表现归 CSS，行为归 JS。

## 1. 范围

- 手写 HTML、Nunjucks/EJS 等模板语言。
- 不覆盖 JSX/TSX（框架专属规则另开 skill）。
- 目标是现代浏览器最近两个版本；不兼容 IE11。

## 2. 文档骨架

```html
<!doctype html>
<html lang="zh-CN">
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>页面标题 - 站点名</title>
  <meta name="description" content="一句话说明页面内容">
  <link rel="stylesheet" href="/styles/reset.css">
  <link rel="stylesheet" href="/styles/tokens.css">
  <link rel="stylesheet" href="/styles/base.css">
  <link rel="stylesheet" href="/styles/utilities.css">
  <body>
    <a class="app-skip-link" href="#main">跳到主要内容</a>
    <header>...</header>
    <main id="main">...</main>
    <footer>...</footer>
    <script type="module" src="/app/main.js"></script>
  </body>
</html>
```

规则：

- `<!doctype html>` 必须在第一行；用小写。
- `<html lang="zh-CN">` 必须有，值和内容一致；局部语言变化用局部 `lang`。
- `<meta charset="utf-8">` 必须在文档最前部，越靠前越好；文件本身存成无 BOM 的 UTF-8。
- `viewport` 不要加 `user-scalable=no`；不要禁用缩放。
- `<title>` 有且只有一个；每个页面标题唯一。
- `<meta name="description">` 一句话说明页面，不堆关键词。
- CSS 放 `<head>`；JS 用 `type="module"` 或 `defer`，不要阻塞渲染。
- `type="module"` 是 ES Module 的必需属性，不属于「省略 type」的范畴。
- 引用样式表和脚本省略 `type` 属性：HTML5 默认就是 `text/css` / `text/javascript`。
- 跨域资源 URL 省略协议头：`src="//cdn.example.com/x.js"`，避免混合协议；同源资源用根相对路径。

### 2.1 省略可选标签

默认省略可选的闭合标签：`</li>`、`</p>`、`</td>`、`</th>`、`</tr>`、`</option>`、`</dt>`、`</dd>` 等；`<head>` 开标签同样可省（`meta`/`title`/`link` 直接置于 `<html>` 下，如上面的骨架示例）。

三个例外：

- `<html>` 标签保留：它是 `lang` 属性的载体，省了就没法声明页面语言（a11y 红线）。
- `<body>` 开标签保留：长文档里 head/body 的边界是重要的导航锚点，隐式 body 会让结构难找。
- 省略让结构变得难读时（如深嵌套表格），宁可写全；和文件内现有风格保持一致优先。

```html
<!-- good：省略可选闭合标签 -->
<ul>
  <li>Apples
  <li>Oranges
</ul>

<table>
  <thead>
    <tr>
      <th scope="col">Income
      <th scope="col">Taxes
  <tbody>
    <tr>
      <td>$5.00
      <td>$4.50
</table>
```

- 省略只限 HTML 规范明确允许的标签；不确定就不省，合法性优先。
- 模板里循环生成的列表同样适用；注意省略 `</li>` 后循环体的缩进要能看出层级。

## 3. 语义化

优先选「表达含义」的元素，而不是「默认样式像」的元素：

| 场景 | 用 | 不用 |
| --- | --- | --- |
| 页面主内容 | `<main>` | `<div class="main">` |
| 导航 | `<nav>` + `<ul>` | `<div class="nav">` |
| 独立内容块 | `<article>` | `<div>` |
| 主题分组 | `<section>` + 标题 | `<div>` |
| 补充内容 | `<aside>` | `<div>` |
| 按钮 | `<button type="button">` | `<div onclick>` |
| 链接 | `<a href>` | `<span onclick>` |
| 列表 | `<ul>/<ol>/<li>` | `<div>` 堆叠 |
| 表格 | `<table>` + `<caption>` + `<th scope>` | `<div>` 网格 |
| 图片说明 | `<figure>` + `<figcaption>` | `<div>` |

- 标题层级按内容结构：`h1` → `h2` → `h3`，不要为了字号跳级。
- 一个页面一个 `h1`；`main` 只有一个。
- 不用 `<br>` 做布局；不用 `<table>` 做页面布局。
- 不用 `<div>` / `<span>` 绑定点击事件；可交互就用 `button` / `a`。
- `button` 默认 `type="submit"`，在表单里不提交时写 `type="button"`。
- `a` 只用于导航；`href` 必须有意义，不用 `href="#"` 占位。

## 4. 属性与格式

- 标签名、属性名、属性值（除 text/CDATA）全部小写；CSS 选择器、属性名同理。
- 属性值用**双引号**。
- void 元素（`img`、`br`、`hr`、`meta`、`link`、`input`、`source` 等）**不自闭合**：写 `<img src="...">`，不写 `<img src="..." />`。HTML5 语法下斜线没有意义，Prettier 默认也不加。
- 布尔属性只写名字：`disabled`、`checked`、`required`、`selected`。
- 自定义属性用 `data-*`。
- 不在 HTML 里写内联 `style`；动态样式用 class 或 CSS 变量。
- 不用内联事件处理器：`onclick="..."` 一律禁止。
- 不用实体引用：文件是 UTF-8，直接写字符（`€`、`—`、`"`），不写 `&mdash;`、`&rdquo;`。唯一例外：HTML 里有特殊含义的字符（`<`、`&`）和不可见控制字符（如不换行空格 `&nbsp;`）。
- class 用 kebab-case；多个类按「block → variant → state」排列：`class="user-card user-card-primary is-disabled"`。
- 属性顺序建议：`id` → `class` → `name` → `data-*` → `src`/`href` → 其他 → `aria-*`。
- 注释写 `<!-- 注释 -->`，注释符和内容之间一个空格。
- 块级、列表、表格元素各自另起一行，子元素缩进——不管 CSS 把 `display` 改成了什么。
- 删除行尾空格；它们只会制造无意义的 diff。

```html
<!-- good -->
<button type="button" class="app-button" data-action="save" disabled>保存</button>
<img src="/avatar.webp" alt="张三的头像" width="40" height="40">
```

- 尽量写合法的 HTML（W3C validator 可验证）；合法性是可度量的质量基线，能暴露拼写错误的标签、放错位置的元素。唯一让步：文件大小导致的确切性能目标。

## 5. 表单

```html
<form action="/users" method="post">
  <div class="app-form-field">
    <label for="email">邮箱</label>
    <input
      id="email"
      name="email"
      type="email"
      autocomplete="email"
      required
      aria-describedby="email-error"
    >
    <!-- 校验失败后向这个 live region 填入错误文案 -->
    <p id="email-error" class="app-form-field-error" aria-live="polite"></p>
  </div>

  <fieldset>
    <legend>通知方式</legend>
    <label><input type="radio" name="notify" value="email" checked> 邮件</label>
    <label><input type="radio" name="notify" value="sms"> 短信</label>
  </fieldset>

  <button type="submit">保存</button>
</form>
```

规则：

- 每个控件都有 `<label for="...">`，`for` 和 `id` 对应。
- 不用 `placeholder` 代替 label；placeholder 只做示例。
- 用正确的 `type`：`email`、`tel`、`url`、`number`、`date`、`password`。
- 用 `autocomplete` 帮助用户；常见值：`name`、`email`、`tel`、`street-address`、`current-password`、`new-password`。
- 移动端用 `inputmode` 指定键盘：`numeric`、`decimal`、`tel`、`search`。
- 错误信息用 `aria-describedby` 关联到控件；动态错误用 `role="alert"`，但不要滥用。
- 单选/多选组用 `<fieldset>` + `<legend>`。
- 必填用 `required`；不要只靠星号文字。
- 不要用 `disabled` 隐藏需要提交的值；用 `readonly` 或隐藏字段。

## 6. 可访问性（WCAG 2.2 AA 关键项）

### 6.1 图片与多媒体

```html
<!-- 有信息量的图片 -->
<img src="/chart.webp" alt="2026 年第一季度销售额增长 12%" width="640" height="360">

<!-- 装饰图片 -->
<img src="/divider.svg" alt="" width="1" height="24">
```

- 有信息量：`alt` 描述图片传达的信息，不是「图片」。
- 纯装饰：`alt=""`；不要省略 `alt`。
- 视频、音频、`canvas` 动画等多媒体提供替代内容：对白、字幕。
- 图标按钮：给按钮 `aria-label`，图标本身 `aria-hidden="true"`。
- 图片写 `width` / `height`，避免布局抖动。
- 首屏以下图片用 `loading="lazy"`；首屏主图不要 lazy。

### 6.2 键盘与焦点

- 所有交互都能用键盘完成。
- 不用正数 `tabindex`；需要调整顺序就改 DOM 顺序。
- 提供「跳到主要内容」链接。
- 模态框打开时把焦点移入，关闭时还给触发元素；焦点不能跑到模态框外。
- 不要用 `onmouseover` 作为唯一交互。

### 6.3 文本与颜色

- 正文对比度至少 4.5:1；大字号至少 3:1。
- 不用颜色作为唯一状态提示；配合文字/图标。
- 支持 200% 缩放和 320px 宽度重排；不要出现横向滚动。
- 不用图片代替文字；必须用时给替代文本。

### 6.4 动态内容

- 局部更新用 `aria-live="polite"`；紧急错误才用 `assertive`。
- 加载状态给 `aria-busy="true"`。
- 展开/收起用 `aria-expanded`；当前项用 `aria-current`。
- 原生语义够用时，不要加多余的 `role` / `aria-*`。

### 6.5 目标尺寸

- 点击目标至少 24×24 CSS px（WCAG 2.2 AA）；移动端建议 44×44。
- 相邻目标之间留出间距，避免误触。

## 7. 资源加载与性能

- CSS 用 `<link rel="stylesheet">` 放 `<head>`；不要用 `@import`。
- JS 用 `<script type="module">` 或 `<script defer>`；不要阻塞解析。
- 需要提前建连的第三方域名用 `preconnect` / `dns-prefetch`，但不要滥用。
- 关键资源才 `preload`；不要给所有资源加。
- 图片用 `srcset` / `sizes` 做响应式；优先 WebP/AVIF。
- 首屏主图可以 `fetchpriority="high"`；不要给所有图片加。
- 视频加 `preload="metadata"`；自动播放必须静音，并提供暂停。
- 不在 HTML 里内联大段 CSS/JS；必要时用 nonce/hash 配合 CSP。

## 8. 模板语言

以 Nunjucks 为例，EJS 同理：

```njk
{{ user.name }}
{{ tags | join(', ') }}

{# 不可信内容默认转义 #}
<p>{{ description }}</p>

{# 只有 sanitize 之后才能 safe #}
<div>{{ sanitizedHtml | safe }}</div>
```

规则：

- 变量、过滤器、关键字两侧留空格：`{{ user.name }}`、`{{ tags | join(', ') }}`。
- 默认开启自动转义；不要全局关闭。
- `| safe` / `<%- %>` 只在内容已经 sanitize 后使用。
- 不把用户输入拼进 `class`、`href`、`style` 等属性而不转义。
- 不在模板里做复杂业务计算；在 TS 层算好再传进来。
- 循环里注意 `loop.index` 等变量的作用域；不要生成重复 `id`。
- TODO 只用 `TODO` 一种标记（不用 `@@` 等变体），格式 `TODO(联系方式): 处理内容`：

```njk
{# TODO(john.doe): 重新处理水平居中 #}
```

```njk
{# good：URL 需要应用层的协议白名单校验，不能只靠 HTML 转义 #}
<a href="{{ user.profileUrl | safeUrl }}" class="app-link">{{ user.name }}</a>

{# bad：直接拼 URL，可能被 javascript: 注入 #}
<a href="{{ user.profileUrl }}">个人主页</a>
```

`safeUrl` 是应用层 helper：只允许 `http:` / `https:` / `mailto:` 等白名单协议，其他一律替换为 `#` 或空值。

## 9. 安全

- 不用 `innerHTML` 插入不可信内容；用 `textContent` 或先 sanitize。
- 不用 `javascript:` URL；不用内联事件处理器。
- `target="_blank"` 配合 `rel="noopener noreferrer"`。
- 用户生成内容（UGC）必须经过 HTML sanitize；Markdown 渲染也要过滤。
- 设置 CSP 时避免 `unsafe-inline`；用 nonce/hash。
- 表单提交做 CSRF 防护（由后端/框架负责，但前端不要绕过）。

## 10. 关注点分离

结构（HTML）、表现（CSS）、行为（JS）严格分离，相互影响最小：

- 文档和模板只放结构；表现性的东西移到样式表，行为性的东西移到脚本。
- 不写内联 `style`、内联事件处理器（已在红线）。
- 尽量减少文档/模板引用的样式表与脚本数量。
- 改 HTML 模板的成本总是高于改 CSS/JS；把会变的东西从 HTML 里挪出去。

> 与阿里规约、Google HTML/CSS 指南的取舍记录见 `provenance.md`；写代码时不需要读。
