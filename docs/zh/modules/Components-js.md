---
title: 组件与JS
---

# 组件与JS

在本指南中，你将了解如何附加组件相关的脚本以及如何处理外部JavaScript库（例如：计数器、画廊、幻灯片等）。

::: warning
本指南适用于 GrapesJS v0.16.34 或更高版本。<br><br>
为了更好地理解本指南中的内容，我们建议首先阅读[组件(Components)](Components.html)和[特征(Traits)]。
:::

::: tip
更喜欢可用于生产环境的现代UI吗？[开始使用 Grapes Studio SDK！](https://app.grapesjs.com/docs-sdk/configuration/components/overview?utm_source=grapesjs-docs&utm_medium=tip)
:::

[[toc]]

## 基础脚本

让我们看看如何创建一个带有脚本的组件。

```js
// 这是我们的自定义脚本 (避免使用箭头函数)
const script = function () {
  alert('Hi');
  // `this` 绑定到组件元素
  console.log('the element', this);
};

// 定义一个新的自定义组件
editor.Components.addType('comp-with-js', {
  model: {
    defaults: {
      script,
      // 添加一些样式，仅为了使组件可见
      style: {
        width: '100px',
        height: '100px',
        background: 'red',
      },
    },
  },
});

// 为组件创建一个区块(block)，以便我们可以轻松拖放它
editor.Blocks.add('test-block', {
  label: 'Test block',
  attributes: { class: 'fa fa-text' },
  content: { type: 'comp-with-js' },
});
```

现在，如果你将新的区块(block)拖到画布(canvas)内，你会看到一个警告框和控制台中的消息，正如你所预期的那样。
值得注意的一点是，`this` 上下文(context)绑定到组件的元素上，因此，例如，如果你需要更改其属性，你会这样做 `this.innerHTML = 'inner content'`。

你需要考虑的一件事是，一旦在画布(canvas)中或最终模板中渲染，脚本是如何绑定到组件的。如果现在你检查编辑器中生成的HTML代码（通过`Export`<sup>导出</sup>按钮或 `editor.getHtml()`），你可能会看到类似这样的内容：

```html
<div id="c764"></div>
<script>
  var items = document.querySelectorAll('#c764');
  for (var i = 0, len = items.length; i < len; i++) {
    (function () {
      // START component code
      alert('Hi');
      console.log('the element', this);
      // END component code
    }).bind(items[i])();
  }
</script>
```

如你所见，编辑器会为所有带有脚本的组件附加一个唯一的ID，并通过 `querySelectorAll` 来检索它们。拖动另一个 `test-block` 将会生成以下内容：

```html
<div id="c764"></div>
<div id="c765"></div>
<script>
  var items = document.querySelectorAll('#c764, #c765');
  for (var i = 0, len = items.length; i < len; i++) {
    (function () {
      // START component code
      alert('Hi');
      console.log('the element', this);
      // END component code
    }).bind(items[i])();
  }
</script>
```

## 重要警告

::: danger
请仔细阅读
:::

请记住，所有组件脚本都在画布(canvas)的 iframe 内部执行（隔离的，就像你的**最终模板**一样），因此它们不属于当前的 `document`。所有外部库（例如，那些你与编辑器一起加载的库）都不在那里（稍后你将看到如何管理带有依赖项的组件）。

这意味着**你不能使用函数作用域之外的东西**。看看这个场景：

```js
const myVar = 'John';

const script = function () {
  alert('Hi ' + myVar);
  console.log('the element', this);
};
```

这将无法工作。你会得到一个 `myVar` 未定义的错误。最终的HTML更清楚地显示了原因：

```html
<div id="c764"></div>
<script>
  var items = document.querySelectorAll('#c764');
  for (var i = 0, len = items.length; i < len; i++) {
    (function () {
      alert('Hi ' + myVar); // <- ERROR: undefined myVar
      console.log('the element', this);
    }).bind(items[i])();
  }
</script>
```

## 向脚本传递属性

假设你需要根据某些组件属性使脚本行为不同，这些属性甚至可以通过[特征(Traits)]进行更改（例如，你想用不同的选项初始化某个库）。你可以通过使用组件上的 `script-props` 属性来实现这一点。

```js
// `props` 参数将只包含你在 `script-props` 中声明的属性
const script = function (props) {
  const myLibOpts = {
    prop1: props.myprop1,
    prop2: props.myprop2,
  };
  alert('My lib options: ' + JSON.stringify(myLibOpts));
};

editor.Components.addType('comp-with-js', {
  model: {
    defaults: {
      script,
      // 为你的自定义属性定义默认值
      myprop1: 'value1',
      myprop2: '10',
      // 定义特征(traits)，以便更改你的属性
      traits: [
        {
          type: 'select',
          name: 'myprop1',
          changeProp: true,
          options: [
            { value: 'value1', name: 'Value 1' },
            { value: 'value2', name: 'Value 2' },
          ],
        },
        {
          type: 'number',
          name: 'myprop2',
          changeProp: true,
        },
      ],
      // 定义要传递哪些属性（这也会在它们更改时重置你的脚本）
      'script-props': ['myprop1', 'myprop2'],
      // ...
    },
  },
});
```

现在，如果你尝试更改特征(traits)，你还会看到脚本将如何带着新的更新属性被触发。

## 依赖管理

正如我们上面提到的，脚本在画布(canvas)内部独立执行，没有任何依赖项，这与编辑器生成的最终HTML完全一样。
如果你想使用外部库，你有两种方法：组件相关和模板相关。

### 组件相关

组件相关的方法无疑是最好的，因为依赖项将动态加载，并且只有当组件存在于画布(canvas)中时，才会打印在最终的HTML中。
你所要做的就是在执行初始化脚本之前加载你的依赖项。

```js
const script = function (props) {
  const initLib = function () {
    const el = this;
    const myLibOpts = {
      prop1: props.myprop1,
      prop2: props.myprop2,
    };
    someExtLib(el, myLibOpts);
  };

  if (typeof someExtLib == 'undefined') {
    const script = document.createElement('script');
    script.onload = initLib;
    script.src = 'https://.../somelib.min.js';
    document.body.appendChild(script);
  } else {
    initLib();
  }
};
```

### 模板相关

某个依赖项可能会在你的所有组件中使用（例如 JQuery），因此，与其在每个脚本内部引入它，你可能希望将其直接注入到画布(canvas)中：

```js
const editor = grapesjs.init({
  ...
  canvas: {
    scripts: ['https://.../somelib.min.js'],
    // 外部样式也是如此
    styles: ['https://.../ext-style.min.css'],
  }
});
```

请记住，编辑器不会在导出的HTML中渲染这些依赖项（例如，通过 `editor.getHtml()`），因此，如何在最终渲染HTML的页面中包含它们取决于你。

[特征(Traits)]: Traits.html