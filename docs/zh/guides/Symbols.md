---
title: 符号
---

# 符号 (Symbols)

::: warning 警告
此功能自 GrapesJS v0.21.11 版本起作为测试版(beta)发布

为了更好地理解本指南中的内容，我们建议您先阅读[组件]部分
:::

符号是一种特殊类型的[组件 (Component)]，使您能够轻松地在整个项目中复用通用元素。它们对于项目中多次出现且需要保持一致的组件尤其有用。通过使用符号，您可以在一个地方轻松更新这些组件，并将更改反映到所有使用它们的地方。

[[toc]]

## 概念

从组件创建的符号保留了相同的形态，并使用相同的[组件 API (Components API)]，但它包含对其他相关符号的引用。当您从一个组件创建一个新符号时，它会创建一个主符号 (Main Symbol)，而原始组件则会成为一个实例符号 (Instance Symbol)。

当您在其他地方复用该符号时，会创建新的实例符号。对主符号所做的任何更新都会自动复制到所有实例符号中，从而确保整个项目的一致性。

下图简单表示了主符号和实例符号之间的连接。

<img :src="$withBase('/symbols-model.svg')">

::: warning 注意
此功能在底层运行，这意味着没有内置的用户界面 (UI) 用于创建和管理符号。开发人员需要实现自己的 UI 来与此功能交互。下面您将看到一个实现示例。
:::

## 编程方式使用

让我们看看如何在您的项目中处理和管理符号。

### 创建符号

从项目中的任何组件创建一个新符号：

```js
const anyComponent = editor.getSelected();
const symbolMain = editor.Components.addSymbol(anyComponent);
```

这将把 `anyComponent` 转换为一个实例，返回的 `symbolMain` 将是主符号。GrapesJS 会在您的项目 JSON 中单独跟踪主符号，并且在您重新加载项目时它们会自动重新连接。

`addSymbol` 方法也处理实例的创建。如果您通过传递 `symbolMain` 或 `anyComponent` 再次调用它，它将创建 `symbolMain` 的一个新实例。

```js
const secondInstance = editor.Components.addSymbol(symbolMain);
```

现在，`symbolMain` 引用了其形态的两个实例。

要获取项目中所有可用的符号，请使用 `getSymbols`：

```js
const symbols = editor.Components.getSymbols();
const symbolMain = symbols[0];
```

### 符号详情

一旦您的项目中有符号，您可能需要知道一个组件何时是符号并获取其详细信息。为此，请使用 `getSymbolInfo` 方法：

```js
// 主符号的详情
const symbolMainInfo = editor.Components.getSymbolInfo(symbolMain);

symbolMainInfo.isSymbol; // true；它是一个符号
symbolMainInfo.isRoot; // true；它是符号的根
symbolMainInfo.isMain; // true；它是主符号
symbolMainInfo.isInstance; // false；它不是实例符号
symbolMainInfo.main; // symbolMainInfo；对主符号的引用
symbolMainInfo.instances; // [anyComponent, secondInstance]；对实例符号的引用
symbolMainInfo.relatives; // [anyComponent, secondInstance]；相关符号

// 实例符号的详情
const secondInstanceInfo = editor.Components.getSymbolInfo(secondInstance);

symbolMainInfo.isSymbol; // true；它是一个符号
symbolMainInfo.isRoot; // true；它是符号的根
symbolMainInfo.isMain; // false；它不是主符号
symbolMainInfo.isInstance; // true；它是实例符号
symbolMainInfo.main; // symbolMainInfo；对主符号的引用
symbolMainInfo.instances; // [anyComponent, secondInstance]；对实例符号的引用
symbolMainInfo.relatives; // [anyComponent, symbolMain]；相关符号
```

### 覆盖 (Overrides)

当您更新符号的属性时，更改会传播到所有相关的符号。为避免传播特定属性，您可以在组件级别指定要跳过的属性：

```js
anyComponent.set('my-property', true);
secondInstance.get('my-property'); // true；更改已传播

anyComponent.setSymbolOverride(['my-property']);
// 获取当前覆盖值：anyComponent.getSymbolOverride();

anyComponent.set('my-property', false);
secondInstance.get('my-property'); // true；更改未传播
```

### 分离符号

一旦您有了符号实例，您可能需要断开其中一个以创建包含其他组件的新自定义形状，在这种情况下，您可以使用 `detachSymbol`。

