# TypeScript 规范

> 第一目标：让人读懂。类型是给未来的读者看的，不只是给编译器看的。

> 级别：**红线**=顶部「本域红线」+ 正文里的「禁止/必须/不得」；其余未标注的祈使句=**默认**（破例说明理由）；标「约定」=按项目/团队。

## 本域红线

- 不用无注释的 `any` 逃逸；边界处用 `unknown` + 收窄。
- 不用 `!` 非空断言，除非不变量已在上文证明并注释。
- 枚举必须用 `enum`，禁用 `const enum`；不用 `namespace`、`module`、`require`、三斜杠引用。
- 不用 `as unknown as T` 双重断言；测试 Mock 构造是唯一例外，且必须注释。
- 类型断言必须用 `as`，不用尖括号语法 `<Foo>x`。
- 导出的函数写明返回类型；`catch` 变量用 `unknown`。
- 不用嵌套三元表达式；`===` 优先（`== null` 是唯一例外）。
- 字符串转数字必须用 `Number()` 并检查 `NaN`；禁止一元 `+` 转换；`parseInt` 只用于非十进制且先校验输入。
- 不用 `@ts-ignore`；每条语句显式分号结束，不依赖自动分号插入（ASI）。
- 不用 `#private` 私有字段；可见性用 `private` / `protected` 修饰符。
- 禁止裸 `for-in`；数组迭代优先 `for-of`（`forEach` 的限制见 §5.2）。

## 1. 格式基线

格式交给 Prettier / EditorConfig，不要手调。团队选择只有两项：**单行 100 字符、单引号**；其余用默认值。配置见 `assets/prettier.config.mjs` 和 `assets/.editorconfig`。

- 每条语句显式以分号结束，不依赖 ASI，避免分号插入歧义和工具兼容问题。
- 文件编码 UTF-8。非 ASCII 字符直接写字符本身（`'μs'`）；只有非输出字符才用转义并加注释（`'\ufeff' // BOM`）。

```ts
// good
const user = {
  id: 'u_1',
  name: 'Ada',
};

export function loadUser(id: string): Promise<User> {
  return fetchUser(id);
}
```

## 2. 类型严格性

具体 tsconfig 以项目为准；本 skill 不提供配置文件，但代码必须按严格类型检查来写。

- `strict` 是底线，不讨论。
- 建议开启 `noUncheckedIndexedAccess`：`items[0]` 会变成 `T | undefined`，逼你处理空值。
- 建议开启 `noImplicitOverride`、`noFallthroughCasesInSwitch`，避免继承和 `switch` 的隐式错误。
- 模块边界要干净：`isolatedModules` 能让导入/导出的问题更早暴露。
- 不要开启 `verbatimModuleSyntax`：它会强制使用 `import type`，与本文「类型用常规导入」的规则冲突。
- 不要开启 `erasableSyntaxOnly`：本规范使用 `enum` 和构造函数参数属性，二者都属于不可擦除的运行时语法。
- `noUnusedLocals` / `noUnusedParameters` 能帮忙，但项目已有对应检查时可能重复；按项目配置取舍。
- 不强制 `exactOptionalPropertyTypes`、`noPropertyAccessFromIndexSignature`、`noImplicitReturns`：它们更严格，但在业务代码里摩擦较大，按项目需要再开。

## 3. 命名

| 对象 | 风格 | 例 |
| --- | --- | --- |
| 变量、函数、参数 | camelCase | `userProfile`、`loadUser` |
| 类型、接口、类、枚举、类型参数 | PascalCase | `UserProfile`、`OrderStatus`、`T` |
| 导出的常量、枚举值 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 局部常量 | camelCase | `defaultTimeout` |
| 布尔值 | `is` / `has` / `can` / `should` 前缀 | `isLoading`、`hasPermission` |
| 事件处理 | `handle` + 事件 | `handleSubmit`、`handleKeyDown` |
| 异步动作 | 动词开头 | `fetchUser`、`saveOrder` |
| 文件名 | kebab-case | `user-profile.ts` |
| CSS 类 | kebab-case | `.user-card` |

规则：

