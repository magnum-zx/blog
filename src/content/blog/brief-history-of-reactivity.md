---
author: MIŠKO HEVERY
pubDatetime: 2023-04-19T16:07:52.000+08:00
title: 响应式系统的前世今生
postSlug: brief-history-of-reactivity
featured: true
ogImage: ""
tags:
  - reactivity
description: "本文将简要介绍响应式系统的前世今生。响应式已经是现代前端框架中最常见的特性了，作为前端小白很有必要了解一下响应式是什么时候出现的，以及它解决了什么问题。"
---

翻译自https://www.builder.io/blog/history-of-reactivity

本文并非权威的`响应式`历史，而是作者的个人经历和感受

# Flex

我的经历从`Macromedia Flex`开始说起，后来被 Adobe 收购了。

`Flex`是一个基于`Flash`的`ActionScript`框架。`ActionScript`是一种与`JavaScript`非常类似的语言，但是`ActionScript`通过注释来让编译器包装字段用于`事件订阅`，大概是类似这样：

```js
class MyComponent {
  [Bindable] public var name: String;
}
```

`[Bindable]`注释创建一个`getter/setter`，可以触发属性变化的`事件`。然后你就可以监听属性的变化。`Flex`使用`.mxml`模板文件来渲染 UI。如果属性通过监听属性的变化而变化，在`.mxml`中的任何数据绑定都是细粒度的响应。

```xml
<mx:Application xmlns:mx="http://www.adobe.com/2006/mxml">
  <mx:MyComponent>
    <mx:Label text="{name}"/></mx:Label>
  </mx:MyComponent>
</mx:Applicatio>
```

我对`Flex`是`Reactivity(响应式)`的起源的说法存疑，但是确实是我第一次接触。

`Reactivity`实际上对`Flex`来说是一个痛苦，因为它很容易制造出`update storms`。`update storms`是指当一个属性发生变化时触发`许多`其他属性发生变化，而这些变化的属性会`进一步`触发其他属性改变，这时就会进入一个无尽的循环当中。由于`Flex`没有将`update property`和`update UI`区分开，以至于会出现频繁的`UI抖动`，也就是会对`中间值`渲染。

虽然经过事后分析，我明白是哪个（架构上的）决策导致这种欠佳的结果。但是，在当时，我是不清楚的，并且我带着对响应式系统的怀疑离开了。

# AngularJS

`AngularJS`最初的目的就是去`拓展HTML`的语法，为了让那些非开发者（设计师）来建立简单的 web 应用。这就是 AngularJS 以`html`标签为结束符号的原因。因为 AngularJS 是对 html 的拓展，所以需要将其绑定到任意一个`JavaScript对象`上。在当时，`Proxy`、`getter/setters`、`Object.observe()`都还不是备选项。所以唯一可行的方案是做一个`dirty checking` 脏检查。

每当浏览器`执行异步任务`，dirty checking 将通过`读取绑定到模板上的所有属性`进行工作。

```html
<!DOCTYPE html>
<html ng-app>
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
  </head>
  <body>
    <div>
      <label>Name:</label>
      <input type="text" ng-model="yourName" placeholder="Enter a name here" />
      <hr />
      <h1>Hello {{yourName}}!</h1>
    </div>
  </body>
</html>
```

这种方法的好处是，`任何JavaScript对象`都可以在模板中作为`数据绑定源`，并且 update 能正常进行。

缺点是，JS 必须执行每个 update，并且`AngularJS`无法知道变化是什么时候发生的，所以`AngularJS`会比理论上需要更频繁的`dirty checking`。

因为 AngularJS 仅与 JS 对象一起工作，并且是 html 语法的拓展，所以 AngularJS 从来`没有提供`任何类型的`状态管理`。

# React

`React`是在 AngularJS 之后出现的，并且做了很多提升。

首先，React 引入了 `setState()`，允许 React 知道什么时候应该执行`vDOM`的`dirty-checking`。这样的好处是，不像 AngularJS 会在`每次异步任务`执行 dirty-checking，React 只有在开发者`"告诉"`它进行 dirty-checking 时才会执行。所以虽然 React vDOM dirty-checking 在计算上要比 Angular 更昂贵，但是执行次数会更少。

