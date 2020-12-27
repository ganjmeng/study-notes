- [ ] **bloc的例子需要重写下,Mark一下**

# 前言

- 首先，有很多的文章在说flutter bloc模式的应用，但是百分之八九十的文章都是在说，使用StreamController+StreamBuilder搭建bloc，提升性能的会加上InheritedWidget，这些文章看了很多，真正写使用bloc作者开发的flutter_bloc却少之又少。没办法，只能去bloc的github上去找使用方式，最后去bloc官网翻文档。

- 蛋痛，各位叼毛，就不能好好说说flutter_bloc的使用吗？非要各种抄bloc模式提出作者的那俩篇文章。现在，搞的杂家这个伸手党要自己去翻文档总结（手动滑稽）。

![表情1](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200804152051.png)

**项目效果（建议PC浏览器打开）**

- [Bloc范例效果](http://cnad666.gitee.io/book_web_manage)

- [Cubit范例效果](https://cnad666.github.io/flutter_use/web/index.html#/)

**下面是Flutter_Bloc历程的一系列链接**

- [Flutter_Bloc起源](https://www.didierboelens.com/2018/08/reactive-programming-streams-bloc/)

- [Flutter_Bloc模式优化](https://www.didierboelens.com/2018/12/reactive-programming-streams-bloc-practical-use-cases/)

- [Flutter_Bloc诞生](https://medium.com/flutter-community/flutter-bloc-package-295b53e95c5c)

- [Flutter_Bloc官网文档](https://bloclibrary.dev/#/)

  前面三个，是bloc作者写的bloc模式文档，典型的观察者模式的应用，最原始的就是java中CallBack形式。前俩篇文章就是咱们这些大抄子的主要“参考”的资料来源，这三篇文章在掘金上有翻译版，搜下bloc就能找到。最后一篇文章就是我主要总结归纳的源泉，作者在官网上写了好几个demo：计时器，登录，Todos，天气等等，大家可以自己去看看。

## 问题

初次使用flutter_bloc框架，可能会有几个疑问

- state里面定义了太多变量，某个事件只需要更新其中一个变量，其它的变量赋相同值麻烦
- 进入某个模块，进行初始化操作：复杂的逻辑运算，网络请求等，入口在哪定义

# 准备工作

**说明**

- 这里说明下，文章里把`BlocBuilder`放在顶层，因为本身页面非常简单，也是为了更好呈现页面结构，所以才放在顶层，如果需要更加颗粒化控件更新区域，请将`BlocBuilder`包裹你需要更新的控件区域即可

**引用**

- 我觉得学习一个模式或者框架的时候，最主要的是把主流程跑通，起码可以符合标准的堆页面，这样的话，就可以把这玩意用起来，再遇到想要的什么细节，就可以自己去翻文档，毕竟大体上已经懂了，写过了几个页面，也有些体会，再去翻文档就很快能理解了
- 实际上Bloc给的API也不多，就几个API，相关API使用说明都写在文章最后

**库**

```dart
flutter_bloc: ^6.1.1 #状态管理框架
equatable: ^1.2.3 #增强组件相等性判断
```

- 看看flutter_bloc都推到6.0了，别再用StreamController手搭Bloc了！

**插件**

在Android Studio设置的Plugins里，搜索：Bloc

![插件搜索](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200804152204.png)

安装重启下，就OK了

- 右击相应的文件夹，选择“Bloc Class”，我在main文件夹新建的，填入的名字：main，就自动生成下面三个文件；：main_bloc，main_event，main_state；main_view是我自己新建，用来写页面的。

![新建bloc文件](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200804152227.png)

![目录结构新建bloc文件](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200804152237.png)

- 是不是觉得，还在手动新建这些bloc文件low爆了；就好像fish_redux，不用插件，让我手动去创建那六个文件，写那些模板代码，真的要原地爆炸。

# Bloc范例

## 效果

- 好了，哔哔了一堆，看下咱们要用flutter_bloc实现的效果。

![bloc演示](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200804152120.gif)

- 直接开Chrome演示，大家在虚拟机上跑也一样。

## 初始化代码

来看下这三个生成的bloc文件：main_bloc，main_event，main_state

- main_bloc：这里就是咱们主要写逻辑的页面了
  - mapEventToState方法只有一个参数，后面自动带了一个逗号，格式化代码就分三行了，建议删掉逗号，格式化代码。

```dart
class MainBloc extends Bloc<MainEvent, MainState> {
  MainBloc() : super(MainInitial());

  @override
  Stream<MainState> mapEventToState(
    MainEvent event,
  ) async* {
    // TODO: implement mapEventToState
  }
}
```

- main_event：这里是执行的各类事件，有点类似fish_redux的action层

```dart
@immutable
abstract class MainEvent {}
```

- main_state：状态数据放在这里保存，中转

```dart
@immutable
abstract class MainState {}

class MainInitial extends MainState {}
```

## 实现

- 说明
  - 这里对于简单的页面，state的使用抽象状态继承实现的方式，未免有点麻烦，这里我进行一点小改动，state的实现类别有很多，官网写demo也有不用抽象类，直接class，类似实体类的方式开搞的。
  - 老夫在代码关键点写上"///"类型注释，大家仔细看看，拷进Android Studio里面，这些地方会变绿！大家好好体会下绿色代码！
- main_bloc
  - state变量是框架内部定义的，会默认保存上一次同步的MainSate对象的值

```dart
class MainBloc extends Bloc<MainEvent, MainState> {
  MainBloc() : super(MainState(selectedIndex: 0, isExtended: false));

  @override
  Stream<MainState> mapEventToState(MainEvent event) async* {
    ///main_view中添加的事件，会在此处回调，此处处理完数据，将数据yield，BlocBuilder就会刷新组件
    if (event is SwitchTabEvent) {
      ///获取到event事件传递过来的值,咱们拿到这值塞进MainState中
      ///直接在state上改变内部的值,然后yield,只能触发一次BlocBuilder,它内部会比较上次MainState对象,如果相同,就不build
      yield MainState()
        ..selectedIndex = event.selectedIndex
        ..isExtended = state.isExtended;
    } else if (event is IsExtendEvent) {
      yield MainState()
        ..selectedIndex = state.selectedIndex
        ..isExtended = !state.isExtended;
    }
  }
}
```

- main_event：在这里就能看见，view触发了那些事件了；维护起来也很爽，看看这里，也很快能懂页面在干嘛了

```dart
@immutable
abstract class MainEvent extends Equatable{
  const MainEvent();
}
///切换NavigationRail的tab
class SwitchTabEvent extends MainEvent{
  final int selectedIndex;

  const SwitchTabEvent({@required this.selectedIndex});

  @override
  List<Object> get props => [selectedIndex];
}
///展开NavigationRail,这个逻辑比较简单,就不用传参数了
class IsExtendEvent extends MainEvent{
  const IsExtendEvent();

  @override
  List<Object> get props => [];
}
```

- main_state：state有很多种写法，在bloc官方文档上，不同项目state的写法也很多
  
  - 这边变量名可以设置为私用，用get和set可选择性的设置读写权限，因为我这边设置的俩个变量全是必用的，读写均要，就设置公有类型，不用下划线“_”去标记私有了。
  - 对于生成的模板代码，我们在这：去掉@immutable注解，去掉abstract；
  
  - 这里说下加上@immutable和abstract的作用，这边是为了标定不同状态，这种写法，会使得代码变得更加麻烦，用state不同状态去标定业务事件，代价太大，这边用一个变量去标定，很容易轻松代替

```dart
class MainState{
   int selectedIndex;
   bool isExtended;
  
   MainState({this.selectedIndex, this.isExtended});
}
```

- main_view
  - 这边就是咱们的界面层了，很简单，将需要刷新的组件，用BlocBuilder包裹起来，使用BlocBuilder：提供的state去赋值就ok了，context去添加执行的事件，context用StatelessWidget中提供的或者BlocBuilder提供的都行

```dart
void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: MainPage(),
    );
  }
}

class MainPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    ///创建BlocProvider的，表明该Page，我们是用MainBloc，MainBloc是属于该页面的Bloc了
    return BlocProvider(
      create: (BuildContext context) => MainBloc(),
      child: BodyPage(),
    );
  }
}

class BodyPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Bloc')),
      body: totalPage(),
    );
  }
}

Widget totalPage() {
  return Row(
    children: [
      navigationRailSide(),
      Expanded(child: Center(
        child: BlocBuilder<MainBloc, MainState>(builder: (context, state) {
          ///看这看这：刷新组件！
          return Text("selectedIndex:" + state.selectedIndex.toString());
        }),
      ))
    ],
  );
}

//增加NavigationRail组件为侧边栏
Widget navigationRailSide() {
  //顶部widget
  Widget topWidget = Center(
    child: Padding(
      padding: const EdgeInsets.all(8.0),
      child: Container(
          width: 80,
          height: 80,
          decoration: BoxDecoration(
            shape: BoxShape.circle,
            image: DecorationImage(
                image: NetworkImage("https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=3383029432,2292503864&fm=26&gp=0.jpg"),
                fit: BoxFit.fill),
          )),
    ),
  );

  //底部widget
  Widget bottomWidget = Container(
    child: BlocBuilder<MainBloc, MainState>(
      builder: (context, state) {
        return FloatingActionButton(
          onPressed: () {
            ///添加NavigationRail展开,收缩事件
            BlocProvider.of<MainBloc>(context).add(IsExtendEvent());
          },
          ///看这看这：刷新组件！
          child: Icon(state.isExtended ? Icons.send : Icons.navigation),
        );
      },
    ),
  );

  return BlocBuilder<MainBloc, MainState>(builder: (context, state) {
    return NavigationRail(
      backgroundColor: Colors.white12,
      elevation: 3,
      ///看这看这：刷新组件！
      extended: state.isExtended,
      labelType: state.isExtended ? NavigationRailLabelType.none : NavigationRailLabelType.selected,
      //侧边栏中的item
      destinations: [
        NavigationRailDestination(
            icon: Icon(Icons.add_to_queue),
            selectedIcon: Icon(Icons.add_to_photos),
            label: Text("测试一")),
        NavigationRailDestination(
            icon: Icon(Icons.add_circle_outline),
            selectedIcon: Icon(Icons.add_circle),
            label: Text("测试二")),
        NavigationRailDestination(
            icon: Icon(Icons.bubble_chart),
            selectedIcon: Icon(Icons.broken_image),
            label: Text("测试三")),
      ],
      //顶部widget
      leading: topWidget,
      //底部widget
      trailing: bottomWidget,
      selectedIndex: state.selectedIndex,
      onDestinationSelected: (int index) {
        ///添加切换tab事件
        BlocProvider.of<MainBloc>(context).add(SwitchTabEvent(selectedIndex: index));
      },
    );
  });
}
```

# Bloc范例优化

## 反思

从上面的代码来看，实际存在几个隐式问题，这些问题，刚开始使用时候，没异常的感觉，但是使用bloc久了后，感觉肯定越来越强烈

- state问题
  - 初始化问题：这边初始化是在bloc里，直接在构造方法里面赋初值的，state中一旦变量多了，还是这么写，会感觉极其难受，不好管理。需要优化
  - 可以看见这边我们只改动selectedIndex或者isExtended；另一个变量不需要变动，需要保持上一次的数据，进行了此类：state.selectedIndex或者state.isExtended赋值，一旦变量达到十几个乃至几十个，还是如此写，是让人极其崩溃的。需要优化
- bloc问题
  - 如果进行一个页面，需要进行复杂的运算或者请求接口后，才能知晓数据，进行赋值，这里肯定需要一个初始化入口，初始化入口需要怎样去定义呢？

## 优化实现

这边完整走一下流程，让大家能有个完整的思路

- state：首先来看看我们对state中的优化，这边进行了俩个很重要优化，增加俩个方法：init()和clone()
  - init()：这里初始化统一用init()方法去管理
  - clone()：这边克隆方法，是非常重要的，一旦变量达到俩位数以上，就能深刻体会该方法是多么的重要

```dart
class MainState {
  int selectedIndex;
  bool isExtended;

  ///初始化方法,基础变量也需要赋初值,不然会报空异常
  MainState init() {
    return MainState()
      ..selectedIndex = 0
      ..isExtended = false;
  }

  ///clone方法,此方法实现参考fish_redux的clone方法
  ///也是对官方Flutter Login Tutorial这个demo中copyWith方法的一个优化
  ///Flutter Login Tutorial（https://bloclibrary.dev/#/flutterlogintutorial）
  MainState clone() {
    return MainState()
      ..selectedIndex = selectedIndex
      ..isExtended = isExtended;
  }
}
```

- event
  - 这边定义一个MainInit()初始化方法，同时去掉Equatable继承，在我目前的使用中，感觉它用处不大。。。

```dart
@immutable
abstract class MainEvent {}

///初始化事件,这边目前不需要传什么值
class MainInitEvent extends MainEvent {}

///切换NavigationRail的tab
class SwitchTabEvent extends MainEvent {
  final int selectedIndex;

  SwitchTabEvent({@required this.selectedIndex});
}

///展开NavigationRail,这个逻辑比较简单,就不用传参数了
class IsExtendEvent extends MainEvent {}
```

- bloc
  - 这增加了初始化方法，请注意，如果需要进行异步请求，同时需要将相关逻辑提炼一个方法，咱们在这里配套Future和await就能解决在异步场景下同步数据问题
  - 这里使用了克隆方法，可以发现，我们只要关注自己需要改变的变量就行了，其它的变量都在内部赋值好了，我们不需要去关注；这就大大的便捷了页面中有很多变量，只需要变动一俩个变量的场景
  - 注意：如果变量的数据未改变，界面相关的widget是不会重绘的；只会重绘变量被改变的widget

```dart
class MainBloc extends Bloc<MainEvent, MainState> {
  MainBloc() : super(MainState().init());

  @override
  Stream<MainState> mapEventToState(MainEvent event) async* {
    ///main_view中添加的事件，会在此处回调，此处处理完数据，将数据yield，BlocBuilder就会刷新组件
    if (event is MainInitEvent) {
      yield await init();
    } else if (event is SwitchTabEvent) {
      ///获取到event事件传递过来的值,咱们拿到这值塞进MainState中
      ///直接在state上改变内部的值,然后yield,只能触发一次BlocBuilder,它内部会比较上次MainState对象,如果相同,就不build
      yield switchTap(event);
    } else if (event is IsExtendEvent) {
      yield isExtend();
    }
  }

  ///初始化操作,在网络请求的情况下,需要使用如此方法同步数据
  Future<MainState> init() async {
    return state.clone();
  }

  ///切换tab
  MainState switchTap(SwitchTabEvent event) {
    return state.clone()..selectedIndex = event.selectedIndex;
  }

  ///是否展开
  MainState isExtend() {
    return state.clone()..isExtended = !state.isExtended;
  }
}
```

- view
  - view层代码太多，这边只增加了个初始化事件，就不重新把全部代码贴出来了，初始化操作直接在创建的时候，在XxxBloc上使用add()方法就行了，就能起到进入页面，初始化一次的效果；add()方法也是Bloc类中提供的，遍历事件的时候，就特地检查了add()这个方法是否添加了事件；说明，这是框架特地提供了一个初始化的方法
  - 这个初始化方式是在官方示例找到的
    - 项目名：Flutter Infinite List Tutorial
    - 项目地址：[flutter-infinite-list-tutorial](https://bloclibrary.dev/#/flutterinfinitelisttutorial?id=flutter-infinite-list-tutorial)

```dart
class MainPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      ///在MainBloc上使用add方法,添加初始化事件
      create: (BuildContext context) => MainBloc()..add(MainInitEvent()),
      child: BodyPage(),
    );
  }
}
///下方其余代码省略...........
```

## 搞定

- OK，经过这样的优化，解决了几个痛点。实际在view中反复是要用BlocBuilder去更新view，写起来有点麻烦，这里我们可以写一个，将其中state和context变量，往提出来的Widget方法传值，也是蛮不错的
- 大家保持观察者模式的思想就行了；观察者（回调刷新控件）和被观察者（产生相应事件，添加事件，去通知观察者），bloc层是处于观察者和被观察者中间的一层，我们可以在bloc里面搞业务，搞逻辑，搞网络请求，不能搞基；拿到Event事件传递过来的数据，把处理好的、符合要求的数据返回给view层的观察者就行了。
- 使用框架，不拘泥框架，在观察者模式的思想上，灵活的去使用flutter_bloc提供Api，这样可以大大的缩短我们的开发时间！

# Cubit范例

- Cubit是Bloc模式的一种简化版，去掉了event这一层，对于简单的页面，用Cubit来实现，开发体验是大大的好啊，下面介绍下该种模式的写法

## 创建

- 首先创建Cubit一组文件，选择“Cubit Class”，点击，新建名称填写：Counter

![image-20201010155420462](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20201010165016.png)

新建好后，他会生成俩个文件：counter_cubit，counter_state，来看下生成的代码

### 原始模板代码

- counter_cubit

```dart
class CounterCubit extends Cubit<CounterState> {
  CounterCubit() : super(CounterInitial());
}
```

- counter_state

```dart
@immutable
abstract class CounterState {}

class CounterInitial extends CounterState {}
```

按照生成的这种state方式去写，比较麻烦，这边调整下

### 模板代码优化

- counter_cubit

```dart
class CounterCubit extends Cubit<CounterState> {
  CounterCubit() : super(CounterState().init());
}
```

- counter_state

```dart
class CounterState {
  ///初始化方法
  CounterState init() {
    return CounterState();
  }

  ///克隆方法,针对于刷新界面数据
  CounterState clone() {
    return CounterState();
  }
}
```

OK，这样调整了下，下面写起来就会舒服很多，也会很省事

## 实现计时器

- 来实现下一个灰常简单的计数器

**效果**

- 来看下实现效果吧，这边不上图了，大家点击下面的链接，可以直接体验Cubit模式写的计时器
- 实现效果：[点我体验实际效果](https://cnad666.gitee.io/flutter_use/#/counter)

**实现**

实现很简单，三个文件就搞定，看下流程：state ->  cubit -> view

- state：这个很简单，加个计时变量

```dart
class CounterState {
  int count;

  CounterState init() {
    return CounterState()..count = 0;
  }

  CounterState clone() {
    return CounterState()..count = count;
  }
}
```

- cubit
  - 这边加了个自增方法：increase()
  - event层实际是所有行为的一种整合，方便对逻辑过于复杂的页面，所有行为的一种维护；但是过于简单的页面，就那么几个事件，还单独维护，就没什么必要了
  - 在cubit层写的公共方法，在view里面能直接调用，更新数据使用：emit()
  - cubit层应该可以算是：bloc层和event层一种结合后的简写

```dart
class CounterCubit extends Cubit<CounterState> {
  CounterCubit() : super(CounterState().init());

  ///自增
  void increase() => emit(state.clone()..count = ++state.count);
}
```

- view
  - view层的代码就非常简单了，点击方法里面调用cubit层的自增方法就ok了

```dart
class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (BuildContext context) => CounterCubit(),
      child: BlocBuilder<CounterCubit, CounterState>(builder: _counter),
    );
  }

  Widget _counter(BuildContext context, CounterState state) {
    return Scaffold(
      appBar: AppBar(title: const Text('Cubit范例')),
      body: Center(
        child: Text('点击了 ${state.count} 次', style: TextStyle(fontSize: 30.0)),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => BlocProvider.of<CounterCubit>(context).increase(),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

## 总结

在Bloc模式里面，如果页面不是过于复杂，使用Cubit去写，基本完全够用了；但是如果业务过于复杂，还是需要用Bloc去写，需要将所有的事件行为管理起来，便于后期维护

OK，Bloc的简化模块，Cubit模式就这样讲完了，对于自己业务写的小项目，我就经常用这个Cubit去写

# 全局Bloc

## 说明

**什么是全局Bloc？**

- BlocProvider介绍里面有这样的形容：BlocProvider should be used to create new blocs which will be made available to the rest of the subtree（BlocProvider应该被用于创建新的Bloc，这些Bloc将可用于其子树）
- 这样的话，我们只需要在主入口地方使用BlocProvider创建Bloc，就能使用全局的XxxBloc了，这里的全局XxxBloc，state状态都会被保存的，除非关闭app，否则state里面的数据都不会被还原！
- **注意**：在主入口创建的XxxBloc，在主入口处创建了一次，在其它页面均不需要再次创建，在任何页面只需要使用BlocBuilder，便可以定点刷新及其获取全局XxxBloc的state数据

**使用场景**

- 全局的主题色，字体样式和大小等等全局配置更改；这种情况，在需要全局属性的地方，使用BlocBuilder对应的全局XxxBloc泛型去刷新数据就行了
- 跨页面去调用事件，既然是全局的XxxBloc，这就说明，我们可以在任何页面，使用` BlocProvider.of<XxxBloc>(context)`调用全局XxxBloc中事件，这就起到了一种跨页面调用事件的效果
  - 使用全局Bloc做跨页面事件时，应该明白，当你关闭Bloc对应的页面，对应全局Bloc中的并不会被回收，下次进入页面，页面的数据还是上次退出页面修改的数据，这里应该使用StatefulWidget，在initState生命周期处，初始化数据；或者在dispose生命周期处，还原数据源
  - 思考下：全局Bloc对象存在周期是在整个App存活周期，必然不能创建过多的全局Bloc，跨页面传递事件使用全局Bloc应当只能做折中方案

## 效果图

- [点我体验一下](https://cnad666.github.io/flutter_use/web/index.html#/spanOne)

![globalBloc](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20201214111143.gif)

## 使用

**来看下怎么创建和使用全局Bloc吧！**

- 主入口配置
  - 全局的Bloc创建还是蛮简单的，这边把`MultiBlocProvider`在Builder里面套在child上面就行了；当然了，把`MultiBlocProvider`套在MaterialApp上也是可以的
  - 这样我们就获得一个全局的`SpanOneCubit`

```dart
void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: MainPage(),
      builder: (BuildContext context, Widget child) {
        return MultiBlocProvider(
          providers: [
            ///此处通过BlocProvider创建的Bloc或者Cubit是全局的
            BlocProvider<SpanOneCubit>(
              create: (BuildContext context) => SpanOneCubit(),
            ),
          ],
          child: child,
        );
      },
    );
  }
}
```

**需要用俩个Bloc模块来演示，这里分别用`SpanOneCubit`和`SpanTwoCubit`来演示，其中`SpanOneCubit`是全局的**

**SpanOneCubit**

- state
  - 先来看看State模块的代码，这里很简单，只定义了count变量

```dart
class SpanOneState {
  int count;

  ///初始化方法
  SpanOneState init() {
    return SpanOneState()..count = 0;
  }

  ///克隆方法,针对于刷新界面数据
  SpanOneState clone() {
    return SpanOneState()..count = count;
  }
}
```

- view
  - 这个页面仅仅是展示计数变量的变化，因为在主入口使用了`BlocProvider`创建了`SpanOneCubit`，所以在这个页面不需要再次创建，直接使用`BlocBuilder`便可以获取其state
  - 可以发现，这个页面使用了StatefulWidget，在initState周期中，初始化了数据源；这样，每次进入页面，数据源就不会保存为上一次改动的来，都会被初始化为我们想要的值；这个页面能接受到任何页面调用其事件，这样就实现类似于广播的一种效果（`伪`）

```dart
class SpanOnePage extends StatefulWidget {
  @override
  _SpanOnePageState createState() => _SpanOnePageState();
}

class _SpanOnePageState extends State<SpanOnePage> {

  @override
  void initState() {
    BlocProvider.of<SpanOneCubit>(context).init();
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<SpanOneCubit, SpanOneState>(builder: _body);
  }

  Widget _body(BuildContext context, SpanOneState state) {
    return Scaffold(
      backgroundColor: Colors.white,
      appBar: AppBar(title: Text('跨页面-One')),
      floatingActionButton: FloatingActionButton(
        onPressed: () =>
            BlocProvider.of<SpanOneCubit>(context).toSpanTwo(context),
        child: const Icon(Icons.arrow_forward_outlined),
      ),
      body: Center(
        child: Text(
          'SpanTwoPage点击了 ${state.count} 次',
          style: TextStyle(fontSize: 30.0),
        ),
      ),
    );
  }
}
```

- cubit
  - cubit里面有三个事件，初始化，跳转页面，计数自增

```dart
class SpanOneCubit extends Cubit<SpanOneState> {
  SpanOneCubit() : super(SpanOneState().init());

  void init() {
    emit(state.init());
  }

  ///跳转到跨页面
  void toSpanTwo(BuildContext context) {
    Navigator.push(context, MaterialPageRoute(builder: (context) => SpanTwoPage()));
  }

  ///自增
  void increase() {
    state..count = ++state.count;
    emit(state.clone());
  }
}
```

**SpanTwoCubit**

- state
  - 使用count，记录下我们点击自增的次数

```dart
class SpanTwoState {
  int count;

  ///初始化方法
  SpanTwoState init() {
    return SpanTwoState()..count = 0;
  }

  ///克隆方法,针对于刷新界面数据
  SpanTwoState clone() {
    return SpanTwoState()..count = count;
  }
}
```

- view
  - 这地方我们需要创建使用BlocProvider一个SpanTwoCubit，这是使用Bloc的常规流程
  - 在自增的点击事件里，我们调用本模块和`SpanOneCubit`中的自增方法，OK，这里我们就能同步的改变`SpanOneCubit`模块的数据了！

```dart
class SpanTwoPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => SpanTwoCubit()..init(context),
      child: BlocBuilder<SpanTwoCubit, SpanTwoState>(builder: _body),
    );
  }

  Widget _body(BuildContext context, SpanTwoState state) {
    return Scaffold(
      backgroundColor: Colors.white,
      appBar: AppBar(title: Text('跨页面-Two')),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          //改变SpanOneCubit模块数据
          BlocProvider.of<SpanOneCubit>(context).increase();

          //改变当前页面数据
          BlocProvider.of<SpanTwoCubit>(context).increase();
        },
        child: const Icon(Icons.add),
      ),
      body: Center(
        child: Text(
          '当前点击了 ${state.count} 次',
          style: TextStyle(fontSize: 30.0),
        ),
      ),
    );
  }
}
```

- cubit
  - 平平无奇的业务代码

```dart
class SpanTwoCubit extends Cubit<SpanTwoState> {
  SpanTwoCubit() : super(SpanTwoState().init());