- 不用 `I` 前缀：`IUser` → `User`。接口名应体现它存在的目的：为 `TodoItem` 定义序列化契约时，叫 `TodoItemStorage` 而不是 `TodoItemInterface`。
- 不用下划线 `_` 前缀或后缀；也不用单个 `_` 作标识符。解构时用空位跳过：`const [a, , b] = [1, 5, 10];`。
- 不用 `opt_` 之类标记可选的前缀；可选性交给 `?` 和类型。
- 缩写视为一个词：`loadHttpUrl`，不是 `loadHTTPURL`（平台惯例除外，如 `XMLHttpRequest`）。
- 标识符一般不含 `$`；仅为与第三方框架命名惯例保持一致时使用（如 RxJS `Observable` 的 `$` 后缀，团队内保持一致即可）。
- 类型参数用单个大写字母（`T`）或 PascalCase（`TKey`、`TValue`）。
- `CONSTANT_CASE` 表示「使用者不应修改」：包括未深度冻结的对象常量和类的 `static readonly` 属性。
- 常量在运行期会被修改或多次实例化的，一律 camelCase。
- 为已有标识符起局部别名时，别名与原名保持一致，用 `const`（变量）或 `readonly`（类字段）声明。
- 不用无意义的缩写：`usr`、`btn`、`cfg` 只在团队通用时使用。
- 名字要能搜索：避免 `data`、`item`、`temp`、`res` 这种到处都有的词。十行以内的小作用域允许 `i`、`j` 这类短名。
- 布尔值不要用否定式：`isNotReady` → `isReady` 或 `isPending`。
- 同一概念只用一个词：`user` / `member` / `account` 不要混用。

## 4. 类型策略

### 4.1 对象类型用 `interface`

对象契约一律用 `interface`，包括会被 `extends` / `implements` 的形状。`type` 只用于非对象形状：联合、交叉、元组、映射、条件、函数类型、原始类型别名。

```ts
// good：对象契约用 interface
interface UserRepository {
  findById(id: string): Promise<User | undefined>;
}

interface FoodPreferences {
  favoriteIcecream: string;
  favoriteChocolate: string;
}

interface User extends FoodPreferences {
  shoeSize: number;
}

// good：联合/别名/函数类型用 type
type UserId = string;
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };
```

不要为了「统一」把对象写成 `type` 字面量：两种写法几乎等价时，只选 `interface` 一种，避免项目内出现无意义变体。

### 4.2 枚举必须用 `enum`，禁用 `const enum`

枚举类型必须用 `enum` 关键字声明。`const enum` 是另一个独立的语言特性（把枚举对 JS 透明化），禁止使用。

```ts
// good
enum OrderStatus {
  Pending = 'pending',
  Paid = 'paid',
  Shipped = 'shipped',
}

// bad：const enum 是独立特性，行为依赖编译选项
const enum OrderStatus {
  Pending = 'pending',
}
```

- 枚举成员用 `CONSTANT_CASE` 命名。
- 枚举本身不可变，不要为了「不可变」再造 `as const` 对象替代 `enum`。
- 连带约束：tsconfig 不能开启 `erasableSyntaxOnly`（见第 2 节）。

### 4.3 `any` 是边界工具，不是日常类型

```ts
// bad：any 一路流进业务
function getName(payload: any) {
  return payload.user.name;
}

// good：边界用 unknown，立刻收窄
function getName(payload: unknown): string {
  if (!isUserPayload(payload)) {
    throw new Error('Invalid user payload');
  }
  return payload.user.name;
}
```

- 默认用 `unknown`。
- `any` 只允许出现在第三方库边界、迁移中的旧代码，并且必须：
  1. 在代码里注释原因；
  2. 在同一个函数内立刻收窄，不让 `any` 流出去。
- 不用 `as any` 绕过类型检查。
- 测试里构造部分实现的 Mock 时，`any` 可以接受，但要加行级抑制注释并说明理由：

```ts
// 这个测试只需要 BookService 的 get()，其余方法用不到。
// 有意使用不安全的部分实现；tslint:disable-next-line:no-any
const mockBookService = ({ get: () => mockBook } as any) as BookService;
```

### 4.4 断言：优先收窄，少用 `as`

```ts
// bad
const user = data as User;

// good
if (!isUser(data)) {
  throw new Error('Invalid user');
}
const user = data;
```

- `as` 只用于：外部契约已经保证、收窄后编译器仍无法推断、测试中的少量构造。
- 断言必须用 `as` 语法，禁止尖括号语法 `<Foo>x`：`as` 强制断言外层加括号，解析无歧义。
- 给对象字面量标类型时，用类型标记 `: Foo` 而非断言 `as Foo`。接口字段改名后，类型标记能立刻报错，断言不会：

