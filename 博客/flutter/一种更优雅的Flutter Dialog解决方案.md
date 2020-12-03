# 前言

系统自带的Dialog实际上就是Push了一个新页面，这样存在很多好处，但是也存在一些很难解决的问题

- 必须传BuildContext
- 无法穿透暗色背景，点击dialog后面的控件
  - 这个是真头痛，想了很多办法都没在自带dialog上面解决
- 系统自带Dialog写成的Loading弹窗，在网络请求和跳转页面的情况，会存在路由混乱的情况
  - 这种情况实在太常见了，解决方法：是定位页面栈的栈顶是否Loading弹窗，选择性Pop，实现麻烦

上面这些痛点，简直个个致命，还存在一些使用其他方法能解决的方案，例如：每个Page顶级使用Stack，使用Overlay；很明显，使用Overlay可移植性最后，目前很多Toast和dialog三方库便是使用该方案，使用了一些loading库，看了其中源码，穿透背景解决方案，和预期的想要的解决大相径庭、一些dialog库自带toast显示，但是toast显示却又不能和dialog共存（toast属于特殊的信息展示，理应能独立存在），没办法这边只能自己实现，花了一些时间，实现了一个Pub包，基本该解决的痛点都已解决，用于实际场景没什么问题

- 引入

```dart
dependencies:
  flutter_smart_dialog: ^0.0.4
```

- 使用

```

```

- 效果图
  - [点我体验一下](https://cnad666.github.io/flutter_use/web/index.html#/smartDialog)

![smartDialog](../../../资料/图片/smartDialog.gif)