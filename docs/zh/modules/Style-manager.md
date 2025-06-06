---
title: 样式管理器
---

# 样式管理器

<p align="center"><img :src="$withBase('/style-manager.jpg')" alt="GrapesJS - 样式管理器"/></p>

样式管理器(Style Manager)模块负责显示和更新与您的[组件(Components)]相关的样式属性。在本指南中，您将了解如何在GrapesJS中设置并充分利用内置的样式管理器用户界面(Style Manager UI)。默认UI是一个包含内置属性的轻量级组件，但正如您将在本指南后续内容中看到的，通过使用[样式管理器 API(Style Manager API)]，您可以轻松地使用自己的元素来扩展它，甚至可以从头开始创建样式管理器UI。

::: warning
为了更好地理解本指南的内容，我们建议您首先阅读[组件(Components)]
:::
::: warning
本指南适用于 GrapesJS v0.18.1 或更高版本
:::

::: tip
正在寻找一个易于定制且即刻可用的UI？[查看 Grapes Studio SDK！](https://app.grapesjs.com/docs-sdk/configuration/components/overview?utm_source=grapesjs-docs&utm_medium=tip)
:::

[[toc]]

## 配置

要更改默认配置，您需要将 `styleManager` 属性与主配置对象一起传递。

```js
const editor = grapesjs.init({
  ...
  styleManager: {
    sectors: [...],
    ...
  }
});
```

在此查看可用选项的完整列表：[样式管理器配置(Style Manager Config)](https://github.com/GrapesJS/grapesjs/blob/master/src/style_manager/config/config.ts)

## 初始化

样式管理器模块组织成多个区块(sectors)，每个区块包含要显示的属性(properties)列表。默认的样式管理器配置已经包含了一系列默认的常用样式属性，您可以通过跳过 `styleManagerConfig.sectors` 选项来使用它们。

```js
grapesjs.init({
  ...
  styleManager: {
    // 未定义区块时，将加载默认列表
    // sectors: [...],
    ...
  },
});
```

::: danger
只有在至少选择了一个组件时，显示样式管理器UI才有意义，因此默认情况下，如果没有选中的组件，样式管理器是隐藏的。
:::

### 区块定义

每个区块由其 `name` 和要显示的 `properties` 列表来标识。您还可以指定 `id` 以便通过API访问区块（如果未指定，则会从 `name` 生成），以及默认的 `open` 状态。

```js
grapesjs.init({
  // ...
  styleManager: {
    sectors: [
      {
        name: '第一个区块',
        properties: [],
        // id: 'first-sector', // Id 从名称生成
        // open: true, // 区块默认打开
      },
      {
        open: false, // 默认渲染为关闭状态
        name: '第二个区块',
        properties: [],
      },
    ],
  },
});
```

### 属性定义

定义好区块后，您就可以开始在 `properties` 中添加属性定义了。每个属性都有一组通用选项（`label`、`default` 等）以及由其 `type` 指定的其他特定选项。

下面我们来看一个控制内边距(padding)样式的简单定义。

```js
sectors: [
  {
    name: '第一个区块',
    properties: [
      {
        // 默认选项
        // id: 'padding', // 属性的id，如果缺失，将与 `property` 值相同
        type: 'number',
        label: '内边距', // 属性的标签
        property: 'padding', // 要更改的CSS属性
        default: '0', // 显示的默认值
        // 额外的 `number` 选项
        units: ['px', '%'], // 单位 (仅适用于 'number' 类型)
        min: 0, // 最小值 (仅适用于 'number' 类型)
      },
    ],
  },
];
```

这将渲染数字输入(number input)用户界面，并更改所选组件的 `padding` CSS属性。

定义的灵活性使您可以为任何可能的CSS属性轻松创建不同的UI输入。您可以自由决定哪种UI最适合您的用户。例如，如果您采用像 `font-size` 这样的数字属性，您可以遵循其CSS规范并将其定义为 `number` 类型。

```js
{
  type: 'number',
  label: '字体大小',
  property: 'font-size',
  units: ['px', '%', 'em', 'rem', 'vh', 'vw'],
  min: 0,
}
```

或者您可以决定将其显示为 `select` 类型，并仅提供一组定义好的值（例如，基于您的设计系统令牌(Design System tokens)）。

```js
{
  type: 'select',
  label: '字体大小',
  property: 'font-size',
  default: '1rem',
  options: [
    { id: '0.7rem', label: '小' },
    { id: '1rem', label: '中' },
    { id: '1.2rem', label: '大' },
  ]
}
```

每种类型定义了特定的UI视图和处理输入及更新的模型(model)。
下面我们来看看所有可用的默认类型及其相关的UI和模型。

#### 默认类型

::: tip
每个**模型(Model)**更详细地描述了可用的属性及其用法。
:::

- `base` - 基础类型，渲染为一个简单的文本输入字段。**模型(Model)**: [属性(Property)](/api/property.html)

  <img :src="$withBase('/sm-base-type.jpg')"/>

  ```js
  // 示例
  {
    // type: 'base', // 定义中省略类型将默认为 'base'
    property: 'some-css-property',
    label: '基础类型',
    default: '默认值',
  },
  ```

- `color` - 与 `base` 相同的属性，但UI是一个颜色选择器。**模型(Model)**: [属性(Property)](/api/property.html)

  <img :src="$withBase('/sm-type-color.jpg')"/>

- `number` - 用于数值的数字输入字段。**模型(Model)**: [数字属性(PropertyNumber)](/api/property_number.html)

  <img :src="$withBase('/sm-type-number.jpg')"/>

  ```js
  // 示例
  {
    type: 'number',
    property: 'width',
    label: '数字类型',
    default: '0%',
    // 额外属性
    units: ['px', '%'],
    min: 0,
    max: 100,
  },
  ```

- `slider` - 与 `number` 相同的属性，但UI是一个滑块。**模型(Model)**: [数字属性(PropertyNumber)](/api/property_number.html)

  <img :src="$withBase('/sm-type-slider.jpg')"/>

- `select` - 带选项的选择输入。**模型(Model)**: [选择属性(PropertySelect)](/api/property_select.html)

  <img :src="$withBase('/sm-type-select.jpg')"/>

  ```js
  // 示例
  {
    type: 'select',
    property: 'display',
    label: '选择类型',
    default: 'block',
    // 额外属性
    options: [
      {id: 'block', label: '块级'},
      {id: 'inline', label: '行内'},
      {id: 'none', label: '无'},
    ]
  },
  ```

- `radio` - 与 `select` 相同的属性，但UI是单选按钮。**模型(Model)**: [选择属性(PropertySelect)](/api/property_select.html)

  <img :src="$withBase('/sm-type-radio.jpg')"/>

- `composite` - 此类型非常适合CSS简写属性，其中最终值是多个子属性的组合。**模型(Model)**: [复合属性(PropertyComposite)](/api/property_composite.html)

  <img :src="$withBase('/sm-type-composite.jpg')"/>

  ```js
  // 示例
  {
    type: 'composite',
    property: 'margin',
    label: '复合类型',
    // 额外属性
    properties: [
      { type: 'number', units: ['px'], default: '0', property: 'margin-top' },
      { type: 'number', units: ['px'], default: '0', property: 'margin-right' },
      { type: 'number', units: ['px'], default: '0', property: 'margin-bottom' },
      { type: 'number', units: ['px'], default: '0', property: 'margin-left' },
    ]
  },
  ```

- `stack` - 此类型非常适合像 `text-shadow`、`box-shadow`、`transform` 等CSS多值属性。**模型(Model)**: [堆叠属性(PropertyStack)](/api/property_stack.html)

  <img :src="$withBase('/sm-type-stack.jpg')"/>

  ```js
  // 示例
  {
    type: 'stack',
    property: 'text-shadow',
    label: '堆叠类型',
    // 额外属性
    properties: [
      { type: 'number', units: ['px'], default: '0', property: 'x' },
      { type: 'number', units: ['px'], default: '0', property: 'y' },
      { type: 'number', units: ['px'], default: '0', property: 'blur' },
      { type: 'color', default: 'black', property: 'color' },
    ]
  },
  ```

#### 内置属性

为了加快样式管理器的配置，GrapesJS 内置了一组已定义的常用CSS属性，您可以重用和扩展它们。

```js
sectors: [
  {
    name: '第一个区块',
    properties: [
      // 将内置CSS属性作为字符串传递
      'width',
      'min-width',
      // 使用您的属性扩展内置属性
      {
        extend: 'max-width',
        units: ['px', '%'],
      },
      // 如果属性不存在，它将被转换为基础类型
      'unknown-property', // -> { type: 'base', property: 'unknown-property' }
    ],
  },
];
```

::: tip
您可以通过运行以下命令检查属性是否可用

```js
editor.StyleManager.getBuiltIn('property-name');
```

或使用以下命令获取所有可用属性的列表

```js
editor.StyleManager.getBuiltInAll();
```
:::

## 国际化(I18n)

如果您计划拥有一个多语言编辑器，您可以轻松地通过它们的ID将区块和属性标签连接到[国际化(I18n)]模块。

```js
grapesjs.init({
  styleManager: {
    sectors: [
      {
        id: 'first-sector-id',
        // 如果i18n定义缺失，您可以保留名称作为后备
        name: 'First sector',
        properties: [
          'width',
          {
            id: 'display-prop-id', // 默认情况下，id与其属性名称相同
            label: 'Display',
            type: 'select',
            property: 'display',
            default: 'block',
            options: [
              { id: 'block', label: 'Block' },
              { id: 'inline', label: 'Inline' },
              { id: 'none', label: 'None' },
            ],
          },
        ],
      },
      // ...
    ],
  },
  i18n: {
    // 使用 `messagesAdd` 来扩展默认设置
    messagesAdd: {
      en: {
        styleManager: {
          sectors: {
            'first-sector-id': 'First sector EN',
          },
          properties: {
            width: 'Width EN',
            'display-prop-id': 'Display EN',
          },
          options: {
            'display-prop-id': {
              block: 'Block EN',
              inline: 'Inline EN',
              none: 'None EN',
            },
          },
        },
      },
    },
  },
});
```

## 组件约束

当您定义自定义组件(custom components)时，您还可以通过 `stylable` 和 `unstylable` 属性指明哪些CSS属性可用于样式设置。在这种情况下，样式管理器将仅显示可用的属性。如果区块不包含任何可用属性，则不会显示该区块。

```js
const customComponents = (editor) => {
  // 组件 A
  editor.Components.addType('cmp-a', {
    model: {
      defaults: {
        // 当此组件被选中时，样式管理器将仅显示以下属性
        stylable: ['width', 'height'],
      },
    },
  });
  // 组件 B
  editor.Components.addType('cmp-b', {
    model: {
      defaults: {
        // 当此组件被选中时，样式管理器将隐藏以下属性
        unstylable: ['color'],
      },
    },
  });
};

grapesjs.init({
  // ...
  plugins: [customComponents],
  components: [
    { type: 'cmp-a', components: '组件 A' },
    { type: 'cmp-b', components: '组件 B' },
  ],
  styleManager: {
    sectors: [
      {
        name: '第一个区块',
        properties: ['width', 'min-width', 'height', 'min-height'],
      },
      {
        name: '第二个区块',
        properties: ['color', 'font-size'],
      },
    ],
  },
});
```

## 程序化用法

对于更高级的用法，您可以依赖[样式管理器 API(Style Manager API)]来执行与模块相关的不同类型的操作。

- 初始化后(post-initialization)管理区块/属性。

  ```js
  // 从编辑器实例获取模块
  const sm = editor.StyleManager;

  // 添加新区块
  const newSector = sm.addSector('sector-id', {
    name: '新区块',
    open: true,
    properties: ['width'],
  });

  // 向区块添加新属性
  sm.addProperty('sector-id', {
    type: 'number',
    property: 'min-width',
  });

  // 移除区块
  sm.removeSector('sector-id');
  ```

- 管理选定目标。

  ```js
  // 选择当前页面中的第一个按钮
  const wrapperCmp = editor.Pages.getSelected().getMainComponent();
  const btnCmp = wrapperCmp.find('button')[0];
  btnCmp && sm.select(btnCmp);

  // 将CSS选择器设置为目标（如果不存在，将创建相关的CSSRule）
  sm.select('.btn > span');
  // 获取最后选定的目标
  const lastTarget = sm.getLastSelected();
  lastTarget?.toCSS && console.log(lastTarget.toCSS());

  // 使用自定义样式更新选定目标
  sm.addStyleTargets({ color: 'red' });
  ```

- 添加/扩展内置属性定义。

  ```js
  const myPlugin = (editor) => {
    editor.StyleManager.addBuiltIn('new-prop', {
      type: 'number',
      label: '新属性',
    })
  };

  grapesjs.init({
    // ...
    plugins: [myPlugin],
    styleManager: {
      sectors: [
        {
          name: '我的区块',
          properties: [ 'new-prop', ... ],
        },
      ],
    },
  })
  ```

- [添加新类型](#adding-new-types)。

## 自定义

默认类型应该能覆盖大多数常见的样式属性，但如果您需要对样式进行更高级的控制，您可以定义自己的类型，甚至可以从头开始创建一个完全自定义的样式管理器UI。

### 添加新类型

为了添加新类型，您必须使用 `styleManager.addType` API调用，并指明所有必要的方法以使其与编辑器正常工作。以下是使用原生 `range` 输入控件的实现示例。

<demo-viewer value="y1mxv6p5" height="500" darkcode/>

<!-- ```js
const customType = (editor) => {
  editor.StyleManager.addType('my-custom-prop', {
    // Create UI
    create({ props, change }) {
      const el = document.createElement('div');
      el.innerHTML = `<input type="range" class="my-input" min="${props.min}" max="${props.max}"/>`;
      const inputEl = el.querySelector('.my-input');
      inputEl.addEventListener('change', event => change({ event })); // `change` will trigger the emit
      inputEl.addEventListener('input', event => change({ event, partial: true }));
      return el;
    },
    // Propagate UI changes up to the targets
    emit({ props, updateStyle }, { event, partial }) {
      const { value } = event.target;
      updateStyle(`${value}px`, { partial });
    },
    // Update UI (eg. when the target is changed)
    update({ value, el }) {
      el.querySelector('.my-input').value = parseInt(value, 10);
    },
    // Clean the memory from side effects if necessary (eg. global event listeners, etc.)
    destroy() {
    },
  });
};

grapesjs.init({
  // ...
  plugins: [customType],
  styleManager: {
    sectors: [
      {
        name: 'My sector',
        properties: [
          {
            type: 'my-custom-prop',
            property: 'font-size',
            default: '15',
            min: 10,
            max: 70,
          },
        ],
      },
    ],
  },
})
``` -->

### 自定义样式管理器

如果您需要一个完全自定义的样式管理器UI（例如，您必须使用您的UI组件），您可以通过依赖相同的区块/属性架构从头开始创建它，甚至可以直接与选定的目标进行交互。

您所要做的就是向编辑器表明您打算使用自定义UI，然后订阅 `style:custom` 事件，该事件将让您知道何时是创建/更新UI的正确时机。

```js
const editor = grapesjs.init({
  // ...
  styleManager: {
    custom: true,
    // ...
  },
});

editor.on('style:custom', (props) => {
  // props.container (HTMLElement)
  //    默认元素，您可以在其中附加自定义UI
  //    以便在默认位置渲染它。
  // 在这里，您将放置逻辑以依赖样式管理器API来渲染/更新您的UI
});
```

下面是一个自定义样式管理器UI的示例，它使用Vuetify (Vue Material组件)渲染，并依赖于通过API实现的默认区块/属性状态架构。

<demo-viewer value="46kf7brn" height="500" darkcode/>

从上面的示例中，您可以看到我们如何订阅 `style:custom` 并在每次触发时更新 `this.sectors = sm.getSectors({ visible: true });`，这足以让框架自动更新模板的其余部分。

如果您需要直接获取/更新选定的样式目标，您也可以依赖这些API。

```js
// 从编辑器实例获取模块
const sm = editor.StyleManager;

// 选择当前页面中的第一个按钮
const wrapperCmp = editor.Pages.getSelected().getMainComponent();
const btnCmp = wrapperCmp.find('button')[0];
btnCmp && sm.select(btnCmp);

// 您也可以选择CSS查询作为目标
sm.select('.btn > span');

// 目标选定后，您可以检查其当前样式对象
console.log(sm.getSelected()?.getStyle());

// 并在必要时更新所有选定目标样式
sm.addStyleTargets({ color: 'red' });
```

<!--
<style>
  .style-manager {
    font-size: 12px;
  }
  .sm-input-color {
    width: 16px !important;
    height: 15px !important;
    opacity: 0 !important;
  }
  .sm-type-cmp {
    background-color: rgba(255,255,255,.05);
    border: 1px solid rgba(255,255,255,.25);
    border-radius: 3px;
    position: relative;
    min-height: 45px;
  }
  .sm-add-layer {
    position: absolute !important;
    top: -20px;
    right: 12px;
  }
  .sm-layer + .sm-layer {
    border-top: 1px solid rgba(255,255,255,.25);
  }
  .sm-layer-prv,
  .sm-btn-prv {
    width: 16px;
    height: 16px;
    border-radius: 3px;
    display: inline-block;
  }

  .sm-layer-prv--text-shadow::after {
    color: #000;
    content: "T";
    font-weight: 900;
    font-size: 10px;
    text-align: center;
    display: block;
  }

  .gjs-pn-views-container, .gjs-pn-views {
    width: 280px;
  }
  .gjs-pn-options {
    right: 280px;
  }
  .gjs-cv-canvas {
    width: calc(100% - 280px);
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
</style>
<div>
  <div class="style-manager">
      <v-app>
        <v-main>
        <v-expansion-panels  accordion multiple>
          <v-expansion-panel v-for="sector in sectors" :key="sector.getId()">
            <v-expansion-panel-header>
              {{ sector.getName() }}
            </v-expansion-panel-header>
            <v-expansion-panel-content>
              <v-row>
              <property-field v-for="prop in sector.getProperties()" :key="prop.getId() + prop.canClear()" :prop="prop"/>
              </v-row>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
        </v-main>
      </v-app>
  </div>

  <div id="property-field" style="display: none;">
    <v-col :class="['py-0 px-1 mb-1', prop.isFull() && 'mb-3']" :cols="prop.get('full') ? '12' : '6'">
        <v-row :class="labelCls">
          <v-col cols="auto pr-0">{{ prop.getLabel() }}</v-col>
          <v-col cols="auto" v-if="prop.canClear()">
            <v-icon @click="prop.clear()" color="indigo accent-1" small>mdi-close</v-icon>
          </v-col>
        </v-row>
        <div v-if="propType === 'number'">
          <v-text-field :placeholder="defValue" :value="inputValue" @change="handleChange" outlined dense/>
        </div>
        <div v-else-if="propType === 'radio'">
          <v-radio-group :value="prop.getValue()" @change="handleChange" row dense>
            <v-radio v-for="opt in prop.getOptions()" :key="prop.getOptionId(opt)" :label="prop.getOptionLabel(opt)" :value="prop.getOptionId(opt)"/>
          </v-radio-group>
        </div>
        <div v-else-if="propType === 'select'">
          <v-select :items="toOptions" :value="prop.getValue()" @change="handleChange" outlined dense/>
        </div>
        <div v-else-if="propType === 'color'">
          <v-text-field :placeholder="defValue" :value="inputValue" @change="handleChange" outlined dense>
            <template v-slot:append>
              <div :style="{ backgroundColor: prop.hasValue() ? prop.getValue() : defValue }">
                <input class="sm-input-color"
                  type="color"
                  :value="prop.hasValue() ? prop.getValue() : defValue"
                  @change="(ev) => handleChange(ev.target.value)"
                  @input="(ev) => handleInput(ev.target.value)"
                />
              </div>
          </template>
          </v-text-field>
        </div>
        <div v-else-if="propType === 'slider'">
          <v-slider
            track-color="white"
            :value="prop.getValue()"
            :min="prop.getMin()"
            :max="prop.getMax()"
            :step="prop.getStep()"
            @change="handleChange"
            @input="(value) => { console.log('trigger input', value) }"
            @start="(value) => { console.log('trigger start', value) }"
          />
        </div>
        <div v-else-if="propType === 'file'">
          <v-btn @click="openAssets(prop)" block>
            <v-row>
              <v-col v-if="prop.getValue() && prop.getValue() !== defValue" cols="auto">
                <div class="sm-btn-prv" :style="{ backgroundImage: `url(${prop.getValue()})` }"></div>
              </v-col>
              <v-col>Select image</v-col>
            </v-row>
          </v-btn>
        </div>
        <div v-else-if="propType === 'composite'">
          <v-row no-gutters class="sm-type-cmp pa-2">
            <property-field v-for="p in prop.getProperties()" :key="p.getId() + p.canClear()" :prop="p"/>
          </v-row>
        </div>
        <div v-else-if="propType === 'stack'">
          <div class="sm-type-cmp pa-3">
            <v-icon @click="prop.addLayer({}, { at: 0 })" class="sm-add-layer" small>mdi-plus</v-icon>
            <v-row class="sm-layer" v-for="layer in prop.getLayers()" :key="layer.getId()">
              <v-col>
                <v-row>
                  <v-col cols="auto" class="pr-1">
                    <v-icon @click="layer.move(layer.getIndex() - 1)" small>mdi-arrow-up</v-icon>
                  </v-col>
                  <v-col cols="auto" class="pl-1">
                    <v-icon @click="layer.move(layer.getIndex() + 1)" small>mdi-arrow-down</v-icon>
                  </v-col>
                  <v-col @click="layer.select()">{{ layer.getLabel() }}</v-col>
                  <v-col cols="auto">
                      <div :class="['sm-layer-prv white', `sm-layer-prv--${propName}`]" :style="layer.getStylePreview({ number: { min: -3, max: 3 } })"></div>
                  </v-col>
                  <v-col cols="auto">
                    <v-icon @click="layer.remove()" small>mdi-close</v-icon>
                  </v-col>
                </v-row>
                <v-row v-if="layer.isSelected()" no-gutters class="sm-type-cmp pa-2 mt-3">
                  <property-field v-for="p in prop.getProperties()" :key="p.getId()" :prop="p"/>
                </v-row>
              </v-col>
            </v-row>
          </div>
        </div>
        <div v-else>
          <v-text-field :placeholder="defValue" :value="inputValue" @change="handleChange" outlined dense/>
        </div>
    </v-col>
  </div>
</div>
<script>

  const sm = editor.StyleManager;
  const Observer = (new Vue()).$data.__ob__.constructor; // obj.__ob__ = new Observer({});

  Vue.mixin({
    data() {
      return { editor };
    }
  });

  Vue.component('property-field', {
    props: { prop: Object },
    template: '#property-field',
    computed: {
      labelCls() {
        const { prop } = this;
        const parent = prop.getParent();
        const hasParentValue = prop.hasValueParent() && (parent ? parent.isDetached() : true);
        return ['flex-nowrap', prop.canClear() && 'indigo--text text--accent-1', hasParentValue && 'orange--text'];
      },
      inputValue() {
        return this.prop.hasValue() ? this.prop.getValue() : '';
      },
      propName() {
        return this.prop.getName();
      },
      propType() {
        return this.prop.getType();
      },
      defValue() {
        return this.prop.getDefaultValue();
      },
      toOptions() {
        const { prop } = this;
        return prop.getOptions().map(o => ({ value: prop.getOptionId(o), text: prop.getOptionLabel(o) }))
      },
    },
    methods: {
      handleChange(value) {
        this.prop.upValue(value);
      },
      handleInput(value) {
        this.prop.upValue(value, { partial: true });
      },
      openAssets(prop) {
        const { Assets } = this.editor;
        Assets.open({
          select: (asset, complete) => {
            prop.upValue(asset.getSrc(), { partial: !complete });
            complete && Assets.close();
          },
          types: ['image'],
          accept: 'image/*',
        })
      }
    }
  })

  const app = new Vue({
    vuetify: new Vuetify({
      theme: { dark: true },
    }),
    el: '.style-manager',
    data: { sectors: [] },
    mounted() {
      editor.on('style:custom', this.handleCustom);
    },
    destroyed() {
      editor.off('style:custom', this.handleCustom);
    },
    methods: {
      handleCustom(props) {
        if (props.container && !props.container.contains(this.$el)) {
          props.container.appendChild(this.$el);
        }
        this.sectors = sm.getSectors({ visible: true });
      },
    }
  });
</script>
-->

## 事件

有关可用事件的完整列表，您可以[在此处](/api/style_manager.html#available-events)查看。

[Components]: Components.html
[I18n]: I18n.html
[Style Manager API]: /api/style_manager.html