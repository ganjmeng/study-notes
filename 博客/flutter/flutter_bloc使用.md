# flutter_bloc使用解析---骚年，你还在手搭bloc吗！

## 前言

首先，有很多的文章在说flutter bloc模式的应用，但是百分之八九十的文章都是在说，使用StreamController+StreamBuilder搭建bloc，提升性能的会加上InheritedWidget，这些文章看了很多，真正写使用bloc作者开发的flutter_bloc却少之又少。没办法，只能去bloc的github上去找使用方式，最后去bloc官网翻文档。
蛋痛，各位叼毛，就不能好好说说flutter_bloc的使用吗？非要各种抄bloc模式提出作者的那俩篇文章。现在，搞的杂家这个伸手党要自己去翻文档总结（手动滑稽）。
![表情1](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200804152051.png)

**项目效果（建议PC浏览器打开）**

- [http://cnad666.gitee.io/book_web_manage](http://cnad666.gitee.io/book_web_manage)

**来来，伸手党们，赶紧保存下面的链接**

- [Flutter_Bloc起源](https://www.didierboelens.com/2018/08/reactive-programming-streams-bloc/)

- [Flutter_Bloc模式优化](https://www.didierboelens.com/2018/12/reactive-programming-streams-bloc-practical-use-cases/)

- [Flutter_Bloc诞生](https://medium.com/flutter-community/flutter-bloc-package-295b53e95c5c)

- [Flutter_Bloc官网文档](https://bloclibrary.dev/#/)

  前面三个，是bloc作者写的bloc模式文档，典型的观察者模式的应用，最原始的就是java中CallBack形式。前俩篇文章就是咱们这些大抄子的主要“参考”的资料来源，这三篇文章在掘金上有翻译版，搜下bloc就能找到。最后一篇文章就是我主要总结归纳的源泉，作者在官网上写了好几个demo：计时器，登录，Todos，天气等等，大家可以自己去看看。

### 问题

初次使用flutter_bloc框架，可能会有几个疑问

- state里面定义了太多变量，某个事件只需要更新其中一个变量，其它的变量赋相同值麻烦
- 进入某个模块，进行初始化操作：复杂的逻辑运算，网络请求等，入口在哪定义

## 效果

- 好了，哔哔了一堆，看下咱们要用flutter_bloc实现的效果。

![bloc演示](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200804152120.gif)

- 直接开Chrome演示，大家在虚拟机上跑也一样。

## 引用

- 先说明下，bloc给的api很多，不同的api针对与解决场景不同，我要是把官网那些api全抄过也没啥意义；不，也有可能可以装币，我要是不说明，大家说不定以为是我自己总结的呢！哈哈。

- OK，大家要是想知道全场景的使用，可以去官网翻翻文档，我觉得学习一个模式或者框架的时候，最主要的是把主流程跑通，起码可以符合标准的堆页面，这样的话，就可以把这玩意用起来，再遇到想要的什么细节，就可以自己去翻文档，毕竟大体上已经懂了，写过了几个页面，也有些体会，再去翻文档就很快能理解了。

#### 库

```dart
flutter_bloc: ^6.0.1 #状态管理框架
equatable: ^1.2.3 #增强组件相等性判断
```

- 看看flutter_bloc都推到6.0了，别再用StreamController手搭Bloc了！

#### 插件

在Android Studio设置的Plugins里，搜索：Bloc

![插件搜索](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200804152204.png)

安装重启下，就OK了

- 右击相应的文件夹，选择“Bloc Class”，我在main文件夹新建的，填入的名字：main，就自动生成下面三个文件；：main_bloc，main_event，main_state；main_view是我自己新建，用来写页面的。

![新建bloc文件](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200804152227.png)

![目录结构新建bloc文件](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200804152237.png)

- 是不是觉得，还在手动新建这些bloc文件low爆了；就好像fish_redux，不用插件，让我手动去创建那六个文件，写那些模板代码，真的要原地爆炸。

## 范例

### 初始化代码

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

### 实现

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

```dart
class MainState{
   int selectedIndex;
   bool isExtended;
  
   MainState({this.selectedIndex, this.isExtended});
}
```

- 对于生成的模板代码，我们在这：去掉@immutable注解，去掉abstract；
  - 这里说下加上@immutable和abstract的作用，这边是为了标定不同状态，拿很典型的列表数据加载说明，列表加载的时候一般有三种状态
    - 获取数据前，列表的布局展示空样式：LoadingBeforeState
    - 获取数据失败，显示出加载失败的布局，或提升重新加载的样式提升：LoadingFailureState
    - 获取数据成功，显示出列表数据：LoadingSuccessState
  - 针对上面三种状态，需要展示不同的布局，这样我们就可以继承抽象的状态类：LoadingState，针对不同状态实现上面三种不同的状态类，不同的状态可以定义不同的参数，然后在view中去判断调用
  - 这种实现不同状态，对不同状态进行管理，有点设计模式中-状态模式的味道
- 下面代码是对上述描述的一种代码展示，可以瞧瞧；跑demo的时候，这下面的代码就不用抄了，仅做演示

```dart
@immutable
abstract class LoadingState extends Equatable {}
class LoadingInitial extends LoadingState {
  @override
  List<Object> get props => [];
}
class LoadingBeforeSate extends LoadingState{
  ///实现相应的字段信息
  @override
  List<Object> get props => [];
}
class LoadingFailureState extends LoadingState{
  ///实现相应的字段信息
  @override
  List<Object> get props => [];
}
class LoadingSuccessState extends LoadingState{
  ///实现相应的字段信息
  @override
  List<Object> get props => [];
}

///在View中使用,伪代码
BlocBuilder<MainBloc, MainState>(builder: (context, state) {
  if(state is LoadingBeforeSate){
	return Beforewidget(state.XX,..);
  } else if(state is LoadingFailureState){
    return FailureWidget(state.XX,state.XX,...);
  } else if(state is LoadingSuccessState){
    return SuccessWidget(state.XX);
  } else {
    return ErrorWidget(...);
  }
})
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
            context.bloc<MainBloc>().add(IsExtendEvent());
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
        context.bloc<MainBloc>().add(SwitchTabEvent(selectedIndex: index));
      },
    );
  });
}
```

## 范例优化

### 反思

从上面的代码来看，实际存在几个隐式问题，这些问题，刚开始使用时候，没异常的感觉，但是使用bloc久了后，感觉肯定越来越强烈

- state问题
  - 初始化问题：这边初始化是在bloc里，直接在构造方法里面赋初值的，state中一旦变量多了，还是这么写，会感觉极其难受，不好管理。需要优化
  - 可以看见这边我们只改动selectedIndex或者isExtended；另一个变量不需要变动，需要保持上一次的数据，进行了此类：state.selectedIndex或者state.isExtended赋值，一旦变量达到十几个乃至几十个，还是如此写，是让人极其崩溃的。需要优化
- bloc问题
  - 如果进行一个页面，需要进行复杂的运算或者请求接口后，才能知晓数据，进行赋值，这里肯定需要一个初始化入口，初始化入口需要怎样去定义呢？

### 优化实现

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
    - 项目地址：[https://bloclibrary.dev/#/flutterinfinitelisttutorial?id=flutter-infinite-list-tutorial](https://bloclibrary.dev/#/flutterinfinitelisttutorial?id=flutter-infinite-list-tutorial)

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

## 最后

- Bloc还有很多Api针对不同的场景非常的实用，例如：MultiBlocProvider，BlocListener，MultiBlocListener，BlocConsumer等等，这里面有些Api和Provider的Api是非常相似的，例如MultiXxxxx，这都是为了减少嵌套，提供多个全局Bloc而提供，大家可以去瞧瞧看，用法也都非常的相似
- 项目代码地址
  - [https://github.com/CNAD666/book_web_manage](https://github.com/CNAD666/book_web_manage)
- flutter_bloc相关Api白嫖地址
  - [https://bloclibrary.dev/#/flutterbloccoreconcepts?id=bloc-widgets](https://bloclibrary.dev/#/flutterbloccoreconcepts?id=bloc-widgets)
- flutter_redux
  - GitHub：[https://github.com/felangel/bloc](https://github.com/felangel/bloc)
  - Pub：[https://pub.dev/packages/flutter_bloc](https://pub.dev/packages/flutter_bloc)