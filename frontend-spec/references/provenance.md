# 规约取舍记录

> 维护用文件，写代码时不需要读。它记录本 skill 对 [alibaba/f2e-spec](https://github.com/alibaba/f2e-spec) 与 [Google TypeScript / HTML/CSS 风格指南](https://zh-google-styleguide.readthedocs.io/en/latest/)（中文版，TS 译于 2024-02）每条规则「保留 / 调整 / 删除」的理由，方便以后复盘。

## 总体关系

- 保留：2 空格、100 列、分号、单引号、尾随逗号、HTML 语义化/a11y、CSS 属性顺序、禁 id；`!important` 仅限第三方覆盖。
- 调整：`interface` 优先 → 按场景选 `type`/`interface`；禁 `++`/`void` 降为默认；不设行数硬上限；CSS 改为「组件前缀 + kebab-case」的原生 CSS（不用 BEM/CSS Modules），取代 Sass/Less（初版 camelCase，2026-10 按 Google 改为连字符，见下）。
- 更新：按 TypeScript 7 的类型规则、Prettier 3 的格式约定；不绑定 ESLint / Stylelint 等 lint 工具。

## 工程

| 阿里规约 | 本 skill |
| --- | --- |
| 文件名小写、用 `-` 连接 | 保留，统一 kebab-case |
| 不使用 `index.ts` 作为默认入口 | 收紧为「只有 feature 公开 API 可用」 |
| `export default` 单导出优先 | 改为 named export 优先 |
| 文件最大 1000 行 | 不设硬上限，按职责拆分；300 行是气味信号 |
| 未涉及目录结构 | 补充 Feature-first + 依赖方向 |

## TypeScript

| 阿里规约 | 本 skill |
| --- | --- |
| `interface` 优先 | 按场景选 `type` / `interface` |
| 禁 `enum`（未单列） | 明确禁 `enum`，用 `as const` 联合（2026-10 按 Google 反转，见下） |
| 禁 `namespace` / `require` / 三斜杠 | 保留 |
| 禁 `!`（部分场景） | 保留，并要求证明不变量 |
| 禁 `++` / `void` | 降为推荐，不升为强制 |
| 显式成员可访问性、成员顺序 | 保留 |
| 单行 100、分号、单引号、尾随逗号 | 保留，由 Prettier 落地 |
| 未涉及严格类型检查 | 补充 `strict` + 关键严格选项（按项目配置） |

## CSS

| 阿里规约 | 本 skill |
| --- | --- |
| 2 空格、分号、属性顺序、禁 id、禁 `!important` | 保留 |
| 禁 `@import` | 保留 |
| 嵌套不超过 3 层 | 收紧为「优先扁平，原生嵌套只用于伪类/伪元素」 |
| BEM / 命名方法论（未规定） | 改为「组件前缀 + camelCase」，不用 BEM（2026-10 按 Google 改为 kebab-case，见下） |
| CSS Modules（未规定） | 不使用；靠全局类名约定隔离 |
| 颜色/间距硬编码（未禁止） | 收紧为 token 优先 |
| Sass/Less 均可 | 收敛为原生 CSS |
| 未涉及原生嵌套/容器查询/`@layer` | 补充为可选现代实践 |

## HTML

| 阿里规约 | 本 skill |
| --- | --- |
| doctype、lang、charset、title、语义化、alt | 保留 |
| 2 空格、双引号、小写标签、布尔属性简写 | 保留 |
| CSS 在 head、JS 在 body 末尾 | 调整为 `type="module"` / `defer` |
| 自闭合标签保留 `/>` | 保留（2026-10 按 Google 改为 void 元素不自闭合，见下） |
| 模板变量两侧空格、默认转义 | 保留并补充 URL/属性上下文 |
| 可访问性（基础） | 扩展到 WCAG 2.2 AA 关键项 |
| 未涉及安全 | 补充 XSS、`target="_blank"`、CSP |

## 与 Google 风格指南的取舍记录（2026-10）

引入 Google TypeScript 与 HTML/CSS 风格指南时，原有规则与之有 9 处冲突。决策：**冲突全部跟随 Google**，同时吸收一批无冲突的补强点。

### 跟随 Google 的 9 处冲突

| # | 原规则 | 现规则（Google） |
| --- | --- | --- |
| 1 | CSS/HTML 类名「组件前缀 + camelCase」（`.userCard`） | 连字符分隔（`.user-card`），结构不变（block/element/variant/state/app-/u- 前缀） |
| 2 | 禁 `enum`，用 `as const` 对象替代 | 枚举必须用 `enum`，禁 `const enum`；连带不开 `erasableSyntaxOnly` |
| 3 | 类型导入用 `import type` | 禁用 `import type` / `export type {}`；常规导入，工具链自动消除；连带不开 `verbatimModuleSyntax` |
| 4 | 禁构造函数参数属性 | 优先参数属性 `constructor(private readonly x: X)` |
| 5 | 运行时私有可用 `#field` | 禁 `#private`，只用 `private` 修饰符 |
| 6 | 自闭合标签保留 `/>` | void 元素不自闭合（`<img>`），与 Prettier 默认一致 |
| 7 | 写全所有标签 | 默认省略可选闭合标签（`</li>` 等）；`<head>` 开标签可省 |
| 8 | CSS 声明按语义分组（定位→盒模型→排版→外观） | 按字母序 |
| 9 | URL 协议未规定 | 跨域 URL 省略协议头（`//cdn.example.com/...`） |

### 跟随 Google 但保留的例外（有明确理由）

- `<html>` 标签不省：它是 `lang` 属性的载体，省略违反 a11y 红线（Google 指南成文早于 a11y 普及）。
- `<body>` 开标签不省：长文档中 head/body 边界是导航锚点，隐式 body 降低可读性。
- `forEach` 吸收为**默认规则**而非红线：数组迭代用 `for-of`，函数式链式转换仍可用 `map`/`filter`；Google 是全面禁止，这里放宽一档。
- `import * as ns` 从「禁止」放开为「按场景选择」（大 API 多符号用命名空间导入），跟随 Google 5.3.1。

### 吸收的无冲突补强点

TS：禁 `Array()` 构造函数；数字转换 `Number()` + NaN 检查、禁一元 `+`、`parseInt` 限非十进制；`if` 条件里不写 `!!x`；禁裸 `for-in`；`switch` 必须 `default`；`new Error()` / `new Foo()` 括号；空构造省略；展开运算符类型匹配；对象字面量用类型标记不用 `as`；禁可空类型别名；`?` 优先于 `|undefined`；索引签名键表意 + 优先 `Map`/`Set`；`T[]` vs `Array<T>`；禁容器类；禁 `export let`；禁 `@ts-ignore`；映射/条件类型最小化；元组替代 `Pair`；缩写视为一个词；类型推导可省略注记；断言用 `as` 不用尖括号；不混用 `x.a`/`x['a']`；JSDoc 言之有物、`@param` 不复述类型、不用 `@override`、JSDoc 在装饰器前；UTF-8 直接写非 ASCII 字符；getter 必须纯；箭头函数表达式体只在需要返回值时用。

CSS：`0` 省单位、前导 0 省略、3 位十六进制；简写属性优先；避免表象命名（`.button-green`）；禁类型选择器限定（`ul#nav`、`div.error`）；避免 hacks 与 UA 检测；分段注释；CSS 字符串单引号、`url()` 不加引号。

HTML：不用实体引用（直接写 UTF-8 字符，`<`/`&`/控制字符除外）；合法 HTML 可用 W3C validator 验证；TODO 统一格式 `TODO(联系方式): 内容`；关注点分离成节；删除行尾空格。

### 仍然比 Google 更严的（保留）

- 导出的函数必须写明返回类型（Google：作者自由决定）。理由：接口变更的影响更快暴露。
- `catch` 变量必须 `unknown`（Google 未规定）。
- TODO 要求带 owner 或 issue（Google 只要求联系方式）。
- 类名仍要求「组件前缀」体系（Google 只给了一般性命名原则）。
- 禁 `as unknown as T` 双重断言（Google 测试 Mock 场景开放，这里保留测试例外但要求注释）。
