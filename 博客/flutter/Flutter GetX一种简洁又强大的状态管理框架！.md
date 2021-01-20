# 前言

> 使用Bloc的时候，有一个让我至今为止十分在意的问题，无法真正的跨页面交互！在反复的查阅官方文档后，使用一个全局Bloc的方式，实现了“伪”跨页面交互，详细可查看：[flutter_bloc使用解析](https://juejin.cn/post/6856268776510504968)；fish_redux的广播机制是可以比较完美的实现跨页面交互的，我也写了一篇近万字介绍如何使用该框架：[fish_redux使用详解](https://juejin.cn/post/6860029460524040199)，对于中小型项目使用fish_redux，这会一定程度上降低开发效率，最近尝试了GetX相关功能，解决了我的相当一部分痛点

**GetX地址**

- Github：[jonataslaw/getx](https://github.com/jonataslaw/getx)
- Pub：[get](https://pub.dev/packages/get)

**GetX相关优势**

- build刷新方法极度简单！
  - getx：Obx(() => Text())
  - 这是我非常非常在意的一个方面，因为bloc的build刷新组件方法要传俩个泛型，加上build方法里面的俩个参数，导致一个build方法如果不使用箭头方法简写，几乎占四五行，用起来实在蛋筒，导致我平时开发直接把`BlocBuilder`方法直接写在页面顶层（不提倡写顶层），一个页面只用写一次了，不用定点到处写`BlocBuilder`了，手动滑稽.jpg
- 跨页面交互
  - 这绝对是GetX的一个优点！对于复杂的生产环境，跨页面交互的场景，实在太常见了，GetX的跨页面交互，几乎和fish_redux一样简单，爱了爱了
- 路由管理
  - 是的，getx内部实现了路由管理，而且用起来，那叫一个简单！bloc没实现路由管理，这让我不得不去找一个star量高的路由管理框架，就选择了fluro，但是让我不得不说，这个fluro用起来真的叫一个折磨人，每次新建一个页面，最让我抗拒的就是去写fluro路由代码，横跨几个文件来回写，真是肝疼
  - GetX实现了动态路由传参，也就是说直接在命名路由上拼参数，然后能拿到这些拼在路由上的参数，也就是说用flutter写H5，直接能通过Url传值（fluro也能做到），OMG！可以无脑舍弃复杂的fluro了
- 实现了全局BuildContext
- 国际化，主题实现

上面单单是build简写的优势，就会让我考虑是否去使用了，而且还能实现跨页面的功能，不想了，开搞！

下来将全面的介绍GetX的使用，文章也不分篇水阅读量了，力求一文写清楚，方便大家随时查阅

# 路由管理

**GetX实现了一套用起来十分简单的路由管理，可以使用一种极其简单的方式导航，也可以使用命名路由导航**

## 简单路由

- 主入口配置

```dart
void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      home: MainPage(),
    );
  }
}
```

- 路由的相关使用
  - 使用是非常简单，使用Get.to()之类api即可，此处简单演示，详细api说明，放在本节结尾

```
//跳转新页面
Get.to(SomePage());
```

## 命名路由导航

**这里是推荐使用命名路由导航的方式**

- 统一管理起了所有页面
- 在app中可能感受不到，但是在web端，加载页面的url地址就是命名路由你所设置字符串，也就是说，在web中，可以直接通过url导航到相关页面

下面说明下，如何使用

- 首先，在主入口出配置下

```dart
void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      initialRoute: RouteConfig.main,
      getPages: RouteConfig.getPages,
    );
  }
}
```

- RouteConfig类
  - 下面是我的相关页面，和其映射的页面，请根据自己的页面进行相关编写

```dart
class RouteConfig{
  ///主页面
  static String main = "/";

  ///dialog页面
  static String dialog = "/dialog";

  ///bloc计数器模块
  static String counter = "/counter";

  ///测试布局页面
  static String testLayout = "/testLayout";

  ///演示SmartDialog控件
  static String smartDialog = "/smartDialog";

  ///Bloc跨页面传递事件
  static String spanOne = "/spanOne";
  static String spanTwo = "/spanOne/spanTwo";

  ///GetX跨页面传递时间
  static String jumpOne = "/jumpOne";
  static String jumpTwo = "/jumpOne/jumpTwo";

  ///别名映射页面
  static List<GetPage> getPages = [
    GetPage(name: main, page: () => MainPage()),
    GetPage(name: dialog, page: () => Dialog()),
    GetPage(name: counter, page: () => CounterPage()),
    GetPage(name: testLayout, page: () => TestLayoutPage()),
    GetPage(name: smartDialog, page: () => SmartDialogPage()),
    GetPage(name: spanOne, page: () => SpanOnePage()),
    GetPage(name: spanTwo, page: () => SpanTwoPage()),
    GetPage(name: jumpOne, page: () => JumpOnePage()),
    GetPage(name: jumpTwo, page: () => JumpTwoPage()),
  ];
}
```

## 路由API

**请注意命名路由，只需要在api结尾加上`Named`即可，举例：**

- 默认：Get.to(SomePage());
- 命名路由：Get.toNamed(“/somePage”);

**详细Api介绍，下面内容来自GetX的README文档，进行了相关整理**

- 导航到新的页面

```dart
Get.to(NextScreen());
Get.toNamed("/NextScreen");
```

- 关闭SnackBars、Dialogs、BottomSheets或任何你通常会用Navigator.pop(context)关闭的东西

```dart
Get.back();
```

- 进入下一个页面，但没有返回上一个页面的选项（用于SplashScreens，登录页面等）

```dart
Get.off(NextScreen());
Get.offNamed("/NextScreen");
```

- 进入下一个界面并取消之前的所有路由（在购物车、投票和测试中很有用）

```dart
Get.offAll(NextScreen());
Get.offAllNamed("/NextScreen");
```

- 发送数据到其它页面

只要发送你想要的参数即可。Get在这里接受任何东西，无论是一个字符串，一个Map，一个List，甚至一个类的实例。

```
Get.to(NextScreen(), arguments: 'Get is the best');
Get.toNamed("/NextScreen", arguments: 'Get is the best');
```

在你的类或控制器上：

```
print(Get.arguments);
//print out: Get is the best
```

- 要导航到下一条路由，并在返回后立即接收或更新数据

```dart
var data = await Get.to(Payment());
var data = await Get.toNamed("/payment");
```

- 在另一个页面上，发送前一个路由的数据

```dart
Get.back(result: 'success');
// 并使用它，例：
if(data == 'success') madeAnything();
```

- 如果你不想使用GetX语法，只要把 Navigator（大写）改成 navigator（小写），你就可以拥有标准导航的所有功能，而不需要使用context，例如：

```dart
// 默认的Flutter导航
Navigator.of(context).push(
  context,
  MaterialPageRoute(
    builder: (BuildContext context) {
      return HomePage();
    },
  ),
);

// 使用Flutter语法获得，而不需要context。
navigator.push(
  MaterialPageRoute(
    builder: (_) {
      return HomePage();
    },
  ),
);

// get语法
Get.to(HomePage());
```

**动态网页链接**

- 这是一个非常重要的功能，在web端，可以`保证通过url传参数到页面`里

Get提供高级动态URL，就像在Web上一样。Web开发者可能已经在Flutter上想要这个功能了，Get也解决了这个问题。

```dart
Get.offAllNamed("/NextScreen?device=phone&id=354&name=Enzo");
```

在你的controller/bloc/stateful/stateless类上：

```dart
print(Get.parameters['id']);
// out: 354
print(Get.parameters['name']);
// out: Enzo
```

你也可以用Get轻松接收NamedParameters。

```dart
void main() {
  runApp(
    GetMaterialApp(
      initialRoute: '/',
      getPages: [
      GetPage(
        name: '/',
        page: () => MyHomePage(),
      ),
      GetPage(
        name: '/profile/',
        page: () => MyProfile(),
      ),
       //你可以为有参数的路由定义一个不同的页面，也可以为没有参数的路由定义一个不同的页面，但是你必须在不接收参数的路由上使用斜杠"/"，就像上面说的那样。
       GetPage(
        name: '/profile/:user',
        page: () => UserProfile(),
      ),
      GetPage(
        name: '/third',
        page: () => Third(),
        transition: Transition.cupertino  
      ),
     ],
    )
  );
}
```