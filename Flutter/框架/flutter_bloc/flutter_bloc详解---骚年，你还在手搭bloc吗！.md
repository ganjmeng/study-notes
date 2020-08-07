# flutter_bloc使用详解---骚年，你还在手搭bloc吗！

## 前言
首先，有很多的文章在说flutter bloc模式的应用，但是百分之八九十的文章都是在说，使用StreamController+StreamBuilder搭建bloc，提升性能的会加上InheritedWidget，这些文章看了很多，真正写使用bloc作者开发的flutter_bloc却少之又少。没办法，只能去bloc的github上去找使用方式，最后去bloc官网翻文档。
蛋痛，各位叼毛，就不能好好说说flutter_bloc的使用吗？非要各种抄bloc模式提出作者的那俩篇文章。现在，搞的杂家这个伸手党要自己去翻文档总结（手动滑稽）。
![](https://raw.githubusercontent.com/CNAD666/MyData/master/pic/study/20200802142049.png)

##### 来来，伸手党们，赶紧保存下面的链接

- Flutter_Bloc起源：[https://www.didierboelens.com/2018/08/reactive-programming-streams-bloc/](https://www.didierboelens.com/2018/08/reactive-programming-streams-bloc/)
- Flutter_Bloc模式优化：[https://www.didierboelens.com/2018/12/reactive-programming-streams-bloc-practical-use-cases/](https://www.didierboelens.com/2018/12/reactive-programming-streams-bloc-practical-use-cases/)
- Flutter_Bloc诞生：[https://medium.com/flutter-community/flutter-bloc-package-295b53e95c5c](https://medium.com/flutter-community/flutter-bloc-package-295b53e95c5c)
- Flutter_Bloc官网文档：[https://bloclibrary.dev/#/](https://bloclibrary.dev/#/)

前面三个，是bloc作者写的bloc模式文档，典型的观察者模式的应用，最原始的就是java中CallBack形式。前俩篇文章就是咱们这些大抄子的主要“参考”的资料来源，这三篇文章在掘金上有翻译版，搜下bloc就能找到。最后一篇文章就是我主要总结归纳的源泉，作者在官网上写了好几个demo：计时器，登录，Todos，天气等等，大家可以自己去看看。
## 效果

- 好了，哔哔了一堆，看下咱们要用flutter_bloc实现的效果。
  <img src="https://raw.githubusercontent.com/CNAD666/MyData/master/pic/study/20200802140721.gif" style="zoom:80%;" />

- 直接开Chrome演示，大家在虚拟机上跑也一样。
## 开搞

- 先说明下，bloc给的api很多，不同的api针对与解决场景不同，我要是把官网那些api全抄过也没啥意义；不，也有可能可以装币，我要是不说明，大家说不定以为是我自己总结的呢！哈哈。
- OK，大家要是想知道全场景的使用，可以去官网翻翻文档，我觉得学习一个模式或者框架的时候，最主要的是把主流程跑通，起码可以符合标准的堆页面，这样的话，就可以把这玩意用起来，再遇到想要的什么细节，就可以自己去翻文档，毕竟大体上已经懂了，写过了几个页面，也有些体会，再去翻文档就很快能理解了。
### 引用
#### 库
```dart
flutter_bloc: ^6.0.1 #状态管理框架爱
equatable: ^1.2.3 #增强组件相等性判断
```

- 看看flutter_bloc都推到6.0了，别再用StreamController手搭Bloc了！
#### 插件
在Android Studio设置的Plugins里，搜索：Bloc
<img src="https://raw.githubusercontent.com/CNAD666/MyData/master/pic/study/20200802171420.png" style="zoom: 67%;" />
安装重启下，就OK了

- 右击相应的文件夹，选择“Bloc Class”，我在main文件夹新建的，填入的名字：main，就自动生成下面三个文件；：main_bloc，main_event，main_state；main_view是我自己新建，用来写页面的。

<img src="https://raw.githubusercontent.com/CNAD666/MyData/master/pic/study/20200802140719.png"  style="zoom:80%;" />

<img src="https://raw.githubusercontent.com/CNAD666/MyData/master/pic/study/20200802141810.png" style="zoom:80%;" />

- 是不是觉得，还在手动新建这些bloc文件low爆了；就好像fish_redux，不用插件，让我手动去创建那五个文件，写那些模板代码，真的要原地爆炸。

### flutter_bloc
#### 生成文件代码
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
#### 实现

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
### 搞定

- OK，这样改了下，简化了state的调用，state里面少些了很多类；这边只是灵活变动了下，具体的实现，大家结合场景去实现就行了。
- 大家保持观察者模式的思想就行了；观察者（回调刷新控件）和被观察者（产生相应事件，添加事件，去通知观察者），bloc层是处于观察者和被观察者中间的一层，我们可以在bloc里面搞业务，搞逻辑，搞网络请求，不能搞基；拿到Event事件传递过来的数据，把处理好的、符合要求的数据返回给view层的观察者就行了。
- 使用框架，不拘泥框架，在观察者模式的思想上，灵活的去使用flutter_bloc提供Api，这样可以大大的缩短我们的开发时间！

<img src="https://raw.githubusercontent.com/CNAD666/MyData/master/pic/study/20200802141412.png" />

## 最后

- Bloc还有很多Api针对不同的场景非常的实用，例如：MultiBlocProvider，BlocListener，MultiBlocListener，BlocConsumer等等，这里面有些Api和Provider的Api是非常相似的，例如MultiXxxxx，这都是为了减少嵌套，提供多个全局Bloc而提供，大家可以去瞧瞧看，用法也都非常的相似
- flutter_bloc相关Api白嫖地址
   - [https://bloclibrary.dev/#/flutterbloccoreconcepts?id=bloc-widgets](https://bloclibrary.dev/#/flutterbloccoreconcepts?id=bloc-widgets)
- flutter_redux
   - GitHub：[https://github.com/felangel/bloc](https://github.com/felangel/bloc)
   - Pub：[https://pub.dev/packages/flutter_bloc](https://pub.dev/packages/flutter_bloc)
