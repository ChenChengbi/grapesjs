---
title: 使用自定义 CSS 解析器
---

# 使用自定义 CSS 解析器

如果你仅仅使用 GrapesJS 从头构建模板，即从一个空白的画布开始，并且编辑时严格依赖生成的 JSON（最终的 HTML/CSS 仅供最终用户使用），那么，你或许可以跳过本指南。另一方面，如果你从已定义的 HTML/CSS 导入模板，或者允许用户嵌入自定义代码（例如，使用 [grapesjs-custom-code](https://github.com/GrapesJS/components-custom-code) 插件 (plugin)），那么你需要知道可能会遇到一些奇怪的行为。

::: warning
本指南要求 GrapesJS 版本为 v0.14.33 或更高
:::

[[toc]]

## 导入 HTML/CSS

导入已定义的 HTML/CSS 是一个非常好的功能，因为它能让你立即开始编辑任何类型的模板 (template)，显然，GrapesJS 本身也提倡这种方法。

```html
<div id="gjs">
  <div class="txt-red">Hello world!</div>
  <style>
    .txt-red {
      color: red;
    }
  </style>
</div>

<script type="text/javascript">
  const editor = grapesjs.init({
    container: '#gjs',
    fromElement: true,
  });
</script>
```

为了更快、更便捷地工作，GrapesJS 需要将一个简单的字符串 (HTML/CSS) 编译成结构化的节点 (node)（嵌套的 JS 对象）。幸运的是，大部分繁重的工作（解析 (parsing)）已经由浏览器本身完成了，浏览器将其字符串转换为自己的对象（[DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) (文档对象模型 Document Object Model) / [CSSOM](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Object_Model) (CSS 对象模型 CSS Object Model)），因此我们只需依赖这些对象，通过遍历它们并创建我们的节点 (node)（不幸的是，浏览器的对象还不够用）。我们能够仅使用浏览器本身来解析我们的字符串，这一点非常酷，我们可以在不需要任何第三方库 (third-party library) 的情况下启用导入功能，那么……问题出在哪里呢？好吧，虽然生成的 DOM 表现相当不错，我们能够提取所需内容，但不幸的是，CSSOM 的情况并非如此，让我们在下一段看看它有什么问题。

## CSSOM 结果不一致

不幸的是，我们发现浏览器生成的 CSSOM 与我们要求解析的内容高度不一致。为了证明这一点，我们将使用内置的解析器 (parser) 创建一个简单的示例，并检查其结果。
因此，在我们的案例中，我们只考虑一个简单的规则，我们将解析它并在屏幕上打印 CSSOM 结果。

```html
<h1>To parse</h1>
<pre id="css-to-parse">
  .simple-class {
    background-image:url("https://image1.png"), url("https://image2.jpg");
    background-attachment: fixed, scroll;
    background-position:left top, center center;
    background-repeat:repeat-y, no-repeat;
    background-size: contain, cover;
    box-shadow: 0 0 5px #9d7aa5, 0 0 10px #e6c3ee;
    border: 2px solid #FF0000;
  }
</pre>

<h1>Result</h1>
<pre id="result"></pre>

<script>
  // We use ES5 just to make it more cross-browser, without the need of being compiled

  function parse(str) {
    var result = [];
    // Create the element which will contain the style to parse
    var el = document.createElement('style');
    el.innerHTML = str;
    // We have to append the style to get its CSSOM
    document.head.appendChild(el);
    var sheet = el.sheet;
    // Now we can remove it
    document.head.removeChild(el);

    return sheet;
  }

  function CSSOMToString(root) {
    // For the sake of brevity we just print what we need
    var styleStr = '';
    var rule = root.cssRules[0];
    var style = rule.style;
    // The only way we have to iterate over CSSStyleDeclaration
    for (var i = 0, len = style.length; i < len; i++) {
      var property = style[i];
      var value = style.getPropertyValue(property);
      styleStr += '\t' + property + ': ' + value + ';\n';
    }
    var result = document.getElementById('result');
    result.innerHTML = rule.selectorText + ' {\n' + styleStr + '}';
  }

  var css = document.getElementById('css-to-parse').innerText;
  CSSOMToString(parse(css));
</script>
```

### 结果

以下是一些结果（使用最新版本 + IE11）

<img :src="$withBase('/cssom-result.jpg')">

如你所见，这就是我们仅请求 7 个属性所得到的结果，有的浏览器增加或减少了属性，有的将颜色转换为 rgba 函数，还有的更改了我们值的顺序（例如 `box-shadow`）。基于 Webkit 的浏览器 (Webkit-based browsers) 甚至会附加它们自己不理解的属性。

<img :src="$withBase('/cssom-devtools.png')">

因此，很明显我们不能依赖 CSSOM 对象，这就是为什么我们添加了通过 `editor.setCustomParserCss` 方法或 `config.Parser.parserCss` 选项在初始化 (initialization) 时设置自定义 CSS 解析器的功能。让我们详细看看它是如何工作的。

## CSSOM 结果可能不直观

根据当前的 [csswg 规范](https://drafts.csswg.org/css-variables-1/#variables-in-shorthands)，简写属性 (shorthand properties) 中的变量可能会序列化 (serialize) 为空字符串。
这意味着虽然 `background-color: var(--my-var)` 会正常序列化，但 `background: var(--my-var)` 则不会。

## 设置 CSS 解析器

你需要使用的自定义解析器只是一个接收2个参数的函数：`css`，作为要解析的 CSS 字符串，以及 `editor`，当前编辑器的实例。作为结果，你应该返回一个包含有效规则对象的数组，这些对象的语法在下面解释。以下是如何设置自定义解析器：

```js
const parserCss = (css, editor) => {
  const result = [];
  // ... parse the CSS string
  result.push({
    selectors: '.someclass, div .otherclass',
    style: { color: 'red' },
  });
  // ...
  return result; // Result should be ALWAYS an array
};

// On initialization
// This is the recommended way, as you gonna use the parser from the beginning
const editor = grapesjs.init({
  //...
  parser: {
    parserCss,
  },
});

// Or later, via editor API
editor.setCustomParserCss(parserCss);
```

## 规则对象

规则对象的语法非常直接，每个对象可能包含以下键：

| 键          | 描述                                                                                   | 示例                         |
| ----------- | -------------------------------------------------------------------------------------- | ------------------------------- |
| `selectors` | 规则的选择器 (selectors)。 <br> **必需** 如果规则没有选择器，则返回空字符串。           | `.class1, div > #someid`        |
| `style`     | 样式 (style) 声明，以对象形式表示。                                                    | `{ color: 'red' }`              |
| `atRule`    | @规则 (atRule) 名称                                                                    | `media`                         |
| `params`    | @规则的参数 (params)。                                                                   | `screen and (min-width: 480px)` |

为了更清楚地说明，让我们看几个例子：

```js
// 输入
`
@font-face {
  font-family: "Font Name";
  src: url("https://font-url.eot");
}
`
// 输出
[
  {
    selectors: '',
    atRule: 'font-face',
    style: {
      'font-family': '"Font Name"',
      src: 'url("https://font-url.eot")',
    },
  }
]

// 输入
`
@keyframes keyframe-name {
  from { opacity: 0; }
  to { opacity: 1; }
}
`
// 输出
[
  {
    params: 'keyframe-name',
    selectors: 'from',
    atRule: 'keyframes',
    style: {
      opacity: '0',
    },
  }, {
    params: 'keyframe-name',
    selectors: 'to',
    atRule: 'keyframes',
    style: {
      opacity: '1',
    },
  }
]

// 输入
`
@media screen and (min-width: 480px) {
    body {
        background-color: lightgreen;
    }

    .class-test, .class-test2:hover {
      color: blue !important;
    }
}
`
// 输出
[
  {
    params: 'screen and (min-width: 480px)',
    selectors: 'body',
    atRule: 'media',
    style: {
      'background-color': 'lightgreen',
    },
  }, {
    params: 'screen and (min-width: 480px)',
    selectors: '.class-test, .class-test2:hover',
    atRule: 'media',
    style: {
      color: 'blue !important',
    },
  }
]

// 输入
`
:root {
  --some-color: red;
  --some-width: 55px;
}
`
// 输出
[
  {
    selectors: ':root',
    style: {
      '--some-color': 'red',
      '--some-width': '55px',
    },
  },
]
```

## 插件

以下是当前可用的 CSS 解析器插件列表，如果你需要创建自己的插件，我们强烈建议你研究它们的源代码。

- [grapesjs-parser-postcss](https://github.com/GrapesJS/parser-postcss) - 使用 [PostCSS](https://github.com/postcss/postcss) 解析器

--- END OF FILE Custom-CSS-parser.md ---