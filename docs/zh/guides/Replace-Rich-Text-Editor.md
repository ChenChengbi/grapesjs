---
title: 替换内置的富文本编辑器
---

# 替换内置的富文本编辑器

您可能已经注意到，默认的富文本编辑器 (RTE, Rich Text Editor) 非常小，因此看起来不像一个完整的文本编辑器解决方案。本文将向您展示如何用另一个编辑器完全替换它，而不是演示如何在默认编辑器内部添加新命令。

在接下来的指南中，我们将集成 CKEditor。要完成此任务，我们只需向 GrapesJS API 方法 `setCustomRte` 提供几个函数作为接口 (Interface)。

::: warning
本指南适用于 GrapesJS v0.21.2 或更高版本
:::

[[toc]]

## 接口 (Interface)

### 启用 (Enable)

第一步是说明如何启用 (enable) 第三方库 (third-party library)，因此我们将从 `enable()` 函数开始。此方法不仅负责自定义 RTE 的首次初始化，还负责后续在同一元素上调用时的处理，这就是为什么会有 `rte` 参数的原因。

```js
// 重要：将代码放在一个新的插件 (plugin) 中
const customRTE = (editor) => {
  const focus = (el, rte) => {
    // 稍后实现
  }

  editor.setCustomRte({
    /**
     * 启用自定义 RTE
     * @param  {HTMLElement} el 这是被选中进行编辑的 HTML 节点
     * @param  {Object} rte 这是您从第一次调用 enable() 时返回的实例。
     *                      在第一次调用时，它将是 undefined。当您需要
     *                      检查 RTE 是否已在组件 (component) 上启用时，这很有用
     * @return {Object} 返回值应该是已初始化的 RTE 实例
     */
    enable(el, rte) {
      // 如果已存在，则仅聚焦 (focus)
      if (rte) {
        focus(el, rte);
        return rte;
      }

      // CKEditor 初始化
      rte = CKEDITOR.inline(el, {
        // 您的配置...
        toolbar: [...],
        // 重要
        // 通常，内联编辑器会精确附加到所选元素的相同位置，
        // 但在这种情况下，只有在您开始滚动画布 (canvas) 之前它才能正常工作。
        // 因此，您必须将 RTE 的工具栏 (toolbar) 移动到 GrapesJS 的工具栏内部。
        // 为此，我们使用了一个插件 (plugin)，该插件简化了此过程，
        // 并将所有后续 CKEditor 的工具栏移动到我们指定的元素内
        sharedSpaces: {
          top: editor.RichTextEditor.getToolbarEl(),
        }
      });

      focus(el, rte);
      return rte;
    },
  });
}

const editor = grapesjs.init({
  ...
  plugins: [customRTE],
});
```

### 禁用 (Disable)

了解如何启用 RTE 后，我们来实现禁用 (disable) 它的方法，即创建 `disable()` 函数。

```js
editor.setCustomRte({
  // ...
  /**
   * 函数签名与 `enable` 相同
   */
  disable(el, rte) {
    el.contentEditable = false;
    rte?.focusManager?.blur(true);
  },
});
```

### 内容 (Content)

每个第三方库处理内容 (content) 状态的方式可能不同，预览中实际渲染为 DOM (Document Object Model) 的内容可能不代表最终的 HTML 输出。因此，默认情况下，GrapesJS 直接从 DOM 元素中获取 `innerHTML` 作为最终输出，但强烈建议指定负责以 HTML 字符串形式返回最终状态的方法（每个第三方库的处理方式可能不同）。

```js
editor.setCustomRte({
  // ...
  getContent(el, rte) {
    const htmlString = rte.getData();
    return htmlString;
  },
});
```

### 聚焦 (Focus)

`focus()` 方法只是 `enable()` 内部使用的一个辅助函数，并非接口 (interface) 所必需。

```js
const focus = (el, rte) => {
  // 如果已经聚焦，则不执行任何操作
  if (rte?.focusManager?.hasFocus) {
    return;
  }
  el.contentEditable = true;
  rte?.focus();
};

editor.setCustomRte({
  // ...
  enable(el, rte) {
    // ...
    focus(el, rte);
    // ...
  },
});
```

## 工具栏位置 (Toolbar position)

工具栏 (toolbar) 默认的左上角位置有时并不符合您的需求。例如，当您滚动画布 (canvas) 且工具栏到达顶部时，您可能希望将其向下移动。为此，您可以通过以下方式添加一个监听器来应用您的逻辑：

```js
editor.on('rteToolbarPosUpdate', (pos) => {
  // 例如，根据 `pos` 中传递的附加数据更新 `pos.top` 和 `pos.left`
});
```

## 内置与第三方 (The built-in vs third-party)

使用自定义 RTE 时，您必须记住一件事：所有内容及其行为都由该库本身处理，GrapesJS 的组件 (component) 只会按原样存储内容。
例如，当您使用内置 RTE 创建链接时，您将能够选择它并通过组件设置编辑其 `href`。而使用自定义 RTE 时，显示用于链接编辑的相应模态框 (modal) 将是其自身的任务。
显然，每个第三方库都有其自身的 API (Application Programming Interface)，并且可能存在一些限制和缺点，因此，对该库有最基本的了解会是一个优势。

### 启用内容解析器 (Enable content parser)

作为一个实验性功能，现在可以告知编辑器解析 (parse) 从自定义 RTE 返回的 HTML 内容，并将结果存储为组件，而不仅仅是简单的 HTML 字符串。这使得 GrapesJS 能够以更接近原生实现的方式处理自定义 RTE，并启用诸如[可编辑文本组件 (textable components)](https://github.com/GrapesJS/grapesjs/issues/2771#issuecomment-1040486056) 之类的功能。

```js
editor.setCustomRte({
  // ...
  // 启用内容解析器
  parseContent: true,
});
```

## 插件 (Plugins)

对于 CKEditor，您可以在这里找到一个完整的插件 (plugin)：[grapesjs-plugin-ckeditor](https://github.com/GrapesJS/ckeditor)。