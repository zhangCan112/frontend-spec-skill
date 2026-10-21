# 格式化配置（可选）

这些文件只是可选的格式化配置，不是「一键安装包」。按项目实际情况取用，不要盲目覆盖已有配置。

本 skill 只约束代码，不规定使用哪种 lint 工具，也不提供 `tsconfig.json` 或 lint 配置。项目已有配置时，以项目配置为准。

## 依赖

```bash
pnpm add -D prettier@^3
```

如果项目用 npm / yarn，把 `pnpm add -D` 换成对应命令。

## 复制清单

| 文件 | 放到 | 说明 |
| --- | --- | --- |
| `.editorconfig` | 仓库根目录 | 编辑器统一缩进/换行/编码 |
| `prettier.config.mjs` | 仓库根目录 | 只写团队选择：100 列、单引号，其余用默认值 |

## package.json 脚本

```json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  }
}
```

## 注意事项

- 本目录不提供 ESLint / Stylelint / tsconfig 配置，也不要求项目安装 lint。
- 格式化统一交给 Prettier / EditorConfig；不要手调空格和换行。
- 该基线与 Google HTML/CSS 指南一致：HTML 属性固定双引号、void 元素不自闭合（Prettier 默认行为），CSS 字符串单引号（`singleQuote: true` 同时作用于 CSS）。
- 如果项目已有格式化配置，优先遵守项目配置；本目录的配置只作参考。
