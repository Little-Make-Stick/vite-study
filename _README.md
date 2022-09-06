#### 下载 vite

```powershell
npm i vite -g
```

#### 使用 vite 初始化一个空项目

npm init vite@latest(使用最新的 vite) my-vue-app(项目名) --template vue(使用的模板)

```powershell
# npm 6.x
npm init vite@latest my-vue-app --template vue

# npm 7+, 需要额外的双横线：
npm init vite@latest my-vue-app -- --template vue

cd my-vue-app

npm i

npm run dev
```

#### 配置 esbuild 选项

```js
// vite.config.js
import { defineConfig } from "vite";

export default defineConfig({
  esbuild: {
    // 调用该函数h
    jsxFactory: "h",
    // 更改 JSX 片段以外的方式使用组件
    jsxFragment: "Fragment",
    // 自动为每一个被 ESbuild 转换的文件注入 JSX helper
    jsxInject: `import React from 'react'`,
  },
});
```

#### 引入 CSS 模块

以'.module.css'后缀命名的 css 文件可以具名导入

```js
// .apply-color -> applyColor
import { applyColor } from "./example.module.css";
document.getElementById("foo").className = applyColor;
```

#### 静态资源处理

```js
// vue2  this.imgUrl = require(./img.png)
import imgUrl from "./img.png";
document.getElementById("hero-img").src = imgUrl;

// 显式加载资源为一个 URL
import assetAsURL from "./asset.js?url";

// 以字符串形式加载资源
import assetAsString from "./shader.glsl?raw";

// 加载为 Web Worker
import Worker from "./worker.js?worker";

// 在构建时 Web Worker 内联为 base64 字符串
import InlineWorker from "./worker.js?worker&inline";
```

#### 导入多个模块

- 异步

  ```js
  const modules = import.meta.glob("./dir/*.js");
  ```

- 同步

  ```js
  const modules = import.meta.globEager("./dir/*.js");
  ```

- 遍历模块

  ```js
  for (const path in modules) {
    modules[path]().then((mod) => {
      console.log(path, mod);
    });
  }
  ```

#### 引入插件

```js
export default defineConfig({
  plugins: [
    vue(),
    legacy({
      targets: ["defaults", "not IE 11"],
      /* ************* enforce ******************
       * pre: 在 Vite 核心插件之前调用该插件
       * default: 在 Vite 核心插件之后调用该插件
       * post: 在 Vite 构建插件之后调用该插件
       * ****************************************/
      enforce: "pre",
      /* *************** apply ******************
       * serve: 开发 (serve)  模式中会调用
       * built: 生产 (build)  模式中会调用
       * ****************************************/
      apply: "build",
    }),
  ],
});
```

#### 引入 public 文件夹下的路径

引入 public 中的资源永远应该使用根绝对路径 —— 举个例子，`public/icon.png` 应该在源码中被引用为 `/icon.png`。

#### new URL

```js
function getImageUrl(name) {
  return new URL(`./dir/${name}.png`, import.meta.url).href;
}
```

#### vite.config.js

* 多页面应用模式

    在开发过程中，简单地导航或链接到 /nested/ - 将会按预期工作，与正常的静态文件服务器表现一致。

    在构建过程中，你只需指定多个 .html 文件作为入口点即可：

    ```js
    const { resolve } = require('path')
    const { defineConfig } = require('vite')

    module.exports = defineConfig({
      build: {
        rollupOptions: {
          input: {
            // 在解析输入路径时，__dirname 的值将仍然是 vite.config.js 文件所在的目录
            main: resolve(__dirname, 'index.html'),
            nested: resolve(__dirname, 'nested/index.html')
          }
        }
      }
    })
    ```


* 库模式

    ```js
    const path = require('path')
    const { defineConfig } = require('vite')

    module.exports = defineConfig({
      build: {
        lib: {
          entry: path.resolve(__dirname, 'lib/main.js'),
          name: 'MyLib',
          fileName: (format) => `my-lib.${format}.js`
        },
        rollupOptions: {
          // 确保外部化处理那些你不想打包进库的依赖
          external: ['vue'],
          output: {
            // 在 UMD 构建模式下为这些外部化的依赖提供一个全局变量
            globals: {
              vue: 'Vue'
            }
          }
        }
      }
    })
    ```

#### 调试

```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview --port 8080"
},

// npm run dev: 构建测试环境，运行在本地

/*
 * npm run build: 构建生产环境，将项目打包成dist
 * npm run preview: 部署在本地，默认搭建在localhost:5000/ 环境下，预览生产环境下的运行结果
 * "vite preview --port 8080": 设置部署在本地 8080 端口下
 */
```

#### .env

```js
  .env                # 所有情况下都会加载
  .env.local          # 所有情况下都会加载，但会被 git 忽略
  .env.[mode]         # 只在指定模式下加载
  .env.[mode].local   # 只在指定模式下加载，但会被 git 忽略
```

为了防止意外地将一些环境变量泄漏到客户端，只有以 VITE_ 为前缀的变量才会暴露给经过 vite 处理的代码。

```js
NODE_ENV=development
VITE_APP_TITLE=My App
# 只有VITE_APP_TITLE 会被暴露为 `import.meta.env.VITE_APP_TITLE` 提供给客户端源码
```

自定义模式及模式配置参数
https://vitejs.cn/guide/env-and-mode.html#modes