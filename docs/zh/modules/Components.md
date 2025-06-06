---
title: 组件管理器
---

# 组件管理器

组件 (Component) 是模板 (template) 的基本元素。它可能是一个简单且原子的元素，比如一个图像或一个文本框，但也可能是复杂的结构，更有可能由其他组件组成，比如区块或页面。组件的概念是为了让开发者能够将不同的行为绑定到不同的元素上。例如，双击图像时打开资源管理器 (Asset Manager) 就是绑定到该特定类型元素的自定义行为。

::: warning
本指南适用于 GrapesJS v0.15.8 或更高版本
:::

::: tip
跳过样板代码——直接使用精细的组件编辑器。[查看 Grapes Studio SDK！](https://app.grapesjs.com/docs-sdk/configuration/components/overview?utm_source=grapesjs-docs&utm_medium=tip)
:::

[[toc]]

## 组件如何工作？

让我们通过查看从向编辑器添加 HTML 字符串开始的所有步骤，来详细了解组件是如何工作的。

::: tip
以下所有代码片段都可以直接在[主要演示](https://grapesjs.com/demo.html)的控制台中运行。
:::

这就是我们如何向画布 (canvas) 添加新组件的方法：

```js
// 直接向画布追加组件
editor.addComponents(`<div>
  <img src="https://path/image" />
  <span title="foo">Hello world!!!</span>
</div>`);

// 或者追加到一些已定义的组件中。
// 例如，追加到选定组件将会是：
editor.getSelected().append(`<div>...`);

// 实际上，editor.addComponents 是以下写法的别名...
editor.getWrapper().append(`<div>...`);
```

::: tip
如果你需要在特定位置追加一个组件，可以使用 `at` 选项。所以，要在所有其他组件（在同一集合中）的顶部添加一个组件，你可以使用

```js
component.append('<div>...', { at: 0 });
```

或者在中间

```js
const { length } = component.components();
component.append('<div>...', { at: parseInt(length / 2, 10) });
```
:::

### 组件定义 (Component Definition)

第一步，HTML 字符串被解析并转换为所谓的**组件定义 (Component Definition)**，因此上述输入的结果将是：

```js
{
  tagName: 'div',
  components: [
    {
      type: 'image',
      attributes: { src: 'https://path/image' },
    }, {
      tagName: 'span',
      type: 'text',
      attributes: { title: 'foo' },
      components: [{
        type: 'textnode',
        content: 'Hello world!!!'
      }]
    }
  ]
}
```

实际的**组件定义 (Component Definition)**会稍微大一些，为了简单起见，我们对 JSON 进行了简化。

你可能会注意到结果类似于通常所说的**虚拟 DOM (Virtual DOM)**，即 DOM 元素的轻量级表示。这实际上有助于编辑器跟踪我们元素的状态并进行性能友好的更改/更新。
像 `tagName`、`attributes` 和 `components` 这样的属性的含义相当明显，但是 `type` 呢？！这个特定的属性指定了我们**组件定义 (Component Definition)** 的**组件类型 (Component Type)**（你可以查看[下文](#built-in-component-types)的默认组件列表），如果省略，则将使用默认的 `type: 'default'`。
此时，一个好问题是，编辑器是如何从一个简单的 HTML 字符串开始分配这些类型的？这个步骤被识别为**组件识别 (Component Recognition)**，并在下一段中详细解释。

### 组件识别 (Component Recognition) 与组件类型栈 (Component Type Stack)

正如我们之前提到的，当你将 HTML 字符串作为组件传递给编辑器时，该字符串会被解析并编译为带有新的 `type` 属性的[组件定义 (Component Definition)]。为了确定应该分配什么 `type`，对于每个解析的 HTML 元素，编辑器会遍历所有已定义的组件，称为**组件类型栈 (Component Type Stack)**，并通过 `isComponent` 方法（我们稍后会看到）检查该组件类型是否适合该元素。组件类型栈只是一个简单的组件类型数组，但重要的是这些类型的顺序。任何新添加的自定义**组件类型 (Component Type)**（我们稍后会看到如何创建它们）都会被放在组件类型栈的顶部，解析器返回的每个元素都会从上到下遍历这个栈（栈的最后一个元素是 `default` 类型），一旦某个组件的 `isComponent` 方法返回一个真值，迭代就会停止。

<img :src="$withBase('/component-type-stack.svg')" class="img-ctr">

::: tip
如果你正在导入大量 HTML 代码块，你可能希望通过直接传递组件定义 (Component Definition) 对象或使用 JSX 语法来跳过解析和组件识别步骤，从而提高性能。
阅读[此处](#setup-jsx-syntax)了解如何设置 JSX 语法解析器。
:::

### 组件实例 (Component instance)

一旦**组件定义 (Component Definition)** 准备就绪并且类型已分配，就可以创建[组件 (Component)]实例（也称为**模型 (Model)**）。让我们回到我们之前的 HTML 字符串示例，`append` 方法的结果是已添加组件的数组。

```js
const component = editor.addComponents(`<div>
  <img src="https://path/image" />
  <span title="foo">Hello world!!!</span>
</div>`)[0];
```

组件实例 (Component instance) 包含属性和方法，允许你获取其数据并更改它们。
你可以使用 `get` 方法读取属性，例如 `type`

```js
const componentType = component.get('type'); // 例如 'image'
```

要更新属性，你会使用 `set` 方法，这可能会改变组件在画布 (canvas) 中的行为方式。

```js
// 使组件不可拖动
component.set('draggable', false);
```

你还可以使用诸如 `getAttributes`、`setAttributes`、`components` 等方法。

```js
const innerComponents = component.components();
innerComponents.forEach((comp) => console.log(comp.toHTML()));
// 更新组件内容
component.components(`<div>Component 1</div><div>Component 2</div>`);
```

每个组件都可以定义自己的属性和方法，但所有组件都将始终至少扩展 `default` 组件（稍后你将看到如何创建新的自定义组件以及如何扩展已定义的组件），因此最好查看[组件 API (Component API)]以了解所有可用的属性和方法。

**组件 (Component) 的主要目的**是跟踪其数据并在必要时返回它们。你可能需要从组件中获取的一项常见内容是显示其当前的 HTML

```js
const componentHTML = component.toHTML();
```

这将返回一个包含组件及其所有子组件 HTML 的字符串。
该组件还实现了 `toJSON` 方法，因此你可以通过这种方式获取其 JSON 结构

```js
JSON.stringify(component);
```

::: tip
要存储/加载所有组件，你应该依赖[存储管理器 (Storage Manager)](/modules/storage.html)
:::

因此，**组件实例 (Component instance)** 负责模板的**最终数据**（例如 HTML、JSON）。如果你需要（例如）更新/添加 HTML 中的某些属性，你需要更新其组件（例如 `component.addAttributes({ title: 'Title added' })`），因此组件/模型 (Component/Model) 是你的**真理之源 (Source of Truth)**。

### 组件渲染 (Component rendering)

组件的另一个重要部分是它们如何在**画布 (canvas)** 中渲染，这方面由其**视图 (View)** 处理。它与**最终的 HTML 数据**无关，你可以返回一个大的 `<div>...</div>` 字符串作为组件的 HTML，但在画布中将其渲染为简单的图像（可以考虑用于复杂/动态数据的占位符）。

默认情况下，组件的视图会自动与其模型 (Model) 的数据同步（你不能拥有没有模型的视图）。如果你更新组件的属性或追加一个新的子组件，视图将在画布中渲染它。

不幸的是，有时你可能需要一些额外的逻辑来更好地处理组件结果。考虑允许用户构建其 `<table>` 元素，对于这个特定情况，你可能希望在画布中添加自定义按钮，以便更容易地添加/删除列/行。要处理这些情况，你可以依赖视图 (View)，在视图中你可以添加额外的 DOM 组件、附加事件等。所有这些都将与 `<table>` 的最终 HTML（用户期望的结果）完全无关，因为它由模型 (Model) 处理。
一旦组件被渲染，你总是可以访问其视图 (View) 和 DOM 元素。

```js
const component = editor.getSelected();
// 获取视图
const view = component.getView();
// 获取 DOM 元素
const el = component.getEl();
```

通常，视图 (View) 是你不需要更改的东西，因为默认视图已经处理了与模型 (Model) 的同步，但如果你需要对元素进行更多控制（例如，画布中的自定义 UI），你可能需要创建一个自定义组件类型并用你的逻辑扩展默认视图 (View)。我们稍后会看到如何创建自定义组件类型。

到目前为止，我们已经了解了组件背后的核心概念以及它们如何工作。**模型/组件 (Model/Component)** 是模板最终代码（例如，HTML 导出依赖于它）的**真理之源 (source of truth)**，而**视图/组件视图 (View/ComponentView)** 是编辑器用来在画布中向用户**预览我们的组件**的内容。

<!--
TODO
A more advanced use case of custom components is an implementation of a custom renderer inside of them
-->

## 内置组件类型 (Built-in Component Types)

下面你可以看到内置组件类型的列表，按其在**组件类型栈 (Component Type Stack)**中的位置排序。

- [`cell`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentTableCell.ts) - 用于处理 `<td>` 和 `<th>` 元素的组件
- [`row`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentTableRow.ts) - 用于处理 `<tr>` 元素的组件
- [`table`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentTable.ts) - 用于处理 `<table>` 元素的组件
- [`thead`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentTableHead.ts) - 用于处理 `<thead>` 元素的组件
- [`tbody`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentTableBody.ts) - 用于处理 `<tbody>` 元素的组件
- [`tfoot`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentTableFoot.ts) - 用于处理 `<tfoot>` 元素的组件
- [`map`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentMap.ts) - 用于处理地图的组件 (GrapesJS `ComponentMap` 似乎是用于地图元素，通常是 `<iframe>` 或 `<div>` 嵌入地图服务，而不是 `<a>` 元素。不过，为了与原文保持一致，此处保留 `<a>`。)
- [`link`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentLink.ts) - 用于处理 `<a>` 元素的组件
- [`label`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentLabel.ts) - 用于正确处理 `<label>` 元素的组件
- [`video`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentVideo.ts) - 用于视频的组件
- [`image`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentImage.ts) - 用于图像的组件
- [`script`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentScript.ts) - 用于处理 `<script>` 元素的组件
- [`svg`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentSvg.ts) - 用于处理 SVG 元素的组件
- [`comment`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentComment.ts) - 用于注释的组件（可能对邮件编辑器有用）
- [`textnode`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentTextNode.ts) - 类似于 DOM 定义中的文本节点，即没有标签元素的文本元素。
- [`text`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentText.ts) - 可以内联编辑的简单文本组件
- [`wrapper`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/ComponentWrapper.ts) - 画布需要包含一个根组件，一个包装器，这个组件是为了识别它而创建的
- [`default`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/dom_components/model/Component.ts) 默认基础组件

## 定义自定义组件类型 (Define Custom Component Type)

既然我们知道了组件是如何工作的，我们就可以开始探索创建自定义**组件类型 (Component Types)**的过程了。

<u>定义新组件类型的第一条规则是将代码放在插件 (plugin) 中</u>。如果你想在开始时，在任何组件初始化（例如，从数据库加载的模板）之前加载你的自定义类型，这是必要的。插件在组件获取（例如，在使用存储 (Storage) 的情况下）之前加载，因此它是定义组件类型的理想位置。

```js
const myNewComponentTypes = (editor) => {
  editor.DomComponents.addType(/* 组件类型定义的 API */);
};

const editor = grapesjs.init({
  container: '#gjs',
  // ...
  plugins: [myNewComponentTypes],
});
```

假设我们想让编辑器更好地理解和处理 `<input>` 元素。这是我们开始定义新组件类型的方式：

```js
editor.DomComponents.addType('my-input-type', {
  // 让编辑器理解何时绑定 `my-input-type`
  isComponent: (el) => el.tagName === 'INPUT',

  // 模型定义
  model: {
    // 默认属性
    defaults: {
      tagName: 'input',
      draggable: 'form, form *', // 只能拖放到 `form` 元素内部
      droppable: false, // 不能将其他元素拖放到内部
      attributes: {
        // 默认属性
        type: 'text',
        name: 'default-name',
        placeholder: '在此处插入文本',
      },
      traits: ['name', 'placeholder', { type: 'checkbox', name: 'required' }],
    },
  },
});
```

通过这段代码，编辑器将能够理解简单的文本 `<input>`，分配默认属性并显示一些特性 (trait) 以更好地处理属性。

::: tip
要更好地理解特性 (Traits) 如何工作，你应该阅读其[专门页面](Traits.html)，但我们强烈建议你在读完这一页后再阅读它。
:::

### isComponent

让我们详细看看我们目前所做的工作。首先要注意的是 `isComponent` 函数，我们已经在[此节](#component-recognition-and-component-type-stack)中提到了它的用法，我们需要它来让编辑器在组件识别步骤中理解 `<input>`。
它只接收 `el` 参数，即解析后的 HTMLElement 节点，并期望在元素满足你的逻辑条件时返回一个真值。所以，如果我们添加这个 HTML 字符串作为组件：

```js
// ...编辑器初始化之后
editor.addComponents(`<input name="my-test" title="hello"/>`);
```

生成的组件定义 (Component Definition) 将是：

```js
{
  type: 'my-input-type',
  attributes: {
    name: 'my-test',
    title: 'hello',
  },
}
```

如果需要，你还可以通过返回一个对象作为结果来自定义生成的组件定义 (Component Definition)：

```js
editor.DomComponents.addType('my-input-type', {
  isComponent: el => {
    if (el.tagName === 'INPUT') {
      // 你应该显式声明结果对象的类型，
      // 否则将使用 `default` 类型
      const result = { type: 'my-input-type' };

      if (/* 某些其他条件 */) {
        result.attributes = { title: 'Hi' };
      }

      return result;
    }
  },
  // ...
});
```

::: danger
保持 `isComponent` 函数尽可能简单
:::

**请注意**，此方法可能会接收来自画布的**任何**已解析元素（例如，在加载或添加时），并且并非所有节点都具有相同的接口（例如，属性/方法）。
如果你这样做：

```js
// ...
// 打印元素
isComponent: (el) => {
  console.log(el);
  return el.tagName === 'INPUT';
},
  // ...
  editor.addComponents(`<div>
  我是一个文本节点
  <!-- 我是一个注释节点 -->
  <img alt="Image here"/>
  <input/>
</div>`);
```

你会看到所有节点都被打印出来，所以在你的 `isComponent` 中执行类似 `el.getAttribute('...')` 的操作（这在 `div` 上有效，但在 `文本节点` 上无效），而没有进行适当的检查，将会破坏代码。

同样重要的是要理解，`isComponent` 仅在需要解析时才执行（例如，通过将组件添加为 HTML 字符串或使用 `fromElement` 初始化编辑器）。如果类型已经定义，则无需执行 `isComponent`。
让我们看一些例子：

```js
// isComponent 将在 some-element 上执行
editor.addComponents('<some-element>...</some-element>');

// isComponent 不会在对象上执行
// 如果对象没有 `type` 键，则将使用 `default` 类型
editor.addComponents({
  type: 'some-component',
});

// isComponent 不会执行，因为我们正在强制指定类型
editor.addComponents('<some-element data-gjs-type="some-component">...');
```

如果你在定义组件类型 (Component Type) 时不使用 `isComponent`，编辑器识别该组件的唯一方法是通过显式声明的类型（通过对象 `{ type: '...' }` 或使用 `data-gjs-type`）。

### 模型 (Model)

既然我们了解了 `isComponent` 的工作原理，我们就可以开始探索 `model` 属性了。
`model` 可能是你使用最多的属性，因为它用于描述你的组件，你能看到的第一个键是 `defaults`，它代表_默认组件属性_，并反映了已经描述过的[组件定义 (Component Definition)]。

模型 (model) 还定义了你将看到的最终 HTML（导出代码），你可能已经注意到模型 (model) 上使用了 `tagName`（如果未指定，则使用 `div`）和 `attributes` 属性。

另一个重要的属性（在我们的输入组件集成中未使用，因为 `<input/>` 不需要它）可能是 `components`，它定义了默认的内部组件。

```js
defaults: {
  tagName: 'div',
  attributes: { title: 'Hello' },
  // 可以是一个字符串
  components: `
    <h1>Header test</h1>
    <p>Paragraph test</p>
  `,
  // 一个组件定义
  components: {
    tagName: 'h1',
    components: 'Header test',
  },
  // 字符串/组件定义的数组
  components: [
    {
      tagName: 'h1',
      components: 'Header test',
    },
    '<p>Paragraph test</p>',
  ],
  // 或者一个函数，它接收当前模型作为参数
  // 并期望返回上述可能值之一
  components: model => {
    return `<h1>Header test: ${model.get('type')}</h1>`;
  },
}
```

#### 读取和更新模型 (model)

无论何时何地，只要你拥有对模型 (model) 的引用，就可以读取和更新其属性。以下是一些最有用的 API 的参考：

```js
// 让我们使用选定的组件
const modelComponent = editor.getSelected();

// 获取所有模型属性
const props = modelComponent.props();

// 获取单个属性
const tagName = modelComponent.get('tagName');

// 更新单个属性
modelComponent.set('tagName', '...');

// 更新多个属性
modelComponent.set({
  tagName: '...',
  // ...
});

// 一些辅助函数

// 获取所有属性
const attrs = modelComponent.getAttributes();

// 添加属性
modelComponent.addAttributes({ title: 'Test' });

// 替换所有属性
modelComponent.setAttributes({ title: 'Test' });

// 获取所有内部组件的集合
modelComponent.components().forEach((inner) => console.log(inner.props()));

// 使用 HTML 字符串/组件定义更新内部内容
const addedComponents = modelComponent.components(`<div>...</div>`);

// 通过查询字符串查找组件
modelComponent.find(`.query-string[example=value]`).forEach((inner) => console.log(inner.props()));
```

你会注意到，任何更改都会相应地改变画布 (canvas) 中的组件及其导出代码。

:::tip
要了解所有可用的方法/属性，请查看[组件 API (Component API)]。
:::

#### 监听属性变化

如果你需要在某些属性更改时完成某种操作，可以在 `init` 方法中设置监听器。

```js
editor.DomComponents.addType('my-input-type', {
  // ...
  model: {
    defaults: {
      // ...
      someprop: 'initial value',
    },

    init() {
      this.on('change:someprop', this.handlePropChange);
      // 监听任何属性变化
      this.on('change:attributes', this.handleAttrChange);
      // 监听 title 属性变化
      this.on('change:attributes:title', this.handleTitleChange);
    },

    handlePropChange() {
      const { someprop } = this.props();
      console.log('someprop 的新值: ', someprop);
    },

    handleAttrChange() {
      console.log('属性已更新: ', this.getAttributes());
    },

    handleTitleChange() {
      console.log('属性 title 已更新: ', this.getAttributes().title);
    },
  },
});
```

你会在[下文](#lifecycle-hooks)找到其他生命周期方法，如 `init`。

现在让我们回到我们的输入组件集成，看看组件定制的另一个有用部分。

### 视图 (View)

通常，当你在 GrapesJS 中创建一个组件时，你期望在画布 (canvas) 中看到你在模型 (model) 中定义的预览。实际上，编辑器默认会执行完全相同的操作，并在模型 (model) 中的某些内容（例如属性、标签等）发生更改时更新画布 (canvas) 中的元素，以获得经典的**所见即所得 (WYSIWYG)** 体验。不幸的是，最简单的事情并不总是正确的，通过为构建器构建组件，你会注意到有时你需要更多：

- 你希望改善组件的编辑体验。
  一个完美的例子是 TextComponent，它的视图 (view) 通过内置的 RTE（富文本编辑器）得到了增强，用户可以通过双击它来更快地编辑文本。

  因此，你可能会觉得需要添加操作来响应某些 DOM 事件，甚至在组件周围添加自定义 UI 元素（例如按钮）。

- 组件的 DOM 表示行为与你期望的不同，因此你需要更改某些行为。
  一个例子可能是 VideoComponent，例如，它是通过 iframe 从 Youtube 加载的。一旦 iframe 加载完毕，其中的所有内容都处于不同的上下文中，编辑器无法看到它，实际上，如果将光标指向 iframe，你将与视频而不是编辑器交互，因此你甚至无法选择你的组件。为了解决这个“问题”，在渲染时，我们禁用了与 iframe 的指针交互，并用另一个元素将其包装起来（如果没有包装器，编辑器会选择父组件）。显然，所有这些更改都与最终代码无关，结果将始终是一个简单的 iframe。

- 你需要自定义内容或用服务器中的某些数据填充它。

对于所有这些情况，你可以在你的组件类型定义 (Component Type Definition) 中使用 `view`。`<input>` 组件可能不是此场景的最佳用例，但我们将尝试通过以下示例涵盖大多数情况：

```js
editor.DomComponents.addType('my-input-type', {
  // ...
  model: {
    // ...
  },
  view: {
    // 默认情况下，元素的标签与模型的标签相同
    tagName: 'div',

    // 使用 `events` 轻松添加组件特定的监听器
    // 由于是组件特定的（例如，你不能在此处附加到 window 的监听器）
    // 你无需担心在组件移除时移除它们，
    // 编辑器会自动管理它们
    events: {
      click: 'clickOnElement',
      // 你也可以使用事件委托
      // 并监听从某些内部元素冒泡的事件
      'dblclick .inner-el': 'innerElClick',
    },

    innerElClick(ev) {
      ev.stopPropagation();
      // ...

      // 如果需要，你可以从视图中的任何函数访问模型
      this.model.components('更新内部组件');
    },

    // 在 init 中，你可以像在模型中一样创建监听器，或者在开始时启动其他一些
    // 函数
    init({ model }) {
      // 在模型属性更改时在视图中执行某些操作
      this.listenTo(model, 'change:prop', this.handlePropChange);

      // 如果你在外部对象上附加了监听器，请记住在 `removed` 函数中解除绑定
      // 它们，以避免内存泄漏
      this.onDocClick = this.onDocClick.bind(this);
      document.addEventListener('click', this.onDocClick);
    },

    // 当元素从画布中移除时触发的回调
    removed() {
      document.removeEventListener('click', this.onDocClick);
    },

    // 元素渲染后对内容执行某些操作。
    // DOM 元素作为 `el` 在参数对象中传递，
    // 但你可以通过 `this.el` 从任何函数访问它
    onRender({ el }) {
      const btn = document.createElement('button');
      btn.value = '+';
      // 这只是一个例子，避免在内部元素上添加事件，
      // 在这些情况下使用 `events`
      btn.addEventListener('click', () => {});
      el.appendChild(btn);
    },

    // 异步内容示例
    async onRender({ el, model }) {
      const asyncContent = await fetchSomething({
        someDataFromModel: model.get('someData'),
      });
      // 请记住，这些更改仅存在于编辑器画布内部
      // 没有任何 DOM 更改存储在你的模板数据中，
      // 如果你需要存储某些内容，请更新模型属性
      el.appendChild(asyncContent);
    },
  },
});
```

## 更新组件类型 (Update Component Type)

更新组件类型非常简单，让我们看看如何操作：

```js
const domc = editor.DomComponents;

domc.addType('some-component', {
  // 你可以更新 isComponent 逻辑或保留 `some-component` 中的逻辑
  // isComponent: (el) => false,

  // 如果需要，更新模型
  model: {
    // `defaults` 属性的处理方式不同
    // 并将与旧的 `defaults` 合并
    defaults: {
      tagName: '...', // 覆盖旧的
      someNewProp: 'Hello', // 添加新属性
    },
    init() {
      // 覆盖 `some-component` 中的 `init` 函数
    },
  },

  // 如果需要，更新视图
  view: {},
});
```

### 扩展组件类型 (Extend Component Type)

有时你需要通过扩展另一个类型来创建一个新类型。只需使用 `extend` 和 `extendView` 来指明要扩展的组件。

```js
comps.addType('my-new-component', {
  isComponent: el => {/* ... */},
  extend: 'other-defined-component',
  model: { ... }, // 将扩展 'other-defined-component' 的模型
  view: { ... }, // 将扩展 'other-defined-component' 的视图
});
```

```js
comps.addType('my-new-component', {
  isComponent: el => {/* ... */},
  extend: 'other-defined-component',
  model: { ... }, // 将扩展 'other-defined-component' 的模型
  extendView: 'other-defined-component-2',
  view: { ... }, // 将扩展 'other-defined-component-2' 的视图
});
```

### 扩展父函数 (Extend parent functions)

当你需要重用你正在扩展的父组件中的函数时，可以避免编写以下代码：

```js
domc.getType('parent-type').model.prototype.init.apply(this, arguments);
```

通过使用 `extendFn` 和 `extendFnView` 选项：

```js
domc.addType('new-type', {
  extend: 'parent-type',
  extendFn: ['init'], // 要从 `parent-type` 扩展的模型函数数组
  model: {
    init() {
      // 执行某些操作
    },
  },
});
```

对于视图，使用 `extendFnView` 也是一样的。

:::tip
如果需要，你还可以使用 `getTypes` 获取所有当前的组件类型。

```js
editor.DomComponents.getTypes().forEach((compType) => console.log(compType.id));
```
:::

## 生命周期钩子 (Lifecycle Hooks)

每个组件都会触发不同的生命周期钩子 (lifecycle hooks)，这允许你在它们的特定阶段添加自定义操作。
我们可以区分两种不同类型的钩子：**全局 (global)** 和 **局部 (local)**。
当你创建/扩展一个组件类型时（通常通过一些 `model`/`view` 方法），你会定义**局部**钩子，其目的是对该特定组件类型的事件做出反应。而**全局**钩子则会对任何组件无差别地调用（你通过 `editor.on` 来监听它们），你可以将它们用于更通用的用例，或者在其他组件内部监听它们。

让我们看看下面所有钩子的流程：

- **局部钩子**：`model.init()` 方法，在组件的模型 (model) 初始化后执行。
- **全局钩子**：`component:create` 事件，在 `model.init()` 之后立即调用。模型 (model) 作为参数传递给回调函数。
  例如：`editor.on('component:create', model => console.log('created', model))`
- **局部钩子**：`view.init()` 方法，在组件的视图 (view) 初始化后执行。
- **局部钩子**：`view.onRender()` 方法，在组件渲染到画布 (canvas) 上后执行。
- **全局钩子**：`component:mount` 事件，在 `view.onRender()` 之后立即调用。模型 (model) 作为参数传递给回调函数。
- **局部钩子**：`model.updated()` 方法，当模型 (model) 的某些属性更新时执行。
- **全局钩子**：`component:update` 事件，在 `model.updated()` 之后调用。模型 (model) 作为参数传递给回调函数。
  你还可以通过 `component:update:{propertyName}` 监听特定属性的更改。
- **局部钩子**：`model.removed()` 方法，在组件被移除时执行。
- **全局钩子**：`component:remove` 事件，在 `model.removed()` 之后调用。模型 (model) 作为参数传递给回调函数。

下面你可以找到所有钩子的示例用法：

```js
editor.DomComponents.addType('test-component', {
  model: {
    defaults: {
      testprop: 1,
    },
    init() {
      console.log('局部钩子: model.init');
      this.listenTo(this, 'change:testprop', this.handlePropChange);
      // 在这里我们可以用 editor.on('...') 监听全局钩子
    },
    updated(property, value, prevValue) {
      console.log('局部钩子: model.updated', '属性', property, '值', value, '先前的值', prevValue);
    },
    removed() {
      console.log('局部钩子: model.removed');
    },
    handlePropChange() {
      console.log('testprop 的值', this.get('testprop'));
    },
  },
  view: {
    init() {
      console.log('局部钩子: view.init');
    },
    onRender() {
      console.log('局部钩子: view.onRender');
    },
  },
});

// 自定义组件的块
editor.BlockManager.add('test-component', {
  label: '测试组件',
  content: '<div data-gjs-type="test-component">测试组件</div>',
});

// 全局钩子
editor.on(`component:create`, (model) => console.log('全局钩子: component:create', model.get('type')));
editor.on(`component:mount`, (model) => console.log('全局钩子: component:mount', model.get('type')));
editor.on(`component:update:testprop`, (model) =>
  console.log('全局钩子: component:update:testprop', model.get('type')),
);
editor.on(`component:remove`, (model) => console.log('全局钩子: component:remove', model.get('type')));
```

## 组件与 CSS (Components & CSS)

::: warning
本节内容适用于 GrapesJS v0.17.27 或更高版本
:::

如果你需要添加与组件相关的样式，可以通过 `styles` 属性来实现。

```js
domc.addType('component-css', {
  model: {
    defaults: {
      attributes: { class: 'cmp-css' },
      components: `
        <span>带样式的组件<span>
        <div class="cmp-css-a">组件 A</div>
        <div class="cmp-css-b">组件 B</div>
      `,
      styles: `
        .cmp-css { color: red }
        .cmp-css-a { color: green }
        .cmp-css-b { color: blue }

        @media (max-width: 992px) {
          .cmp-css{ color: darkred; }
          .cmp-css-a { color: darkgreen }
          .cmp-css-b { color: darkblue }
        }
      `,
    },
  },
});
```

这种方法允许编辑器将这些样式（[CssRule] 实例）分组，并在同一组件的所有引用都被移除时相应地移除它们。

::: danger 重要注意事项
&nbsp;
:::

在上面的例子中，我们使用了一个自定义组件和默认的子组件。样式仅在我们的自定义组件上声明，这意味着如果你从画布 (canvas) 中移除了所有 `.cmp-css-a` 和 `.cmp-css-b` 实例，它们的 CssRules 仍将存储在项目中（**这里我们讨论的不是能够跳过未使用规则的 CSS 导出，而是存储在项目 JSON 中的实例**）。

最纯粹的方法是遵循面向组件的样式设计，即仅在组件自身的作用域内声明样式。下面是使用上述示例的实现方式。

```js
domc.addType('cmp-a', {
  model: {
    defaults: {
      attributes: { class: 'cmp-css-a' },
      components: '组件 A',
      styles: `
        .cmp-css-a { color: green }
        @media (max-width: 992px) {
          .cmp-css-a { color: darkgreen }
        }
      `,
    },
  },
});
domc.addType('cmp-b', {
  model: {
    defaults: {
      attributes: { class: 'cmp-css-b' },
      components: '组件 B',
      styles: `
        .cmp-css-b { color: blue }
        @media (max-width: 992px) {
          .cmp-css-b { color: darkblue }
        }
      `,
    },
  },
});
domc.addType('component-css', {
  model: {
    defaults: {
      attributes: { class: 'cmp-css' },
      components: ['<span>带样式的组件<span>', { type: 'cmp-a' }, { type: 'cmp-b' }],
      styles: `
        .cmp-css { color: red }
        @media (max-width: 992px) {
          .cmp-css{ color: darkred; }
        }
      `,
    },
  },
});
```

::: tip 组件优先样式 (Component-first styling)
默认情况下，当你在画布 (canvas) 中选择一个组件并对其应用样式时，更改将应用于其现有的类。这将导致所有具有这些应用类的组件发生更改。如果你需要样式仅应用于特定的选定组件，则必须通过以下方式选择 `componentFirst` 策略。

```js
grapesjs.init({
  ...
  selectorManager: {
    componentFirst: true,
  },
})
```
:::

### 外部 CSS (External CSS)

如果你需要加载特定于组件的外部 CSS，则必须依赖 `script` 属性。更多详细信息，请参阅[组件与 JS (Components & JS)](Components-js.html)。

## 组件与 JS (Components & JS)

如果你想知道如何创建带有附加 JavaScript 的组件（例如计数器、画廊、幻灯片等），请查看专门的页面
[组件与 JS (Components & JS)](Components-js.html)

## 技巧 (Tips)

### JSX 语法 (JSX syntax)

如果你正在将大量 HTML 字符串块导入编辑器（例如，通过块 (Blocks) 定义），JSX 可能是性能和代码可读性之间的一个很好的折衷方案，因为它允许你通过保持 HTML 语法来跳过解析和组件识别步骤。
默认情况下，GrapesJS 理解从 React JSX 预设生成的对象，因此，如果你在 React 应用程序中工作，你可能已经在使用 JSX，并且不需要做任何其他事情，你的环境已经配置为解析 JavaScript 文件中的 JSX。

所以，与其这样写：

```js
// 我正在添加一个字符串，因此将执行解析和组件识别步骤
editor.addComponents(`<div>
  <span data-gjs-type="custom-component" data-gjs-prop="someValue" title="foo">
    你好！
  </span>
</div>`);
```

或者这样

```js
// 我正在传递组件定义 (Component Definition)，因此将跳过繁重的步骤，但代码可读性较差
editor.addComponents({
  tagName: 'div',
  components: [
    {...}
  ],
});
```

你可以使用这种格式

```js
editor.addComponents(
  <div>
    <custom-component data-gjs-prop="someValue" title="foo">
      你好！
    </custom-component>
  </div>,
);
```

通过使用 JSX，你将获得的另一个很酷的功能是能够将组件类型作为元素标签 `<custom-component>` 传递，而不是 `data-gjs-type="custom-component"`。

#### 设置 JSX 语法 (Setup JSX syntax)

对于那些不使用 React 的人，你有以下选项：

- GrapesJS 有一个选项 `config.domComponents.processor`，它允许你轻松实现其他 JSX 预设。如果你使用不同于 React 但使用 JSX 的框架（例如 Vue），则此场景很有用。在这种情况下，JSX pragma 函数（React 使用 `React.createElement`）的结果对象将不同（你可以记录 JSX 以查看结果对象），你必须将其转换为 GrapesJS [组件定义 (Component Definition)] 对象。以下是一个用法示例：

```js
grapesjs.init({
  // ...
  domComponents: {
    processor: (obj) => {
     if (obj.$$typeof) { // 例如，这是一个 React 元素
        const compDef = {
         type: obj.type,
         components: obj.props.children,
         ...
        };
        ...
        return compDef;
     }
    }
  }
})
```

- 如果你需要从头开始支持 JSX（你不使用支持 JSX 的框架），你首先必须实现将文件中的 JSX 转换为 JS 可读内容的解析器。

对于 Babel 用户，只需添加几个插件：`@babel/plugin-syntax-jsx` 和 `@babel-plugin-transform-react`。然后更新你的 `.babelrc` 文件

```json
{
  "plugins": [
    "@babel/plugin-syntax-jsx",
    "@babel/plugin-transform-react-jsx"
  ]
}
```

你还可以自定义执行转换的 pragma 函数 `[“@babel/plugin-transform-react-jsx”, { “pragma”: “customCreateEl” }]`，默认情况下使用 `React.createElement`（你需要在文件中有一个可用的 React 实例才能使其工作）。

这种方法的完整示例可以在[这里](https://codesandbox.io/s/x07xf)找到。

[Component Definition]: #component-definition
[Component]: /api/component.html
[CssRule]: /api/css_rule.html
[Component API]: /api/component.html