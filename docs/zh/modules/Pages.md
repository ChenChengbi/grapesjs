---
title: 页面
---

# 页面

GrapesJS 中的页面模块 (Pages module) 允许您创建一个包含多个页面的项目 (project)。默认情况下，即使您不需要多页面支持，系统也会在内部 (under the hood) 创建一个页面。这样做可以保持 API (Application Programming Interface) 的一致性 (consistent)，并且在您以后需要添加多个页面时更容易扩展 (extend)。

::: warning
本指南适用于 GrapesJS v0.21.1 或更高版本
:::

::: tip
想让页面功能开箱即用，并拥有一个精致的 UI (User Interface)？[了解 Grapes Studio SDK 是如何实现的！](https://app.grapesjs.com/docs-sdk/configuration/pages?utm_source=grapesjs-docs&utm_medium=tip)
:::
[[toc]]

## 初始化 (Initialization)

默认的编辑器 (Editor) 初始化不需要任何关于页面的知识，这主要是为了在引入页面模块时避免引入破坏性变更 (breaking changes)。

以下是一个典型的编辑器初始化示例：

```js
const editor = grapesjs.init({
  container: '#gjs',
  height: '100%',
  storageManager: false,
  // CSS 或样式 JSON
  style: '.my-el { color: red }',
  // HTML 字符串或组件 JSON
  components: '<div class="my-el">Hello world!</div>',
  // ...其他配置选项
});
```

实际发生的是，此配置会自动迁移到页面管理器 (Page Manager)。

```js
const editor = grapesjs.init({
  container: '#gjs',
  height: '100%',
  storageManager: false,
  pageManager: {
    pages: [
      {
        // 如果没有明确的 ID，将创建一个随机 ID
        id: 'my-first-page',
        // CSS 或样式 JSON
        styles: '.my-el { color: red }',
        // HTML 字符串或组件 JSON
        component: '<div class="my-el">Hello world!</div>',
      },
    ],
  },
});
```

::: warning
值得注意的是，之前的键是 `style` 和 `components`，而在页面中您应该使用 `styles` 和 `component`。
:::

您可能已经猜到，以下是如何使用多个页面初始化编辑器的示例：

```js
const editor = grapesjs.init({
  // ...
  pageManager: {
    pages: [
      {
        id: 'my-first-page',
        styles: '.my-page1-el { color: red }',
        component: '<div class="my-page1-el">Page 1</div>',
      },
      {
        id: 'my-second-page',
        styles: '.my-page2-el { color: blue }',
        component: '<div class="my-page2-el">Page 2</div>',
      },
    ],
  },
});
```

GrapesJS 没有为页面管理器提供任何默认 UI，但您可以利用其 [APIs][页面 API (Pages API)] 轻松构建一个。查看 [自定义](#customization) 部分，了解有关如何创建自己的页面管理器 UI 的更多详细信息。

## 编程方式使用 (Programmatic usage)

如果您需要以编程方式管理页面，可以使用其 [APIs][页面 API (Pages API)]。

以下是一些常用的方法：

```js
// 首先获取 Pages 模块
const pages = editor.Pages;

// 获取所有页面的数组
const allPages = pages.getAll();

// 获取当前选定的页面
const selectedPage = pages.getSelected();

// 添加一个新页面
const newPage = pages.add({
  id: 'new-page-id',
  styles: '.my-class { color: red }',
  component: '<div class="my-class">My element</div>',
});

// 通过 ID 获取页面
const page = pages.get('new-page-id');

// 通过 ID 选择另一个页面
pages.select('new-page-id');
// 或者通过传递 Page 实例
pages.select(page);

// 从页面组件获取 HTML/CSS 代码
const component = page.getMainComponent();
const htmlPage = editor.getHtml({ component });
const cssPage = editor.getCss({ component });

// 通过 ID（或 Page 实例）移除页面
pages.remove('new-page-id');
```

## 自定义 (Customization)

通过使用 [页面 API (Pages API)]，可以轻松创建您自己的页面管理器 UI。

最简单的方法是订阅通用的 `page` 事件，该事件在与页面模块相关的任何更改（与页面内容如组件 (components) 或样式 (styles) 无关）时触发，并相应地更新您的 UI。

```js
const editor = grapesjs.init({
  // ...
});

editor.on('page', () => {
  // 更新您的 UI
});
```

在下面的示例中，您可以看到页面管理器 UI 的快速实现。

<demo-viewer value="1y6bgeo3" height="500" darkcode/>

<!-- 演示模板，此处供参考
<style>
  .app-wrap {
    height: 100%;
    width: 100%;
    display: flex;
  }
  .editor-wrap  {
    widtH: 100%;
    height: 100%;
  }
  .pages-wrp, .pages {
    display: flex;
    flex-direction: column
  }
  .pages-wrp {
      background: #333;
      padding: 5px;
  }
  .add-page {
    background: #444444;
    color: white;
    padding: 5px;
    border-radius: 2px;
    cursor: pointer;
    white-space: nowrap;
    margin-bottom: 10px;
  }
  .page {
    background-color: #444;
    color: white;
    padding: 5px;
    margin-bottom: 5px;
    border-radius: 2px;
    cursor: pointer;

    &.selected {
      background-color: #706f6f
    }
  }

  .page-close {
    opacity: 0.5;
    float: right;
    background-color: #2c2c2c;
    height: 20px;
    display: inline-block;
    width: 17px;
    text-align: center;
    border-radius: 3px;

    &:hover {
      opacity: 1;
    }
  }
</style>

<div style="height: 100%">
  <div class="app-wrap">
    <div class="pages-wrp">
        <div class="add-page" @click="addPage">Add new page</div>
        <div class="pages">
          <div v-for="page in pages" :key="page.id" :class="{page: 1, selected: isSelected(page) }" @click="selectPage(page.id)">
            {{ page.get('name') || page.id }} <span v-if="!isSelected(page)" @click="removePage(page.id)" class="page-close">&Cross;</span>
          </div>
        </div>
    </div>
    <div class="editor-wrap">
      <div id="gjs"></div>
    </div>
  </div>
</div>

<script>
const editor = grapesjs.init({
  container: '#gjs',
  height: '100%',
  storageManager: false,
  plugins: ['gjs-blocks-basic'],
  pageManager: {
    pages: [{
      id: 'page-1',
      name: 'Page 1',
      component: '<div id="comp1">Page 1</div>',
      styles: `#comp1 { color: red }`,
    }, {
      id: 'page-2',
      name: 'Page 2',
      component: '<div id="comp2">Page 2</div>',
      styles: `#comp2 { color: green }`,
    }, {
      id: 'page-3',
      name: 'Page 3',
      component: '<div id="comp3">Page 3</div>',
      styles: `#comp3 { color: blue }`,
    }]
  },
});

const pm = editor.Pages;

const app = new Vue({
  el: '.pages-wrp',
  data: { pages: [] },
  mounted() {
    this.setPages(pm.getAll());
    editor.on('page', () => {
      this.pages = [...pm.getAll()];
    });
  },
  methods: {
    setPages(pages) {
      this.pages = [...pages];
    },
    isSelected(page) {
      return pm.getSelected().id == page.id;
    },
    selectPage(pageId) {
      return pm.select(pageId);
    },
    removePage(pageId) {
      return pm.remove(pageId);
    },
    addPage() {
      const len = pm.getAll().length;
      pm.add({
        name: `Page ${len + 1}`,
        component: '<div>New page</div>',
      });
    },
  }
});
</script>
-->

## 事件 (Events)

有关可用事件的完整列表，您可以[在此处](/api/pages.html#available-events)查看。

[页面 API (Pages API)]: /api/pages.html