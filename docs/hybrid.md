---
layout: default
title: 混合开发
nav_order: 5
---

# 混合开发

## Flutter

[Flutter 面试题整理 - 掘金](https://juejin.cn/post/7067828374344826887)

### 渲染流程

1. **组件的创建**：在 Flutter 中，你可以通过创建 Widget（小部件）来构建应用界面。Widget 是 UI 构建块，可以表示一切，从简单的文本和图像到复杂的布局和动画。当你创建一个 Widget，实际上是在构建一个描述界面的声明式树。

2. **Widget 树构建**：Flutter 使用 Widget 树来表示应用界面。当应用启动时，Flutter 引擎会调用你的 main() 方法，然后你可以通过构建一棵 Widget 树来描述整个应用的界面布局。Widget 树是声明式的，这意味着你只需要描述你想要的界面是什么样子，而不需要关心底层的 UI 布局细节。

3. **Diffing 和重建**：Flutter 使用一种称为“差异引擎”的技术来优化 UI 更新。当 Widget 树发生变化时，Flutter 引擎会通过比较前后两棵树的差异来确定需要更新的部分。然后，Flutter 会重新构建需要更新的部分，并将这些更改应用到底层渲染引擎中。

4. **布局和绘制**：Flutter 使用 Skia 图形引擎进行绘制，它能够在不同平台上实现高性能的绘制操作。在布局和绘制阶段，Flutter 会根据 Widget 树的布局信息计算每个 Widget 的位置和大小，然后将它们绘制到屏幕上。

5. **事件处理**：Flutter 通过 Widget 树中的 GestureDetector、InkWell 等组件来处理用户输入事件，例如触摸、点击等。这些组件可以捕获事件并调用相应的回调函数来响应用户操作。

6. **动画和交互**：Flutter 提供了丰富的动画和交互支持，你可以在 Widget 树中使用动画控制器和过渡来创建复杂的动画效果。Flutter 的动画是基于 Widget 树的，因此可以很容易地在界面元素之间添加动态效果。

7. **销毁和资源释放**：当 Widget 不再需要时，Flutter 会自动调用 Widget 的 `dispose` 方法来执行资源释放和清理操作，以防止内存泄漏和资源浪费。

## React Native

### 控制元素显示隐藏
    `{isShow && [element]}`

### 什么是`Promise`，有几种状态

Promise是 Javascript 实现异步编程的一种范式，使用Promise让代码更加容易阅读，书写起来也更加的方便。

Promise 对象代表一个异步操作，有三种状态：*pending: 初始状态，不是成功或失败状态。fulfilled: 意味着操作成功完成。rejected: 意味着操作失败*；

### 箭头函数

**什么是箭头函数？**

箭头函数是一种匿名函数，类似于 lambda 表达式（python）或者 block (Ruby)。它们接受固定数量的参数，并在其封闭范围的上下文内运行。这个封闭范围可能定义它们的函数或其他代码。

**箭头函数的语法**

```
(argument1, argument2, ... argumentN) => {
  // function body
}
```

还有其他几种简写形式，这里不再列举，可以到下面的英文文章链接中查看。

**封闭范围上下文**

与其他类型的函数不同，箭头函数没有它自己的执行上下文。

这就意味着，`this` 和 `parameters` 继承自它的父函数。

```
const test = {
  name: 'test object',
  createAnonFunction: function() {
    return function() {
      console.log(this.name);
      console.log(arguments);
    };
  },

  createArrowFunction: function() {
    return () => {
      console.log(this.name);
      console.log(arguments);
    };
  }
};

> const anon = test.createAnonFunction('hello', 'world');
> const arrow = test.createArrowFunction('hello', 'world');

> anon();
undefined
{}

> arrow();
test object
{ '0': 'hello', '1': 'world' }
```

### FlatView 和 ScrollView 的区别

ScrollView 会一次性把所有内容渲染出来，而 FlatView 会惰性渲染子元素，只有它们展示在屏幕中时才开始渲染。

它们都是基于 `VirtualizeList` 的封装。

### React Native 组件的生命周期

1. `constructor(props)`：组件的构造函数，在组件被创建时调用。通常在此方法中初始化状态和绑定成员函数，在 RN 中需要调用`super(props)`。
2. `componentDidMount()`：组件已经渲染到屏幕上后调用，通常在此方法中发起网络请求、订阅事件等操作。
3. `shouldComponentUpdate(nextProps, nextState)`：在组件接收新的 props 或状态更新之前调用，返回一个布尔值，表示组件是否需要重新渲染。
4. `componentDidUpdate(prevProps, prevState)`：在组件已经重新渲染后调用，通常用于处理更新后的 DOM 操作。
5. `componentWillUnmount()`：在组件即将从屏幕移除之前调用，通常用于清理定时器、取消订阅等操作。
6. `getDerivedStateFromProps(nextProps, prevState)`：在组件接收新的 props 时调用，返回一个对象类更新状态。
7. `getSnapshotBeforeUpdate(prevProps, prevState)`：在组件更新之前获取 DOM 快照，通常在组件需要根据之前的 DOM 状态做一些操作时使用。
8. `static getDrivedStateFromError(error)`：在渲染过程中捕获子组件抛出的错误，并返回一个对象来更新状态，以便显示错误信息。
9. `componentDidCatch(error, info)`：在子组件抛出错误时调用，通常用于记录错误信息，或返回错误页面。

### React Native 组件的渲染流程

1. **组件的创建**：当你在应用中使用一个 React Native 组件时，首先会调用该组件的构造函数（constructor）来创建组件实例，并传入初始的 props。在构造函数中，通常会初始化组件的状态（state）和绑定成员函数。
2. **初始渲染**：一旦组件实例被创建，React Native 将调用组件的 `render` 方法，该方法返回一个描述组件界面的虚拟 DOM 树。React Native 的虚拟 DOM 与浏览器的不同，因为它需要与原生组件交互。
3. **原生组件创建**：React Native 会把虚拟 DOM 转换成原生 UI 组件。这些原生组件会和 Javascript 中的虚拟 DOM 进行关联，并在屏幕上进行渲染。
4. **布局计算和绘制**：原生 UI 组件根据其属性和样式进行布局计算和绘制。
5. **事件处理**：React Native 组件可以注册原生组件的事件处理，例如触摸事件、滚动事件。当事件发生时，React Native 将处理事件并调用相关的 Javascript 代码。
6. **更新流程**：当事件的 props 或状态发生变化时，React Native 会触发重新渲染流程。在更新流程中，React Native 会通过比较前后两次渲染的虚拟 DOM，计算出需要执行的最小更新操作，然后更新原生 UI 组件。
7. **卸载组件**：当组件不再需要时，例如从屏幕上移除或销毁组件实例时，React Native 会调用组件的 `componentWillUnmount` 方法执行清理操作。


### 参考链接
[记一次react native面试 - 掘金](https://juejin.cn/post/6992919281625202719)
[JavaScript Arrow Functions: How, Why, When (and WHEN NOT) to Use Them](https://zendev.com/2018/10/01/javascript-arrow-functions-how-why-when.html)
