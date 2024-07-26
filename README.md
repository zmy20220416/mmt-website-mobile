# 前端通用工程化脚手架 eslint + prettier + stylelint + husky 项目规范搭建

## eslint

vite 在安装 react 时就带了 eslint 的很多依赖，省去了对 eslint 的配置，其他脚手架可以参考 vite 的 eslint 规则进行配置。

### eslint 忽略校验的文件

```.eslintignore
*.sh
node_modules
dist
*.woff
*.ttf
.vscode
.idea
.husky
.local
/bin
mock

# cache
.eslintcache
.stylelintcache
```

### eslint 校验缓存

缓存是为了减少 eslint 校验的时间，若项目过于庞大，每次都对整个项目进行 eslint 校验是相对浪费时间的。

修改 `package.json`，在原命令的基础上添加 `--fix --cache` 自动修复并缓存校验结果。

意思就是 eslint 校验 `ts,tsx` 文件，并尝试自动修复这些文件的错误，且 `warning` 警告级别的数量需要为 0（`error` 级别的无法通过校验，也就是不能出现错误和警告），最终结果缓存到项目中，下次就可以忽略未变动的文件校验。

```json
{
  "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0 --fix --cache"
}
```

### 自定义 eslint 规则

eslint 默认在 TS 中不允许出现 `any`，可以通过在 `.eslintrc.cjs` 中添加以下内容：

```cjs
module.exports = {
  // ...
  rules: {
    // ...
    '@typescript-eslint/no-explicit-any': ['off'],
  },
};
```

就是关闭对不允许 `any` 这个规则的校验，其他的约束结果不合理，也可以选择 `off` `warn` `error` 等不同等级。

## prettier

### 安装相关依赖

安装三个包 `prettier eslint-config-prettier eslint-plugin-prettier`

```bash
bun add prettier eslint-config-prettier eslint-plugin-prettier
```

- prettier：代码格式化插件，统一代码格式
- eslint-config-prettier：解决 eslint 和 prettier 中的规则冲突，禁用所有与代码格式有关的 eslint 规则。具体来说，它会关闭所有可能与 prettier 格式化冲突的 eslint 规则，以确保 prettier 的格式化结果不会被 eslint 检查所干扰
- eslint-plugin-prettier：prettier 的格式化规则集成到 eslint 中，使得 prettier 的格式化规则作为 eslint 规则的一部分进行简介，让 eslint 可以识别到 prettier 的不规范写法，从而提示警告或者错误

### 配置 prettier

### 集成到 eslint 中

