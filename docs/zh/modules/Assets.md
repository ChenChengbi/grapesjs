---
title: 资产管理器
---

# 资产管理器

<p align="center"><img src="http://grapesjs.com/img/sc-grapesjs-assets-1.jpg" alt="GrapesJS - 资产管理器" align="center"/></p>

在本节中，您将了解如何在 GrapesJS 中设置并充分利用内置的资产管理器(Asset Manager)。资产管理器是轻量级的，其核心仅实现了一个 `image` 类型，但正如您接下来将看到的，扩展和创建您自己的资产类型非常容易。

::: tip
想要一个开箱即用且外观出色的资产管理器吗？[试试 Grapes Studio SDK！](https://app.grapesjs.com/docs-sdk/configuration/assets/overview?utm_source=grapesjs-docs&utm_medium=tip)
:::

[[toc]]

## 配置

要更改默认配置，您需要在主配置对象中传递 `assetManager` 属性

```js
const editor = grapesjs.init({
  ...
  assetManager: {
    assets: [...],
    ...
  }
});
```

您稍后可以使用模块(module)内的 `getConfig` 方法更新大部分配置

```js
const amConfig = editor.AssetManager.getConfig();
```

在此处查看可用选项的完整列表：[资产管理器配置 (Asset Manager Config)](https://github.com/GrapesJS/grapesjs/blob/master/src/asset_manager/config/config.ts)

## 初始化

资产管理器默认即可工作，因此传递一些 URL 即可看到它们被加载

```js
const editor = grapesjs.init({
  ...
  assetManager: {
    assets: [
     'http://placehold.it/350x250/78c5d6/fff/image1.jpg',
     // 传递带有自定义属性的对象
     {
       type: 'image',
       src: 'http://placehold.it/350x250/459ba8/fff/image2.jpg',
       height: 350,
       width: 250,
       name: 'displayName'
     },
     {
       // 由于 'image' 是资产的基本类型，省略它将
       // 默认设置为 `image`
       src: 'http://placehold.it/350x250/79c267/fff/image3.jpg',
       height: 350,
       width: 250,
       name: 'displayName'
     },
    ],
  }
});
```

如果您想要可用属性的完整列表，请查看源代码 [AssetImage 模型 (AssetImage Model)](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/asset_manager/model/AssetImage.ts)

内置的资产管理器模态框(modal)已经实现，并在请求时显示。默认情况下，您可以通过在画布(canvas)中拖动图像组件(Image Components)、双击图像以及所有其他与图像相关的事项（例如 CSS 样式）来使其出现。

<img :src="$withBase('/assets-builtin-modal.png')">

<!--
使模态框出现已注册为一个命令，因此您可以使用此命令使其出现

```js
// 此命令仅显示 `image` 类型的资产
editor.runCommand('open-assets');
```


值得注意的是，这样做您无法对资产进行太多操作（如果您双击它们，则不会发生任何事情），这是因为您尚未指定任何目标。尝试在画布中选择一个图像并在控制台中运行此命令（您应首先在脚本中使编辑器全局可用 `window.editor = editor;`）

```js
editor.runCommand('open-assets', {
  target: editor.getSelected()
});
```

现在您应该能够更改组件的图像了。
-->

## 上传资产

默认的资产管理器还包含一个易于使用的拖放上传器(uploader)，并带有一些 UI 辅助功能。默认的上传器在您打开资产管理器时就已经可见。

<img :src="$withBase('/assets-uploader.png')">

您可以点击上传器选择文件，或者直接从计算机拖动文件来触发上传器。显然，在它工作之前，您必须设置服务器以接收您的资产，并在配置中指定上传端点(endpoint)。

```js
const editor = grapesjs.init({
  ...
  assetManager: {
    ...
    // 上传端点，设置为 `false` 以禁用上传，默认为 `false`
    upload: 'https://endpoint/upload/assets',

    // POST 请求中用于传递已上传文件的名称，默认为 `'files'`
    uploadName: 'files',
    ...
  },
  ...
});
```

### 监听器 (Listeners)

如果您想在上传过程之前/之后执行操作（例如加载动画）甚至在收到响应时执行操作，可以使用这些监听器

```js
// 上传开始
editor.on('asset:upload:start', () => {
  startAnimation();
});

// 上传结束（无论是否完成）
editor.on('asset:upload:end', () => {
  endAnimation();
});

// 错误处理
editor.on('asset:upload:error', (err) => {
  notifyError(err);
});

// 对响应执行某些操作
editor.on('asset:upload:response', (response) => {
  ...
});
```

### 响应 (Response)

上传结束后，默认情况下（通过配置参数 `autoAdd: 1`），编辑器期望在响应(response)的 `data` 键中接收到已上传资产的 JSON，并尝试将它们添加到主集合中。该 JSON 可能如下所示：

```js
{
  data: [
    'https://.../image.png',
    // ...
    {
      src: 'https://.../image2.png',
      type: 'image',
      height: 100,
      width: 200,
    },
    // ...
  ];
}
```

<!-- 已弃用
### 设置拖放区域 (Dropzone)

还有另一个辅助功能可以改进资产上传：一个全宽的编辑器拖放区域。

<img :src="$withBase('/assets-full-dropzone.gif')">


您所要做的就是激活它，并可能设置自定义内容（您可能还想隐藏默认的上传器）

```js
const editor = grapesjs.init({
  ...
  assetManager: {
    ...,
    dropzone: 1,
    dropzoneContent: '<div class="dropzone-inner">将您的资产拖放到此处</div>'
  }
});
``` -->

## 程序化使用

如果您需要以编程方式管理资产，则必须使用其 [API][API-Asset-Manager]

```js
// 首先获取资产管理器模块
const am = editor.AssetManager;
```

首先，值得注意的是，资产管理器维护着两个资产集合：

-   **全局 (global)** - 包含所有可用资产的集合，您可以使用 `am.getAll()` 获取它
-   **可见 (visible)** - 这是资产管理器当前渲染的集合，您可以使用 `am.getAllVisible()` 获取它

这使您可以决定何时显示哪些资产。假设我们想要一个类别切换器，首先，您需要将所有资产添加到**全局**集合中（您可能已经在初始化时通过 `config.assetManager.assets = [...]` 定义了这些资产）

```js
am.add([
  {
    // 您可以传递任何想要的自定义属性
    category: 'c1',
    src: 'http://placehold.it/350x250/78c5d6/fff/image1.jpg',
  },
  {
    category: 'c1',
    src: 'http://placehold.it/350x250/459ba8/fff/image2.jpg',
  },
  {
    category: 'c2',
    src: 'http://placehold.it/350x250/79c267/fff/image3.jpg',
  },
  // ...
]);
```

现在，如果您调用 `render()` 方法且不带参数，您将看到所有资产都被渲染出来

```js
// 不带任何参数
am.render();

am.getAll().length; // <- 3
am.getAllVisible().length; // <- 3
```

好的，现在我们只显示第一个类别的资产

```js
const assets = am.getAll();

am.render(assets.filter((asset) => asset.get('category') == 'c1'));

am.getAll().length; // 仍然有 3 个资产
am.getAllVisible().length; // 但只显示了 2 个
```

您还可以混合使用资产数组

```js
am.render([...assets1, ...assets2, ...assets3]);
```

<!--
如果您想自定义资产管理器容器，可以获取其 `HTMLElement`

```js
am.getContainer().insertAdjacentHTML('afterbegin', '<div><button type="button">点击</button></div>');
```
-->

如果您想更新或删除资产，可以使用这些方法

```js
// 通过 `src` 获取资产
const asset = am.get('http://.../img.jpg');

// 更新资产属性
asset.set({ src: 'http://.../new-img.jpg' });

// 删除资产
am.remove(asset); // 或者通过 src, am.remove('http://.../new-img.jpg');
```

更多 API 方法，请查看 [API 参考][API-Asset-Manager]。

### 自定义选择逻辑

::: warning
本节内容适用于 GrapesJS v0.17.26 或更高版本
:::

您可以使用自己的选择逻辑打开资产管理器。

```js
am.open({
  types: ['image'], // 这是默认选项
  // 如果没有 select，资产选择时不会发生任何事情
  select(asset, complete) {
    const selected = editor.getSelected();

    if (selected && selected.is('image')) {
      selected.addAttributes({ src: asset.getSrc() });
      // 默认的 AssetManager UI 将在单击资产时触发 `select(asset, false)`
      // 并在双击时触发 `select(asset, true)`
      complete && am.close();
    }
  },
});
```

## 自定义

默认的资产管理器 UI 非常适合简单任务，但除了调整某些 CSS 样式的可能性之外，添加更复杂的功能（如搜索输入框、过滤器等）则需要替换默认 UI。

您所要做的就是向编辑器表明您打算使用自定义 UI，然后订阅 `asset:custom` 事件，该事件将为您提供有关任何请求更改的所有信息。

```js
const editor = grapesjs.init({
  // ...
  assetManager: {
    // ...
    custom: true,
  },
});

editor.on('asset:custom', (props) => {
  // `props` 将包含更新 UI 所需的所有信息。
  // props.open (boolean) - 指示资产管理器是否打开
  // props.assets (Array<Asset>) - 所有资产的数组
  // props.types (Array<String>) - 请求的资产类型数组，例如 ['image']
  // props.close (Function) - 关闭资产管理器的回调函数
  // props.remove (Function<Asset>) - 删除资产的回调函数
  // props.select (Function<Asset, boolean>) - 选择资产的回调函数
  // props.container (HTMLElement) - 您应该将 UI 附加到的元素
  // 在这里您将放置渲染/更新 UI 的逻辑。
});
```

以下是一个使用 Vue 组件实现自定义资产管理器的示例。

<demo-viewer value="wbj4tmqk" height="500" darkcode/>

如果您需要替换默认 UI，上述示例是正确的方法，但您可能注意到我们将挂载的元素附加到了容器 `props.container.appendChild(this.$el);`。
这是必需的，因为默认情况下，资产管理器位于[模态框(Modal)](/modules/Modal.html)中。

如果您的资产管理器是一个完全独立/外部的模块（例如，应该在自己的自定义模态框中显示），该如何处理？没问题，您可以通过 `assetManager.custom.open` 绑定资产管理器的状态。

```js
const editor = grapesjs.init({
  // ...
  assetManager: {
    // ...
    custom: {
      open(props) {
        // `props` 与 `asset:custom` 事件中使用的相同
        // ...
        // 初始化并打开您的外部资产管理器
        // ...
        // 重要提示：
        // 当外部库关闭时，您必须将此状态传回编辑器，
        // 否则 GrapesJS 会认为资产管理器仍处于打开状态。
        // 示例：myAssetManager.on('close', () => props.close())
      },
      close(props) {
        // 关闭外部资产管理器
      },
    },
  },
});
```

声明 `close` 函数也很重要，编辑器应该能够通过 `am.close()` 关闭资产管理器。

<!--
### 定义新的资产类型

一般来说，图像并不是您将使用的唯一资产，它可能是 `video`、`svg-icon` 或任何其他类型的 `document`。每种类型的资产在我们的模板/页面中的应用方式都不同。如果您需要更改组件的图像，您只需要 `src` 属性中的另一个 `url`。但是，对于 `svg-icon` 的情况则不同，您可能希望用新的 `<svg>` 内容替换该元素。除此之外，您还必须处理资产在面板/模态框中的表示/预览。例如，为大图像显示缩略图或预览视频的可能性。


定义新的资产类型意味着我们必须在“类型栈 (Stack of Types)”的顶部推入一个新的层。编辑器在每次添加资产时都会迭代此堆栈，并尝试关联正确的类型。

```js
am.add('https://.../image.png');
// 字符串，url，以 '.png' 结尾 -> 它是 'image' 类型

am.add('<svg ...');
// 字符串且以 '<svg...' 开头 -> 'svg' 类型

am.add({type: 'video', src: '...'});
// 对象，具有 'video' 类型键 -> 'video' 类型
```

您需要告诉编辑器如何识别您的类型，为此您应该使用 `isType()` 方法。
现在让我们看一个如何开始定义像 `svg-icon` 这样的类型的示例


```js
am.addType('svg-icon', {
  // `value` 是例如传递给 `am.add(VALUE);` 的参数
  isType(value) {
    // 条件故意很简单
    if (value.substring(0, 5) == '<svg ') {
      return {
        type: 'svg-icon',
        svgContent: value
      };
    }
    // 也许您已经传递了 `svg-icon` 对象
    else if (typeof value == 'object' && value.type == 'svg-icon') {
      return value;
    }
  }
})
```

有了这个代码片段，您就已经可以添加 SVG 了，资产管理器将分配适当的类型。

```js
// 添加一些随机的 SVG
am.add(`<svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
  <path d="M22,9 C22,8.4 21.5,8 20.75,8 L3.25,8 C2.5,8 2,8.4 2,9 L2,15 C2,15.6 2.5,16 3.25,16 L20.75,16 C21.5,16 22,15.6 22,15 L22,9 Z M21,15 L3,15 L3,9 L21,9 L21,15 Z"></path>
  <polygon points="4 10 5 10 5 14 4 14"></polygon>
</svg>`);
```


默认的 `open-assets` 命令仅显示 `image` 资产，因此要渲染 `svg-icon`，请运行此命令

```js
am.render(am.getAll().filter(
  asset => asset.get('type') == 'svg-icon'
));
```


您应该会看到类似这样的内容

<img :src="$withBase('/assets-empty-view.png')">


SVG 资产未正确渲染，这是因为我们尚未配置其视图(view)

```js
am.addType('svg-icon', {
  view: {
    // `getPreview()` 和 `getInfo()` 只是一些辅助方法，您可以使用
    // `template()` 方法覆盖整个模板
    // 在此处查看基本的 `template()`：
    // https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/asset_manager/view/AssetView.ts
    getPreview() {
      return `<div style="text-align: center">${this.model.get('svgContent')}</div>`;
    },
    getInfo() {
      // 如果您传递了模型的属性，则可以使用它们：
      // am.add({
      //  type: 'svg-icon',
      //  svgContent: '<svg ...',
      //  name: 'Some name'
      //  })
      //  ... 然后
      //  this.model.get('name');
      return '<div>SVG 描述</div>';
    },
  },
  isType(value) {...}
})
```


结果如下

<img :src="$withBase('/assets-svg-view.png')">


现在我们必须处理如何将我们的 `svgContent` 分配给所选元素


```js
am.addType('svg-icon', {
  view: {
    // 在我们的例子中，目标是选定的组件
    updateTarget(target) {
      const svg = this.model.get('svgContent');

      // 为了让事情更有趣一点，如果它是图像类型
      // 我将 svg 作为数据 URI 放入，否则放入内容
      if (target.get('type') == 'image') {
        // 提示：您也可以使用 `data:image/svg+xml;utf8,<svg ...` 但您
        // 必须转义一些字符
        target.set('src', `data:mime/type;base64,${btoa(svg)}`);
      } else {
        target.set('content', svg);
      }
    },
    ...
  },
  isType(value) {...}
})
```


我们的自定义 `svg-icon` 资产已准备就绪。您还可以向 `addType` 定义中添加一个 `model` (模型)来对资产的业务逻辑进行分组，但这通常是可选的。


```js
// 只是一个模型使用的示例
am.addType('svg-icon', {
  model: {
    // 使用 `default` 定义模型的默认属性
    defaults: {
      type:  'svg-icon',
      svgContent: '',
      name: '默认 SVG 名称',
    },

    // 您可以在视图中调用模型的方法：
    // const name = this.model.getName();
    getName() {
      return this.get('name');
    }
  },
  view: {...},
  isType(value) {...}
})
```


### 扩展资产类型

扩展资产类型与添加它们基本相同，您可以选择要扩展的类型以及如何扩展。

```js
// svgIconType 将包含定义（模型、视图、isType）
const svgIconType = am.getType('svg-icon');

// 添加新类型并扩展另一种类型
am.addType('svg-icon2', {
  view: svgIconType.view.extend({
    getInfo() {
      return '<div>SVG2 描述</div>';
    },
  }),
  // `isType` 很重要，但如果省略它，将添加默认的
  // isType(value) {
  //  if (value && value.type == id) {
  //    return {type: value.type};
  //  }
  // };
})
```


您还可以扩展已定义的类型（为确保加载具有旧扩展类型的资产，请为您的定义创建一个插件）

```js
// 扩展原始 `image` 类型并在删除它之前添加一个确认对话框
am.addType('image', {
  // 由于您是在已定义的类型之上添加，因此可以避免指明
  // `am.getType('image').view.extend({...` 编辑器会默认执行此操作
  // 但您最终可以扩展其他一些类型
  view: {
    // 如果您想查看更多可扩展的方法，请查看
    // https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/asset_manager/view/AssetImageView.ts
    onRemove(e) {
      e.stopPropagation();
      const model = this.model;

      if (confirm('您确定吗？')) {
        model.collection.remove(model);
      }
    }
  },
})
``` -->

## 事件

有关可用事件的完整列表，您可以在[此处](/api/assets.html#available-events)查看。

[API-Asset-Manager]: /api/assets.html