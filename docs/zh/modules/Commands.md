---
title: 命令
---

# 命令

GrapesJS 中的基础命令(command)是一个简单的函数，但在本指南中您将看到它们能有多么强大。命令模块(Command module)的主要目标是集中管理函数，并使其易于在整个编辑器(editor)中复用。使用命令的另一个巨大优势是能够追踪、扩展甚至在特定条件下中断它们。

::: warning
本指南适用于 GrapesJS v0.14.61 或更高版本
:::

[[toc]]

## 基本配置

您可以在初始化(initialization)步骤中通过在 `commands.defaults` 选项中传递它们来创建命令：

```js
const editor = grapesjs.init({
  ...
  commands: {
    defaults: [
      {
        // id 和 run 在这种情况下是强制性的
        id: 'my-command-id',
        run() {
          alert('This is my command');
        },
      }, {
        id: '...',
        // ...
      }
    ],
  }
});
```

所有其他可用选项，请直接查看[配置文件源码](https://github.com/GrapesJS/grapesjs/blob/master/src/commands/config/config.ts)。

更常见的情况是，命令是在初始化后动态创建的，这种情况下，您需要使用[命令 API(Commands API)](/api/commands.html)（例如，如果您创建插件(plugin)，就需要这样做）

```js
const commands = editor.Commands;
commands.add('my-command-id', (editor) => {
  alert('This is my command');
});

// 或者这样做也一样...
commands.add('my-command-id', {
  run(editor) {
    alert('This is my command');
  },
});
```

如您所见，定义非常简单，只需添加一个 ID 和回调函数(callback function)即可。[编辑器(Editor)](/api/editor.html)实例作为第一个参数传递给回调函数，因此您可以访问任何其他模块(module)或 API 方法。

现在，如果您想调用该命令，只需运行：

```js
editor.runCommand('my-command-id');
```

::: tip
方法 `editor.runCommand` 是 `editor.Commands.run` 的别名(alias)
:::

如果需要，您还可以传递选项(options)：

```js
editor.runCommand('my-command-id', { some: 'option' });
```

然后，您可以将相同的对象作为回调函数的第三个参数获取。

```js
commands.add('my-command-id', (editor, sender, options = {}) => {
  alert(`This is my command ${options.some}`);
});
```

第二个参数 `sender`，仅指示谁请求了该命令，在我们的例子中，它始终是 `editor`。

到目前为止，除了函数的通用入口点之外，没有什么特别令人兴奋的地方，但稍后我们将看到它的真正优势。

## 默认命令

GrapesJS 自带一些默认命令集，您可以通过 `editor.Commands.getAll()` 获取当前所有可用命令的列表。这将为您提供一个包含所有可用命令的对象，因此，也包括那些稍后添加的命令，例如通过插件添加的命令。您可以通过它们的命名空间(namespace) `core:*` 来识别默认命令，我们也建议在您自己的自定义命令中使用命名空间，但让我们更详细地看一下这里：

- [`core:canvas-clear`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/CanvasClear.ts) - 清空画布中的所有内容（HTML 和 CSS）
- [`core:component-delete`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/ComponentDelete.ts) - 删除一个组件
- [`core:component-enter`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/ComponentEnter.ts) - 选择所选组件的第一个子组件
- [`core:component-exit`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/ComponentExit.ts) - 选择当前所选组件的父组件
- [`core:component-next`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/ComponentNext.ts) - 选择下一个同级组件
- [`core:component-prev`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/ComponentPrev.ts) - 选择上一个同级组件
- [`core:component-outline`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/SwitchVisibility.ts) - 在组件上启用轮廓边框
- [`core:component-offset`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/ShowOffset.ts) - 启用组件偏移（外边距、内边距）
- [`core:component-select`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/SelectComponent.ts) - 启用在画布中选择组件的过程
- [`core:copy`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/CopyComponent.ts) - 复制当前选定的组件
- [`core:paste`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/PasteComponent.ts) - 粘贴复制的组件
- [`core:preview`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/Preview.ts) - 在画布中显示模板的预览
- [`core:fullscreen`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/Fullscreen.ts) - 将编辑器设置为全屏
- [`core:open-code`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/ExportTemplate.ts) - 打开一个包含模板代码的默认面板
- [`core:open-layers`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/OpenLayers.ts) - 打开一个包含图层的默认面板
- [`core:open-styles`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/OpenStyleManager.ts) - 打开一个包含样式管理器的默认面板
- [`core:open-traits`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/OpenTraitManager.ts) - 打开一个包含特征管理器的默认面板
- [`core:open-blocks`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/OpenBlocks.ts) - 打开一个包含区块的默认面板
- [`core:open-assets`](https://github.com/GrapesJS/grapesjs/blob/dev/packages/core/src/commands/view/OpenAssets.ts) - 打开一个包含资源的默认面板
- `core:undo` - 调用撤销操作
- `core:redo` - 调用重做操作
  <!-- * `core:canvas-move` -->
  <!-- * `core:component-drag` -->
  <!-- * `core:component-style-clear` -->
  <!-- tlb-clone tlb-delete tlb-move -->

## 状态命令

正如我们已经看到的，命令只是一个函数，一旦执行完毕，就不会留下任何东西。但在某些情况下，我们希望跟踪已执行的命令。GrapesJS 默认可以处理这种情况，要启用它，您只需将命令声明为一个包含 `run` 和 `stop` 方法的对象。

```js
commands.add('my-command-state', {
  run(editor) {
    alert('This command is now active');
  },
  stop(editor) {
    alert('This command is disabled');
  },
});
```

因此，如果我们现在运行 `editor.runCommand('my-command-state')`，该命令将被注册为活动状态。要检查命令的状态，您可以使用 `commands.isActive('my-command-state')`，或者您甚至可以通过 `commands.getActive()` 获取所有活动命令的列表，在我们的例子中，结果会是这样的：

```js
{
  ...
  'my-command-state': undefined
}
```

结果对象的键(key)告诉您活动的命令，值(value)是 `run` 命令的最后返回值，在我们的例子中是 `undefined`，因为我们没有返回任何东西，但这取决于您的实现来决定返回什么以及您是否真的需要它。

```js
// 让我们返回一些东西
...
run(editor) {
    alert('This command is now active');
    return {
      activated: new Date(),
    }
},
...
// 现在，您将看到来自 run 方法的对象，而不是 `undefined`
```

要禁用该命令，请使用 `editor.stopCommand` 方法，因此在我们的例子中，它将是 `editor.stopCommand('my-command-state')`。与 `runCommand` 一样，您可以将选项对象作为第二个参数传递，并在 `stop` 方法中使用它们。

一旦命令处于活动状态，如果您尝试再次运行 `editor.runCommand('my-command-state')`，您会注意到 `run` 不会触发。这种行为有助于防止多次执行激活过程，这可能导致状态不一致（例如，考虑一个计数器，它应该在 `run` 时增加，在 `stop` 时减少）。如果您需要多次运行一个命令，那么您可能正在处理一个非状态命令(stateful command)，所以尝试在没有 `stop` 方法的情况下使用它。但如果您清楚您的应用程序状态，您实际上可以使用 `editor.runCommand('my-command-state', { force: true })` 强制执行。同样的逻辑也适用于 `stopCommand` 方法。

<br/>

::: danger 警告
&nbsp;
:::

如果您在状态命令中处理 UI，请注意保持状态与逻辑的一致性。让我们以使用模态框(Modal)作为命令状态指示器为例。

```js
commands.add('my-command-modal', {
  run(editor) {
    editor.Modal.open({
      title: 'Modal example',
      content: 'My content',
    });
  },
  stop(editor) {
    editor.Modal.close();
  },
});
```

如果您运行它，关闭模态框（例如，通过单击顶部的“x”），然后尝试再次运行它，您会发现模态框不再打开。这是因为该命令仍然处于活动状态（您应该可以在 `commands.getActive()` 中看到它），要修复此问题，您必须在模态框关闭后禁用它。

```js
...
  run(editor) {
    editor.Modal.open({
      title: 'Modal example',
      content: 'My content',
    }).onceClose(() => this.stopCommand());
  },
...
```

在上面的示例中，我们使用了 Modal 模块的一些辅助方法（`onceClose`）和命令本身的（`stopCommand`），但显然，由于您的需求和特定 UI，逻辑可能会有所不同。

## 扩展

命令的另一个巨大优势是可以轻松地用另一个命令扩展或覆盖它们。
让我们看一个简单的例子：

```js
commands.add('my-command-1', (editor) => {
  alert('This is command 1');
});
```

如果您需要用另一个命令覆盖此命令，只需添加它并保持相同的 ID。

```js
commands.add('my-command-1', (editor) => {
  alert('This is command 1 overwritten');
});
```

现在让我们看看如何扩展一个命令：

```js
commands.add('my-command-2', {
  someFunction1() {
    alert('This is function 1');
  },
  someFunction2() {
    alert('This is function 2');
  },
  run() {
    this.someFunction1();
    this.someFunction2();
  },
});
```

要扩展它，只需使用 `extend` 方法并传递 ID：

```js
commands.extend('my-command-2', {
  someFunction2() {
    alert('This is function 2 extended');
  },
});
```

## 事件

命令模块还提供了一组事件(Events)，您可以使用它们来拦截命令流，以添加更多功能甚至中断它。

### 拦截 run 和 stop

使用我们之前创建的 `my-command-modal` 命令，让我们看看我们可以监听哪些事件：

```js
editor.on('command:run:my-command-modal', () => {
  console.log('After `my-command-modal` execution');
  // 例如，您可以向模态框添加额外内容
  const modalContent = editor.Modal.getContentEl();
  modalContent.insertAdjacentHTML('beforeEnd', '<div>Some content</div>');
});
editor.on('command:run:before:my-command-modal', () => {
  console.log('Before `my-command-modal` execution');
});
// 对于状态命令
editor.on('command:stop:my-command-modal', () => {
  console.log('After `my-command-modal` is stopped');
});
editor.on('command:stop:before:my-command-modal:before', () => {
  console.log('Before `my-command-modal` is stopped');
});
```

如果需要，您还可以监听所有命令：

```js
editor.on('command:run', (commandId) => {
  console.log('Run', commandId);
});

editor.on('command:stop', (commandId) => {
  console.log('Stop', commandId);
});
```

### 中断命令流

有时，您可能需要根据某些条件中断现有命令的执行。在这种情况下，您必须使用 `command:run:before:{COMMAND-ID}` 事件并将 abort 选项设置为 `true`。

```js
const condition = 1;

editor.on('command:run:before:my-command-modal', (options) => {
  if (condition) {
    options.abort = true;
    console.log('Prevent `my-command-modal` from execution');
  }
});
```

## 总结

命令模块非常简单，但如果使用得当，功能也非常强大。因此，如果您正在为 GrapesJS 创建插件，请尽可能多地使用命令，这将为您的逻辑带来更高的可复用性(reusability)和控制能力。