```ts
interface Foo {
  bar: number;
  baz?: string;
}

// bad：接口把 bam 改成 baz 后，这里不会报错
const foo = { bar: 123, bam: 'abc' } as Foo;

// good：多余字段直接报错
const foo: Foo = { bar: 123, baz: 'abc' };
```

- 禁止 `as unknown as T` 双重断言；如果必须，把它隔离在一个有注释的适配函数里。
- 禁止 `!` 非空断言，除非不变量已在上文证明：

```ts
// bad
user!.profile!.name;

// good
if (!user?.profile) {
  return;
}
user.profile.name;
```

- 断言确实安全且理由不明显时，写注释说明为什么可以接受。

### 4.5 用 `satisfies` 校验配置对象

```ts
const routes = {
  home: '/',
  user: '/users/:id',
} as const satisfies Record<string, string>;
```

这样既保留字面量类型，又能在编译期发现拼写/类型错误。

### 4.6 用可辨识联合表达状态

不要用一堆布尔值表示互斥状态。用带 `status` 字段的联合类型，示例见 §4.1 的 `RequestState<T>`；每个状态只携带该状态需要的字段。

### 4.7 `readonly` 和不可变数据

- 不打算修改的字段加 `readonly`；不需要深层不可变。
- 不打算修改的数组参数用 `readonly T[]`。
- 不要为了「性能」到处浅拷贝；先保证语义清楚。

```ts
interface User {
  readonly id: string;
  name: string;
}

function sortUsers(users: readonly User[]): User[] {
  return [...users].sort((a, b) => a.name.localeCompare(b.name));
}
```

### 4.8 `undefined` 与 `null`

- 内部代码优先用 `undefined` 表示「没有」；DOM / 第三方 API 返回 `null` 的场景跟随上下文。
- 不要在同一层混用两者；如果必须转换，在边界处显式转换。
- 禁止定义带 `| null` / `| undefined` 的类型别名。可空性在使用处联合，让空值在出现位置附近被处理：

```ts
// bad：别名把 undefined 层层传递，掩盖空值源头
type CoffeeResponse = Latte | Americano | undefined;

// good
type CoffeeResponse = Latte | Americano;
class CoffeeService {
  getLatte(): CoffeeResponse | undefined { ... }
}
```

- 可选字段 / 可选参数用 `?`，不要手写联合 `| undefined`：`?` 允许调用方直接省略。
- 类的字段尽量不用 `?`，尽量在声明处初始化：`field = '';`。

### 4.9 泛型

- 单个泛型用 `T`；多个用有意义的名字：`TKey`、`TValue`、`TResult`。
- 泛型要有约束，不要写「万能泛型」。
- 不要为了少写一个类型而泛型化；先写具体类型，出现第二个使用方再抽象。
- 不要创建「返回类型只有泛型」的 API；调用这类既有 API 时显式标明类型参数。

```ts
// good
function groupBy<T, TKey extends string>(
  items: readonly T[],
  getKey: (item: T) => TKey,
): Record<TKey, T[]> { ... }
```

### 4.10 避免的类型

- `Object`、`String`、`Number`、`Boolean` 包装类型：用 `object` / `string` / `number` / `boolean`；也不要实例化包装类（`new Boolean(false)` 在布尔表达式里为真）。
- `Function`：写出具体签名，如 `(id: string) => Promise<User>`。
- `{}`：它几乎匹配所有非空值，容易掩盖问题；用具体结构或 `Record<string, unknown>`。
- `object`：只在确实只关心「非原始值」时使用。

### 4.11 类型推导：能推导则省略

`string` / `number` / `boolean` / `RegExp` 字面量和 `new` 表达式能直接推导类型，省略注记：

```ts
// bad：注记对可读性没有任何帮助
const x: boolean = true;
const s: Set<string> = new Set();

// good
const x = true;
const s = new Set<string>();
```

复杂表达式的注记由审查决定。返回类型遵循红线：导出的函数必须标明，局部函数可省略（显式返回类型能让接口变更的影响更快暴露）。

### 4.12 数组与索引类型

- 简单类型（名称只含字母、数字、点）用语法糖 `T[]`；复杂类型用 `Array<T>`。`readonly T[]` 与 `ReadonlyArray<T>` 同理：

```ts
// good
const a: string[] = [];
const b: readonly string[] = [];
const d: Array<string | number> = [];
const e: ReadonlyArray<string | number> = [];

// bad：大括号加中括号难以阅读
const h: { n: number; s: string }[] = [];
```

