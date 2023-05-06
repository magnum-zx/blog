---
author: magnum-zx
pubDatetime: 2023-05-06T20:07:52.000+08:00
title: Astro 2.4 is coming！
postSlug: astro-240
featured: true
ogImage: https://astro.build/_astro/blog-hero-2_4-release.dc6af8fa.jpg
tags:
  - astro
description: "2023年5月4日 Astro 发布了2.4版本，来看看又新增了哪些特性"
---

# Astro 2.4

[Astro](https://astro.build/) 框架可以用于快速构建一个以内容为中心的网站。在 Astro 中可以使用你喜欢的任意 UI 组件，不限于 Vue，React，Solid 等。你可以查看 [Astro docs](https://docs.astro.build/en/getting-started/) 来学习如何快速构建你的网站，以及 Astro 的特性。

Astro 在 2.4 版本新增以下特性：

- `更强的 CSS 作用域`：新增了一个配置项 `scopedStyleStrategy`，允许开发者为 Astro 组件的作用域样式配置更高的特异性。
- `改进的 <Code/> 组件`：升级了 Shiki 并支持内联。
- `站点地图（sitemap）的 SSR 支持`：已知路由现在将在构建过程中内置到站点地图中。
- `中间件（实验性）`：定义在页面组件和端点之前运行的代码，以拦截请求和修改响应。
- `CSS 内联（实验性）`：允许你配置何时应通过 <style> 标记内联样式表。

## `scopedStyleStrategy`

Astro 使用 [`:where`](https://docs.astro.build/en/guides/styling/#scoped-styles) 伪选择器用于作用域内的 Astro 组件样式。这种方法意味着你的组件样式具有相同的特异性，就好像它们是用纯 CSS 编写的一样。缺点是不能保证组件样式总是会覆盖全局样式，而在 2.4 版本中 Astro 对此进行了改进。

Astro 新增了 `scopedStyleStrategy` 选项，可用于启用高特异性策略，该策略使用基于 `class` 的选择器：

```js
// astro.config.mjs
import { defineConfig } from "astro/config";

export default defineConfig({
  scopedStyleStrategy: "class",
});
```

此策略可能会用作 Astro 3.0 中的默认策略。

## 改进的 `<Code/>` 组件

[`<Code/>`](https://docs.astro.build/en/reference/api-reference/#code) 组件有一些新的改进：

- 升级至 Shiki 0.14 - Astro 之前使用的是 Shiki 0.11。这次升级带来了许多改进，最显着的是新主题和语言支持。
- 新的 `inline` 属性避免添加包装 `<pre>`。如果你希望在段落中内联突出显示语法，这将很有用。

## `站点地图（sitemap）`的 SSR 支持

对`@astrojs/sitemaps` 包进行了更新，增加了对服务器端渲染（SSR）的支持。以前，只有在使用 `output: 'static'` 时才支持站点地图，因为动态路由在构建时是未知的。

新版本现在也可以与 `output: 'server'` 一起使用，并将仅为静态路由输出站点地图，动态路由被排除在外。无需更改配置即可启用此功能。

## `中间件`（实验性）

Astro 的中间件支持的早期预览已发布于 Astro 2.4。中间件将允许你拦截请求和响应并动态注入行为。要启用实验性中间件支持，请在 `astro.config.mjs` 中添加 `experimental` 配置：

```js
// astro.config.mjs
import { defineConfig } from "astro/config";

export default defineConfig({
  experimental: {
    middleware: true,
  },
});
```

除了拦截请求和响应之外，使用中间件还可以改变所有 Astro 组件和 API 端点中可用的`local`对象。这是身份验证中间件的示例。使用以下内容创建 src/middlware.ts 文件：

```js
// src/middlware.ts
import type { MiddlewareResponseHandler } from "astro/middleware";

const auth: MiddlewareResponseHandler = async ({ cookies, locals }, next) => {
  if (!cookies.has("sid")) {
    return new Response(null, {
      status: 405, // Not allowed
    });
  }

  let sessionId = cookies.get("sid");
  let user = await getUserFromSession(sessionId);
  if (!user) {
    return new Response(null, {
      status: 405, // Not allowed
    });
  }

  locals.user = user;
  return next();
};

export { auth as onRequest };
```

然后可以通过 Astro 组件访问：

```js
// a certain .astro file
---
const { user } = Astro.locals
---

<h1>Hello {user.name}</h1>
```

此功能仍处于试验阶段，想了解更多，请参阅[中间件指南](https://docs.astro.build/en/guides/middleware/)。

## `CSS 内联`（实验性）

现在可以通过启用 `experimental` 的配置项 `inlineStylesheets`，将样式表设为 `<style>` 标签内联模式：

```js
// astro.config.mjs
import { defineConfig } from "astro/config";

export default defineConfig({
  experimental: {
    inlineStylesheets: "auto",
  },
});
```

使用 `“auto”` 选项，低于 `vite.assetInlineLimit` 的样式表将作为 `<style>` 标签添加，而不是通过 `<link>` 获取。

也可以将该配置项设为 `“always”` 以强制内联。目前，默认值为 `“never”`，但在 Astro 3.0 中可能会更改为 `“auto”`。

## 参考

- [https://astro.build/blog/astro-240/](https://astro.build/blog/astro-240/)