  void init(BuildContext context){
    emit(state.init());
  }

  ///自增
  void increase() => emit(state.clone()..count = ++state.count);
}
```

## 总结

OK，这样便用全局Bloc实现了类似广播的一种效果

- 使用全局去刷新：主题，字体样式和大小之类，每个页面都要使用BlocBuilder对应的全局bloc去刷新对应的全局view模块

# Bloc API说明

## BlocBuilder

**BlocBuilder**是Flutter窗口小部件，需要`Bloc`和`builder`函数。`BlocBuilder`处理构建小部件以响应新状态。`BlocBuilder`与非常相似，`StreamBuilder`但具有更简单的API，可以减少所需的样板代码量。该`builder`函数可能会被多次调用，并且应该是一个纯函数，它会根据状态返回小部件。

看看`BlocListener`是否要响应状态更改“执行”任何操作，例如导航，显示对话框等。

如果省略cubit参数，`BlocBuilder`将使用`BlocProvider`和当前函数自动执行查找`BuildContext`。

```dart
BlocBuilder<BlocA, BlocAState>(
  builder: (context, state) {
    // return widget here based on BlocA's state
  }
)
```

仅当您希望提供一个范围仅限于单个窗口小部件且无法通过父级`BlocProvider`和当前类访问的bloc时，才指定该bloc `BuildContext`。

```dart
BlocBuilder<BlocA, BlocAState>(
  cubit: blocA, // provide the local cubit instance
  builder: (context, state) {
    // return widget here based on BlocA's state
  }
)
```

为了对何时`builder`调用该函数进行细粒度的控制，`buildWhen`可以提供一个可选的选项。`buildWhen`获取先前的块状态和当前的块状态并返回一个布尔值。如果`buildWhen`返回true，`builder`将使用进行调用，`state`并且小部件将重新生成。如果`buildWhen`返回false，`builder`则不会调用`state`且不会进行重建。

```dart
BlocBuilder<BlocA, BlocAState>(
  buildWhen: (previousState, state) {
    // return true/false to determine whether or not
    // to rebuild the widget with state
  },
  builder: (context, state) {
    // return widget here based on BlocA's state
  }
)
```

## BlocProvider

**BlocProvider**是Flutter小部件，可通过为其子**元素**提供块`BlocProvider.of<T>(context)`。它用作依赖项注入（DI）小部件，以便可以将一个块的单个实例提供给子树中的多个小部件。

在大多数情况下，`BlocProvider`应使用它来创建新的bloc，这些bloc将可用于其余子树。在这种情况下，由于`BlocProvider`负责创建块，它将自动处理关闭bloc。

```dart
BlocProvider(
  create: (BuildContext context) => BlocA(),
  child: ChildA(),
);
```

默认情况下，`BlocProvider`将懒惰地创建bloc，这意味着`create`当通过查找块时将执行该bloc `BlocProvider.of<BlocA>(context)`。

要覆盖此行为并强制`create`立即运行，`lazy`可以将其设置为`false`。

```dart
BlocProvider(
  lazy: false,
  create: (BuildContext context) => BlocA(),
  child: ChildA(),
);
```

在某些情况下，`BlocProvider`可用于向小部件树的新部分提供现有的bloc。当需要将现有bloc用于新路线时，这将是最常用的。在这种情况下，`BlocProvider`由于不会创建bloc，因此不会自动关闭该bloc。

```dart
BlocProvider.value(
  value: BlocProvider.of<BlocA>(context),
  child: ScreenA(),
);
```

然后从`ChildA`或`ScreenA`中检索`BlocA`：

```dart
// with extensions
context.read<BlocA>();

