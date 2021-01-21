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

上面单单是build简写的优势，就会让我考虑是否去使用了，而且还能实现跨页面的功能，这还考虑啥，开搞！

下来将全面的介绍GetX的使用，文章也不分篇水阅读量了，力求一文写清楚，方便大家随时查阅

# 计数器

## 准备

- 首页导入GetX的插件

```dart
# getx 状态管理框架 https://pub.flutter-io.cn/packages/get
get: ^3.24.0
```

- 主入口配置
  - 只需要将`MaterialApp`改成`GetMaterialApp`即可

```dart
void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      home: CounterGetPage(),
    );
  }
}
```

- 各模块导包，均使用下面包即可

```dart
import 'package:get/get.dart';
```

## 效果图



## 实现

**首页，当然是实现一个简单的计数器，来看GetX怎么将逻辑层和界面层解耦的**

这里我们新建一个`counter_get`文件夹，在`counter_get`文件夹下创建`view`和`logic`文件

### 响应式状态管理

> 当数据源变化时，将自动执行刷新组件的方法

- logic层
  - 因为是处理页面逻辑的，加上Controller单词过长，也防止和Flutter自带的一些控件控制器弄混，所以该层用`logic`结尾，这里就定为了`logic`层，当然这点随个人意向，写Event，Controller均可
  - 这里变量数值后写`.obs`操作，是说明定义了该变量为响应式变量，当该变量数值变化时，页面的刷新方法将自动刷新；基础类型，List，类都可以加`.obs`，使其变成响应式变量

```dart
class CounterGetLogic extends GetxController {
  var count = 0.obs;

  ///自增
  void increase() => ++count;
}
```

- view层

  - 这地方获取到Logic层的实例后，就可进行操作了，大家可能会想：WTF，为什么实例的操作放在build方法里？逗我呢？---------  实际不然，stl是无状态组件，说明了他就不会被二次重组，所以实例操作只会被执行一次，而且Obx()方法是可以刷新组件的，完美解决刷新组件问题了

    ```dart
    class CounterGetPage extends StatelessWidget {
      @override
      Widget build(BuildContext context) {
        CounterGetLogic logic = Get.put(CounterGetLogic());
    
        return Scaffold(
          appBar: AppBar(title: const Text('GetX计数器')),
          body: Center(
            child: Obx(
              () => Text('点击了 ${logic.count.value} 次',
                  style: TextStyle(fontSize: 30.0)),
            ),
          ),
          floatingActionButton: FloatingActionButton(
            onPressed: () => logic.increase(),
            child: const Icon(Icons.add),
          ),
        );
      }
    }
    ```

  - 当然，也可以这样写

    ```dart
    class CounterGetPage extends StatelessWidget {
      final CounterGetLogic logic = Get.put(CounterGetLogic());
    
      @override
      Widget build(BuildContext context) {
        return Scaffold(
          appBar: AppBar(title: const Text('GetX计数器')),
          body: Center(
            child: Obx(
              () => Text('点击了 ${logic.count.value} 次',
                  style: TextStyle(fontSize: 30.0)),
            ),
          ),
          floatingActionButton: FloatingActionButton(
            onPressed: () => logic.increase(),
            child: const Icon(Icons.add),
          ),
        );
      }
    }
    ```

  - 可以发现刷新组件的方法极其简单：`Obx()`，这样可以愉快的到处写定点刷新操作了

- Obx()方法刷新的条件

  - 只有当响应式变量的值发生变化时，才会会执行刷新操作，当某个变量初始值为：“test”，再赋值为：“test”，并不会执行刷新操作
  - 当你定义了一个响应式变量，该响应式变量改变时，包裹该响应式变量的Obx()方法才会执行刷新操作，其它的未包裹该响应式变量的Obx()方法并不会执行刷新操作，Cool！

- 来看下如果把整个类对象设置成响应类型，如何实现更新操作呢？

  - 下面解释来自官方README文档

```dart
// model
// 我们将使整个类成为可观察的，而不是每个属性。
class User() {
    User({this.name = '', this.age = 0});
    String name;
    int age;
}

// controller
final user = User().obs;
//当你需要更新user变量时。
user.update( (user) { // 这个参数是你要更新的类本身。
    user.name = 'Jonny';
    user.age = 18;
});
// 更新user变量的另一种方式。
user(User(name: 'João', age: 35));

// view
Obx(()=> Text("Name ${user.value.name}: Age: ${user.value.age}"))
    // 你也可以不使用.value来访问模型值。
    user().name; // 注意是user变量，而不是类变量（首字母是小写的）。
```

### 简单状态管理器

> GetBuilder：这是一个极其轻巧的状态管理器，占用资源极少！

- logic：先来看看logic层

```dart
class CounterEasyGetLogic extends GetxController {
  var count = 0;

  void increase() {
    ++count;
    update();
  }
}
```

- view

```dart
class CounterEasyGetPage extends StatelessWidget {
  final CounterEasyGetLogic logic = Get.put(CounterEasyGetLogic());

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('计数器-简单式')),
      body: Center(
        child: GetBuilder<CounterEasyGetLogic>(
          builder: (logicGet) => Text(
            '点击了 ${logicGet.count} 次',
            style: TextStyle(fontSize: 30.0),
          ),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => logic.increase(),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

- 分析下：GetBuilder这个方法
  - init：虽然上述代码没用到，但是，这个参数是存在在GetBuilder中的，因为在加载变量的时候就使用`Get.put()`生成了`CounterEasyGetLogic`对象，GetBuilder会自动查找该对象，所以，就可以不使用init参数
  - builder：方法参数，拥有一个入参，类型便是GetBuilder所传入泛型的类型
  - initState，dispose等：GetBuilder拥有StatefulWidget所有周期回调，可以在相应回调内做一些操作

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
  static final String main = "/";

  ///dialog页面
  static final String dialog = "/dialog";

  ///bloc计数器模块
  static final String counter = "/counter";

  ///测试布局页面
  static final String testLayout = "/testLayout";

  ///演示SmartDialog控件
  static final String smartDialog = "/smartDialog";

  ///Bloc跨页面传递事件
  static final String spanOne = "/spanOne";
  static final String spanTwo = "/spanOne/spanTwo";

  ///GetX 计数器  跨页面交互
  static final String counterGet = "/counterGet";
  static final String jumpOne = "/jumpOne";
  static final String jumpTwo = "/jumpOne/jumpTwo";

  ///别名映射页面
  static final List<GetPage> getPages = [
    GetPage(name: main, page: () => MainPage()),
    GetPage(name: dialog, page: () => Dialog()),
    GetPage(name: counter, page: () => CounterPage()),
    GetPage(name: testLayout, page: () => TestLayoutPage()),
    GetPage(name: smartDialog, page: () => SmartDialogPage()),
    GetPage(name: spanOne, page: () => SpanOnePage()),
    GetPage(name: spanTwo, page: () => SpanTwoPage()),
    GetPage(name: counterGet, page: () => CounterGetPage()),
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

发送命名路由数据

```dart
Get.toNamed("/profile/34954");
```

在第二个页面上，通过参数获取数据

```dart
print(Get.parameters['user']);
// out: 34954
```

现在，你需要做的就是使用Get.toNamed()来导航你的命名路由，不需要任何context(你可以直接从你的BLoC或Controller类中调用你的路由)，当你的应用程序被编译到web时，你的路由将出现在URL中。

