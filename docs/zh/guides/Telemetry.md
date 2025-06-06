---
title: GrapesJS 遥测
---

# GrapesJS 遥测

我们收集并使用数据来改进 GrapesJS。本页面解释了我们收集哪些数据以及如何使用这些数据。

## 我们收集哪些数据

我们收集以下数据：

- **域名(Domain)**：GrapesJS 使用所在网站的域名。
- **版本(Version)**：使用的 GrapesJS 版本。
- **时间戳(Timestamp)**：编辑器(Editor)加载的时间。

## 我们如何使用数据

我们使用数据来：

- **改进 GrapesJS**：我们使用数据来改进 GrapesJS。例如，我们使用数据来识别缺陷(Bug)并修复它们。
- **分析使用情况**：我们使用数据来分析 GrapesJS 的使用方式。例如，我们使用数据来了解哪些功能(Feature)最常被使用。
- **提供支持**：我们使用数据为用户提供支持。例如，我们使用数据来了解用户如何与 GrapesJS 互动。

## 如何选择退出

您可以在初始化 GrapesJS 时，通过将 `telemetry` 选项(Option)设置为 `false` 来选择退出数据收集：

```js
const editor = grapesjs.init({
  // ...
  telemetry: false,
});
```