- 索引类型的键标签要表意：`{ [userName: string]: number }`，不写无意义的 `key`。
- 关联容器优先 `Map` / `Set`：键可以是 string 以外的类型，语义也更明确。键集合静态已知时用 `Record<Keys, ValueType>`。
- 不要用 `Pair` 类风格的接口返回两个值；用元组 `[string, string]` 或内联对象类型 `{ host: string; port: number }`，后者通常更清晰。

### 4.13 映射类型与条件类型：能用最简单的写法就不用类型运算

- 优先用 `interface` + 继承表达类型关系；`Pick<T, K>` 这类运算符让读者在头脑里求值，IDE 的查找引用也看不到里面的属性。
- 一定的重复，好过复杂的类型表达式带来的长期维护成本。
- 确实需要映射/条件类型时再用，并保证同事能看懂。

```ts
// 可用，但读者需要自己求值
type FoodPreferences = Pick<User, 'favoriteIcecream' | 'favoriteChocolate'>;

// 更好：结构一目了然，IDE 支持完整
interface FoodPreferences {
  favoriteIcecream: string;
  favoriteChocolate: string;
}
interface User extends FoodPreferences {
  shoeSize: number;
}
```

### 4.14 展开运算符类型必须匹配

- 创建对象时只能展开对象，创建数组时只能展开可迭代类型；同一个键后面的值覆盖前面的值。
- 禁止展开可能为 `undefined` 的条件表达式：

```ts
// bad：...(cond && foo) 可能展开 undefined
const bar = { num: 5, ...(shouldUseFoo && foo) };

// good
const foo = shouldUseFoo ? { num: 7 } : {};
const bar = { num: 5, ...foo };
```

- 禁止把数组展开进对象字面量（会得到 `{0: 'a', 1: 'b'}` 这种没有 `length` 的东西）。

## 5. 函数与可读性

- 一个函数做一件事；用早返回压平嵌套。

```ts
// bad
function getDiscount(user: User) {
  if (user) {
    if (user.isVip) {
      if (user.years > 3) {
        return 0.8;
      } else {
        return 0.9;
      }
    } else {
      return 1;
    }
  }
  return 1;
}

// good
function getDiscount(user: User): number {
  if (!user.isVip) return 1;
  if (user.years > 3) return 0.8;
  return 0.9;
}
```

- 嵌套不超过 3 层；超过就拆函数或早返回。
- 80 行是提醒，不是硬门；函数需要滚动才能读完时，通常该拆。
- 不用嵌套三元；用 `if` 或查表。
- 不用魔法数字；给常量起名字。
- 布尔参数最多一个；多个布尔参数改成 options 对象，或拆成两个函数。
- 参数超过 4 个用 options 对象。
- 异步优先 `async/await`；并行用 `Promise.all`。
- 不要 floating promise：要么 `await`，要么显式 `void promise` 并注释为什么忽略。
- 不要用 `arguments`；用 rest 参数。
- `++` / `--` 只作推荐避免：不要在表达式里用，单独一行可以接受。
- 不用 `void` 运算符做类型转换；`void promise` 是唯一例外。

### 5.1 声明与表达式

- 顶层具名函数用函数声明 `function foo() { ... }`；不要把函数表达式赋给局部变量（`const x = function () {}`）。TypeScript 不允许重新绑定函数声明，`const` 是多余的。
- 函数表达式一律用箭头函数；只有需要动态重绑 `this` 时才用 `function` 关键字，而常规代码不应重绑 `this`。
- 箭头函数需要用到返回值时才用表达式体；只为执行副作用时用块体：

```ts
// bad：不需要返回值却用了表达式体
myPromise.then((v) => console.log(v));

// good
myPromise.then((v) => {
  console.log(v);
});
```

- 类的属性一般不初始化为箭头函数：它让 `this` 指向变得费解。回调里直接包一层匿名箭头函数调用方法；需要稳定引用做卸载的事件句柄是例外（见 §8.5）。
- 不要持有或传递实例方法的裸引用（`const handler = this.listener`）；调用处直接 `() => this.listener(x)`。

### 5.2 迭代

- 数组迭代用 `for-of`；需要下标时用传统 `for` 或 `entries()`：

```ts
// good
for (const x of someArr) { ... }
for (let i = 0; i < someArr.length; i++) { ... }
for (const [i, x] of someArr.entries()) { ... }
```