```js
function Counter() {
  const [count, setCount] = useState();
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

其次，React 引入了从`parent`到`child`的`严格数据流`。这些是迈向框架认可的`状态管理`的第一步，而 AngularJS 没有。

# 粗粒度响应式

React 和 AngularJS 都是粗粒度的响应式。这意味着数据的变化会触发大量 JavaScript 的执行。该框架最终会将所有更改协调到 UI 中。这意味着快速变化的属性（例如动画）会导致性能问题。

# 细粒度响应式

上述问题的解决方案是细粒度的反应性，其中状态更改仅更新状态绑定到的 UI 部分。难点是知道如何以具有好的 `DX` 的方式监听属性变化。（DX：`Developer experience`，describes how developers feel about a system while working on it）

# Backbone.js

Backbone 是早于 AngularJS 的一种框架，它是具有细粒度响应性的，但是语法非常繁琐。

```js
var MyModel = Backbone.Model.extend({
  initialize: function () {
    // Listen to changes on itself.
    this.on("change:name", this.onAsdChange);
  },
  onNameChange: function (model, value) {
    console.log("Model: Name was changed to:", value);
  },
});
var myModel = new MyModel();
myModel.set("name", "something");
```

我认为冗长的语法是 Backbone 被像 AngularJS 和后来的 React 接管的部分原因。因为开发人员可以简单地使用点符号来访问和设置状态，而不是一组复杂的函数回调。在这些更新的框架中开发应用程序更加容易和快捷。

# Knockout

Knockout 是一个与 AngularJS 同时出现的框架。虽然我没有使用过，但是以我的理解，但是它同样被`update storms`困扰。虽然，Knockout 是对 Backbone.js 的改进，但是它使用`可观测属性`依旧非常笨重。这也是为什么我认为开发者更喜欢“点符号”框架（AngularJS 和 React 等）的原因。

但是 Knockout 有一个有趣的创新——`计算属性`，也许在之前就出现了，但确实是我第一次听说。计算属性会根据输入数据自动创建一个订阅。

```js
var ViewModel = function (first, last) {
  this.firstName = ko.observable(first);
  this.lastName = ko.observable(last);
  this.fullName = ko.pureComputed(function () {
    // Knockout自动追踪依赖
    // 知道fullName依赖firstName，lastName
    // 因为在计算fullName时会调用他们
    return this.firstName() + " " + this.lastName();
  }, this);
};
```

当`ko.pureComputed()`调用`this.firstName()`时，该值会隐式创建一个订阅。这是通过`ko.pureComputed()`设置一个全局变量实现的，允许`this.firstName()`与`ko.pureComputed()`进行通信，并且无需开发者做任何额外的工作就可以将订阅信息传递给`ko.pureComputed()`。

# Svelte

Svelte 使用一个`编译器`整合了`响应式`。这么做的好处是，语法可以更加灵活，不必限制于 JavaScript。
Svelte 具有非常自然的响应式组件的语法。但是 Svelte 无法编译所有文件，只能处理`.svelte`文件。如果你想在未编译文件中使用响应式，Svelte 提供了一个`store API`，它没有编译文件的响应式特性，而是要求使用`subscribe`和`unsubscribe`来进行明确的注册。

```js
const count = writable(0);
const unsubscribe = count.subscribe(value => {
  countValue = value;
});
```

我认为用两种不同的方式做同一件事并不理想，因为你必须在头脑中保留两种不同的心智模型并在它们之间进行选择。因此，更加推荐只使用一种方法。

# RxJS

RxJS 是一个响应式库，`没有与任何其他底层渲染系统进行捆绑`。这似乎是一个优势，但是它有一个缺陷。导航到一个新页面时，需要将旧 UI 清除，再构建新的 UI。对于 RxJS，这意味着需要进行许多`取消订阅`和`订阅`。这种额外的工作意味着粗粒度的响应式系统在这种情况下更快，因为清除只是丢弃 UI（垃圾收集），而构建不需要任何`注册/分配`监听器。我们需要的是一种`批量`取消订阅/订阅的方法。

```js
const observable1 = interval(400);
const observable2 = interval(300);
const subscription = observable1.subscribe(x =>
  console.log("[first](https://rxjs.dev/api/index/function/first): " + x)
);
const childSubscription = observable2.subscribe(x =>
  console.log("second: " + x)
);
subscription.add(childSubscription);
setTimeout(() => {
  // Unsubscribes BOTH subscription and childSubscription
  subscription.unsubscribe();
}, 1000);
```

# Vue and Mobx

大约在同一时间，Vue 和 MobX 都开始尝试基于`Proxy`的响应式。

`Proxy` 的优点是你可以使用点符号语法（开发者更喜欢），并且可以使用与 `Knockout` 相同的技巧来创建`自动订阅`——这是一个巨大的胜利！

```html
<template>
  <button @click="count = count + 1">{{ count }}</button>