可以结合文档进行配置 [集成 prettier 到 eslint 中](https://github.com/prettier/eslint-plugin-prettier)

完成安装后，需要让 eslint 集成 prettier 的这些规则，让 eslint 可以提示给开发者 prettier 的不规范代码，在 `.eslintrc.cjs` 中添加如下内容：

```cjs
module.exports = {
  // ...
  extends: [
    // ...
    'plugin:prettier/recommended',
  ],
  rules: {
    // ...

    'prettier/prettier': 'warn', // 让 eslint 提示 prettier 错误格式的级别 warn | error
    'arrow-body-style': 'off', // eslint-plugin-prettier 建议关闭，原因可查看文档说明
    'prefer-arrow-callback': 'off',
  },
};
```

### prettier 规则

如果每个人的 prettier 格式化规则不一样，也会导致代码风格迥异，此时为了统一项目的整体代码风格，可以在根目录新建 `.prettierrc.cjs`，这样编辑器会优先读取该文件下的配置。

```cjs
module.exports = {
  semi: true,
  singleQuote: false,
  printWidth: 100,
  jsxSingleQuote: false,
  useTabs: false,
  tabWidth: 2,
};
```

## stylelint

### 安装 stylelint 相关插件

```bash
bun add stylelint stylelint-config-standard stylelint-order stylelint-config-recess-order stylelint-config-css-modules -D
```

- stylelint：css样式校验的基础插件
- stylelint-config-standard：stylelint 推荐的 css 校验规则
- stylelint-order：css 排序，可自定义 css 排序规则
- stylelint-config-recess-order：现成的 css 排序规则，不用再去手写
- stylelint-config-css-modules：识别 `css modules` 的样式

### stylelint 配置

在根目录下新建 `.stylelintrc.cjs` 文件，写入以下内容：

```cjs
module.exports = {
  plugins: ['stylelint-order'],
  extends: [
    'stylelint-config-standard',
    'stylelint-config-css-modules',
    'stylelint-config-recess-order',
  ],
};
```

### stylelint 忽略部分文件的校验

新建 `.stylelintignore` 文件， 加入不需要校验的文件：

```.stylelintignore
**/*.js
**/*.ts
**/*.tsx
**/*.jsx

node_modules
dist
```

### 集成 less

vite 经过特殊处理，不需要像 `webpack` 那样安装 `less-loader`，安装 less 即可使用。

```bash
bun add less postcss-less -D
```

由于 stylelint 无法理解 less 的部分语法，所以可能会识别为错误语法，为了防止这种情况，需要安装 `postcss-less`，并在 `.stylelintrc.cjs` 中添加如下内容：

```cjs
module.exports = {
  plugins: [
    /**/
  ],
  extends: [
    /**/
  ],
  customSyntax: 'postcss-less',
};
```

在 `package.json` 中添加 `stylelint` 校验命令：

```json
{
  "scripts": {
    // ...
    "eslint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0 --fix --cache",
    "stylelint": "stylelint \"**/*.{less,css,scss}\" --fix --cache"
  }
}
```

### 集成 postcss autoprefixer 实现自动添加浏览器前缀

```bash
bun add postcss autoprefixer -D
```

新建 `postcss.config.js` 启用 `autoprefixer` 插件

```javascript
export default {
  plugins: {
    autoprefixer: {},
  },
};
```

## 提交规范

- 使用 `pretty-quick` 进行代码提交前的自动公式化，可参考 [官方文档](https://github.com/prettier/pretty-quick#readme)
- 使用 `simple-git-hooks` 进行类 `husky` 的提交方式。

### 安装相关插件

```bash
bun add pretty-quick simple-git-hooks -D
```

#### 配置 git 提交校验

`package.json` 配置

```json
{
  "scripts": {
    // ...
    "eslint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0 --fix --cache",
    "stylelint": "stylelint \"**/*.{less,css,scss,vue}\" --fix --cache",
    "postinstall": "simple-git-hooks"
  },
  "simple-git-hooks": {
    "pre-commit": "npx pretty-quick --staged && bun run eslint && bun run stylelint",
    "commit-msg": "node scripts/verify-commit.js"
  }
}
```

**simple-git-hooks 简单介绍**

- `pre-commit` 是代码 commit 提前前预先对代码进行校验，格式化代码、eslint 校验、stylelint 校验，校验不通过不允许提交
- `commit-msg` 是对 commit 的信息进行校验，执行 scripts 文件夹下的 `verify-commit.ts`，可以参考 [vuejs](https://github.com/vuejs/core/blob/main/package.json) 对 `simple-git-hooks` 的用法

**simple-git-hooks 初始化**

`postinstall` 是让项目在初次安装过程完成自动执行 `npx simple-git-hooks` 命令。

如果初次没有执行，需要手动执行该命令，不执行的话没有 git hook 的钩子校验。

npx 手动初始化

![npx simple-git-hooks](https://img2024.cnblogs.com/blog/3395093/202406/3395093-20240630210543754-39401956.png)

git 拉取下来安装市场自带 .git 文件，会自动执行 `postinstall` 命令进行初始化。

**对 git commit 的信息进行校验**

首先安装 `@types/node`

```bash
bun add @types/node
```

在 `scripts` 文件夹下新增 `verify-commit.ts` 文件，写入如下内容，在这里需要安装 `picocolors`。

```bash
bun add picocolors -D
```

```typescript
import pico from 'picocolors';
import { readFileSync } from 'node:fs';
import path from 'node:path';
import process from 'node:process';

const msgPath = path.resolve('.git/COMMIT_EDITMSG');
const msg = readFileSync(msgPath, 'utf-8').trim();

// by https://github.com/vuejs/core/blob/main/scripts/verify-commit.js
const commitRE =
  /^(revert: )?(feat|fix|docs|dx|style|refactor|perf|test|workflow|build|ci|chore|types|wip|release)(\(.+\))?: .{1,50}/;

if (!commitRE.test(msg)) {
  console.log();
  console.error(
    `  ${pico.white(pico.bgRed(' ERROR '))} ${pico.red(`invalid commit message format.`)}\n\n` +
      pico.red(`  you should submit good message format. Examples:\n\n`) +
      `    ${pico.green(`feat: add a new feature`)}\n` +
      `    ${pico.green(`fix: fixed a logic bug`)}\n\n` +
      pico.red(`or you can install <git-commit-plugin> to complete commit.\n`),
  );
  process.exit(1);
}

/**
-   feat：新增功能。
-   fix：修复 bug。
-   docs：更新文档。
-   dx：改进开发者体验。
-   style：修改样式。
-   refactor：重构代码。
-   perf：性能优化。
-   test：添加或修改测试。
-   workflow：改进工作流程。
-   build：修改构建系统或外部依赖。
-   ci：修改持续集成配置文件或脚本。
-   chore：其他杂项任务。
-   types：修改类型定义文件（如 TypeScript）。
-   wip：进行中的工作，尚未完成。
-   release：发布新版本。
 */
```

## git 提交忽略文件

新建 `.gitignore` 文件， 将你不想提交到 git 仓库的文件在这里写入，支持通配符格式。

```.gitignore
# Logs
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*

node_modules
dist
build
dist-ssr
*.local

# lock 需保留可从此处删除
package-lock.json
yarn.lock
pnpm-lock.yaml

# .vscode/*
!.vscode/extensions.json
.idea
.DS_Store
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?

stats.html

# cache
.eslintcache
.stylelintcache
```

## 统一编码格式 CRLF 或 LF

LF 是 Unix 编码，CRLF 是 windows 编码。

在开发时，绝大部分时候换行符格式都是 `LF`，但是从仓库拉取下来代码后，经常会遇到变成 `CRLF` 格式，此时 prettier 就会报错。

为了避免这种现象，可以从两方面杜绝换行符不一致的问题。

**1. 开发时直接将文件编码换成 `LF` 格式**

在根目录下新建 `.editorconfig` 文件，写入如下内容：

```.editorconfig
# http://editorconfig.org
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf # 重点是这个，指定行尾结束符为 lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false
```

**2. 提交时对换行符进行统一，统一为 LF 行尾符**

新建文件 `.gitattributes`，写入如下内容（这也是 V8 的 `.gitattributes` 写法）

```.gitattributes
# 自动将所有基于文本的文件的行尾统一为 lf
* text=auto eol=lf

# 不修改二进制文件的行尾，被 git 识别为文本文件修改会造成文件损坏
*.png binary
```

## 配置路径别名

现在 `vite.config.ts` 中添加如下内容，让项目识别改别名

```typescript
import { defineConfig, loadEnv } from "vite";
import react from "@vitejs/plugin-react";
import { fileURLToPath } from "node:url";

// https://vitejs.dev/config/
export default defineConfig((mode) => {
  return {
    plugins: [react()],
    resolve: {
      alias: {
        "@": fileURLToPath(new URL("./src", import.meta.url)),
      },
    },
});
```

然后让编辑器识别，需要再向 `tsconfig.json` 或 `tsconfig.app.json` 中添加对应的别名

```json
{
  "compilerOptions": {
    // ...
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

## 动态读取 env 环境变量

### env 内容

可以结合 [vite文档](https://vitejs.dev/guide/env-and-mode.html) 进行使用

这里分为三种 `env` 文件，通用 `.env` 开发环境 `.env.development` 生产环境 `.env.production`

以下是三种 env 的例子：

**.env**

```.env
# 网站标题
VITE_APP_TITLE = "my project"

# port
VITE_PORT = 3000

# open 运行 npm run dev 时自动打开浏览器
VITE_OPEN = false

# 是否删除生产环境 console
VITE_DROP_CONSOLE = true

# 是否生成打包体积分析预览
VITE_REPORT = true

# 是否开启 gzip 压缩
VITE_BUILD_GZIP = false
```

**.env.development**

```.env.development
# proxy 代理 / axios baseURL
VITE_API_BASE_URL = '/devapi'

# 本地环境接口地址
VITE_API_URL = 'http://localhost:8888/myapi'
```

**.env.production**

```.env.production
VITE_API_BASE_URL = '/prodapi'

VITE_API_URL = 'http://xxx:1111/api'
```

**vite.config.ts 读取 env**

使用 `loadEnv` 读取即可，需要注意的是，读取出来的值都是字符串，所以需要进行一些转换

```typescript
import { defineConfig, loadEnv } from 'vite';
import react from '@vitejs/plugin-react';
import { fileURLToPath } from 'node:url';

// https://vitejs.dev/config/
export default defineConfig((mode) => {
  const env = loadEnv(mode.mode, process.cwd());
  const viteEnv = wrapperEnv(env);

  return {
    plugins: [react()],
    resolve: {
      alias: {
        '@': fileURLToPath(new URL('./src', import.meta.url)),
      },
    },

    // proxy 代理
    server: {
      port: viteEnv.VITE_PORT,
      cors: true,
      proxy: {
        [viteEnv.VITE_API_BASE_URL]: {
          target: viteEnv.VITE_API_URL,
          changeOrigin: true,
          rewrite: (path: string) => path.replace(/^\/devapi/, ''),
        },
      },
    },
  };
});

/** 对获取的环境变量做类型转换 */
function wrapperEnv(envConf: Record<string, string>) {
  const ret: Record<string, any> = {};
  for (const envName of Object.keys(envConf)) {
    let realName: any = envConf[envName].replace(/\\n/g, '\n');
    realName = realName === 'true' ? true : realName === 'false' ? false : realName;

    if (envName === 'VITE_PORT') realName = Number(realName);

    ret[envName] = realName;
    process.env[envName] = realName;
  }
  return ret;
}
```

### 配置类型提示

env 的环境变量在 ts 和 tsx 等文件中都可以直接访问，出来 `vite.config.ts`

为了有更好的类型提示，可以在 `vite.env.d.ts` 中添加以下类型

```typescript
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_BASE_URL: string;
  readonly VITE_API_URL: string;
  readonly VITE_PORT: number;
  readonly VITE_DROP_CONSOLE: boolean;
  readonly VITE_BUILD_GZIP: boolean;
  readonly VITE_REPORT: boolean;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

### index.html 标题也是用 env 配置

vite 本身都支持在 html 中读取 env 的环境变量

![HTML Env Replacemnt](https://img2024.cnblogs.com/blog/3395093/202407/3395093-20240701103843307-298970531.png)

## 打包相关内容

### 手动分包

```typescript
import { defineConfig, loadEnv } from 'vite';
import react from '@vitejs/plugin-react';
import { fileURLToPath } from 'node:url';
import { compression } from 'vite-plugin-compression2';
import { visualizer } from 'rollup-plugin-visualizer';

// https://vitejs.dev/config/
export default defineConfig((mode) => {
  return {
    build: {
      outDir: 'dist',
      rollupOptions: {
        output: {
          // Static resource classification and packaging
          chunkFileNames: 'js/chunk-[name]-[hash].js',
          entryFileNames: 'js/[name]-[hash].js',
          assetFileNames: '[ext]/[name]-[hash].[ext]', // 可自定义生成的文件夹名，可接收函数的返回值
        },
      },
    },
  };
});
```

### gzip 压缩，去掉 console，代码体积分析

```bash
bun add rollup-plugin-visualizer vite-plugin-compression2 terser -D
```

- rollup-plugin-visualizer 用于在打包完成后是否生成体积预览页面
- vite-plugin-compression2 基于 vite-plugin-compression 升级的压缩插件
- terser 打包时自动去除 console

配置如下

```typescript
import { defineConfig, loadEnv } from 'vite';
import react from '@vitejs/plugin-react';
import { fileURLToPath } from 'node:url';
import { compression } from 'vite-plugin-compression2';
import { visualizer } from 'rollup-plugin-visualizer';

// https://vitejs.dev/config/
export default defineConfig((mode) => {
  return {
    plugins: [
      react(),
      viteEnv.VITE_REPORT && visualizer(), // 体积分析
      viteEnv.VITE_BUILD_GZIP && compression({ threshold: 1025 }), // 大于 1kb 压缩
    ],
    esbuild: {
      pure: viteEnv.VITE_DROP_CONSOLE ? ['console.log', 'debugger'] : [],
    },
    build: {
      outDir: 'dist',
      // esbuild 打包更快，但是不能去除 console.log，去除 console 使用 terser 模式 npm i terser -D
      // minify: "esbuild",
      minify: 'terser',
      terserOptions: {
        compress: {
          drop_console: viteEnv.VITE_DROP_CONSOLE,
          drop_debugger: true,
        },
      },
    },
  };
});
```

### vite.config.ts 的完整配置

```typescript
import { defineConfig, loadEnv } from 'vite';
import react from '@vitejs/plugin-react';
import { fileURLToPath } from 'node:url';
import { compression } from 'vite-plugin-compression2';
import { visualizer } from 'rollup-plugin-visualizer';

// https://vitejs.dev/config/
export default defineConfig((mode) => {
  const env = loadEnv(mode.mode, process.cwd());
  const viteEnv = wrapperEnv(env);

  return {
    plugins: [
      react(),
      viteEnv.VITE_REPORT && visualizer(),
      viteEnv.VITE_BUILD_GZIP && compression({ threshold: 1025 }),
    ],
    resolve: {
      alias: {
        '@': fileURLToPath(new URL('./src', import.meta.url)),
      },
    },

    server: {
      port: viteEnv.VITE_PORT,
      cors: true,
      proxy: {
        [viteEnv.VITE_API_BASE_URL]: {
          target: viteEnv.VITE_API_URL,
          changeOrigin: true,
          rewrite: (path: string) => path.replace(/^\/api/, ''),
        },
      },
    },

    esbuild: {
      pure: viteEnv.VITE_DROP_CONSOLE ? ['console.log', 'debugger'] : [],
    },
    build: {
      outDir: 'dist',
      // esbuild 打包更快，但是不能去除 console.log，去除 console 使用 terser 模式 npm i terser -D
      // minify: "esbuild",
      minify: 'terser',
      terserOptions: {
        compress: {
          drop_console: viteEnv.VITE_DROP_CONSOLE,
          drop_debugger: true,
        },
      },
      rollupOptions: {
        output: {
          // Static resource classification and packaging
          chunkFileNames: 'js/chunk-[name]-[hash].js',
          entryFileNames: 'js/[name]-[hash].js',
          assetFileNames: '[ext]/[name]-[hash].[ext]', // 可自定义生成的文件夹名，可接收函数的返回值
        },
      },
    },
  };
});

/** 对获取的环境变量做类型转换 */
function wrapperEnv(envConf: Record<string, string>) {
  const ret: Record<string, any> = {};
  for (const envName of Object.keys(envConf)) {
    let realName: any = envConf[envName].replace(/\\n/g, '\n');
    realName = realName === 'true' ? true : realName === 'false' ? false : realName;

    if (envName === 'VITE_PORT') realName = Number(realName);

    ret[envName] = realName;
    process.env[envName] = realName;
  }
  return ret;
}
```