- 不用 `forEach`：闭包让编译器的控制流分析失效（可空的 `x` 在回调里会被误判），断点调试也更麻烦。函数式链式转换用 `map` / `filter`，终结迭代用 `for-of`。
- 禁止裸 `for-in`：它遍历原型链上的属性。对象迭代用 `Object.keys()` / `Object.entries()` 的 `for-of`，或 `for-in` + `hasOwnProperty` 过滤：

```ts
// bad
for (const x in someObj) { ... }          // x 可能来自原型链
for (const x in someArray) { ... }        // x 是 string 类型的下标！

// good
for (const [key, value] of Object.entries(someObj)) { ... }
```

### 5.3 类型转换

- 转 `string` 用 `String()` 或模板字符串；转 `boolean` 用 `Boolean()` 或 `!!`；不要用字符串拼接做隐式转换。
- 转 `number` 必须用 `Number()`；解析可能失败时必须检查 `NaN`（注意 `Number('')` 和 `Number(' ')` 返回 `0` 而不是 `NaN`）。
- 解析整数用 `Number()` + `Math.floor` / `Math.trunc`，不用 `parseInt`；`parseInt(s, 10)` 只用于非十进制场景（如十六进制），且调用前先校验输入格式。
- `if` / `while` 条件本身就是布尔上下文，不要再写 `!!x`：

```ts
// bad
if (!!foo) { ... }

// good
if (foo) { ... }
// 显式比较 0 或依赖隐式转换都可以
if (arr.length > 0) { ... }
if (arr.length) { ... }
```

### 5.4 控制流

- 多行控制流必须加大括号；只有能写在一行的 `if` 可以省略。
- `switch` 必须有 `default` 分支（可以为空）；非空 `case` 不允许穿透执行，空 `case` 可以合并：

```ts
switch (x) {
  case Y:
    doSomethingElse();
    break;
  default:
  // 什么也不做。
}
```

## 6. 错误处理

- 抛异常必须用 `new Error(...)`，不是调用 `Error(...)`：与其它对象实例化形式一致。
- `catch` 变量用 `unknown`，先收窄再使用。

```ts
try {
  await saveUser(user);
} catch (error: unknown) {
  if (error instanceof HttpError) {
    logger.warn('save user failed', { status: error.status });
    throw error;
  }
  throw new Error('save user failed', { cause: error });
}
```

- 不要吞错：空 `catch` 是 bug。
- 不要只 `console.log(error)` 然后继续；要么恢复，要么向上抛。
- 给错误加 context，但不要泄露密钥/个人信息。
- 自定义错误用 `class extends Error`，并设置 `name`。

## 7. 模块

- 只用 ESM：`import` / `export`。不用 `require`、`namespace`、`module`、三斜杠引用。
- 不用 `import type { ... }` / `export type { ... }`；类型和值都用常规导入导出，工具链会自动消除仅类型导入。定义类型别名 `export type Foo = ...` 不受影响：

```ts
// bad
import type { Foo } from './foo';
export type { Bar } from './bar';

// good
import { Foo } from './foo';
export { Bar } from './bar';
export type RequestState = ...;
```

- 模块导入与解构导入按场景选择：
  - 从大型 API 导入多个符号，或想要命名空间下的自动补全：`import * as tableview from './tableview'`；
  - 少数高频符号（如测试的 `describe` / `it` / `expect`）：解构导入。
- 重命名导入用于：命名冲突、自动生成的名字、原名不表意（RxJS `from` → `observableFrom`）。
- 具名导出是唯一默认；`export default` 只用于框架/工具硬性要求（配置文件、路由懒加载）。默认导出让导入方随意起名（`import Foo from './bar'` 和 `import Bar from './bar'` 都合法），还容易长成充当命名空间的巨型对象。
- 禁止可变导出（`export let`）：多模块重导出后行为不可预测。确需暴露可变值时导出显式取值器 `export function getFoo()`。

```ts
// good
export function loadUser(id: string): Promise<User> { ... }
export type RequestState = ...;

// avoid
export default loadUser;
export let count = 0;
```

- 不要为命名空间创建装满 `static` 成员的容器类；用具名导出的常量和函数：

```ts
// bad
export class Container {
  static FOO = 1;
  static bar() { return 1; }
}

// good
export const FOO = 1;
export function bar() { return 1; }
```

- 只导出模块外真正使用的符号；公开 API 面积越小越好。
- 避免循环依赖；发现时的处理见 `engineering.md` §1.1。

