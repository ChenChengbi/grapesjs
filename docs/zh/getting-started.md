---
title: 入门指南
pageClass: page__getting-started
meta:
  - name: keywords
    content: grapesjs 入门
---

# 入门指南

本指南将手把手地指导您使用 GrapesJS 创建自己的构建器(builder)。这不是一份详尽的指南，仅对最常用的模块(modules)进行了简明扼要的概述。请跟随本指南从零开始创建一个页面构建器。您可以直接跳转到页面末尾查看[最终成果](#final-result)。

::: tip

正在寻找 GrapesJS 的可定制版本，并希望它拥有可嵌入、生产就绪的用户界面(UI)吗？[探索 Grapes Studio SDK！](https://app.grapesjs.com/docs-sdk/overview/getting-started)

:::

## 导入库

在开始使用 GrapesJS 之前，您需要先导入它。让我们导入最新版本：

```html
<link rel="stylesheet" href="//unpkg.com/grapesjs/dist/css/grapes.min.css" />
<script src="//unpkg.com/grapesjs"></script>
<!--
If you need plugins, put them below the main grapesjs script
<script src="/path/to/some/plugin.min.js"></script>
-->
```

或者，如果您在 Node 环境中

```js
import 'grapesjs/dist/css/grapes.min.css';
import grapesjs from 'grapesjs';
// If you need plugins, put them below the main grapesjs script
// import 'grapesjs-some-plugin';
```

## 从画布(Canvas)开始

第一步是定义我们编辑器的界面(interface)。为此，我们将从基本的 HTML 布局(layouts)开始。为任何项目找到通用的 UI 结构并非易事。因此，GrapesJS 倾向于使这个过程尽可能简单。我们提供了一些辅助工具(helpers)，但允许用户自定义界面。这保证了最大的灵活性(flexibility)。
GrapesJS 编辑器的主要部分是画布(canvas)，您在这里创建模板(templates)的结构，这是不可或缺的。让我们尝试初始化一个只有画布而没有面板(panels)的编辑器。

<<< @/.vuepress/components/demos/DemoCanvasOnly.html
<<< @/.vuepress/components/demos/DemoCanvasOnly.js
<<< @/.vuepress/components/demos/DemoCanvasOnly.css
<Demo>
<DemoCanvasOnly/>
</Demo>

仅使用画布，您就已经可以移动、复制和删除结构中的组件(components)了。目前，我们看到的是从容器(container)中获取的示例模板。接下来，让我们看看如何创建自定义区块(blocks)并将其拖拽到画布中。

## 添加区块(Blocks)

GrapesJS 中的区块(block)只是一个可复用的 HTML 片段，您可以将其拖放到画布中。区块可以是一张图片、一个按钮，或者一个包含视频、表单(forms)和内联框架(iframes)的完整区域(section)。让我们先创建另一个容器，并在其中追加一些基本区块。稍后，我们可以使用这种技术来构建更复杂的结构。

```html{4}
<div id="gjs">
  ...
</div>
<div id="blocks"></div>
```

```js
const editor = grapesjs.init({
  // ...
  blockManager: {
    appendTo: '#blocks',
    blocks: [
      {
        id: 'section', // id is mandatory
        label: '<b>Section</b>', // You can use HTML/SVG inside labels
        attributes: { class: 'gjs-block-section' },
        content: `<section>
          <h1>This is a simple title</h1>
          <div>This is just a Lorem text: Lorem ipsum dolor sit amet</div>
        </section>`,
      },
      {
        id: 'text',
        label: 'Text',
        content: '<div data-gjs-type="text">Insert your text here</div>',
      },
      {
        id: 'image',
        label: 'Image',
        // Select the component once it's dropped
        select: true,
        // You can pass components as a JSON instead of a simple HTML string,
        // in this case we also use a defined component type `image`
        content: { type: 'image' },
        // This triggers `active` event on dropped components and the `image`
        // reacts by opening the AssetManager
        activate: true,
      },
    ],
  },
});
```

```css
.gjs-block {
  width: auto;
  height: auto;
  min-height: auto;
}
```

<Demo>
 <DemoBasicBlocks/>
</Demo>

如您所见，我们通过初始配置添加区块。显然，在某些情况下，您可能希望动态添加它们，这时您需要使用[区块管理器 API(Block Manager API)](api/block_manager.html)：

```js
editor.BlockManager.add('my-block-id', {
  label: '...',
  category: '...',
  // ...
});
```

::: tip
如果您想了解更多关于区块的信息，我们建议您阅读其专门的文章：[区块管理器模块(Block Manager Module)](modules/Blocks.html)。
:::

## 定义组件(Components)

从技术上讲，一旦您将 HTML 区块拖放到画布内，内容的每个元素都会转换成一个 GrapesJS 组件(Component)。GrapesJS 组件是一个对象，它包含了元素如何在画布中渲染（在视图(View)中管理）以及它在最终代码中可能呈现的样子（由模型(Model)中的属性(properties)创建）等信息。通常，所有模型属性都会反映在视图中。因此，如果您向模型添加一个新属性(attribute)，它将在导出代码中可用（我们稍后会详细了解），并且您在画布中看到的元素也将使用新属性进行更新。
这并非完全不同寻常，但组件的独特之处在于您可以创建一个完全解耦(decoupled)的视图。这意味着无论模型中有什么，您都可以向用户显示任何您想要的内容。例如，通过拖动占位符文本(placeholder text)，您可以获取并显示动态内容。如果您想了解更多关于自定义组件的信息，您应该查阅[组件管理器模块(Component Manager Module)](modules/Components.html)。

GrapesJS 自带一些[内置组件(built-in Components)](modules/Components.html#built-in-component-types)，这些组件在画布中渲染后会启用不同的功能。例如，双击图像组件，您将看到默认的[资源管理器(Asset Manager)](modules/Assets.html)，您可以对其进行自定义或集成您自己的资源管理器。双击文本组件，您可以通过内置的富文本编辑器(Rich Text Editor)对其进行编辑，该编辑器也是可自定义和[可替换的](guides/Replace-Rich-Text-Editor.html)。

正如我们之前所见，您可以直接将区块创建为组件：

```js
editor.BlockManager.add('my-block-id', {
  // ...
  content: {
    tagName: 'div',
    draggable: false,
    attributes: { 'some-attribute': 'some-value' },
    components: [
      {
        tagName: 'span',
        content: '<b>Some static content</b>',
      },
      {
        tagName: 'div',
        // use `content` for static strings, `components` string will be parsed
        // and transformed in Components
        components: '<span>HTML at some point</span>',
      },
    ],
  },
});
```

::: tip
查阅[组件 API(Components API)](api/components.html)以了解如何动态地与组件交互。
:::

以下示例展示了如何选择某个内部组件并用新内容替换其子组件：

```js
// The wrapper is the root Component
const wrapper = editor.DomComponents.getWrapper();
const myComponent = wrapper.find('div.my-component')[0];
myComponent.components().forEach(component => /* ... do something ... */);
myComponent.components('<div>New content</div>');
```

## 面板(Panels)与按钮(Buttons)

现在我们有了画布和自定义区块，让我们看看如何创建一个新的自定义面板，并在其中添加一些按钮（使用[面板 API(Panels API)](api/panels.html)），这些按钮可以触发命令(commands)（核心命令或自定义命令）。

```html{1,2,3}
<div class="panel__top">
    <div class="panel__basic-actions"></div>
</div>
<div id="gjs">
  ...
</div>
<div id="blocks"></div>
```

```css
.panel__top {
  padding: 0;
  width: 100%;
  display: flex;
  position: initial;
  justify-content: center;
  justify-content: space-between;
}
.panel__basic-actions {
  position: initial;
}
```

```js
editor.Panels.addPanel({
  id: 'panel-top',
  el: '.panel__top',
});
editor.Panels.addPanel({
  id: 'basic-actions',
  el: '.panel__basic-actions',
  buttons: [
    {
      id: 'visibility',
      active: true, // active by default
      className: 'btn-toggle-borders',
      label: '<u>B</u>',
      command: 'sw-visibility', // Built-in command
    },
    {
      id: 'export',
      className: 'btn-open-export',
      label: 'Exp'<sup>导出</sup>,
      command: 'export-template',
      context: 'export-template', // For grouping context of buttons from the same panel
    },
    {
      id: 'show-json',
      className: 'btn-show-json',
      label: 'JSON'<sup>JSON</sup>,
      context: 'show-json',
      command(editor) {
        editor.Modal.setTitle('Components JSON'<sup>组件JSON</sup>)
          .setContent(
            `<textarea style="width:100%; height: 250px;">
            ${JSON.stringify(editor.getComponents())}
          </textarea>`,
          )
          .open();
      },
    },
  ],
});
```

<Demo>
 <DemoCustomPanels/>
</Demo>

我们使用 `el: '#basic-panel'` 定义了面板的渲染位置，然后为每个按钮添加了 `command` 属性。该命令可以是 ID、一个包含 `run` 和 `stop` 函数的对象，或者仅仅是一个函数。
尽可能使用[命令(Commands)](api/commands.html)，它们允许您全局跟踪操作。命令还会在其执行前后执行回调(callbacks)（您甚至可以中断它们）。

```js
editor.on('run:export-template:before', (opts) => {
  console.log('Before the command run');
  if (0 /* some condition */) {
    opts.abort = 1;
  }
});
editor.on('run:export-template', () => console.log('After the command run'));
editor.on('abort:export-template', () => console.log('Command aborted'));
```

::: tip
查阅[面板 API(Panels API)](api/panels.html)以查看所有可用的方法。
:::

## 图层(Layers)

在处理 Web 元素时，您可能会发现另一个有用的工具是图层管理器(layer manager)。它是结构节点(nodes)的树状概览，使您能够更轻松地管理结构。要启用它，您只需指定要渲染它的位置。

```html{4,5,6,7,8,9,10,11}
<div class="panel__top">
    <div class="panel__basic-actions"></div>
</div>
<div class="editor-row">
  <div class="editor-canvas">
    <div id="gjs">...</div>
  </div>
  <div class="panel__right">
    <div class="layers-container"></div>
  </div>
</div>
<div id="blocks"></div>
```

<<< @/.vuepress/components/demos/DemoLayers.css

```js
const editor = grapesjs.init({
  // ...
  layerManager: {
    appendTo: '.layers-container',
  },
  // We define a default panel as a sidebar to contain layers
  panels: {
    defaults: [
      {
        id: 'layers',
        el: '.panel__right',
        // Make the panel resizable
        resizable: {
          maxDim: 350,
          minDim: 200,
          tc: false, // Top handler
          cl: true, // Left handler
          cr: false, // Right handler
          bc: false, // Bottom handler
          // Being a flex child we need to change `flex-basis` property
          // instead of the `width` (default)
          keyWidth: 'flex-basis',
        },
      },
    ],
  },
});
```

<Demo>
 <DemoLayers/>
</Demo>

## 样式管理器(Style Manager)

定义了模板结构之后，下一步就是对其进行样式设置。为了满足这一需求，GrapesJS 包含了样式管理器(Style Manager)模块，该模块由 CSS 样式属性和区域(sectors)组成。为了更清楚地说明，让我们看看如何定义一个基本集合。
让我们先在 `panel__right` 内添加一个面板，并在 `panel__top` 中添加另一个面板，用于包含图层/样式管理器切换器：

```html{3,8}
<div class="panel__top">
    <div class="panel__basic-actions"></div>
    <div class="panel__switcher"></div>
</div>
...
  <div class="panel__right">
    <div class="layers-container"></div>
    <div class="styles-container"></div>
  </div>
...
```

```css
.panel__switcher {
  position: initial;
}
```

```js
const editor = grapesjs.init({
  // ...
  panels: {
    defaults: [
      // ...
      {
        id: 'panel-switcher',
        el: '.panel__switcher',
        buttons: [
          {
            id: 'show-layers',
            active: true,
            label: 'Layers'<sup>图层</sup>,
            command: 'show-layers',
            // Once activated disable the possibility to turn it off
            togglable: false,
          },
          {
            id: 'show-style',
            active: true,
            label: 'Styles'<sup>样式</sup>,
            command: 'show-styles',
            togglable: false,
          },
        ],
      },
    ],
  },
  // The Selector Manager allows to assign classes and
  // different states (eg. :hover) on components.
  // Generally, it's used in conjunction with Style Manager
  // but it's not mandatory
  selectorManager: {
    appendTo: '.styles-container',
  },
  styleManager: {
    appendTo: '.styles-container',
    sectors: [
      {
        name: 'Dimension'<sup>尺寸</sup>,
        open: false,
        // Use built-in properties
        buildProps: ['width', 'min-height', 'padding'],
        // Use `properties` to define/override single property
        properties: [
          {
            // Type of the input,
            // options: integer | radio | select | color | slider | file | composite | stack
            type: 'integer',
            name: 'The width'<sup>宽度</sup>, // Label for the property
            property: 'width', // CSS property (if buildProps contains it will be extended)
            units: ['px', '%'], // Units, available only for 'integer' types
            defaults: 'auto', // Default value
            min: 0, // Min value, available only for 'integer' types
          },
        ],
      },
      {
        name: 'Extra'<sup>额外</sup>,
        open: false,
        buildProps: ['background-color', 'box-shadow', 'custom-prop'],
        properties: [
          {
            id: 'custom-prop',
            name: 'Custom Label'<sup>自定义标签</sup>,
            property: 'font-size',
            type: 'select',
            defaults: '32px',
            // List of options, available only for 'select' and 'radio'  types
            options: [
              { value: '12px', name: 'Tiny'<sup>小号</sup> },
              { value: '18px', name: 'Medium'<sup>中号</sup> },
              { value: '32px', name: 'Big'<sup>大号</sup> },
            ],
          },
        ],
      },
    ],
  },
});

// Define commands
editor.Commands.add('show-layers', {
  getRowEl(editor) {
    return editor.getContainer().closest('.editor-row');
  },
  getLayersEl(row) {
    return row.querySelector('.layers-container');
  },

  run(editor, sender) {
    const lmEl = this.getLayersEl(this.getRowEl(editor));
    lmEl.style.display = '';
  },
  stop(editor, sender) {
    const lmEl = this.getLayersEl(this.getRowEl(editor));
    lmEl.style.display = 'none';
  },
});
editor.Commands.add('show-styles', {
  getRowEl(editor) {
    return editor.getContainer().closest('.editor-row');
  },
  getStyleEl(row) {
    return row.querySelector('.styles-container');
  },

  run(editor, sender) {
    const smEl = this.getStyleEl(this.getRowEl(editor));
    smEl.style.display = '';
  },
  stop(editor, sender) {
    const smEl = this.getStyleEl(this.getRowEl(editor));
    smEl.style.display = 'none';
  },
});
```

<Demo>
  <DemoStyle/>
</Demo>

在样式管理器定义中，我们使用 `buildProps` 来帮助我们从[可用的内置对象](modules/Style-manager.html#built-in-properties)创建通用属性，然后在 `properties` 中，我们可以覆盖由 `property` 名称标识的相同对象（例如，传递另一个 `name` 来更改标签）。从 `custom-prop` 示例中可以看出，这实际上就是定义 CSS `property` 和输入 `type`。我们建议您从[网页预设演示](https://github.com/GrapesJS/grapesjs/blob/gh-pages/demo.html#L1000)中查看更完整的样式管理器属性用法示例。

::: tip
查阅[样式管理器 API(Style Manager API)](api/panels.html)以了解如何动态更新区域和属性。
:::

<!--
To get more about style manager extension check out this guide.
Each component can also indicate what to style and what not.

-- Example component with limit styles
-->

## 特性(Traits)

大多数情况下，您会为组件设置样式并将其放置在结构中的某个位置，但有时您的组件可能需要自定义属性甚至自定义行为，为此您可以使用特性(traits)。特性通常用于更新 HTML 元素属性（例如，输入框的 `placeholder` 或图像的 `alt`），但您也可以定义自己的自定义特性。访问所选组件模型并执行任何您想要的操作。在本指南中，我们将向您展示如何渲染可用的特性，有关如何扩展它们的更多详细信息，建议您阅读[特性管理器模块页面(Trait Manager Module page)](modules/Traits.html)。
让我们为特性创建一个新容器。告诉编辑器在哪里渲染它并更新侧边栏切换器：

```html{5}
...
  <div class="panel__right">
    <div class="layers-container"></div>
    <div class="styles-container"></div>
    <div class="traits-container"></div>
  </div>
...
```

```js
const editor = grapesjs.init({
  // ...
  panels: {
    defaults: [
      // ...
      {
        id: 'panel-switcher',
        el: '.panel__switcher',
        buttons: [
          // ...
          {
            id: 'show-traits',
            active: true,
            label: 'Traits'<sup>特性</sup>,
            command: 'show-traits',
            togglable: false,
          },
        ],
      },
    ],
  },
  traitManager: {
    appendTo: '.traits-container',
  },
});

// Define command
// ...
editor.Commands.add('show-traits', {
  getTraitsEl(editor) {
    const row = editor.getContainer().closest('.editor-row');
    return row.querySelector('.traits-container');
  },
  run(editor, sender) {
    this.getTraitsEl(editor).style.display = '';
  },
  stop(editor, sender) {
    this.getTraitsEl(editor).style.display = 'none';
  },
});
```

<Demo>
  <DemoTraits/>
</Demo>

现在，如果您切换到特性面板并选择其中一个内部组件，您应该会看到其默认特性。

## 响应式模板(Responsive templates)

GrapesJS 实现了一个模块，使您可以轻松处理响应式模板。让我们看看如何定义不同的设备(devices)以及用于设备切换的按钮：

```html{3}
<div class="panel__top">
    <div class="panel__basic-actions"></div>
    <div class="panel__devices"></div>
    <div class="panel__switcher"></div>
</div>
...
```

```css
.panel__devices {
  position: initial;
}
```

```js
const editor = grapesjs.init({
  // ...
  deviceManager: {
    devices: [
      {
        name: 'Desktop',
        width: '', // default size
      },
      {
        name: 'Mobile',
        width: '320px', // this value will be used on canvas width
        widthMedia: '480px', // this value will be used in CSS @media
      },
    ],
  },
  // ...
  panels: {
    defaults: [
      // ...
      {
        id: 'panel-devices',
        el: '.panel__devices',
        buttons: [
          {
            id: 'device-desktop',
            label: 'D'<sup>桌面</sup>,
            command: 'set-device-desktop',
            active: true,
            togglable: false,
          },
          {
            id: 'device-mobile',
            label: 'M'<sup>移动</sup>,
            command: 'set-device-mobile',
            togglable: false,
          },
        ],
      },
    ],
  },
});

// Commands
editor.Commands.add('set-device-desktop', {
  run: (editor) => editor.setDevice('Desktop'),
});
editor.Commands.add('set-device-mobile', {
  run: (editor) => editor.setDevice('Mobile'),
});
```

<Demo>
  <DemoDevices/>
</Demo>

从命令定义中可以看出，我们使用 `editor.setDevice` 方法来更改视口(viewport)的大小。如果您需要在设备更改时触发操作，可以像这样设置一个监听器(listener)：

```js
editor.on('change:device', () => console.log('Current device: ', editor.getDevice()));
```

那么移动优先(mobile-first)的方法呢？您可以通过以下方式更改配置来实现：

```js
const editor = grapesjs.init({
  // ...
  mediaCondition: 'min-width', // default is `max-width`
  deviceManager: {
    devices: [
      {
        name: 'Mobile',
        width: '320',
        widthMedia: '',
      },
      {
        name: 'Desktop',
        width: '',
        widthMedia: '1024',
      },
    ],
  },
  // ...
});

// Set initial device as Mobile
editor.setDevice('Mobile');
```

::: tip
查阅[设备管理器 API(Device Manager API)](api/device_manager.html)以查看所有可用的方法。
:::

## 存储和加载数据

完成构建器界面的定义后，下一步是设置存储和加载过程。
GrapesJS 在其存储管理器(Storage Manager)中实现了两种简单的存储类型：本地存储（使用 `localStorage`，默认激活）和远程存储。这些足以涵盖大多数情况，但也可以添加新的实现（[grapesjs-indexeddb](https://github.com/GrapesJS/storage-indexeddb) 是一个很好的例子）。
让我们看看默认选项是如何工作的：

```js
grapesjs.init({
  // ...
  storageManager: {
    type: 'local', // Type of the storage, available: 'local' | 'remote'
    autosave: true, // Store data automatically
    autoload: true, // Autoload stored data on init
    stepsBeforeSave: 1, // If autosave enabled, indicates how many changes are necessary before store method is triggered
    options: {
      local: {
        // Options for the `local` type
        key: 'gjsProject', // The key for the local storage
      },
    },
  },
});
```

让我们看一下设置远程存储所需的配置：

```js
grapesjs.init({
  // ...
  storageManager: {
    type: 'remote',
    // ...
    stepsBeforeSave: 10,
    options: {
      remote: {
        headers: {}, // Custom headers for the remote storage request
        urlStore: 'https://your-server/endpoint/store', // Endpoint URL where to store data project
        urlLoad: 'https://your-server/endpoint/load', // Endpoint URL where to load data project
      },
    },
  },
});
```

您可能已经注意到，我们保留了一些默认选项不变，增加了触发自动保存所需的更改次数，并传递了远程端点(endpoints)。
如果您愿意，也可以禁用自动保存并使用自定义命令来触发存储：

```js
// ...
  storageManager: {
    type: 'remote',
    autosave: false,
    // ...
  },
  // ...
  commands: {
    defaults: [
      // ...
      {
        id: 'store-data',
        run(editor) {
          editor.store();
        },
      }
    ]
  }
// ...
```

要更好地了解存储管理器以及如何存储/加载模板，或如何定义新的存储，您应该阅读[存储管理器模块(Storage Manager Module)](modules/Storage.html)页面。

## 主题化(Theming)

最后一个可能会极大地改善编辑器个性的步骤是其视觉外观。为了实现简单的主题化，我们为此采用了原子设计(atomic design)。因此，例如，要自定义主调色板(palette)，您所要做的就是将自定义 CSS 规则放在 GrapesJS 样式之后。
为了完成我们的构建器，让我们自定义其调色板，并为了使其在视觉上更“易读”，我们可以用 SVG 图标替换所有按钮标签：

```css
/* We can remove the border we've set at the beginning */
#gjs {
  border: none;
}
/* Theming */

/* Primary color for the background */
.gjs-one-bg {
  background-color: #78366a;
}

/* Secondary color for the text color */
.gjs-two-color {
  color: rgba(255, 255, 255, 0.7);
}

/* Tertiary color for the background */
.gjs-three-bg {
  background-color: #ec5896;
  color: white;
}

/* Quaternary color for the text color */
.gjs-four-color,
.gjs-four-color-h:hover {
  color: #ec5896;
}
```

还有许多[CSS 自定义属性 (变量)](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)可用于自定义编辑器的样式。

例如，您可以通过执行以下操作来达到与上述相同的结果：

```css
:root {
  --gjs-primary-color: #78366a;
  --gjs-secondary-color: rgba(255, 255, 255, 0.7);
  --gjs-tertiary-color: #ec5896;
  --gjs-quaternary-color: #ec5896;
}
```

这是我们的最终成果：

<Demo id="final-result">
  <DemoTheme/>
</Demo>