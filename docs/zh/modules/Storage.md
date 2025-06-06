---
title: 存储管理器
---

# 存储管理器

存储管理器 (Storage Manager) 是一个内置模块 (module)，它允许您持久化 (persistence) 您的项目数据 (project data)。

::: warning
本指南需要 GrapesJS v0.19.\* 或更高版本。
:::

::: tip
需要更强大和可定制的存储选项吗？[Grapes Studio SDK 为您提供了解决方案。](https://app.grapesjs.com/docs-sdk/configuration/projects?utm_source=grapesjs-docs&utm_medium=tip#storage)
:::

[[toc]]

## 配置

要更改默认配置 (configuration)，您需要将 `storageManager` 属性与主配置对象一起传递。

```js
const editor = grapesjs.init({
  ...
  // 默认配置
  storageManager: {
    type: 'local', // 存储类型。可选值：local | remote
    autosave: true, // 自动保存数据
    autoload: true, // 初始化时自动加载存储的数据
    stepsBeforeSave: 1, // 如果启用了自动保存，表示在触发存储方法之前需要多少次更改
    // ...
    // 默认存储选项
    options: {
      local: {/* ... */},
      remote: {/* ... */},
    }
  },
});
```

如果您不需要任何持久化，可以这样禁用该模块：

```js
const editor = grapesjs.init({
  ...
  storageManager: false,
});
```

在此处查看可用选项的完整列表：[存储管理器配置](https://github.com/GrapesJS/grapesjs/blob/master/src/storage_manager/config/config.ts)

## 项目数据

项目数据是一个 JSON 对象 (JSON object)，包含有关编辑器 (editor) 中项目的所有必要信息（样式 (styles)、页面 (pages) 等），并且是存储管理器方法中用于存储和加载项目（本地或远程数据库/文件）的数据。

::: tip
您可以通过以下方式获取当前数据状态并手动加载：

```js
// 获取当前项目数据
const projectData = editor.getProjectData();
// ...
// 加载项目数据
editor.loadProjectData(projectData);
```

:::

::: danger
您应该仅依赖 JSON 项目数据来正确加载编辑器中的项目。

编辑器能够解析和使用 HTML/CSS 代码，您可以将其用作项目初始化的一部分，但绝不能将其作为加载项目时的持久化层，因为许多信息可能会丢失。
:::

<!-- If necessary, the JSON can be also enriched with your data of choice, but as the data schema might differ in time we highly recommend to store them in your domain specific keys-->

## 存储策略

每当更改量 (`editor.getDirtyCount()`) 达到保存前步数 (`editor.Storage.getStepsBeforeSave()`) 时，项目数据会自动存储。任何成功的数据存储都会重置更改计数器 (`editor.clearDirtyCount()`)。

::: tip
必要时，您始终可以手动触发存储/加载。

```js
// 存储数据
const storedProjectData = await editor.store();

// 加载数据
const loadedProjectData = await editor.load();
```

:::

## 设置本地存储

默认情况下，GrapesJS 使用内置的 `local` 存储将数据保存在本地，该存储利用了 [localStorage API]。

对于本地存储，您可能唯一关心的选项是用于存储数据的 `key` (键)。如果用户在您的应用程序中加载不同的项目，您可能需要通过项目的 ID（此处的 ID 旨在成为您应用程序域的一部分）来区分本地存储。

```js
// 获取您的项目 ID（例如，从路由中获取）
const projectId = getProjectId();

const editor = grapesjs.init({
  ...
  storageManager: {
    type: 'local',
    options: {
      local: { key: `gjsProject-${projectId}` }
    }
  },
});
```

## 设置远程存储

项目数据通常可能远程保存在您的服务器（数据库、文件等）上，因此您需要设置服务器端 (server-side) API 调用以存储/加载项目数据。

为简单起见，我们可以依赖 [json-server] 来设置一个伪 REST API (REST API) 服务器。

```sh
mkdir my-server
cd my-server
npm init
npm i json-server
echo '{"projects": [ {"id": 1, "data": {"assets": [], "styles": [], "pages": [{"component": "<div>Initial content</div>"}]} } ]}' > db.json
npx json-server --watch db.json
```

这将启动一个本地服务器，其中一个项目可在 `http://localhost:3000/projects/1` 上访问。数据将在 `db.json` 文件中更新。

以下是如何在 GrapesJS 中配置 `remote` 存储的示例。

```js
const projectID = 1;
const projectEndpoint = `http://localhost:3000/projects/${projectID}`;

const editor = grapesjs.init({
  ...
  storageManager: {
    type: 'remote',
    stepsBeforeSave: 3,
    options: {
      remote: {
        urlLoad: projectEndpoint,
        urlStore: projectEndpoint,
        // `remote` 存储在存储数据时使用 POST 方法，
        // 但 json-server API 需要 PATCH。
        fetchOptions: opts => (opts.method === 'POST' ?  { method: 'PATCH' } : {}),
        // 由于 API 以 `{id: 1, data: projectData }` 格式存储项目，
        // 我们必须在存储前正确更新请求体，并从响应结果中提取项目数据。
        onStore: data => ({ id: projectID, data }),
        onLoad: result => result.data,
      }
    }
  }
});
```

::: danger
请确保在您的服务器 API 上正确配置 [CORS](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)。[json-server] 不适用于生产环境，因此会自动启用所有 CORS 策略。
:::

<div id="setup-the-server"></div>

### 服务器设置

服务器配置可能因情况而异，因此通常由您来决定如何正确配置它。
默认的远程存储遵循简单的 REST API 方法，项目数据以 JSON (`Content-Type: application/json`) 格式交换。

-   在 **加载** (`GET`<sup>获取</sup> 方法) 时，JSON 项目数据应直接在响应中返回。如上例所示，如果响应包含其他元数据，您可以使用 `options.remote.onLoad` 来提取项目数据。
-   在 **存储** (`POST`<sup>提交</sup> 方法) 时，编辑器不期望任何特定结果，只需要服务器的有效响应（状态码 `200`）。

<!-- ## Store and load templates

Even without a fully working endpoint, you can see what is sent from the editor by triggering the store and looking in the network panel of the inspector. GrapesJS sends mainly 4 types of parameters and it prefixes them with the `gjs-` key (you can disable it via `storageManager.id`). From the parameters, you will get the final result in 'gjs-html' and 'gjs-css' and this is what actually your end-users will gonna see on the final template/page. The other two, 'gjs-components' and 'gjs-styles', are a JSON representation of your template and therefore those should be used for the template editing. **So be careful**, GrapesJS is able to start from any HTML/CSS but use this approach only for importing already existent HTML templates, once the user starts editing, rely always on JSON objects because the HTML doesn't contain information about your components. You can achieve it in a pretty straightforward way and if you load your page by server-side you don't even need to load asynchronously your data (so you can turn off the `autoload`).

```js
// Lets say, for instance, you start with your already defined HTML template and you'd like to
// import it on fly for the user
const LandingPage = {
  html: `<div>...</div>`,
  css: null,
  components: null,
  style: null,
};
// ...
const editor = grapesjs.init({
  ...
  // If set to true, then the content within the wrapper element overrides the following config,
  fromElement: false,
  // The `components` accepts HTML string or a JSON of components
  // Here, at first, we check and use components if are already defined, otherwise
  // the HTML string gonna be used
  components: LandingPage.components || LandingPage.html,
  // We might want to make the same check for styles
  style: LandingPage.style || LandingPage.css,
  // As we already initialize the editor with the template we can skip the `autoload`
  storageManager: {
    ...
    autoload: false,
  },
});
``` -->

## 存储 API

存储管理器模块也有其自身的 [API 集合](/api/storage_manager.html)，允许您扩展和添加新功能。

### 定义新存储

定义新存储只需将两个异步方法传递给 `editor.Storage.add` API。为简单起见，下面的示例通过使用 [sessionStorage API](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/sessionStorage) 来演示定义 `session` 存储的 API 用法。

```js
const sessionStoragePlugin = (editor) => {
  // 由于 sessionStorage 不是异步 API，
  // `async` 关键字可以省略
  editor.Storage.add('session', {
    async load(options = {}) {
      return JSON.parse(sessionStorage.getItem(options.key));
    },

    async store(data, options = {}) {
      sessionStorage.setItem(options.key, JSON.stringify(data));
    }
  });
};

const editor = grapesjs.init({
  ...
  plugins: [sessionStoragePlugin],
  storageManager: {
    type: 'session',
    options: {
      session: { key: 'myKey' }
    }
  },
});
```

### 扩展存储

除其他需求外，您可能需要使用现有存储将它们组合成更复杂的用例。
例如，假设我们想将本地存储和远程存储混合到另一个存储中。它看起来会是这样：

```js
const { Storage } = editor;

Storage.add('remote-local', {
  async store(data) {
    const remoteStorage = Storage.get('remote');

    try {
      await remoteStorage.store(data, Storage.getStorageOptions('remote'));
    } catch (err) {
      // 远程错误时，将数据存储在本地
      const localStorage = Storage.get('local');
      await localStorage.store(data, Storage.getStorageOptions('local'));
    }
  },

  async load() {
    // ...
  },
});
```

### 替换存储

您还可以通过在 `Storage.add` 方法中传递相同的存储类型来替换已定义的存储。例如，您可以将依赖 [localStorage API] 的默认 `local` 存储替换为更具可扩展性的方案，如 [IndexedDB API]。

您也可能已经在使用一些 HTTP 客户端库（例如 [axios](https://github.com/axios/axios)），它为您处理应用程序中的所有必要 HTTP 标头（CSRF 令牌、会话数据等），因此您可以简单地用您选择的实现替换默认的 `remote` 存储，而无需关心默认配置。

```js
editor.Storage.add('remote', {
  async load() {
    return await axios.get(`projects/${projectId}`);
  },

  async store(data) {
    return await axios.patch(`projects/${projectId}`, { data });
  },
});
```

<!-- ### Examples

Here you can find some of the plugins extending the Storage Manager

* [grapesjs-indexeddb] - Storage wrapper for IndexedDB
* [grapesjs-firestore] - Storage wrapper for [Cloud Firestore](https://firebase.google.com/docs/firestore) -->

## 常见用例

### 跳过初始加载

如果您使用 `remote` 存储，您可能希望通过立即加载项目来跳过初始的远程调用。在这种情况下，您可以在初始化时指定 `projectData`。

```js
// 在初始化编辑器之前获取数据（例如，在服务器端打印）。
const projectData = {...};
// ...
grapesjs.init({
  // ...
  // 如果 projectData 未定义，我们可能希望为项目加载一些初始数据。
  projectData: projectData || {
    pages: [
        {
          component: `
            <div class="test">Initial content</div>
            <style>.test { color: red }</style>
          `
        }
    ]
  },
  storageManager: {
    type: 'remote',
    // ...
  },
})
```

如果定义了 `projectData`，初始存储加载将自动跳过。

### 项目数据中的 HTML 代码

项目数据不包含页面的 HTML/CSS，因为其主要目的是仅收集绝对必要的信息。
如果您有严格的要求，需要在存储项目数据时执行其他逻辑（例如，将 HTML/CSS 结果部署到预演环境），您可以使用远程配置中的 `onStore` 选项来丰富您的远程调用。

```js
grapesjs.init({
  // ...
  storageManager: {
    type: 'remote',
    options: {
      remote: {
        // 丰富存储调用
        onStore: (data, editor) => {
          const pagesHtml = editor.Pages.getAll().map((page) => {
            const component = page.getMainComponent();
            return {
              html: editor.getHtml({ component }),
              css: editor.getCss({ component }),
            };
          });
          return { id: projectID, data, pagesHtml };
        },
        // 如果在加载时，您返回的是与上述相同的 JSON...
        onLoad: (result) => result.data,
      },
    },
  },
});
```

### 内联项目数据

在某些情况下，编辑器可能未连接到任何存储，而只是在表单中的输入字段中读取/写入数据。对于这种情况，您可以创建一个内联存储。

```html
<form id="my-form">
  <input id="project-html" type="hidden" />
  <input id="project-data" type="hidden" value='{"pages": [{"component": "<div>Initial content</div>"}]}' />
  <div id="gjs"></div>
  <button type="submit">Submit<sup>提交</sup></button>
</form>

<script>
  // 提交时显示数据
  document.getElementById('my-form').addEventListener('submit', (event) => {
    event.preventDefault();
    const projectDataEl = document.getElementById('project-data');
    const projectHtmlEl = document.getElementById('project-html');
    alert(`HTML: ${projectHtmlEl.value}\n------\nDATA: ${projectDataEl.value}`);
  });

  // 内联存储
  const inlineStorage = (editor) => {
    const projectDataEl = document.getElementById('project-data');
    const projectHtmlEl = document.getElementById('project-html');

    editor.Storage.add('inline', {
      load() {
        return JSON.parse(projectDataEl.value || '{}');
      },
      store(data) {
        const component = editor.Pages.getSelected().getMainComponent();
        projectDataEl.value = JSON.stringify(data);
        projectHtmlEl.value = `<html>
          <head>
            <style>${editor.getCss({ component })}</style>
          </head>
          ${editor.getHtml({ component })}
        <html>`;
      },
    });
  };

  // 初始化编辑器
  grapesjs.init({
    container: '#gjs',
    height: '500px',
    plugins: [inlineStorage],
    storageManager: { type: 'inline' },
  });
</script>
```

在上面的示例中，我们依赖两个隐藏的输入字段，一个用于包含项目数据，另一个用于 HTML/CSS。

## 事件

有关可用事件 (events) 的完整列表，您可以在[此处](/api/storage_manager.html#available-events)查看。

[grapesjs-indexeddb]: https://github.com/GrapesJS/storage-indexeddb
[grapesjs-firestore]: https://github.com/GrapesJS/storage-firestore
[localStorage API]: https://developer.mozilla.org/zh-CN/docs/Web/API/Window/localStorage
[IndexedDB API]: https://developer.mozilla.org/zh-CN/docs/Web/API/IndexedDB_API
[json-server]: https://github.com/typicode/json-server