## 8. 类

- 公开成员省略 `public`；`private` / `protected` 显式写。默认可见性就是 `public`，`public` 修饰符只在「公开且非只读的参数属性」处可写。
- 尽量收紧可见性：只在本文件使用的逻辑可以移出类，做成模块级内部函数。
- 不用 `#private` 私有字段（编译体积与性能问题、抬高编译目标下限、对静态检查无增益）；用 `private`。
- 不打算修改的字段加 `readonly`。
- 构造函数：
  - 调用必须带括号：`new Foo()`，即使无参。
  - 空构造函数、仅转发 `super()` 的构造函数必须省略；含参数属性、参数装饰器、私有构造函数的除外。
  - 类成员初始化优先用参数属性 `constructor(private readonly bar: Bar) {}`，不要手动声明再赋值：

```ts
// bad：样板代码
class Foo {
  private readonly barService: BarService;
  constructor(barService: BarService) {
    this.barService = barService;
  }
}

// good
class Foo {
  constructor(private readonly barService: BarService) {}
}
```

- 非参数属性的字段在声明处初始化，常常可以整个省掉构造函数：`private readonly userList: string[] = [];`。
- 成员声明用 `;` 分隔（与类一致）；内联对象类型（`type X = {...}` 成员、内联字面量）用 `,` 分隔。
- 成员顺序：static 字段 → static 方法 → 实例字段 → constructor → 实例方法 → getter/setter。
- 存取器（getter/setter）：getter 必须是纯函数；两侧都只是直传的平凡存取器不要写，直接 `public` 字段；只有 getter 的考虑改 `readonly` 字段。
- 属性访问风格统一：不要混用 `x.someField` 与 `x['someField']`；不要用方括号语法绕过可见性检查。
- 为外部数据（JSON、协议响应）声明对应接口，访问点类型安全、重命名安全：

```ts
declare interface ServerInfoJson {
  appVersion: string;
  user: UserJson;
}
const data = JSON.parse(serverResponse) as ServerInfoJson;
console.log(data.appVersion);
```

- 优先组合，不要为了复用强行继承。

### 8.5 事件句柄

- 事件由类自身发送、句柄不需要卸载的，用匿名箭头函数：`this.addEventListener('click', () => this.listener())`。
- 句柄必须可卸载的（如 `window` 的事件），用箭头函数属性提供稳定引用，注册和卸载用同一个引用：

```ts
class Component {
  onAttached() {
    window.addEventListener('beforeunload', this.listener);
  }
  onDetached() {
    // 卸载必须移除同一引用，否则内存泄漏
    window.removeEventListener('beforeunload', this.listener);
  }
  // 箭头函数属性自动正确捕获 this
  private listener = () => {
    confirm('Do you want to exit the page?');
  };
}
```

- 不要在注册表达式中 `bind`：每次 `bind` 都创建新的、无法卸载的引用。

## 9. 注释

- 文档（给使用者读的）用 `/** JSDoc */`；实现说明（只和实现细节有关的）用 `//` 行注释。JSDoc 能被编辑器、文档生成器识别。
- 注释解释 **why**，不解释 **what**。
- 导出的函数/类/复杂类型用 JSDoc；但不要复述类型系统能表达的信息：`@param` / `@return` 里不写类型，不写与参数名同义的废话，只在有额外信息时才加标签：

```ts
/**
 * 发送 POST 请求，开始煮咖啡
 * @param amountLitres 煮咖啡的量，注意必须和煮锅的尺寸对应！
 */
brew(amountLitres: number, logger: Logger) { ... }

// bad：没有任何有意义的内容
/** @param fooBarService Foo 应用的 Bar 服务 */
```

- 参数属性（`constructor(private readonly bar: Bar)`）的文档写在构造函数的 JSDoc `@param` 上，编辑器会在两处都显示。
- 不用 `@override`：编译器不强制它，注释与实现容易不一致。
- 同时有装饰器和 JSDoc 时，JSDoc 放在装饰器之前；禁止夹在装饰器和被装饰符号之间。
- `TODO` / `FIXME` 必须带 owner 或 issue：

```ts
// TODO(@ada, #123): 接口支持批量查询后删除这个循环
```

- 不写「增加 1」这种复述代码的注释。
- 注释用中文，标识符用英文。

> 与阿里规约、Google TS 指南的取舍记录见 `provenance.md`；写代码时不需要读。