// without extensions
BlocProvider.of<BlocA>(context)复制到剪贴板错误复制的
```

## MultiBlocProvider

**MultiBlocProvider**是Flutter小部件，可将多个`BlocProvider`小部件合并为一个。 `MultiBlocProvider`提高了可读性，消除了嵌套多个元素的需求`BlocProviders`。通过使用，`MultiBlocProvider`我们可以从：

```dart
BlocProvider<BlocA>(
  create: (BuildContext context) => BlocA(),
  child: BlocProvider<BlocB>(
    create: (BuildContext context) => BlocB(),
    child: BlocProvider<BlocC>(
      create: (BuildContext context) => BlocC(),
      child: ChildA(),
    )
  )
)
```

至：

```dart
MultiBlocProvider(
  providers: [
    BlocProvider<BlocA>(
      create: (BuildContext context) => BlocA(),
    ),
    BlocProvider<BlocB>(
      create: (BuildContext context) => BlocB(),
    ),
    BlocProvider<BlocC>(
      create: (BuildContext context) => BlocC(),
    ),
  ],
  child: ChildA(),
)
```

## BlocListener

**BlocListener**是Flutter小部件，它带有`BlocWidgetListener`和可选，`Bloc`并调用，`listener`以响应bloc中的状态变化。它应用于需要在每次状态更改时发生一次的功能，例如导航，显示a `SnackBar`，显示a`Dialog`等。

```
listener`与in和函数不同，每次状态更改（**不**包括初始状态）仅被调用一次。`builder``BlocBuilder``void
```

如果省略cubit参数，`BlocListener`将使用`BlocProvider`和当前函数自动执行查找`BuildContext`。

```dart
BlocListener<BlocA, BlocAState>(
  listener: (context, state) {
    // do stuff here based on BlocA's state
  },
  child: Container(),
)
```

仅当您希望提供无法通过`BlocProvider`和当前访问的bloc时，才指定该bloc `BuildContext`。

```dart
BlocListener<BlocA, BlocAState>(
  cubit: blocA,
  listener: (context, state) {
    // do stuff here based on BlocA's state
  },
  child: Container()
)
```

为了对何时`listener`调用该函数进行细粒度的控制，`listenWhen`可以提供一个可选的选项。`listenWhen`获取先前的bloc状态和当前的bloc状态并返回一个布尔值。如果`listenWhen`返回true，`listener`将使用调用`state`。如果`listenWhen`返回false，`listener`则不会调用`state`。

```dart
BlocListener<BlocA, BlocAState>(
  listenWhen: (previousState, state) {
    // return true/false to determine whether or not
    // to call listener with state
  },
  listener: (context, state) {
    // do stuff here based on BlocA's state
  },
  child: Container(),
)
```

## MultiBlocListener

**MultiBlocListener**是Flutter小部件，可将多个`BlocListener`小部件合并为一个。 `MultiBlocListener`提高了可读性，消除了嵌套多个元素的需求`BlocListeners`。通过使用，`MultiBlocListener`我们可以从：

```dart
BlocListener<BlocA, BlocAState>(
  listener: (context, state) {},
  child: BlocListener<BlocB, BlocBState>(
    listener: (context, state) {},
    child: BlocListener<BlocC, BlocCState>(
      listener: (context, state) {},
      child: ChildA(),
    ),
  ),
)
```

至：

```dart
MultiBlocListener(
  listeners: [
    BlocListener<BlocA, BlocAState>(
      listener: (context, state) {},
    ),
    BlocListener<BlocB, BlocBState>(
      listener: (context, state) {},
    ),
    BlocListener<BlocC, BlocCState>(
      listener: (context, state) {},
    ),
  ],
  child: ChildA(),
)
```

## BlocConsumer

**BlocConsumer**公开`builder`和`listener`以便对新状态做出反应。`BlocConsumer`与嵌套类似`BlocListener`，`BlocBuilder`但减少了所需的样板数量。`BlocConsumer`仅应在需要重建UI和执行其他对状态更改进行响应的情况下使用`cubit`。`BlocConsumer`取需要`BlocWidgetBuilder`和`BlocWidgetListener`和任选的`cubit`，`BlocBuilderCondition`和`BlocListenerCondition`。

如果`cubit`省略该参数，`BlocConsumer`将使用`BlocProvider`和当前函数自动执行查找 `BuildContext`。

```dart
BlocConsumer<BlocA, BlocAState>(
  listener: (context, state) {
    // do stuff here based on BlocA's state
  },
  builder: (context, state) {
    // return widget here based on BlocA's state
  }
)
```

可选的`listenWhen`，`buildWhen`可以实现，以更精细地控制何时`listener`和`builder`被调用。在`listenWhen`和`buildWhen`将在每个被调用`cubit` `state`的变化。它们各自采用先前的`state`和当前的，`state`并且必须返回a `bool`，以确定是否将调用`builder`and / or`listener`函数。以前`state`会被初始化为`state`的`cubit`的时候`BlocConsumer`被初始化。`listenWhen`并且`buildWhen`是可选的，如果未实现，则默认为`true`。

```dart
BlocConsumer<BlocA, BlocAState>(
  listenWhen: (previous, current) {
    // return true/false to determine whether or not
    // to invoke listener with state
  },
  listener: (context, state) {
    // do stuff here based on BlocA's state
  },
  buildWhen: (previous, current) {
    // return true/false to determine whether or not
    // to rebuild the widget with state
  },
  builder: (context, state) {
    // return widget here based on BlocA's state
  }
)
```

## RepositoryProvider

**RepositoryProvider**是Flutter小部件，它通过为其子节点提供存储库`RepositoryProvider.of<T>(context)`。它用作依赖项注入（DI）小部件，以便可以将存储库的单个实例提供给子树中的多个小部件。`BlocProvider`应该用于提供块，而`RepositoryProvider`只能用于存储库。

```dart
RepositoryProvider(
  create: (context) => RepositoryA(),
  child: ChildA(),
);
```

然后`ChildA`我们可以通过以下方式检索`Repository`实例：

```dart
// with extensions
context.read<RepositoryA>();

