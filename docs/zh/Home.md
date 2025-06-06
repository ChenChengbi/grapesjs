# 开始入门

本页面将向您介绍 GrapesJS 的主要选项及其工作原理，以便您能够创建自定义编辑器(editor)。

以非常简洁的方式实例化(instantiate)编辑器的代码如下所示：

```html
<link rel="stylesheet" href="path/to/grapes.min.css" />
<script src="path/to/grapes.min.js"></script>

<div id="gjs"></div>

<script type="text/javascript">
  var editor = grapesjs.init({
    container: '#gjs',
    components: '<div class="txt-red">Hello world!</div>',
    style: '.txt-red{color: red}',
  });
</script>
```

仅需几行代码，使用默认配置(configurations)，您就已经能看到一些可以操作的内容了。

[[img/default-gjs.jpg]]

您将在左上角看到组件(components)命令，这些命令便于创建和管理您的区块(blocks)，其下方是一些需要高亮和导出它们的选项。当您选择组件（“鼠标指针”图标）时，右侧应该会弹出类管理器(Class Manager)和样式管理器(Style Manager)选项，允许您自定义组件的样式。还有一个图层管理器/导航器(Layer Manager/Navigator)（“汉堡”图标），可帮助轻松管理结构。

当然，所有这些元素（面板(panels)、按钮(buttons)、命令等）都只是默认设置，因此您可以覆盖(overwrite)它们并添加更多其他内容。在开始创建之前，您应该知道 GrapesJS UI 主要由一个画布(canvas)（您将在其中“绘制”）和面板（将包含按钮）组成。

[[img/canvas-panels.jpg]]

如果您想扩展已经实例化的编辑器，您需要查看 [API 参考]。也请查看如何使用相同的 API [创建插件](./Creating-plugins)。
在本指南中，我们将重点介绍如何从头开始使用所有自定义 UI 初始化编辑器。

让我们用一些基本的工具栏面板(toolbar panel)来启动编辑器：

```js
...
var editor = grapesjs.init({
    container : '#gjs',
    height: '100%',

    panels: {
      defaults: [{
          id: 'commands',
      }],
    }
});
...
```

在此示例中，我们设置了一个 id 为 `commands` 的面板，在渲染(render)之后，我们只会看到一个空 div 添加到我们的面板中。新的面板已经设置了样式，因为 id `commands` 是默认的之一，但您可以使用任何您喜欢的 id，并使用 CSS 将其放置在任何您想要的位置。刷新后，我们可能会看到如下图所示的内容，新的面板在左侧：

[[img/new-panel.png]]

> 有关编辑器配置的更多详细信息，请查看 [编辑器 API 参考]。

现在让我们在里面放一些按钮：

```js
...
  panels: {
    defaults  : [{
        id      : 'commands',
        buttons : [{
            id          : 'smile',
            className   : 'fa fa-smile-o',
            attributes  : { title: 'Smile' }
        }],
    }],
  }
...
```

页面刷新后可能会出现一些变化（`fa fa-smile-o` 来自 FontAwesome 字体集，因此请确保已正确放置字体目录）：

[[img/new-btn.png]]

是的，这个按钮非常漂亮且令人愉悦，但没有分配任何命令(command)就毫无用处，如果您点击它，什么也不会发生。

> 有关面板和按钮的更多详细信息，请查看 [面板 API 参考]。

分配命令非常简单，但在此之前，您应该定义一个或使用一个默认的（[内置命令](./Built-in-commands)）。因此，在这种情况下，我们将创建一个新的命令。

```js
...
  panels: {
    defaults  : [{
        id      : 'commands',
        buttons : [{
            id          : 'smile',
            className   : 'fa fa-smile-o',
            attributes  : { title: 'Smile' },
            command     : 'helloWorld',
        }],
    }],
  },
  commands: {
    defaults: [{
        id: 'helloWorld',

        run:  function(editor, senderBtn){
          alert('Hello world!');
          // Deactivate button
          senderBtn.set('active', false);
        },

        stop:  function(editor, senderBtn){
        },
    }]
  }
...
```

如您所见，我们添加了一个新的命令 `helloWorld`，并将其 `id` 用作 `button.command` 内的标识符(identifier)。除此之外，我们还实现了两个必需的方法，`run` 和 `stop`，以使按钮执行(execute)命令。

[[img/btn-clicked.png]]

> 查看 [命令 API 参考]。

