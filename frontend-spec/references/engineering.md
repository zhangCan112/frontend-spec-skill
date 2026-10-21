# 工程规范：目录、命名、导入、拆文件

> 目标：让代码的「位置」可预测。看到一个新文件，知道它该放哪；看到一条 import，知道依赖方向；看到一个文件，知道它为什么存在。

> 级别：**红线**=顶部「本域红线」+ 正文里的「禁止/必须/不得」；其余未标注的祈使句=**默认**（破例说明理由）；标「约定」=按项目/团队。

## 本域红线

- `app → features → shared`；features 不互相 import；`shared` 不 import `features` / `app`。
- 禁止循环依赖。
- `shared/` 里不放业务规则。
- 不用 `utils.ts` / `helpers.ts` / `common.ts` 这类垃圾桶文件。

## 1. 目录结构

单应用、非 monorepo，默认使用 **Feature-first**：

```text
src/
  app/                  # 应用壳：启动、路由、全局 provider
    main.ts
    router.ts
  features/             # 业务功能域，每个 feature 自包含
    user/
      components/
      styles/
      api.ts
      types.ts
      index.ts          # 可选：feature 的公开 API
  shared/               # 跨 feature 的通用能力，不含业务规则
    components/         # 共享组件；组件样式和组件放在一起
    utils/              # 按能力命名文件：date.ts、string.ts、url.ts
    types/
  styles/               # 全局 reset、token、字体、第三方覆盖
    reset.css
    tokens.css
    base.css
    utilities.css
    vendor-overrides.css
  assets/               # 图片、字体等静态资源
  types/                # 全局环境/第三方类型声明
```

### 1.1 依赖方向

```text
app      → features → shared
app      → shared
features → shared
shared   → 无（不得反向依赖 features/app）
features A ↛ features B
```

- `app` 可以 import `features` 和 `shared`。
- 一个 feature 只能 import 自己的子树和 `shared`；不得 import 另一个 feature。
- `shared` 不得 import `features` 或 `app`。
- 禁止循环依赖。发现循环时，先把真正共享的部分下沉到 `shared`，或重新划边界。
- 跨 feature 复用：先判断是否真的通用；通用则下沉到 `shared`，否则通过 `app` 层组合。

### 1.2 目录职责

| 目录 | 放什么 | 不放什么 |
| --- | --- | --- |
| `app/` | 启动、路由、全局 provider、布局壳 | 具体业务规则 |
| `features/<name>/` | 该功能域的组件、样式、API、类型 | 其他 feature 的逻辑 |
| `shared/` | 真正跨域复用的组件/工具/类型 | 只在一个 feature 用的东西 |
| `styles/` | reset、token、全局排版、第三方覆盖 | 组件私有样式 |
| `assets/` | 图片、字体、图标源文件 | 运行时生成的资源 |
| `types/` | 全局声明、环境类型 | 业务类型（放 feature 内） |

### 1.3 什么时候不用 Feature-first

- 项目很小、只有一个功能域：可以用扁平的 `src/components`、`src/utils`，但一旦出现第二个域就迁移。
- 已有固定目录：优先遵守现有结构，不要为了规范重排整个仓库。
- monorepo：按 package 划分边界，本文的 feature/shared 规则下沉到单个 package 内。

## 2. 文件与目录命名

统一 **kebab-case**：

```text
user-card.ts
user-card.css
user-card.test.ts
user-card.types.ts
use-user.ts
```

- 目录名也用 kebab-case：`user-profile/`。
- 不用空格、下划线、大写字母。
- 不用 `index.ts` 作为「万能出口」；只有 feature 的公开 API 可以用。
- 不用 `utils.ts` / `helpers.ts` / `common.ts` 这种垃圾桶文件；按能力命名：`date.ts`、`string.ts`、`url.ts`。
- 目录可以叫 `shared/utils/`，但里面的文件仍要按能力命名，不要变成第二个垃圾桶。
- 类型文件用 `.types.ts`；测试用 `.test.ts`；样式文件用 `.css`，与组件同名。
- 文件名要能回答「它做什么」，不要用 `data.ts`、`temp.ts`、`new.ts`。
- 一个文件围绕一个主题组织；可以导出多个紧密相关的符号，但不要变成杂物间。

## 3. 导入

### 3.1 路径别名

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@shared/*": ["./src/shared/*"]
    }
  }
}
```

TypeScript 7 已移除 `baseUrl`，`paths` 直接写相对路径（`./src/*`）。

- `@/*` 用于 `app` / `features` 内部跨目录引用。
- `@shared/*` 明确表示「跨域共享」。
- feature 内部优先相对路径（`./`、`../`）。
- 禁止 `../../../` 这种深层相对路径；超过两层就用别名。
- 不要为每个 feature 建别名；别名越多，依赖方向越模糊。

### 3.2 导入顺序

```ts
// 1. Node/运行时内置
import { readFile } from 'node:fs/promises';

// 2. 外部依赖
import { z } from 'zod';

// 3. 内部别名
import { Button } from '@shared/components/button';
import { formatDate } from '@shared/utils/date';

// 4. 相对路径
import { UserCard } from './user-card';
import { User } from './types';

// 5. 样式/资源
import './user-card.css';
```

- 组间空一行；组内按字母序（工具可自动排）。
- 导入写法（是否用 `import type`、命名空间还是解构、重命名）见 `typescript.md` §7；这里只约束顺序与路径。

## 4. 导出

导出写法（named 优先、`export default` 的适用范围、禁可变导出、容器类、`export type` 转发）见 `typescript.md` §7。这里只约束结构：

- feature 的 `index.ts` 只导出公开 API；内部实现不通过 barrel 暴露。
- 不要 `export *`（feature 公开 API 除外）。
- 不要在 import 时直接 export；先 import，再显式 export。
- 不要让 `shared` 的 barrel 把整个目录暴露出去；按文件导入。

## 5. 拆文件：按职责，不按行数

一个文件只承担一个变化原因。出现以下信号就拆：

- 文件名需要用「and」连接：`user-and-order.ts`。
- 一个文件有多个不相关的导出，且它们不共享私有实现。
- 修改 A 功能时，经常需要动到 B 功能的代码。
- 文件里出现多组 import，来自不同领域。
- 阅读时需要反复上下跳转才能理解一条路径。
- 行数不是问题，问题是一个文件里塞了多个变化原因。超过 300 行时，先检查是否混入了多个职责；不要为了行数把内聚逻辑切碎。

拆分方式：

- 按「变化原因」拆，不按「代码长度」拆。
- 先拆出纯函数/类型/常量，再拆 UI 或编排。
- 共享代码下沉到 `shared` 前，确认至少有两个真实使用方；不要为假设中的复用提前抽象。
- 拆完后，公开 API 要更窄，不是更宽。

函数拆分的细则（早返回、嵌套层数、参数个数、行数提醒）见 `typescript.md` §5。

> 与阿里规约的取舍记录见 `provenance.md`；写代码时不需要读。
