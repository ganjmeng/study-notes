

# 前言

> 使用Bloc的时候，有一个让我至今为止十分在意的问题，无法真正的跨页面交互！在反复的查阅官方文档后，使用一个全局Bloc的方式，实现了“伪”跨页面交互，详细可查看：[flutter_bloc使用解析](https://juejin.cn/post/6856268776510504968)；fish_redux的广播机制是可以比较完美的实现跨页面交互的，我也写了一篇近万字介绍如何使用该框架：[fish_redux使用详解](https://juejin.cn/post/6860029460524040199)，对于中小型项目使用fish_redux，这会一定程度上降低开发效率，最近尝试了GetX相关功能，解决了我的相当一部分痛点

> 把整篇文章写完后，我马上把自己的一个demo里面所有Bloc代码全用GetX替换，且去掉了Fluro框架；感觉用Getx虽然会省掉大量的模板代码，但还是有些重复工作：创建文件夹，创建几个必备文件，写那些必须要写的初始化代码和类；略微繁琐，**为了对得起GetX给我开发带来的巨大便利，我就花了一些时间，给它写了一个插件！** 上面这重复的代码，文件，文件夹统统能一键生成！

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

# 准备

## 引入

- 首先导入GetX的插件

```dart
# getx 状态管理框架 https://pub.flutter-io.cn/packages/get
get: ^3.24.0
```

**GetX地址**

- Github：[jonataslaw/getx](https://github.com/jonataslaw/getx)
- Pub：[get](https://pub.dev/packages/get)

## 主入口配置

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

## 插件

> 吐槽下写插件的过程，实际写这种模板代码生成插件，其实也不难，网上有很多人写了范例，参考参考思路，能较快的整出来，就是有些配置比较蛋筒。
>
> 一开始选择Plugin DevKit模式整的，都已经写好，但是看官网文档的时候，官方文档开头就说了：建议使用Gradle模式开发插件，又巴拉巴拉列了一堆好处；考虑良久，决定用Gradle模式重写。
>
> 这个Gradle模式，最烦的还是开头新建项目的时候，那个Gradle死活下载不下来，科学全局上网都不行，然后手动下载了Gradle，指定本地Gradle，开全局再次同步时，会下载一个较大的社区版IDEA，但是使用本地Gradle加载完，存在一个很大的BUG！main文件夹下，不会自动生成Java文件夹！我真是佛了，点击其它的文件夹，右击：New -> Plugin DevKit 居然不会没有Action选项，差点把我劝退了，换了了七八个版本IDEA试了都不行！Action选项出不来，过了俩天后，晚上无意尝试在main文件夹下面新建了一个Java文件，然后在这个java文件上右击：New -> Plugin DevKit，Action选项出现了！真几把佛了。。。
>
> 还有个巨坑的问题，在Gradle模式下开发插件，把模板代码文件放在main文件下、放在src下、放在根目录下，都获取不到文件里面的内容，这个真是坑了我不少时间，搜了很多博客，都发现没写这个问题，官方文档范例看了几遍也没发现有啥说明，后来找到了一个三年前的项目，翻了翻代码发现，所有的资源文件都必须放在resources文件夹下，才能读取到文件内容。。。我勒个去。。。

### 说明

- 插件地址
  - Github：[getx_template](https://github.com/CNAD666/getx_template)
  - Jetbrains：[getx_template](https://plugins.jetbrains.com/plugin/15919-getx/versions)
- 插件效果
  - 看下插件使用的效果图吧，样式参考了fish_redux插件样式
  - 有一些可选择的功能，所以做成多按钮的样式，大家可以按照自己的需求进行操作
- 说下插件的功能含义
  - Model：生成GetX的模式，
    - Default：默认模式，生成三个文件：state，logic，view
    - Easy：简单模式，生成三个文件：logic，view
  - Function：功能选择
    - useFolder：使用文件，选择后会生成文件夹，大驼峰命名自动转换为：小写+下划线
    - usePrefix：使用前缀，生成的文件前加上前缀，前缀为：大驼峰命名自动转换为：小写+下划线
  - Module Name：模块的名称，请使用大驼峰命名

### 效果图

- 来看下使用的效果图

![getx_template](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/getx_plugin_show.gif)

### 安装

- 在设置里面选择：Plugins ---> 输入“getx”搜索 ---> 选择名字为：“GeX” ---> 然后安装 ---> 最后记得点击下“Apply”
- 如果在使用该插件的过程中有什么问题，请在该项目的github上给我提issue，我看到后，会尽快处理

![image-20210130182527494](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20210130182833.png)



# 计数器

## 效果图

- [体验一下](https://cnad666.github.io/flutter_use/web/index.html#/counterGet)

![counter_getx](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/counter_getx.gif)

## 实现

**首页，当然是实现一个简单的计数器，来看GetX怎么将逻辑层和界面层解耦的**

- 来使用插件生成下简单文件
  - 模式选择：Easy
  - 功能选择：useFolder

![image-20210126175019383](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/getx_plugin_user01.png)

来看下生成的默认代码，默认代码十分简单，详细解释放在俩种状态管理里

- logic

```dart
import 'package:get/get.dart';

class CounterGetLogic extends GetxController {

}
```

- view

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';

import 'logic.dart';

class CounterGetPage extends StatelessWidget {
  final CounterGetLogic logic = Get.put(CounterGetLogic());

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

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
  - 这里尝试了下，将整个类对象设置为响应类型，当你改变了类其中一个变量，然后执行更新操作，`只要包裹了该响应类变量的Obx()，都会实行刷新操作`，将整个类设置响应类型，需要结合实际场景使用

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

### 简单状态管理

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

## 总结

**分析**

- `GetBuilder`内部实际上是对StatefulWidget的封装，所以占用资源极小
- 响应式变量，因为使用的是`StreamBuilder`，会消耗一定资源

**使用场景**

- 一般来说，对于大多数场景都是可以使用响应式变量的
- 但是，在一个包含了大量对象的List，都使用响应式变量，将生成大量的`StreamBuilder`，必将对内存造成较大的压力，该情况下，就要考虑使用简单状态管理了

# 跨页面交互

> 跨页面交互，在复杂的场景中，是非常重要的功能，来看看GetX怎么实现跨页面事件交互的

## 效果图

- [体验一下](https://cnad666.github.io/flutter_use/web/index.html#/jumpOne)
- Cool，这才是真正的跨页面交互！下级页面能随意调用上级页面事件，且关闭页面后，下次重进，数据也很自然重置了（全局Bloc不会重置，需要手动重置）

![jump_getx](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/getx_jump.gif)

## 实现

### 页面一

常规代码

- logic
  - 这里的自增事件，是供其它页面调用的，该页面本身没使用

```dart
class JumpOneLogic extends GetxController {
  var count = 0.obs;

  ///跳转到跨页面
  void toJumpTwo() {
    Get.toNamed(RouteConfig.jumpTwo);
  }

  ///跳转到跨页面
  void increase() => count++;
}
```

- view
  - 此处就一个显示文字和跳转功能

```dart
class JumpOnePage extends StatelessWidget {
  /// 使用Get.put()实例化你的类，使其对当下的所有子路由可用。
  final JumpOneLogic logic = Get.put(JumpOneLogic());

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.white,
      appBar: AppBar(title: Text('跨页面-One')),
      floatingActionButton: FloatingActionButton(
        onPressed: () => logic.toJumpTwo(),
        child: const Icon(Icons.arrow_forward_outlined),
      ),
      body: Center(
        child: Obx(
          () => Text('跨页面-Two点击了 ${logic.count.value} 次',
              style: TextStyle(fontSize: 30.0)),
        ),
      ),
    );
  }
}
```

### 页面二

这个页面就是重点了，将演示怎么调用前一个页面的事件

- logic

```dart
class JumpTwoLogic extends GetxController {
  var count = 0.obs;

  ///跳转到跨页面
  void increase() => count++;
}
```

- view
  - 加号的点击事件，点击时，能实现俩个页面数据的变换
  - 重点来了，这里通过`Get.find()`，获取到了之前实例化GetXController，获取某个模块的GetXController后就很好做了，可以通过这个GetXController去调用相应的事件，也可以通过它，拿到该模块的数据！

```dart
class JumpTwoPage extends StatelessWidget {
  final JumpOneLogic oneLogic = Get.find();
  final JumpTwoLogic twoLogic = Get.put(JumpTwoLogic());

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.white,
      appBar: AppBar(title: Text('跨页面-Two')),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          oneLogic.increase();
          twoLogic.increase();
        },
        child: const Icon(Icons.add),
      ),
      body: Center(
        child: Obx(
          () => Text('跨页面-Two点击了 ${twoLogic.count.value} 次',
              style: TextStyle(fontSize: 30.0)),
        ),
      ),
    );
  }
}
```

## 总结

GetX这种的跨页面交互事件，真的是非常简单了，侵入性也非常的低，不需要在主入口配置什么，在复杂的业务场景下，这样简单的跨页面交互方式，就能实现很多事了

# 进阶吧！计数器

> 我们可能会遇到过很多复杂的业务场景，在复杂的业务场景下，单单某个模块关于变量的初始化操作可能就非常多，在这个时候，如果还将state（状态层）和logic（逻辑层）写在一起，维护起来可能看的比较晕，这里将状态层和逻辑层进行一个拆分，这样在稍微大一点的项目里使用GetX，也能保证结构足够清晰了！

在这里就继续用计数器举例吧！

## 实现

此处需要划分三个结构了：state（状态层），logic（逻辑层），view（界面层）

- 这里使用插件生成下模板代码
  - Model：选择Default（默认）
  - Function：useFolder（默认中）

![image-20210127093925934](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/getx_plugin_use02.png)

**来看下生成的模板代码**

- state

```dart
class CounterHighGetState {
  CounterHighGetState() {
    ///Initialize variables
  }
}
```

- logic

```dart
import 'package:get/get.dart';

import 'state.dart';

class CounterHighGetLogic extends GetxController {
  final state = CounterHighGetState();
}
```

- view

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';

import 'logic.dart';
import 'state.dart';

class CounterHighGetPage extends StatelessWidget {
  final CounterHighGetLogic logic = Get.put(CounterHighGetLogic());
  final CounterHighGetState state = Get.find<CounterHighGetLogic>().state;

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

为什么写成这样三个模块，需要把State单独提出来，请速速浏览下方

### 改造

- state
  - 这里可以发现，count类型使用的是`RxInt`，没有使用`var`，使用该变量类型的原因，此处是将所有的操作都放在构造函数里面初始化，如果直接使用`var`没有立马赋值，是无法推导为`Rx`类型，所以这里直接定义为`RxInt`，实际很简单，基础类型将开头字母大写，然后加上`Rx`前缀即可
  - 实际上直接使用`var`也是可以的，但是，使用该响应变量的时候`.value`无法提示，需要自己手写，所以还是老老实实的写明Rx具体类型吧
  - 详细可查看：[声明响应式变量](https://github.com/jonataslaw/getx/blob/master/documentation/zh_CN/state_management.md#%E5%A3%B0%E6%98%8E%E4%B8%80%E4%B8%AA%E5%93%8D%E5%BA%94%E5%BC%8F%E5%8F%98%E9%87%8F)

```dart
class CounterHighGetState {
  RxInt count;

  CounterHighGetState() {
    count = 0.obs;
  }
}
```

- logic
  - 逻辑层就比较简单，需要注意的是：开始时需要实例化状态类

```dart
class CounterHighGetLogic extends GetxController {
  final state = CounterHighGetState();

  ///自增
  void increase() => ++state.count;
}
```

- view
  - 实际上view层，和之前的几乎没区别，区别的是把状态层给独立出来了
  - 因为`CounterHighGetLogic`被实例化，所以直接使用`Get.find<CounterHighGetLogic>()`就能拿到刚刚实例化的逻辑层，然后拿到state，使用单独的变量接收下
  - ok，此时：logic只专注于触发事件交互，state只专注数据

```dart
class CounterHighGetPage extends StatelessWidget {
  final CounterHighGetLogic logic = Get.put(CounterHighGetLogic());
  final CounterHighGetState state = Get.find<CounterHighGetLogic>().state;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('计数器-响应式')),
      body: Center(
        child: Obx(
              () => Text('点击了 ${state.count.value} 次',
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

### 对比

> 看了上面的改造，屏幕前的你可能想吐槽了：坑比啊，之前简简单单的逻辑层，被拆成俩个，还搞得这么麻烦，你是猴子请来的逗比吗？
>
> 大家先别急着吐槽，当业务过于复杂，state层，也是会维护很多东西的，让我看看下面的一个小栗子，下面实例代码是不能直接运行的，想看详细运行代码，请查看项目：[flutter_use](https://github.com/CNAD666/flutter_use)

- state

```dart
class MainState {
  ///选择index - 响应式
  RxInt selectedIndex;

  ///控制是否展开 - 响应式
  RxBool isUnfold;

  ///分类按钮数据源
  List<BtnInfo> list;

  ///Navigation的item信息
  List<BtnInfo> itemList;

  ///PageView页面
  List<Widget> pageList;
  PageController pageController;

  MainState() {
    //初始化index
    selectedIndex = 0.obs;
    //默认不展开
    isUnfold = false.obs;
    //PageView页面
    pageList = [
      keepAliveWrapper(FunctionPage()),
      keepAliveWrapper(ExamplePage()),
      keepAliveWrapper(Center(child: Container())),
    ];
    //item栏目
    itemList = [
      BtnInfo(
        title: "功能",
        icon: Icon(Icons.bubble_chart),
      ),
      BtnInfo(
        title: "范例",
        icon: Icon(Icons.opacity),
      ),
      BtnInfo(
        title: "设置",
        icon: Icon(Icons.settings),
      ),
    ];
    //页面控制器
    pageController = PageController();
  }
}
```

- logic

```dart
class MainLogic extends GetxController {
  final state = MainState();

  ///切换tab
  void switchTap(int index) {
    state.selectedIndex.value = index;
  }

  ///是否展开侧边栏
  void onUnfold(bool unfold) {
    state.isUnfold.value = !state.isUnfold.value;
  }
}
```

- view

```dart
class MainPage extends StatelessWidget {
  final MainLogic logic = Get.put(MainLogic());
  final MainState state = Get.find<MainLogic>().state;

  @override
  Widget build(BuildContext context) {
    return BaseScaffold(
      backgroundColor: Colors.white,
      body: Row(children: [
        ///侧边栏区域
        Obx(
          () => SideNavigation(
            selectedIndex: state.selectedIndex.value,
            sideItems: state.itemList,
            onItem: (index) {
              logic.switchTap(index);
              state.pageController.jumpToPage(index);
            },
            isUnfold: state.isUnfold.value,
            onUnfold: (unfold) {
              logic.onUnfold(unfold);
            },
          ),
        ),

        ///Expanded占满剩下的空间
        Expanded(
          child: PageView.builder(
            physics: NeverScrollableScrollPhysics(),
            itemCount: state.pageList.length,
            itemBuilder: (context, index) => state.pageList[index],
            controller: state.pageController,
          ),
        )
      ]),
    );
  }
}
```

从上面可以看出，state层里面的状态已经较多了，当某些模块涉及到大量的：提交表单数据，跳转数据，展示数据等等，state层的代码会相当的多，相信我，真的是非常多，一旦业务发生变更，还要经常维护修改，就蛋筒了

在复杂的业务下，将状态层（state）和业务逻辑层（logic）分开，绝对是个明智的举动

## 最后

- 该模块的效果图就不放了，和上面计数器效果一模一样，想体验一下，可点击：[体验一下]()
- 简单的业务模块，可以使用俩层结构：logic，view；复杂的业务模块，推荐使用三层结构：state，logic，view

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

# 最后

## 相关地址

- 文中DEMO地址：[flutter_use](https://github.com/CNAD666/flutter_use)
- GetX插件地址：[getx_template](https://github.com/CNAD666/getx_template)

## 系列文章

> 引流了，手动滑稽.jpg

- Dialog解决方案，墙裂推荐

[一种更优雅的Flutter Dialog解决方案](https://juejin.cn/post/6902331428072390663)

- 状态管理篇

[fish_redux使用详解---看完就会用！](https://juejin.cn/post/6860029460524040199)

[flutter_bloc使用解析---骚年，你还在手搭bloc吗！](https://juejin.cn/post/6856268776510504968)

