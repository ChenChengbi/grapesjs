---
title: 选择器管理器
---

# 选择器管理器

<p align="center"><img :src="$withBase('/selector-manager.jpg')" alt="GrapesJS - 选择器管理器"/></p>

[选择器(Selector)] 允许在项目中所有 [组件(Components)] 之间复用样式（这与 HTML 中的类(class)完全相同），而选择器管理器(Selector Manager)的主要目标是收集这些选择器并指示当前的选择状态。

::: warning
本指南适用于 GrapesJS v0.17.28 或更高版本
:::

[[toc]]

## 配置

要更改默认配置，您需要传递 `selectorManager` 属性(property)以及主配置对象。

```js
const editor = grapesjs.init({
  ...
  selectorManager: {
    ...
  }
});
```

在此处查看可用选项(option)的完整列表：[选择器管理器配置(Selector Manager Config)](https://github.com/GrapesJS/grapesjs/blob/master/src/selector_manager/config/config.ts)

## 初始化

一旦组件和样式加载完毕，选择器管理器便开始收集数据。默认的 用户界面(UI) 会与 GrapesJS 核心提供的默认 面板(panel) 一同显示。如果您需要使用自己的面板来设置编辑器，我们建议您遵循 [入门指南(Getting Started)]。

在下面的示例中，我们使用已提供的组件和样式来初始化编辑器。

```js
const editor = grapesjs.init({
  container: '#gjs',
  height: '100%',
  storageManager: false,
  components: `
    <div class="class-a">Element A</div>
    <div class="class-a class-b">Element A-B</div>
    <div class="class-a class-b class-c">Element A-B-C</div>
  `,
  style: `
    .class-a { color: red }
    .class-b { color: green }
    .class-c { color: blue }
  `,
});
```

在内部，上面的示例将向选择器管理器提供 3 个选择器：`class-a`、`class-b` 和 `class-c`。

在未选择任何组件的情况下，选择器管理器 UI 默认是隐藏的（与 样式管理器(Style Manager) 一同隐藏）。通过选择 `Element A-B-C`，您将看到当前实际将被应用样式的选择。

<img :src="$withBase('/sm-selected-component.jpg')" alt="选中的组件" style="display: block; margin: auto"/>

标签 **Selected**<sup>已选择</sup> 指示了样式将应用于哪个 CSS 查询，因此如果您尝试更改当前所选内容的颜色，您将在最终代码中得到以下结果：

```css
.class-a.class-b.class-c {
  color: #483acb;
}
```

您还可以禁用特定的选择器并更改 状态(state)（例如 Hover<sup>悬停</sup>）以切换样式目标。

<img :src="$withBase('/sm-disable-selector.jpg')" alt="禁用的选择器" style="display: block; margin: auto"/>

## 组件优先选择器

默认情况下，选择带有类的组件会将其选择器指示为样式目标。这意味着样式管理器中的任何更改都将应用于包含这些 **Selected**<sup>已选择</sup> 类的所有组件。

如果您需要选择单个组件作为样式目标，可以启用 `componentFirst` 选项。

```js
const editor = grapesjs.init({
  // ...
  selectorManager: {
    componentFirst: true,
  },
});
```

此选项还支持对多个组件进行样式设置，并能够将通用选择器与当前组件样式同步（通过刷新图标<sup>refresh icon</sup>）。

<img :src="$withBase('/sm-component-first.jpg')" alt="组件优先" style="display: block; margin: auto"/>

::: warning
进行多选时，样式管理器始终显示最后选定组件的样式。
:::

## 程序化使用

如果您需要以编程方式管理选择器，可以使用其 [API][Selector API]。

## 自定义

默认 UI 可以处理大多数常见任务，但如果您需要更高级的逻辑/元素，则需要替换默认 UI。

您所要做的就是向编辑器表明您打算使用自定义 UI，然后订阅 `selector:custom` 事件，该事件将在 UI 需要任何更新时触发。

```js
const editor = grapesjs.init({
  // ...
  selectorManager: {
    custom: true,
    // ...
  },
});

editor.on('selector:custom', (props) => {
  // props.container (HTMLElement) - 您可以在其中附加 UI 的默认元素
  // 您可以在此处放置渲染/更新 UI 的逻辑。
});
```

在下面的示例中，我们将仅使用选择器管理器 API 来复制大部分默认功能。

<demo-viewer value="v8cgkLfr" height="500" darkcode/>

## 事件

要获取可用事件的完整列表，您可以在[此处](/api/selector_manager.html#available-events)查看。

[选择器]: /api/selector.html
[样式管理器]: Style-manager.html
[组件]: Components.html
[入门指南]: /getting-started.html
[选择器 API]: /api/selector_manager.html