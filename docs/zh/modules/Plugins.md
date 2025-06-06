---
title: 插件
---

# 插件

在 GrapesJS 中创建插件(Plugin)非常简单，本文将向您介绍如何实现。

::: warning
本指南适用于 GrapesJS v0.21.2 或更高版本。
:::

::: tip
正在寻找经过测试、验证且可扩展的插件？[在 Grapes Studio SDK 中浏览所有插件！](https://app.grapesjs.com/docs-sdk/plugins/overview?utm_source=grapesjs-docs&utm_medium=tip)
:::

[[toc]]

## 基础插件

插件是在编辑器(Editor)初始化时运行的简单函数。

```js
function myPlugin(editor) {
  // Use the API: https://grapesjs.com/docs/api/
  editor.Blocks.add('my-first-block', {
    label: 'Simple block',
    content: '<div class="my-block">This is a simple block</div>',
  });
}

const editor = grapesjs.init({
  container: '#gjs',
  plugins: [myPlugin],
});
```

这意味着可以将插件移动到单独的文件夹以保持代码整洁，或者从 NPM (Node包管理器)导入。

```js
import myPlugin from './plugins/myPlugin';
import npmPackage from '@npm/package';

const editor = grapesjs.init({
  container: '#gjs',
  plugins: [myPlugin, npmPackage],
});
```

<!--
## 命名插件

如果您要全局(Globally)分发(Distribute)您的插件，您可能希望创建一个命名插件。为了保持代码整洁，您可能会得到类似的结构：

```
/your/path/to/grapesjs.min.js
/your/path/to/grapesjs-plugin.js
```

**重要提示：** 文件加载顺序至关重要。GrapesJS 必须在插件之前加载。这样会设置 `grapesjs` 全局变量。

因此，在您的 `grapesjs-plugin.js` 文件中：

```js
export default grapesjs.plugins.add('my-plugin-name', (editor, options) => {
  /*
  * Here you should rely on GrapesJS APIs, so check 'API Reference' for more info
  * For example, you could do something like this to add some new command:
  *
  * editor.Commands.add(...);
  */
})
```

名称 `my-plugin-name` 是您插件的 ID (标识符)，您将使用它来告知编辑器加载该插件。

这是一个完整的通用示例：

```html
<script src="http://code.jquery.com/jquery-2.2.0.min.js"></script>
<link rel="stylesheet" href="path/to/grapes.min.css">
<script src="path/to/grapes.min.js"></script>
<script src="path/to/grapesjs-plugin.js"></script>

<div id="gjs"></div>

<script type="text/javascript">
  var editor = grapesjs.init({
      container : '#gjs',
      plugins: ['my-plugin-name']
  });
</script>
```
-->

## 带选项的插件

也可以向插件传递自定义参数(Parameter)，以使其更加灵活。

<!--
```js
  var editor = grapesjs.init({
      container : '#gjs',
      plugins: ['my-plugin-name'],
      pluginsOpts: {
        'my-plugin-name': {
          customField: 'customValue'
        }
      }
  });
```

在您的插件内部，您将通过 `options` 参数获取这些选项(Option)：

```js
export default grapesjs.plugins.add('my-plugin-name', (editor, options) => {
  console.log(options);
  //{ customField: 'customValue' }
})
```

这也适用于未命名的插件。

-->

```js
const myPluginWithOptions = (editor, options) => {
  console.log(options);
  // { customField: 'customValue' }
};

const editor = grapesjs.init({
  container: '#gjs',
  plugins: [myPluginWithOptions],
  pluginsOpts: {
    [myPluginWithOptions]: {
      customField: 'customValue',
    },
  },
});
```

<!--
## 命名插件与非命名插件

当您使用命名插件时，该名称在所有其他插件中必须是唯一的。

```js
grapesjs.plugins.add('my-plugin-name', fn);
```

在此示例中，插件名称为 `my-plugin-name`，不能被其他插件使用。为避免命名空间(Namespace)限制，请使用纯函数的基础插件。

-->

## TS 用法

如果您正在使用 TypeScript (TS)，为了获得更好的类型安全(Type safety)，我们建议使用 `usePlugin` 辅助函数(Helper)。

```ts
import grapesjs, { usePlugin } from 'grapesjs';
import type { Plugin } from 'grapesjs';

interface MyPluginOptions {
  opt1: string;
  opt2?: number;
}

const myPlugin: Plugin<MyPluginOptions> = (editor, options) => {
  // ...
};

grapesjs.init({
  // ...
  plugins: [
    // no need for `pluginsOpts`
    usePlugin(myPlugin, { opt1: 'A', opt2: 1 }),
  ],
});
```

## 样板代码

为了快速开发插件，我们强烈建议使用 [grapesjs-cli](https://github.com/GrapesJS/cli) (命令行界面)。它可以帮助您避免为开发和构建设置所有依赖(Dependency)和配置(Configuration)的麻烦（无需接触 Webpack 或 Babel 配置）。更多信息请查看该仓库。