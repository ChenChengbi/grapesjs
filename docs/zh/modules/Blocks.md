---
title: 区块管理器
---

# 区块管理器 (Block Manager)

<p align="center"><img src="http://grapesjs.com/img/sc-grapesjs-blocks-prp.jpg" alt="GrapesJS - 区块管理器" height="400" align="center"/></p>

一个[区块 (Block)]是一个简单的对象，它允许最终用户重用您的[组件 (Components)]。它可以连接到单个[组件 (Component)]或它们的复杂组合。在本指南中，您将了解如何在 GrapesJS 中设置和充分利用内置的区块管理器 UI。
默认 UI 是一个轻量级组件，内置了拖放 (Drag & Drop) 支持，但正如您将在本指南后面看到的，扩展和创建自己的 UI 管理器非常容易。

::: warning
为了更好地理解本指南的内容，我们建议您先阅读[组件 (Components)]。
:::
::: warning
本指南指的是 GrapesJS v0.17.27 或更高版本。
:::

::: tip
需要一个易于扩展和自定义的时尚区块 UI 吗？[探索 Grapes Studio SDK！](https://app.grapesjs.com/docs-sdk/configuration/blocks?utm_source=grapesjs-docs&utm_medium=tip)
:::

[[toc]]

## 配置 (Configuration)

要更改默认配置，您需要将 `blockManager` 属性与主配置对象一起传递。

```js
const editor = grapesjs.init({
  ...
  blockManager: {
    blocks: [...],
    ...
  }
});
```

在此处查看可用选项的完整列表：[区块管理器配置 (Block Manager Config)](https://github.com/GrapesJS/grapesjs/blob/master/src/block_manager/config/config.ts)

## 初始化 (Initialization)

默认情况下，区块管理器 UI 被认为是一个隐藏组件。目前，GrapesJS 核心会渲染默认面板 (Panel) 和按钮，允许您显示它们，但从长远来看，这种情况可能会改变。下面您可以看到如何在没有默认面板的情况下初始化编辑器，并立即渲染区块管理器 UI。

::: tip
请遵循[入门指南 (Getting Started)]以正确设置带有自定义面板的编辑器。
:::

```js
const editor = grapesjs.init({
  container: '#gjs',
  height: '100%',
  storageManager: false,
  panels: { defaults: [] }, // 避免默认面板
  blockManager: {
    appendTo: '.myblocks',
    blocks: [
      {
        id: 'image',
        label: 'Image',
        media: `<svg style="width:24px;height:24px" viewBox="0 0 24 24">
            <path d="M8.5,13.5L11,16.5L14.5,12L19,18H5M21,19V5C21,3.89 20.1,3 19,3H5A2,2 0 0,0 3,5V19A2,2 0 0,0 5,21H19A2,2 0 0,0 21,19Z" />
        </svg>`,
        // 使用 `image` 组件
        content: { type: 'image' },
        // `image` 组件是可激活的（显示资源管理器 (Asset Manager)）。
        // 我们希望在它被拖放到画布 (Canvas) 后立即激活它。
        activate: true,
        // select: true, // `activate: true` 时的默认值
      },
    ],
  },
});
```

## 区块内容类型

将区块连接到组件的关键是 `block.content` 属性，我们在上面示例中传递的是[组件定义 (Component Definition)]。这是创建区块的面向组件的方式，我们强烈建议您以这种方式创建区块。

### 面向组件

`content` 可以接受不同的格式，比如 HTML 字符串 (HTML string)（它将被解析并转换为组件），但面向组件的方法是最精确的，因为您可以控制画布中每个拖放的区块。另一个建议是让您的区块的[组件定义 (Component Definition)]尽可能轻量。如果您定义了许多冗余属性，那么创建一个专用的组件可能更有意义，这可能会减少项目 JSON 文件的大小。下面是一个例子：

```js
// 您的组件
editor.Components.addType('my-cmp', {
  model: {
    defaults: {
      prop1: 'value1',
      prop2: 'value2',
    }
  }
});
// 您的区块
[
  { ..., content: { type: 'my-cmp', prop1: 'value1-EXT', prop2: 'value2-EXT' } }
  { ..., content: { type: 'my-cmp', prop1: 'value1-EXT', prop2: 'value2-EXT' } }
  { ..., content: { type: 'my-cmp', prop1: 'value1-EXT', prop2: 'value2-EXT' } }
]
```

这里我们多次重用同一个组件，并使用相同的属性集（这只是一个示例，对于组件的组合内容更有意义），这可以简化为如下形式：

```js
// 您的组件
editor.Components.addType('my-cmp', { ... });
editor.Components.addType('my-cmp-alt', {
  extend: 'my-cmp',
  model: {
    defaults: {
      prop1: 'value1-EXT',
      prop2: 'value2-EXT'
    }
  }
});
// 您的区块
[
  { ..., content: { type: 'my-cmp-alt' } }
  { ..., content: { type: 'my-cmp-alt' } }
  { ..., content: { type: 'my-cmp-alt' } }
]
```

### HTML 字符串

使用 HTML 字符串作为 `content` 并没有错，在某些情况下，您不需要对组件进行最精细的控制，并希望让用户在模板组合方面拥有完全的自由（例如，静态站点构建器编辑器，其 HTML 从像 [Tailwind Components](https://tailwindcomponents.com/) 这样的框架中复制粘贴而来）。

```js
// 您的区块
{
  // ...
  content: `<div class="el-X">
    <div class="el-Y el-A">Element A</div>
    <div class="el-Y el-B">Element B</div>
    <div class="el-Y el-C">Element C</div>
  </div>`;
}
```

在这种情况下，所有渲染的元素都将被转换为最合适的默认组件（例如，`.el-Y` 元素将被视为 `text` 组件）。用户将能够对它们进行样式化和拖动，没有特别的限制。

得益于组件的 [isComponent](Components.html#iscomponent) 特性（在解析后执行），您仍然可以将渲染的元素绑定到组件并强制执行额外的逻辑。以下示例说明了如何在不触及 `content` 中使用的原始 HTML 的任何部分的情况下，强制所有 `.el-Y` 元素仅放置在 `.el-X` 元素内部。

```js
// 您的组件
editor.Components.addType('cmp-Y', {
  // 检测 '.el-Y' 元素
  isComponent: (el) => el.classList?.contains('el-Y'),
  model: {
    defaults: {
      name: 'Component Y', // 简单的自定义名称
      draggable: '.el-X', // 添加 `draggable` 逻辑
    },
  },
});
```

另一种方法是利用 `data-gjs-*` 属性 (attributes) 将属性附加到组件上。

::: tip
您可以使用大多数可用的[组件属性 (/api/component.html#properties)]。
:::

```js
// -- [选项 1]: 在 HTML 字符串中声明类型 --
{
  // ...
  content: `<div class="el-X">
    <div data-gjs-type="cmp-Y" class="el-Y el-A">Element A</div>
    <div data-gjs-type="cmp-Y" class="el-Y el-B">Element B</div>
    <div data-gjs-type="cmp-Y" class="el-Y el-C">Element C</div>
  </div>`;
}
// 组件
editor.Components.addType('cmp-Y', {
  // 您不再需要 `isComponent`，因为您已经在元素上声明了类型
  model: {
    defaults: {
      name: 'Component Y', // 简单的自定义名称
      draggable: '.el-X', // 添加 `draggable` 逻辑
    },
  },
});

// -- [选项 2]: 在 HTML 字符串中声明属性 (不太推荐的选项) --
{
  // ...
  content: `<div class="el-X">
    <div data-gjs-name="Component Y" data-gjs-draggable=".el-X" class="el-Y el-A">Element A</div>
    <div data-gjs-name="Component Y" data-gjs-draggable=".el-X" class="el-Y el-B">Element B</div>
    <div data-gjs-name="Component Y" data-gjs-draggable=".el-X" class="el-Y el-C">Element C</div>
  </div>`;
}
// 不需要自定义组件。
// 您已经在定义每个元素的属性。
```

这里我们展示了 HTML 字符串的所有可能性，但我们强烈建议不要滥用 `选项 2`，并坚持采用更面向组件的方法。
如果没有合适的组件类型，不仅您的 HTML 更难阅读，而且所有这些定义的属性都将“硬编码”到这些元素的通用组件中。因此，如果有一天您决定“升级”组件的逻辑（例如，`draggable: '.el-X'` -> `draggable: '.el-X, .el-Z'`），您将无法做到。

### 混合

通过传递一个数组，也可以将组件与 HTML 字符串混合使用。

```js
{
  // ...
  // 像 `activate`/`select` 这样的选项将仅在第一个组件上触发。
  activate: true,
  content: [
    { type: 'image' },
    `<div>Extra</div>`
  ]
}
```

## 重要注意事项

::: danger 仔细阅读
&nbsp;
:::

### 避免不可序列化的属性

不要在区块中放置不可序列化属性 (non serializable properties)，例如函数，只将它们保留在您的组件中。

```js
// 您的区块
{
  content: {
    type: 'my-cmp',
    script() {...},
  },
}
```

这会起作用，但如果您尝试保存并重新加载存储的项目，这些属性将会消失。

### 避免样式

不要在区块中放置样式 (Styles)，始终将它们保留在您的组件中。

```js
// 您的区块
{
  content: [
    // 糟糕：您可能会创建冲突的样式
    { type: 'my-cmp', styles: '.cmp { color: red }' },
    { type: 'my-cmp', styles: '.cmp { color: green }' },

    // 非常糟糕：如果所有相关组件都被移除，
    // 编辑器没有安全的方法来知道如何连接
    // 和清理您的样式。
    `<div class="el">Element</div>
    <div class="el2">Element 2</div>
    <style>
      .el { color: blue }
      .el2 { color: violet }
    </style>`,
  ],
}
```

<!-- 通过组件定义导入的样式，使用 `styles` 属性，会连接到该特定组件类型。这使得编辑器能够在所有相关组件被删除时自动移除这些样式。 -->

使用面向组件的方法，您会面临样式冲突的风险，并在项目 JSON 中产生大量无用的冗余样式定义。

使用 HTML 字符串，如果您移除了所有相关元素，编辑器无法从项目 JSON 中清理这些样式，因为没有安全的方法来连接它们。

## 编程式使用 (Programmatic usage)

如果您需要以编程方式管理您的区块，您可以使用其 [API][Blocks API]。

::: warning
所有区块 API 方法主要更新您的区块管理器 UI，与画布中已拖放的组件无关。
:::

以下是常用方法的示例。

```js
// 首先获取 BlockManager 模块
const bm = editor.Blocks; // `Blocks` 是 `BlockManager` 的别名

// 添加一个新区块
const block = bm.add('BLOCK-ID', {
  // 您的区块属性...
  label: 'My block',
  content: '...',
});

// 获取区块
const block2 = bm.get('BLOCK-ID-2');

// 更新区块属性
block2.set({
  label: 'Updated block',
});

// 移除区块
const removedBlock = bm.remove('BLOCK-ID-2');
```

要了解更多关于可用区块属性的信息，请查看[区块 API 参考 (Block API Reference)][Block]。

## 自定义 (Customization)

默认的区块管理器 UI 非常适合简单的事物，但除了调整某些 CSS 样式的可能性之外，添加更复杂的元素需要替换默认 UI。

您所要做的就是向编辑器表明您打算使用自定义 UI，然后订阅 `block:custom` 事件 (Event)，该事件将为您提供有关任何请求更改的所有信息。

```js
const editor = grapesjs.init({
  // ...
  blockManager: {
    // ...
    custom: true,
  },
});

editor.on('block:custom', (props) => {
  // `props` 将包含更新 UI 所需的所有信息。
  // props.blocks (Array<Block>) - 所有区块的数组
  // props.dragStart (Function<Block>) - 触发区块拖动开始的回调。
  // props.dragStop (Function<Block>) - 触发区块拖动停止的回调。
  // props.container (HTMLElement) - 您可以附加 UI 的默认元素
  // 在这里您将放置渲染/更新 UI 的逻辑。
});
```

以下是使用自定义区块管理器与 Vue 组件的示例。

<demo-viewer value="xyofm1qr" height="500" darkcode/>

从上面的演示中您还可以看到我们如何决定隐藏自定义区块管理器并将其附加到默认容器，但这取决于您的偏好。

## 事件 (Events)

有关可用事件的完整列表，您可以在[此处](/api/block_manager.html#available-events)查看。

<!--
## 自定义渲染 <Badge text="0.14.55+"/>

如果您需要自定义每个区块的外观，可以在区块定义中传递一个 `render` 回调函数。让我们看看它是如何工作的。

作为第一个选项，您可以返回一个简单的 HTML 字符串，它将用作区块新的内部内容。作为回调的参数，您将得到一个包含以下属性的对象：

* `model` - 区块的模型（因此您可以使用传递给它的任何属性）
* `el` - 区块当前渲染的 HTMLElement
* `className` - 用于区块的基本类名（如果您遵循 BEM，则很有用，因此您可以创建像 `${className}__elem` 这样的类）

```js
blockManager.add('some-block-id', {
  label: `<div>
      <img src="https://picsum.photos/70/70"/>
      <div class="my-label-block">Label block</div>
    </div>`,
  content: '<div>...</div>',
  render: ({ model, className }) => `<div class="${className}__my-wrap">
      Before label
      ${model.get('label')}
      After label
    </div>`,
});
```

<img :src="$withBase('/block-custom-render.jpg')">


另一种选择是不从回调返回（在这种情况下，任何内容都不会被替换），而只编辑当前的 `el` 区块元素

```js
blockManager.add('some-block-id', {
  // ...
  render: ({ el }) => {
    const btn = document.createElement('button');
    btn.innerHTML = 'Click me';
    btn.addEventListener('click', () => alert('Do something'))
    el.appendChild(btn);
  },
});
```
<img :src="$withBase('/block-custom-render2.jpg')">
-->

[Block]: /api/block.html
[Component]: /api/component.html
[Components]: Components.html
[Getting Started]: /getting-started.html
[Blocks API]: /api/block_manager.html
[Component Definition]: Components.html#component-definition