</template>

<script setup>
  import { ref } from "vue";

  const count = ref(1);
</script>
```

在上面的示例中，模板（template）通过在渲染期间读取`count`，自动创建对`count`的订阅。这期间，开发人员不需要额外的工作。

# SolidJS

`Proxy` 的缺点是不能传递对 `getter/setter` 的引用。您可以传递整个 `Proxy` 或 `属性的值`，但你可以从 `store` 中剥离一个 `getter` 并传递它。以这个问题为例。

```js
function App() {
  const state = createStateProxy({ count: 1 });
  return (
    <>
      <button onClick={() => state.count++}>+1</button>\
      <Wrapper value={state.count} />
    </>
  );
}

function Wrapper(props) {
  return <Display value={state.value} />;
}
function Display(props) {
  return <span>Count: {props.value}</span>;
}
```

当我们读取 `state.count` 时，得到的数字是初始值的，并且不再可观察。这意味着 `Middle` 和 `Child` 都需要在 `state.count` 更改时重新渲染。我们失去了细粒度的反应性。理想情况下，只有 `Count:` 应该被更新。我们需要一种方法，传递对值的引用而不是值本身。

# Enter Signals

`Signal` 允许你引用该值，还允许你引用该值的 `getter/setter`。所以你可以用 `Signal` 解决上面的问题：

```js
function App() {
  const [count, setCount] = createSignal(1);
  return (
    <>
      <button onClick={() => setCount(count() + 1)}>+1</button>
      <Wrapper value={count} />
    </>
  );
}
function Wrapper(props: { value: Accessor<number> }) {
  return <Display value={props.value} />;
}
function Display(props: { value: Accessor<number> }) {
  return <span>Count: {props.value}</span>;
}
```

这种解决方案的好处是，我们没有传递`值`而是传递了一个`Accessor（一个getter）`，这意味着当`count`值改变时，我们不必通过`Wrapper`和`Display`，而是直接对`DOM`进行一个`更新`。在工作方式上与`Knockout`非常相似，但是在语法上与`Vue/Mobx`非常相似.

但是这存在一个 DX 问题，作为组件的使用者，假设我们想要绑定一个常量

```js
<Display value={10} />
```

由于`Display`被定义为`Accessor（访问器）`，所以这是不起作用的。

```js
function Display(props: {value: Accessor<number>});
```

这很不幸，因为组件的作者现在定义了消费者是否可以发送 `getter` 或`值`。无论作者选择什么，总会有未涵盖的用例。这两件事都是合理的。

```js
<Display value={10}/>
<Display value={createSignal(10)}/>
```

以上是使用 `Display` 组件的两种有效方式，但它们都不是对的！我们需要的是一种将类型声明为`原始类型`但同时适用于`原始类型`和`访问器`的方法。

```js
function App() {
  const [count, setCount] = createSignal(1);
  return (
    <>
      <button onClick={() => setCount(count() + 1)}>+1</button>
      <Wrapper value={count()} />
    </>
  );
}
function Wrapper(props: { value: number }) {
  return <Display value={props.value} />;
}
function Display(props: { value: number }) {
  return <span>Count: {props.value}</span>;
}
```

请注意，现在我们声明的是`数字`，而不是`访问器`。这意味着这段代码可以正常工作。

```js
<Display value={10}/>
<Display value={createSignal(10)()}/> // Notice the extra ()
```

但这是否意味着我们现在已经打破了反应性？答案是肯定的，除非我们可以让编译器执行一个技巧来恢复我们的反应性。问题是这一行：

```js
<Wrapper value={count()} />
```

调用 `count()` 将`访问器`转换为`原始属性`并创建订阅。所以编译器做了这个把戏。

```js
Wrapper({
  get value() {
    return count();
  },
});
```

当 `count()` 作为一个 `prop` 传递给子组件时，通过将其包装在一个`getter`中，使得编译器可以延长 `count()` 的执行时机，直到 `DOM` 需要使用这个值。

这样的好处是：

- 简洁的语法
- 自动订阅与取消订阅
- 组件接口不必 `原始类型` 和 `Accessor` 二选一
- 即使开发人员将 `Accessor` 转换为 `原始类型`，响应式仍然有效。

# 响应式与渲染

让我们想象一个场景，一个带有购买按钮和购物车的页面。
![](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2F462fa726ceaa45bbac48c9a1a6e7714b?format=webp&width=2000)

在上面的示例中，有一系列组件组成的树。用户可能的行为是点击购买按钮，这时需要更新购物车。存在两种不同的结果。

在粗粒度的响应式系统中，它是这样的：
![](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2Fa04cf84ae4864600874910b1349ce9fb?format=webp&width=2000)

我们必须找到 `Buy` 和 `Cart` 组件之间的公共根，因为这是最有可能存放状态的地方。然后，在更改状态时，与该状态关联的树必须重新渲染。使用记忆化（memoization），可以将树修剪成两条最小路径，如上所示。许多代码仍然需要执行，尤其是当应用程序变得越来越复杂时。

在一个细粒度的响应式系统中，它看起来像这样：
![](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2F958e7be34d234dcfa7c90ca17e8d3858?format=webp&width=2000)

可以观察到只有目标 `Cart` 组件需要执行。无需查看声明状态的位置或共同祖先是什么。也不必担心数据记忆化并而去剪枝。细粒度响应式系统的好处在于，无需开发人员的任何努力，运行时只执行最少量的代码！

细粒度响应式系统的精度使其非常适合延迟执行代码，因为系统只需要执行状态的侦听器（在我们的例子中是 `Cart` ）。

但是细粒度的响应式系统有一个意想不到的极端情况。为了让系统至少建立一次响应式图，必须执行所有组件才能了解这些关系！一旦建立起来，该系统就可以进行`外科手术`。这是初始执行的样子：
![](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2F5523a7449125407c910a564412acaac1?format=webp&width=2000)

这个问题就是我们想 lazy 下载和执行，但是响应图的初始化强制应用程序完全执行（下载）

# Qwik

这就是 Qwik 的用武之地。Qwik 是细粒度的反应式，类似于 SolidJS，这意味着状态的变化会直接更新 DOM。（在某些极端情况下，Qwik 可能需要执行整个组件。）但是 Qwik 有一个小窍门。还记得细粒度的反应性要求所有组件至少执行一次以创建反应性图吗？Qwik 利用了组件已经在 `SSR/SSG` 期间在服务器上执行的事实。Qwik 可以将该图序列化为 HTML。这允许客户端完全跳过最初的“执行全部组件以了解响应图”。我们称之为`可恢复性（resumability）`。因为组件不在客户端执行或下载，所以 Qwik 的好处是应用程序的即时启动。应用程序运行后，反应性就像外科手术一样，就像 `SolidJS` 一样。

# 总结

作为一个行业，我们经历了多次反应性迭代。我们从粗粒度反应系统开始，因为它们对开发人员更友好。但我们一直关注拥有细粒度反应系统的价值，并且一直在努力解决这个问题一段时间。最新一代框架解决了开发者体验、更新风暴、状态管理等诸多问题。我不知道您接下来会选择哪个框架，但我敢打赌它会是细粒度的响应式框架！
