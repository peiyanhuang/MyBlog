---
layout: post
title:  typeScript 配置
date:   2018-02-10 19:59:00 +0800
categories: TypeScript
tag: TypeScript
---

* content
{:toc}

### 1. 安装使用

npm 安装:

```bash
> npm install -g typescript
```

编译方式:

```bash
tsc ***.ts
```

输出结果为一个`***.js`文件，它包含了和输入文件中相同的JavsScript代码。

### 2. 配置文件 tsconfig.json

[tsconfig.json](https://www.tslang.cn/docs/handbook/tsconfig-json.html)

TypeScript 使用 `tsc` 来编译 `.ts` 文件。

- 不带任何输入文件的情况下调用 `tsc`，编译器会从当前目录开始去查找 `tsconfig.json` 文件，逐级向上搜索父目录。
- 不带任何输入文件的情况下调用 `tsc`，且使用命令行参数 `--project`（或-p）指定一个包含 `tsconfig.json` 文件的目录。

当命令行上指定了输入文件时，`tsconfig.json` 文件会被忽略。

```json
{
    "compilerOptions": {
        "module": "system",
        "noImplicitAny": true,
        "removeComments": true,
        "preserveConstEnums": true,
        "outFile": "../../built/local/tsc.js",
        "sourceMap": true
    },

    // 使用 include 和 exclude 属性
    "include": [
        "src/**/*"
    ],
    "exclude": [
        "node_modules",
        "**/*.spec.ts"
    ],

    // 或者使用 files 属性
    "files": [
        "index.ts",
        "sys.ts",
        "types.ts",
    ]
}
```

#### 2.1 compilerOptions

`compilerOptions`: 编译规则；若被忽略，这时编译器会使用默认值。

常用的选项如下：

-`target`: 指定ECMAScript目标版本 "ES3"（默认）, "ES5", "ES6", "ES2015", "ES2016", "ES2017"或"ESNext"。

-`module`: 指定生成哪个模块系统代码： "None"， "CommonJS"， "AMD"， "System"， "UMD"， "ES6"或 "ES2015"。

-`removeComments`: 删除所有注释，除了以 `/!*` 开头的版权信息，默认 `false`。

-`sourceMap`: 生成相应的 `.map` 文件，默认 `false`。

-`sourceRoot`: 指定TypeScript源文件的路径，以便调试器定位。

-`outDir`: 输出目录。

完整配置项请查看[这里](https://www.tslang.cn/docs/handbook/compiler-options.html)

#### 2.2 files 和 include、exclude

`files` 指定一个包含相对或绝对文件路径的列表。

`include` 和 `exclude` 属性指定一个文件 `glob` 匹配模式列表。 支持的 `glob` 通配符有：

-`*`: 匹配0或多个字符（不包括目录分隔符）

-`?`: 匹配一个任意字符（不包括目录分隔符）

-`**/`: 递归匹配任意子目录

如果一个 `glob` 模式里的某部分只包含`*`或`.*`，那么仅有支持的文件扩展名类型被包含在内（比如默认`.ts`，`.tsx`，和`.d.ts`， 如果 `allowJs`设置能`true`还包含`.js`和`.jsx`）。

如果 `files` 和 `include` 都没有被指定，编译器默认包含当前目录和子目录下所有的TypeScript文件（.ts, .d.ts 和 .tsx）。如果指定了 `files`或`include`，编译器会将它们结合一并包含进来。

使用 `outDir` 指定的目录下的文件永远会被编译器排除，除非你明确地使用 `files` 将其包含进来。通过 `files` 属性明确指定的文件却总是会被包含在内，不管 `exclude` 如何设置。