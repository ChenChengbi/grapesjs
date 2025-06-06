---
title: 特性管理器
---

# 特性管理器

在 GrapesJS 中，特性(Trait)定义了组件(Component)的不同参数和行为。用户通常会将特性视为组件的_设置_。特性常用于自定义元素属性(Attribute)（例如 `<input>` 的 `placeholder`），或者你也可以将它们绑定到组件的属性(Property)并对其变化做出反应。

::: warning
本指南适用于 GrapesJS v0.21.9 或更高版本。<br><br>
为了更好地理解本指南的内容，我们建议先阅读[组件](Components.html)。
:::

::: tip
想要开箱即用且外观出色的特性吗？[了解 Grapes Studio SDK 如何处理它。](https://app.grapesjs.com/docs-sdk/configuration/components/properties?utm_source=grapesjs-docs&utm_medium=tip#traits)
:::

[[toc]]

## 向组件添加特性

通常，你可以在新的自定义组件的定义中（或通过扩展另一个组件）定义特性。让我们在这个例子中看看如何使输入框能被编辑器更好地自定义。

默认情况下，所有组件都包含两个特性：`id` 和 `title`（在撰写本文时）。因此，如果你选择一个输入框并打开“设置”<sup>Settings</sup>面板，你将看到以下内容：

<img :src="$withBase('/default-traits.png')" class="img-ctr-rad" style="max-width: 200px;" alt="Default traits">

我们可以通过这种方式创建一个新的自定义 `input` 组件：

```js
editor.Components.addType('input', {
  isComponent: (el) => el.tagName === 'INPUT',
  model: {
    defaults: {
      traits: [
        // 字符串会自动转换成文本类型
        'name', // 等同于：{ type: 'text', name: 'name' }
        'placeholder',
        {
          type: 'select', // 特性的类型
          name: 'type', // (必需) 用于组件属性/特性的名称
          label: 'Type', // 你将在“设置”中看到的标签
          options: [
            { id: 'text', label: 'Text' },
            { id: 'email', label: 'Email' },
            { id: 'password', label: 'Password' },
            { id: 'number', label: 'Number' },
          ],
        },
        {
          type: 'checkbox',
          name: 'required',
        },
      ],
      // 由于特性默认绑定到 HTML 属性，因此要定义
      // 它们的初始值，我们可以使用 HTML 属性
      attributes: { type: 'text', required: true },
    },
  },
});
```

现在的结果将是

<img :src="$withBase('/input-custom-traits.png')" class="img-ctr-rad" style="max-width: 230px;" alt="Input with custom traits">

如果你愿意，也可以通过函数动态定义特性，这些函数将在组件初始化时创建。如果你需要根据组件的其他特性来创建特性，这可能会很有用。

```js
editor.Components.addType('input', {
  isComponent: (el) => el.tagName === 'INPUT',
  model: {
    defaults: {
      traits(component) {
        const result = [];

        // 一些逻辑示例
        if (component.get('draggable')) {
          result.push('name');
        } else {
          result.push({
            type: 'select',
            // ....
          });
        }

        return result;
      },
    },
  },
});
```

如果你需要对特性的某些变化做出反应，可以订阅其属性(attribute)侦听器

```js
editor.Components.addType('input', {
  model: {
    defaults: {
      // ...
    },

    init() {
      this.on('change:attributes:type', this.handleTypeChange);
    },

    handleTypeChange() {
      console.log('Input type changed to: ', this.getAttributes().type);
    },
  },
});
```

如前所述，默认情况下，特性修改的是模型的 HTML 属性(attributes)，但你也可以使用 `changeProp` 选项将它们绑定到组件的属性(properties)。

```js
editor.Components.addType('input', {
  model: {
    defaults: {
      // ...
      traits: [
        {
          name: 'placeholder',
          changeProp: 1,
        },
        // ...
      ],
      // 由于我们从 HTML 属性切换到了组件属性，
      // 初始值应从组件属性设置
      placeholder: 'Initial placeholder',
    },

    init() {
      // 侦听器也从 `change:attributes:*` 变为 `change:*`
      this.on('change:placeholder', this.handlePlhChange);
    },
    // ...
  },
});
```

### 分类

可以将你的特性分组到类别中，如下所示。

<img :src="$withBase('/trait-categories.png')" class="img-ctr-rad" style="max-width: 250px;" alt="Traits with categories">

```js
const category1 = { id: 'first', label: 'First category' };
const category2 = { id: 'second', label: 'Second category', open: false };

editor.Components.addType('input', {
  model: {
    defaults: {
      // ...
      traits: [
        { name: 'trait-1', category: category1 },
        { name: 'trait-2', category: category1 },
        { name: 'trait-3', category: category2 },
        { name: 'trait-4', category: category2 },
        // 没有分类的特性将呈现在底部
        { name: 'trait-5' },
        { name: 'trait-6' },
      ],
    },
  },
});
```

## 内置特性类型

GrapesJS 提供了一些内置类型，你可以用它们来定义特性：

### 文本 (Text)

简单文本输入框

```js
{
  type: 'text', // 如果不指定类型，`text` 是默认类型
  name: 'my-trait', // 必需，所有特性都可用
  label: 'My trait', // 你将在输入框旁边看到的标签
  // label: false, // 如果将标签设置为 `false`，标签列将被移除
  placeholder: 'Insert text', // 在输入框内显示的占位符
}
```

### 数字 (Number)

数字输入框

```js
{
  type: 'number',
  // ...
  placeholder: '0-100',
  min: 0, // 最小数值
  max: 100, // 最大数值
  step: 5, // 步长
}
```

### 复选框 (Checkbox)

简单复选框

```js
{
  type: 'checkbox',
  // ...
  valueTrue: 'YES', // 选中时分配的值，默认为：`true`
  valueFalse: 'NO', // 未选中时分配的值，默认为：`false`
}
```

### 下拉选择 (Select)

带选项的下拉选择框

```js
{
  type: 'select',
  // ...
  options: [ // 选项数组
    { id: 'opt1', label: 'Option 1'},
    { id: 'opt2', label: 'Option 2'},
  ]
}
```

### 颜色 (Color)

颜色选择器

```js
{
  type: 'color',
  // ...
}
```

### 按钮 (Button)

可分配命令的按钮

```js
{
  type: 'button',
  // ...
  text: 'Click me',
  full: true, // 全宽按钮
  command: editor => alert('Hello'),
  // 或者你可以直接指定命令 ID
  command: 'some-command',

}
```

## 运行时更新特性

如果你需要在组件上更改某些特性，可以使用[组件 API](/api/component.html)在任何需要的地方更新它。

特性是组件的一个简单属性(property)，因此要获取当前特性的完整列表，可以使用：

```js
const component = editor.getSelected(); // 画布中选中的组件
const traits = component.get('traits');
traits.forEach((trait) => console.log(trait.props()));
```

如果你只需要一个：

```js
const component = editor.getSelected();
console.log(component.getTrait('type').props()); // 通过特性的 `name` 查找
```

例如，如果你想更新特性的某个属性，请执行以下操作：

```js
// 让我们更新 Input 组件中定义的 `type` 特性的 `options`
const component = editor.getSelected();
component.getTrait('type').set('options', [
  { id: 'opt1', label: 'New option 1'},
  { id: 'opt2', label: 'New option 2'},
]);
// 或者使用多个值
component.getTrait('type').set({
  label: 'My type',
  options: [...],
});
```

你还可以使用 [`addTrait`](/api/component.html#addtrait)/[`removeTrait`](/api/component.html#removetrait) 轻松添加新特性或删除其他特性。

```js
// 添加新特性
const component = editor.getSelected();
component.addTrait({
  name: 'type',
  ...
}, { at: 0 });
// `at` 选项指示放置新特性的索引，
// 如果没有它，特性将被追加到列表末尾

// 移除特性
component.removeTrait('type');
```

## 国际化(I18n)

要利用[国际化(I18n)模块](I18n.html)，你可以参考此结构：

```js
{
  en: {
    traitManager: {
      empty: 'Select an element before using Trait Manager',
      label: 'Component settings',
      categories: {
        categoryId: 'Category label',
      },
      traits: {
        // 特性的 `name` 属性用作键
        labels: {
          href: 'Href label',
        },
        // 对于像 `text` 这样的内置类型，这些用于输入框的 DOM 属性
        attributes: {
          href: { placeholder: 'eg. https://google.com' },
        },
        // 对于 `select` 类型，这些用于翻译选项标签
        options: {
          target: {
            // 这里的键是选项的 `id`
            _blank: 'New window',
          },
        },
      },
    },
  }
}
```

## 自定义

默认类型应能覆盖大多数常见属性，但如果你需要更高级的用户界面(UI)，可以[定义自己的类型](#define-new-trait-type)，甚至从头开始创建一个完全[自定义的特性管理器界面](#custom-trait-manager)。

### 定义新的特性类型

在大多数情况下，默认类型应该足够了，但有时你可能需要更多功能。
在这种情况下，你可以定义一种新的特性类型，并将任何类型的元素绑定到它。

#### 创建元素

让我们用一种新的特性来更新默认的 `link` 组件。这是简单链接的默认特性情况。

<img :src="$withBase('/default-link-comp.jpg')" class="img-ctr-rad" style="max-width: 210px;" alt="Default link component">

我们将其所有特性替换为一个新的特性 `href-next`，它将允许用户选择 href 的类型（例如 'url'、'email' 等）。

```js
// 更新组件
editor.Components.addType('link', {
  model: {
    defaults: {
      traits: [
        {
          type: 'href-next',
          name: 'href',
          label: 'New href',
        },
      ],
    },
  },
});
```

现在你会看到一个简单的文本输入框，因为我们还没有定义新的特性类型，现在我们来定义它：

```js
editor.Traits.addType('href-next', {
  // 期望返回一个简单的 HTML 字符串或 HTML 元素
  createInput({ trait }) {
    // 这里我们可以决定使用特性中的属性
    const traitOpts = trait.get('options') || [];
    const options = traitOpts.length
      ? traitOpts
      : [
          { id: 'url', label: 'URL' },
          { id: 'email', label: 'Email' },
        ];

    // 创建一个新的元素容器并添加一些内容
    const el = document.createElement('div');
    el.innerHTML = `
      <select class="href-next__type">
        ${options.map((opt) => `<option value="${opt.id}">${opt.label}</option>`).join('')}
      </select>
      <div class="href-next__url-inputs">
        <input class="href-next__url" placeholder="Insert URL"/>
      </div>
      <div class="href-next__email-inputs">
        <input class="href-next__email" placeholder="Insert email"/>
        <input class="href-next__email-subject" placeholder="Insert subject"/>
      </div>
    `;

    // 让我们的内容具有交互性
    const inputsUrl = el.querySelector('.href-next__url-inputs');
    const inputsEmail = el.querySelector('.href-next__email-inputs');
    const inputType = el.querySelector('.href-next__type');
    inputType.addEventListener('change', (ev) => {
      switch (ev.target.value) {
        case 'url':
          inputsUrl.style.display = '';
          inputsEmail.style.display = 'none';
          break;
        case 'email':
          inputsUrl.style.display = 'none';
          inputsEmail.style.display = '';
          break;
      }
    });

    return el;
  },
});
```

在上面的例子中，我们简单地创建了自定义输入（同时也提供了使用 `option` 特性属性的可能性），并定义了一些在类型更改时的输入切换行为。现在结果会是这样：

<img :src="$withBase('/docs-init-link-trait.jpg')" class="img-ctr-rad" style="max-width: 215px;" alt="Link with traits">

#### 更新布局

在继续并使我们的特性工作之前，让我们讨论一下特性的布局结构。你可能已经注意到，特性由标签列和输入列组成，因此，GrapesJS 允许你自定义它们两者。

对于标签自定义，你可以使用 `createLabel`

```js
editor.Traits.addType('href-next', {
  // 期望返回一个简单的 HTML 字符串或 HTML 元素
  createLabel({ label }) {
    return `<div>
      <div>Before</div>
      ${label}
      <div>After</div>
    </div>`;
  },
  // ...
});
```

你可能已经看到，在特性定义中，你可以设置 `label: false` 来完全移除标签列，但如果你需要在此特性类型的所有实例中强制此行为，可以使用 `noLabel` 属性

```js
editor.Traits.addType('href-next', {
  noLabel: true,
  // ...
});
```

你可能还会注意到，默认情况下 GrapesJS 会在你的输入周围应用一种包装器，这对于简单的输入通常是可以的，但当你创建复杂的自定义特性时，这可能不是你所需要的。要移除默认包装器，你可以使用 `templateInput` 选项

```js
editor.Traits.addType('href-next', {
  // 完全移除包装器
  templateInput: '',
  // 使用一个新的，通过 `data-input` 属性指定放置输入容器的位置
  templateInput: `<div class="custom-input-wrapper">
    Before input
    <div data-input></div>
    After input
  </div>`,
  // 它也可以是一个函数，期望返回一个 HTML 字符串
  templateInput({ trait }) {
    return '<div ...';
  },
});
```

<img :src="$withBase('/docs-link-trait-raw.jpg')" class="img-ctr-rad" style="max-width: 215px;" alt="Basic custom link trait">

在这种情况下，结果会相当原始且没有样式，但自定义特性类型的意义在于允许你重用自己设计的、可能已经定义好（或在某个 UI 框架中实现）的样式化输入。
现在，我们保留默认的输入包装器，并继续集成我们的自定义特性。

#### 绑定到组件

在当前状态下，我们在 `createInput` 中创建的元素尚未绑定到组件，因此更新输入时不会发生任何事情，现在我们来处理它

```js
editor.Traits.addType('href-next', {
  // ...

  // 根据元素更改更新组件
  // `elInput` 是从 `createInput` 获取的 HTMLElement 结果
  onEvent({ elInput, component, event }) {
    const inputType = elInput.querySelector('.href-next__type');
    let href = '';

    switch (inputType.value) {
      case 'url':
        const valUrl = elInput.querySelector('.href-next__url').value;
        href = valUrl;
        break;
      case 'email':
        const valEmail = elInput.querySelector('.href-next__email').value;
        const valSubj = elInput.querySelector('.href-next__email-subject').value;
        href = `mailto:${valEmail}${valSubj ? `?subject=${valSubj}` : ''}`;
        break;
    }

    component.addAttributes({ href });
  },
});
```

现在，大部分功能应该已经可以工作了（你可以更新特性并在代码预览中检查 HTML）。你可能想知道编辑器如何捕获输入更改以及如何控制它。
默认情况下，基础特性包装器会在 `change` 事件上应用一个侦听器，并在任何捕获到的事件上调用 `onEvent`（要捕获事件，该事件需要能够[冒泡](https://stackoverflow.com/questions/4616694/what-is-event-bubbling-and-capturing)）。如果你想，例如，在 `input` 事件上更新组件，可以更改 `eventCapture` 属性

```js
editor.Traits.addType('href-next', {
  eventCapture: ['input'], // 你可以在数组中使用多个事件
  // ...
});
```

最后，你可能已经注意到我们的特性初始渲染不正确，在已定义 `href` 属性的情况下输入未被填充。这一步应该在 `onUpdate` 方法中完成

```js
editor.Traits.addType('href-next', {
  // ...

  // 组件更改时更新元素
  onUpdate({ elInput, component }) {
    const href = component.getAttributes().href || '';
    const inputType = elInput.querySelector('.href-next__type');
    let type = 'url';

    if (href.indexOf('mailto:') === 0) {
      const inputEmail = elInput.querySelector('.href-next__email');
      const inputSubject = elInput.querySelector('.href-next__email-subject');
      const mailTo = href.replace('mailto:', '').split('?');
      const email = mailTo[0];
      const params = (mailTo[1] || '').split('&').reduce((acc, item) => {
        const items = item.split('=');
        acc[items[0]] = items[1];
        return acc;
      }, {});
      type = 'email';

      inputEmail.value = email || '';
      inputSubject.value = params.subject || '';
    } else {
      elInput.querySelector('.href-next__url').value = href;
    }

    inputType.value = type;
    inputType.dispatchEvent(new CustomEvent('change'));
  },
});
```

现在，即使组件发生如下更改，特性也会更新：

```js
editor.getSelected().addAttributes({ href: 'mailto:new-email@test.com?subject=NewSubject' });
```

总结一下我们到目前为止所做的，要创建一个自定义特性类型，你将需要以下 3 个方法：

- `createInput` - 我们在这里定义自定义 HTML 元素
- `onEvent` - 输入更改时如何更新组件
- `onUpdate` - 组件更改时如何更新输入

#### 结果

我们所做工作的最终结果可以在这里看到
<demo-viewer value="yf6amdqb/10"/>

#### 集成外部 UI 组件

看上面的例子可能觉得代码很多，但归根结底，它只涉及一些逻辑和原生的 DOM API，后者并不那么美观。如果你使用现代 UI 客户端框架（例如 Vue、React 等），你会发现集成更加容易。下面是如何将自定义的 [Vue Slider Component](https://github.com/NightCatSama/vue-slider-component) 集成为一个特性：

```js
editor.Traits.addType('slider', {
  createInput({ trait }) {
    const vueInst = new Vue({ render: (h) => h(VueSlider) }).$mount();
    const sliderInst = vueInst.$children[0];
    sliderInst.$on('change', (ev) => this.onChange(ev)); // 使用 onChange 触发 onEvent
    this.sliderInst = sliderInst;
    return vueInst.$el;
  },

  onEvent({ component }) {
    const value = this.sliderInst.getValue() || 0;
    component.addAttributes({ value });
  },

  onUpdate({ component }) {
    const value = component.getAttributes().value || 0;
    this.sliderInst.setValue(value);
  },
});
```

<demo-viewer value="x9sw2udv"/>

通过遵循以下这些简单的核心要点，可以实现与外部组件的集成：

1.  **组件渲染(Component rendering)**：`new Vue({ render: ...`<br/>
    取决于框架，例如，在 React 中，它应该是 `ReactDOM.render(element, ...`
2.  **变更传播(Change propagation)**：`sliderInst.$on('change', ev => this.onChange(ev))`<br/>
    框架应该有一个订阅变更的机制，并且组件[应该暴露该变更](https://nightcatsama.github.io/vue-slider-component/#/api/events)<br/>
    我们还使用了 `onChange` 方法，当需要手动触发 `onEvent` 事件时它非常方便（你不应该直接调用 `onEvent` 方法，而只应在需要时通过 `onChange` 调用）
3.  **属性(Property)获取器/设置器(getters/setters)**：[`sliderInst.getValue()`](https://nightcatsama.github.io/vue-slider-component/#/api/methods?hash=getvalue)/ [`sliderInst.setValue(value)`](https://nightcatsama.github.io/vue-slider-component/#/api/methods?hash=setvaluevalue)<br/>
    组件应允许从实例中读取和写入数据

### 自定义特性管理器

默认的特性管理器用户界面(UI)应能处理大多数常见任务，但如果你需要更高级的逻辑/元素，可以从头开始创建自定义的界面。

你所要做的就是向编辑器指明你打算使用自定义 UI，然后订阅 `trait:custom` 事件，该事件将在 UI 需要任何更新时触发。

```js
const editor = grapesjs.init({
  // ...
  traitManager: {
    custom: true,
    // ...
  },
});

editor.on('trait:custom', (props) => {
  // props.container (HTMLElement) - 你可以在其中附加自定义 UI 的默认元素
  // 你可以在这里放置渲染/更新 UI 的逻辑。
});
```

在下面的示例中，我们将使用[特性 API]复制大部分默认功能。

<demo-viewer value="hszpw2rb" height="500" darkcode/>

<!--
<style>
  .trait-input-color {
    width: 16px !important;
    height: 15px !important;
    opacity: 0 !important;
  }
  /* Vuetify overrides */
  .v-application {
    background: transparent !important;
  }
  .v-application--wrap {
    min-height: auto;
  }
  .v-input__slot {
    font-size: 12px;
    min-height: 10px !important;
    color-scheme: dark;
  }
  .v-select__selections {
    flex-wrap: nowrap;
  }
  .v-text-field .v-input__slot {
    padding: 0 10px !important;
  }
  .v-input--selection-controls {
    margin-top: 0;
  }
  .v-text-field__details, .v-messages {
    display: none;
  }
  .no-cat-header {
    opacity: 0;
    padding: 10px;
    max-height: 10px;
    pointer-events: none;
    min-height: auto !important;
  }
  .trait-color-prv {
    border: 1px solid rgba(255,255,255,0.5);
    border-radius: 3px;
  }
</style>
<div>
  <div class="vue-app">
    <v-app>
      <v-main>
        <v-expansion-panels accordion multiple v-model="panels">
          <v-expansion-panel v-for="trc in traitCategories">
            <v-expansion-panel-header v-if="trc.category">
              {{ trc.category.getLabel() }}
            </v-expansion-panel-header>
            <v-expansion-panel-header class="no-cat-header" v-else></v-expansion-panel-header>
            <v-expansion-panel-content>
              <v-row>
                <trait-field v-for="trait in trc.items" :key="trait.id" :trait="trait"/>
              </v-row>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-main>
    </v-app>
  </div>

  <div id="trait-field" style="display: none;">
    <v-col :class="['py-0 px-1 mb-1', trait.get('full') && 'mb-3']" :cols="12">
      <v-row class="flex-nowrap" v-if="isTypeNeedLabel">
        <v-col cols="auto pr-0">{{ trait.getLabel() }}</v-col>
      </v-row>
      <div v-if="type === 'number'">
        <v-text-field :placeholder="placeholder" :value="inputValue" @change="handleChange" outlined dense/>
      </div>
      <div v-else-if="type === 'checkbox'">
        <v-checkbox :label="trait.getLabel()" :input-value="inputValue" @change="handleChange"></v-checkbox>
      </div>
      <div v-else-if="type === 'select'">
        <v-select :items="toOptions" :value="inputValue" @change="handleChange" outlined dense/>
      </div>
      <div v-else-if="type === 'color'">
        <v-text-field :placeholder="placeholder" :value="inputValue" @change="handleChange" outlined dense>
          <template v-slot:append>
            <div :style="{ backgroundColor: inputValue || placeholder }" class="trait-color-prv">
              <input class="trait-input-color"
                      type="color"
                      :value="inputValue || placeholder"
                      @change="(ev) => handleChange(ev.target.value)"
                      @input="(ev) => handleInput(ev.target.value)"
                      />
            </div>
          </template>
        </v-text-field>
      </div>
      <div v-else-if="type === 'button'">
        <v-btn block @click="trait.runCommand()">
          <v-row>
            <v-col>{{ trait.get('labelButton') }}</v-col>
          </v-row>
        </v-btn>
      </div>
      <div v-else>
        <v-text-field :placeholder="placeholder" :type="type" :value="inputValue" @change="handleChange" outlined dense/>
      </div>
    </v-col>
  </div>
</div>
<script>
  const myPlugin = (editor) => {
    const category1 = { id: 'first', label: 'First category' };
    const category2 = { id: 'second', label: 'Second category', open: false };

    editor.Components.addType('demo-cmp', {
      model: {
        defaults: {
          traits: [
            {
              type: 'date',
              name: 'date-trait',
              label: 'Date trait',
              placeholder: 'Insert date',
            },
            {
              type: 'text',
              name: 'text-trait',
              label: false,
              placeholder: 'Insert text',
              category: category1,
            },
            {
              type: 'select',
              name: 'select-trait',
              label: 'Select 1',
              category: category1,
              default: 'opt1',
              options: [
                { id: 'opt1', name: 'Option 1'},
                { id: 'opt2', name: 'Option 2'},
                { id: 'opt3', name: 'Option 3'},
              ]
            },
            {
              type: 'select',
              name: 'select-trait2',
              label: 'Select 2',
              category: category1,
              default: 'opt2',
              options: [
                { id: 'opt1', name: 'Option 1'},
                { id: 'opt2', name: 'Option 2'},
                { id: 'opt3', name: 'Option 3'},
              ]
            },
            {
              type: 'number',
              name: 'number-trait',
              placeholder: '0-100',
              min: 0,
              max: 100,
              step: 5,
            },
            {
              type: 'color',
              label: 'Color trait',
              name: 'color-trait',
            },
            {
              type: 'checkbox',
              label: 'Checkbox trait',
              name: 'checkbox-trait',
              valueTrue: 'YES',
              valueFalse: 'NO',
            },
            {
              type: 'checkbox',
              name: 'open',
              category: category2,
            },
            {
              type: 'button',
              label: 'Button trait',
              labelButton: 'Alert',
              name: 'button-trait',
              category: category2,
              full: true,
              command: () => alert('hello'),
            },
            {
              type: 'button',
              label: false,
              full: true,
              category: category2,
              labelButton: 'Open code',
              name: 'button-trait2',
              command: 'core:open-code',
            },
          ],
        },
      },
    });
  };

  const editor = grapesjs.init({
    container: '#gjs',
    height: '100%',
    storageManager: false,
    fromElement: true,
    plugins: ['gjs-blocks-basic', myPlugin],
    traitManager: {
      custom: true
    }
  });

  Vue.component('trait-field', {
    props: { trait: Object },
    template: '#trait-field',
    computed: {
      inputValue() {
        return this.trait.getValue({ useType: true });
      },
      type() {
        return this.trait.getType();
      },
      placeholder() {
        return this.trait.get('placeholder');
      },
      toOptions() {
        const { trait } = this;
        return trait.getOptions().map(o => ({ value: trait.getOptionId(o), text: trait.getOptionLabel(o) }))
      },
      isTypeNeedLabel() {
        return !['checkbox', 'button'].includes(this.type);
      }
    },
    methods: {
      handleChange(value) {
        this.trait.setValue(value);
      },
      handleInput(value) {
        this.trait.setValue(value, { partial: true });
      },
    }
  });

  const app = new Vue({
    el: '.vue-app',
    vuetify: new Vuetify({
      theme: { dark: true },
    }),
    data: {
      traitCategories: [],
      panels: [],
    },
    mounted() {
      const { Traits } = editor;
      // Catch-all event for any spot update
      editor.on('trait:custom', this.onTraitCustom);
    },
    destroyed() {
      editor.off('trait:custom', this.handleCustom);
    },
    methods: {
      onTraitCustom(props) {
        const { container } = props;
        if (container && !container.contains(this.$el)) {
          container.appendChild(this.$el);
        }
        const traitsByCategory = editor.Traits.getTraitsByCategory();
        const noCategoryIndex = traitsByCategory.findIndex(trc => !trc.category) || 0;
        // Keep items without categories open
        if (!this.panels.includes(noCategoryIndex)) {
          this.panels.push(noCategoryIndex);
        }
        this.traitCategories = traitsByCategory;
      },
      categoryId(traitCategory) {
        return traitCategory.category?.id || 'none';
      },
    }
  });
</script>
-->

## 事件

有关可用事件的完整列表，你可以在[此处](/api/trait_manager.html#available-events)查看。

[特性 API]: /api/trait_manager.html