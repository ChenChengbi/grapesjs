---
title: 模态框
---

# 模态框

**模态框(Modal)** 模块允许您轻松地在对话窗口(dialog window)中显示内容。

::: warning
本指南涉及 GrapesJS v0.17.26 或更高版本。
:::

[[toc]]

## 基本用法

您可以通过调用单个 API (应用程序接口) 来轻松显示您的内容。

```js
// 初始化编辑器
const editor = grapesjs.init({ ... });
// 打开模态框
const openModal = () => {
    editor.Modal.open({
        title: 'My title', // 字符串 | HTMLElement
        content: 'My content', // 字符串 | HTMLElement
    });
};
// 创建一个简单的自定义按钮，用于打开模态框
document.body.insertAdjacentHTML('afterbegin',`
    <button onclick="openModal()">Open Modal</button>
`);
```

## 使用 API

通过使用其他[可用的 API](/api/modal_dialog.html)，您可以完全控制模态框（例如，更新内容/标题、关闭模态框等）。

以下是一些示例：

```js
const { Modal } = editor;

// 关闭模态框
Modal.close();

// 检查模态框是否打开
Modal.isOpen();

// 更新标题
Modal.setTitle('New title');

// 更新内容
Modal.setContent('New content');

// 在模态框关闭时执行一次性回调(callback)
Modal.onceClose(() => {
  console.log('My last modal is closed');
});
```

## 自定义

模态框可以完全自定义，并且您有不同的可用选项。
最快且最简单的方法是为模态框元素使用您特定的 CSS (层叠样式表)。只需几行 CSS (层叠样式表)，您的模态框就可以完全根据您的选择进行调整。

```css
.gjs-mdl-dialog {
  background-color: white;
  color: #333;
}
```

如果您需要对特定模态框进行不同的自定义，可以依赖您的自定义类属性(attributes)。

```js
editor.Modal.open({
  title: 'My title',
  content: 'My content',
  attributes: {
    class: 'my-small-modal',
  },
});
```

```css
.my-small-modal .gjs-mdl-dialog {
  max-width: 300px;
}
```

::: warning
您的自定义 CSS 必须在 GrapesJS 的 CSS 之后加载。
:::

### 自定义模态框

对于更高级的用法，您可以完全用自己的模态框替换默认的模态框。您所要做的就是向编辑器指明您打算使用自定义模态框，然后订阅 `modal` 事件(event)，该事件(event)将为您提供有关任何请求更改的所有信息。

```js
const editor = grapesjs.init({
  // ...
  modal: { custom: true },
});

editor.on('modal', (props) => {
  // `props` 将包含更新自定义模态框所需的所有信息。
  // props.open (boolean) - 指示模态框是否应打开
  // props.title (Node) - 模态框标题 (Node<sup>节点</sup>)
  // props.content (Node) - 模态框内容 (Node<sup>节点</sup>)
  // props.attributes (Object) - 模态框自定义属性 (Object<sup>对象</sup>) （例如 class）
  // props.close (Function) - 当您想以编程方式关闭模态框时使用的回调 (Function<sup>函数</sup>)
  // 在这里，您将放置控制模态框的逻辑。
});
```

以下是使用 Bootstrap 模态框的示例。

<demo-viewer value="x70amv3f" height="500" darkcode/>

## 事件

有关可用事件(event)的完整列表，您可以在[此处](/api/modal_dialog.html#available-events)查看。