```js
editor.Components.detachSymbol(anyComponent);

const info = editor.Components.getSymbolInfo(anyComponent);
info.isSymbol; // false；不再是符号

const infoMain = editor.Components.getSymbolInfo(symbolMain);
infoMain.instances; // [secondInstance]；移除了引用
```

### 移除符号

要移除主符号并分离所有相关的实例：

```js
const symbolMain = editor.Components.getSymbols()[0];
symbolMain.remove();
```

## 事件 (Events)

编辑器会触发几个与符号相关的事件，您可以利用这些事件进行集成：

- `symbol:main:add` 添加了新的根主符号。

```js
editor.on('symbol:main:add', ({ component }) => { ... });
```

- `symbol:main:update` 根主符号已更新。

```js
editor.on('symbol:main:update', ({ component }) => { ... });
```

- `symbol:main:remove` 根主符号已移除。

```js
editor.on('symbol:main:remove', ({ component }) => { ... });
```

- `symbol:main` 与根主符号更新相关的捕获所有事件。

```js
editor.on('symbol:main', ({ event, component }) => { ... });
```

- `symbol:instance:add` 添加了新的根实例符号。

```js
editor.on('symbol:instance:add', ({ component }) => { ... });
```

- `symbol:instance:remove` 根实例符号已移除。

```js
editor.on('symbol:instance:remove', ({ component }) => { ... });
```

- `symbol:instance` 与根实例符号更新相关的捕获所有事件。

```js
editor.on('symbol:instance', ({ event, component }) => { ... });
```

- `symbol` 任何符号更新（主符号或实例）的捕获所有事件。

```js
editor.on('symbol', () => { ... });
```

## 示例

以下是一个利用符号 API 的基本 UI 实现：

<demo-viewer value="ta19s6go" height="500" darkcode show/>

<!-- 演示模板，此处供参考
<style>
.app-wrapper {
  height: 100vh;
  display: flex;
  flex-direction: column;
}
.vue-app {
  padding: 10px;
  display: flex;
  gap: 10px;
}
.symbols-wrp {
  display: flex;
  gap: 10px;
  width: 100%;
  padding: 10px;
  flex-direction: column;
  border-radius: 3px;
}
.symbols {
  display: flex;
  gap: 10px;
  width: 100%;
}
.symbol {
  cursor: pointer;
  flex-basis: 100px;
  text-align: left;
  margin: 0;
}
</style>

<div class="app-wrapper">
  <div class="vue-app">
    <button @click="createSymbol">Create Symbol</button>
    <div class="symbols-wrp gjs-one-bg gjs-two-color">
      <div v-if="symbols.length">Click on symbol to append</div>
      <div class="symbols">
        <div
          v-for="symbol in symbols"
          class="gjs-block symbol"
          @click="createInstance(symbol)"
          :key="symbol.getId()"
        >
          Name: {{ symbol.getName() }}
          Instances: {{ getInstancesLength(symbol) }}
        </div>
      </div>
    </div>
  </div>
  <div id="gjs"></div>
</div>

<script>
const editor = grapesjs.init({
  container: '#gjs',
  height: '100%',
  storageManager: false,
  components: `<div style="display: flex">
    <article class="card" style="max-width: 300px; padding: 20px">
      <img src="https://placehold.co/600x400/000000/FFF" style="max-width: 100%"/>
      <h1>Title</h1>
      <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua</p>
    </article>
  </div>`,
  plugins: ['gjs-blocks-basic'],
  selectorManager: { componentFirst: true },
});

const { Components } = editor;

const app = new Vue({
  el: '.vue-app',
  data: { symbols: [] },
  mounted() {
		editor.on('symbol', this.updateMainSymbolsList);
  },
  destroyed() {
    editor.off('symbol', this.updateMainSymbolsList);
  },
  methods: {
    updateMainSymbolsList() {
      this.symbols = Components.getSymbols();
    },
    createSymbol() {
      const selected = editor.getSelected();
      if (!selected) return alert('Select a component first!');

      const info = Components.getSymbolInfo(selected);
      if (info.isSymbol) return alert('Selected component is already a symbol!');

      Components.addSymbol(selected);
    },
    getInstancesLength(symbolMain) {
      return Components.getSymbolInfo(symbolMain).instances.length;
    },
    createInstance(symbolMain) {
      const instance = Components.addSymbol(symbolMain);
      editor.getWrapper().append(instance, { at: 0 });
    }
  }
});
</script>
-->

[Component]: /modules/Components.html
[Components]: /modules/Components.html
[Components API]: /api/component.html