// without extensions
RepositoryProvider.of<RepositoryA>(context)
```

## MultiRepositoryProvider

**MultiRepositoryProvider**是Flutter小部件，将多个`RepositoryProvider`小部件合并为一个。 `MultiRepositoryProvider`提高了可读性，消除了嵌套多个元素的需求`RepositoryProvider`。通过使用，`MultiRepositoryProvider`我们可以从：

```dart
RepositoryProvider<RepositoryA>(
  create: (context) => RepositoryA(),
  child: RepositoryProvider<RepositoryB>(
    create: (context) => RepositoryB(),
    child: RepositoryProvider<RepositoryC>(
      create: (context) => RepositoryC(),
      child: ChildA(),
    )
  )
)
```

至：

```dart
MultiRepositoryProvider(
  providers: [
    RepositoryProvider<RepositoryA>(
      create: (context) => RepositoryA(),
    ),
    RepositoryProvider<RepositoryB>(
      create: (context) => RepositoryB(),
    ),
    RepositoryProvider<RepositoryC>(
      create: (context) => RepositoryC(),
    ),
  ],
  child: ChildA(),
)
```

# 最后

## 相关地址

- Cubit范例代码地址
  - [Cubit范例代码](https://github.com/CNAD666/ExampleCode/tree/master/Flutter/flutter_use)
- Bloc范例代码地址
  - [Bloc范例代码](https://github.com/CNAD666/book_manage)
- flutter_bloc相关Api白嫖地址
  - [flutter_bloc相关Api](https://bloclibrary.dev/#/flutterbloccoreconcepts?id=bloc-widgets)
- flutter_bloc
  - GitHub：[https://github.com/felangel/bloc](https://github.com/felangel/bloc)
  - Pub：[https://pub.dev/packages/flutter_bloc](https://pub.dev/packages/flutter_bloc)

## Flutter状态管理和Dialog问题

- 可以看看下面的文章，相信对你应该有所帮助的

[fish_redux使用详解---看完就会用！](https://juejin.cn/post/6860029460524040199)

[一种更优雅的Flutter Dialog解决方案](https://juejin.cn/post/6902331428072390663)