有关面板、按钮和内置命令的更完整用法，请查看[演示](http://grapesjs.com/demo.html)。

## 组件

组件是画布内的元素，可以通过命令绘制或通过配置直接注入。简单来说，组件代表了我们 HTML 文档(HTML document)的结构。您可以通过传递 HTML 字符串来初始化编辑器中的组件：

```js
...
  // Disable default local storage in case you've already used GrapesJS
  storageManager: {type: 'none'},

  components: '<div style="width:300px; min-height:100px; margin: 0 auto"></div>' +
              '<div style="width:400px; min-height:100px; margin: 0 auto"></div>' +
              '<div style="width:500px; min-height:100px; margin: 0 auto"></div>',
...
```

我们添加了 3 个带有基本样式的简单组件。如果您刷新页面，可能仍会看到相同的空白页，但它们实际上在那里，您只需要高亮(highlight)它们。
为此目的，已经存在一个命令，因此请按以下方式将其添加到您的面板中：

```js
...
  panels: {
    defaults  : [{
        id      : 'commands',
        buttons : [
          {
            id: 'smile',
            ...
          },
          {
            id        : 'vis',
            className : 'fa fa-eye',
            command   : 'sw-visibility',
            context   : 'some-random-context', // For grouping context of buttons in the same panel
            active    : true,
          },
        ],
    }],
  },
...
```

值得注意的是 `context` 选项的使用（尝试在没有它的情况下点击 `smile`<sup>微笑</sup> 命令）和 `active` 以在渲染后启用它。
现在您应该能够看到画布内的区块了。

[[img/blocks3.jpg]]

您可以添加其他命令以启用与区块的交互(interactions)。查看[内置命令](./Built-in-commands)以获取更多信息。

> 查看 [组件 API 参考]。

## 样式管理器

任何 HTML 结构在某个时候都需要适当的样式，因此为了满足这一需求，样式管理器(Style Manager)作为 GrapesJS 的一个内置功能(built-in feature)被添加进来。样式管理器由多个区域(sectors)组成，这些区域内部组织了不同类型的 CSS 属性(CSS properties)。因此，您可以例如为 `width` 和 `height` 添加一个 `Dimension`<sup>维度</sup> 区域，并为 `font-size` 和 `color` 添加另一个名为 `Typography`<sup>排版</sup> 的区域。因此，由您来决定如何组织这些区域。

要启用此模块(module)，我们依赖一个内置命令 `open-sm`，它会显示样式管理器，我们将把它绑定(bind)到另一个面板中的另一个按钮上：

```js
...
panels: {
    defaults  : [
      {
        id      : 'commands',
        ...
      },{
        // If you use this id the default CSS will place this panel on top right corner for you
        id      : 'views',
        buttons : [{
            id        : 'open-style-manager',
            className : 'fa fa-paint-brush',
            command   : 'open-sm',
            active    : true,
        }]
      }
    ],
},
...
```

之后，您将能够看到如下图所示的内容：

[[img/enabled-sm.jpg]]

如您所见，样式管理器已启用，但在使用它之前，您必须在画布中选择一个元素，为此，我们可以通过以下方式添加另一个带有内置命令 `select-comp` 的按钮：

```js
...
  panels: {
    defaults  : [{
        id      : 'commands',
        buttons : [
          {
            id: 'smile',
            ...
          },{
            id         : 'select',
            className : 'fa fa-mouse-pointer',
            command   : 'select-comp',
          }
        ],
    }],
  },
...
```

选择其中一个组件将显示样式管理器，其中包含默认的区域、属性以及一个可以管理类的输入框。您看到的默认类 (cXX) 是通过从组件中提取样式生成的。

[[img/default-sm.jpg]]

当我们探索 GrapesJS 内部的不同配置时，我们将覆盖所有默认区域以创建一些自定义区域。

让我们使用 `buildProps` 来放置一些区域，它可以帮助我们构建常见的属性：

```js
...
  styleManager : {
    sectors: [{
      name: 'Dimension',
      buildProps: ['width', 'min-height']
    },{
      name: 'Extra',
      buildProps: ['background-color', 'box-shadow']
    }]
  }
...
```

现在您应该能够为组件设置样式了。

[[img/style-comp.jpg]]

您可以在此处查看 `buildProps` 中可用的属性列表：[内置属性](./Built-in-properties)。
否则，也可以自己构建它们，让我们看看在没有 `buildProps` 辅助函数的情况下，我们如何完成之前的配置：

```js
...
styleManager : {
  sectors: [
    {
      name: 'Dimension',
      properties:[
        {
            // Just the name
            name      : 'Width',
            // CSS property
            property  : 'width',
            // Type of the input, options: integer | radio | select | color | file | composite | stack
            type      : 'integer',
            // Units, available only for 'integer' types
            units     : ['px', '%'],
            // Default value
            defaults  : 'auto',
            // Min value, available only for 'integer' types
            min       : 0,
        },{
            // Here I'm going to be more original
            name      : 'Minimum height',
            property  : 'min-height',
            type      : 'select',
            defaults  : '100px',
            // List of options, available only for 'select' and 'radio'  types
            list    : [{
                      value   : '100px',
                      name    : '100',
                    },{
                      value   : '200px',
                      name    : '200',
                    },{
                      value   : '300px',
                      name    : '300',
                    }],
        }
      ]
    },{
      name: 'Extra',
      // Sectors are expanded by default so put this one closed
      open: false,
      properties:[
        {
          name      : 'Background',
          property  : 'background-color',
          type      : 'color',
          defaults:   'none'
        },{
          name    : 'Box shadow',
          property  : 'box-shadow',
          type    : 'stack',
          preview   : true,
          // List of nested properties, available only for 'stack' and 'composite'  types
          properties  : [{
                  name:     'Shadow type',
                  // Nested properties with stack/composite type don't require proper 'property' name
                  // as all of them will be merged to parent property, eg. box-shadow: X Y ...;
                  property:   'shadow-type',
                  type:     'select',
                  defaults:   '',
                  list:   [ { value : '', name : 'Outside', },
                              { value : 'inset', name : 'Inside', }],
                },{
                  name:     'X position',
                  property:   'shadow-x',
                  type:     'integer',
                  units:    ['px','%'],
                  defaults :  0,
                },{
                  name:     'Y position',
                  property:   'shadow-y',
                  type:     'integer',
                  units:    ['px','%'],
                  defaults :  0,
                },{
                  name:     'Blur',
                  property: 'shadow-blur',
                  type:     'integer',
                  units:    ['px'],
                  defaults :  0,
                  min:    0,
                },{
                  name:     'Spread',
                  property:   'shadow-spread',
                  type:     'integer',
                  units:    ['px'],
                  defaults :  0,
                },{
                  name:     'Color',
                  property:   'shadow-color',
                  type:     'color',
                  defaults:   'black',
                },],
        }
      ]
    }
  ]
}
...
```

如您所见，使用 `buildProps` 实际上会为您节省大量工作。您也可以混合使用这些技术，以在更短的时间内获得自定义属性。例如，让我们看看如何设置相同的宽度但使用不同的 `min` 值：

```js
...
  styleManager : {
    sectors: [{
      name: 'Dimension',
      buildProps: ['width', 'min-height'],
      properties:[{
        property: 'width', // Use 'property' as id
        min: 30
      }]
    },
    ...
  }
...
```

> 查看 [样式管理器 API 参考]。

## 存储/加载数据

在最后一部分，我们将了解如何在 GrapesJS 内部存储和加载模板数据(template data)。您可能已经注意到，即使在画布上进行更改后刷新页面，您的数据也不会丢失，这是因为 GrapesJS 自带了一些内置的存储实现。
默认的是本地存储(localStorage)，它非常简单，所有数据都存储在您的计算机本地。让我们看看此存储可用的选项：

```js
...
var editor = grapesjs.init({
    container : '#gjs',
    ...
    // Default configuration
    storageManager: {
      id: 'gjs-',             // Prefix identifier that will be used inside storing and loading
      type: 'local',          // Type of the storage
      autosave: true,         // Store data automatically
      autoload: true,         // Autoload stored data on init
      stepsBeforeSave: 1,     // If autosave enabled, indicates how many changes are necessary before store method is triggered
      storeComponents: false, // Enable/Disable storing of components in JSON format
      storeStyles: false,     // Enable/Disable storing of rules/style in JSON format
      storeHtml: true,        // Enable/Disable storing of components as HTML string
      storeCss: true,         // Enable/Disable storing of rules/style as CSS string
    }
});
...
```

值得注意的是默认的 `id` 参数，它为所有要存储的键添加了前缀。如果您检查 DOM 面板中的 localStorage，您会看到类似 `{ 'gjs-components': '<div>....' ...}` 的内容，这样可以防止在大规模应用程序中使用 localStorage 时常见的冲突风险。

在本地存储数据既简单又快速，但在某些常见情况下却毫无用处。在下一个示例中，我们将看到如何设置远程存储(remote storage)，这与前一个示例相差不远：

```js
...
var editor = grapesjs.init({
    container : '#gjs',
    ...
    storageManager: {
      type: 'remote',
      stepsBeforeSave: 10,
      urlStore: 'http://store/endpoint',
      urlLoad: 'http://load/endpoint',
      params: {},   // For custom values on requests
    }
});
...
```

如您所见，我们保留了一些默认选项不变，增加了触发(trigger)自动保存(autosave)所需的更改次数，并传递了远程端点(endpoints)。

如果您愿意，也可以禁用自动保存，并使用自定义命令自行完成，方法如下：

```js
...
  storageManager: {
    type: 'remote',
    autosave: false,
  },
  ...
  commands: {
    defaults: [{
        id: 'storeData',
        run:  function(editor, senderBtn){
          editor.store();
        },
    }]
  }
...
```

> 查看 [存储管理器 API 参考]。

[API Reference]: API-Reference
[Panels API Reference]: API-Panels
[Commands API Reference]: API-Commands
[Components API Reference]: API-Components
[Style Manager API Reference]: API-Style-Manager
[Editor API Reference]: API-Editor
[Storage Manager API Reference]: API-Storage-Manager