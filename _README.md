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

