## fish_redux使用解析---看完就会用！

## 前言

来学学难搞的fish_redux框架吧，这个框架，官方的文档真是一言难尽，比flutter_bloc官网的文档真是逊色太多了，但是一旦知道怎么写，页面堆起来也是非常爽呀，结构分明，逻辑也会错落有致。

其实在当时搞懂这个框架的时候，就一直想写一篇文章记录下，但是因为忙（lan），导致一直没写，现在觉得还是必须把使用的过程记录下，毕竟刚上手这个框架是个蛋痛的过程，必须要把这个过程做个记录。

这不仅仅是记录的文章，文中所给出的示例，也是我重新构思去写的，过程也是力求阐述清楚且详细。

![img](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/bookWeb/20200808161402.jpg)

### 几个问题点

- 页面切换的转场动画
- 页面怎么更新数据
- fish_redux各个模块之间，怎么传递数据
- 页面跳转传值，及其接受下个页面回传的值
- 怎么配合ListView使用
- ListView怎么使用adapter，数据怎么和item绑定
- 怎么将Page当做widget使用（BottomNavigationBar，NavigationRail等等导航栏控件会使用到）
  - 这个直接使用：XxxPage.buildPage(null) 即可

如果你在使用fish_redux的过程中遇到过上述的问题，那就来看看这篇文章吧！这里，会解答上面所有的问题点！

**偶现bug**

- 如果大家运行在web上面，遇到下面错误，建议把flutter版本切换到dev版：flutter channel dev
  - 我在beta版上遇到了这个问题，在github上找到了一个解决办法，别人是切到master分支上，我切到dev分支也解决了这个问题，这个bug非必现

```dart
The method 'toFilePath' was called on null.
```

## 准备

### 引入

**fish_redux相关地址**

- GitHub地址：https://github.com/alibaba/fish-redux
- Pub地址：https://pub.dev/packages/fish_redux

我用的是0.3.X的版本，算是第三版，相对于前几版，改动较大

- 引入fish_redux插件，想用最新版插件，可进入pub地址里面查看

```
fish_redux: ^0.3.4
#演示列表需要用到的库
dio: ^3.0.9    #网络请求框架
json_annotation: ^2.4.0 #json序列化和反序列化用的
```

### 开发插件

- 此处我们需要安装代码生成插件，可以帮我们生成大量文件和模板代码

- 在Android Studio里面搜索”fish“就能搜出插件了，插件名叫：FishReduxTemplate

  ![image-20200808181112391](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/bookWeb/20200808233038.png)

- BakerJQ编写：[Android Studio的Fish Redux模板](https://github.com/BakerJQ/FishReduxTemplateForAS)。
- huangjianke编写：[VSCode的Fish Redux模板](https://github.com/huangjianke/fish-redux-template)

### 创建

- 这里我在新建的count文件夹上，选择新建文件，选择：New ---> FishReduxTemplate

![image-20200808181242775](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/bookWeb/20200808233050.png)

- 此处选择：Page，底下的“Select Fils”全部选择，这是标准的redux文件结构；这边命名建议使用大驼峰：Count
  - Component：这个一般是可复用的相关的组件；列表的item，也可以选择这个
  - Adapter：这里有三个Adapter，都可以不用了；fish_redux第三版推出了功能更强大的adapter，更加灵活的绑定方式

![image-20200808181325258](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/bookWeb/20200808233056.png)

- 创建成功后，记得在创建的文件夹上右击，选择：Reload From Disk；把创建的文件刷新出来

![image-20200808181410600](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/bookWeb/20200808233101.png)

- 创建成功的文件结构
  - page：总页面，注册effect，reducer，component，adapter的功能，相关的配置都在此页面操作
  - state：这地方就是我们存放子模块变量的地方；初始化变量和接受上个页面参数，也在此处，是个很重要的模块
  - view：主要是我们写页面的模块
  - action：这是一个非常重要的模块，所有的事件都在此处定义和中转
  - effect：相关的业务逻辑，网络请求等等的“副作用”操作，都可以写在该模块
  - reducer：该模块主要是用来更新数据的，也可以写一些简单的逻辑或者和数据有关的逻辑操作

![image-20200808181550186](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/bookWeb/20200808233107.png)

- OK，至此就把所有的准备工作搞定了，下面可以开搞代码了

![img](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/bookWeb/20200808233113.jpg)

## 开发流程

### redux流程

- 下图是阮一峰老师博客上放的redux流程图

![img](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/bookWeb/20200808233125.jpeg)

### fish_redux流程

- 在写代码前，先看写下流程图，这图是凭着自己的理解画的
  - 可以发现，事件的传递，都是通过dispatch这个方法，而且action这层很明显是非常关键的一层，事件的传递，都是在该层定义和中转的
  - 这图在语雀上调了半天，就在上面加了个自己的github水印地址

  ![fish_redux流程](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/bookWeb/20200808233137.jpg)
  
- 通过俩个流程图对比，其中还是有一些差别的
  - redux里面的store是全局的。fish_redux里面也有这个全局store的概念，放在子模块里面理解store，react；对应fish_redux里的就是：state，view
  - fish_redux里面多了effect层：这层主要是处理逻辑，和相关网络请求之类
  - reducer里面，理论上也是可以处理一些和数据相关，简单的逻辑；但是复杂的，会产生相应较大的“副作用”的业务逻辑，还是需要在effect中写

## 范例说明

这边写俩个个示例，来演示fish_redux的使用

- 计数器
  - fish_redux正常情况下的流转过程
  - fish_redux各模块怎么传递数据
- 列表文章
  - 网络请求数据，列表加载
  - fish_redux在ListView中使用

## 计数器

### 效果图

![fish_redux中count](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/bookWeb/20200808233256.gif)

- 这个例子演示，view中点击此操作，然后更新页面数据；下述的流程，在effect中把数据处理好，通过action中转传递给reducer更新数据
  - view ---> action ---> effect ---> reducer（更新数据）
- 注意：该流程将展示，怎么将数据在各流程中互相传递

### 标准模式

- main
  - 这地方需要注意，cupertino，material这类系统包和fish_redux里包含的“Page”类名重复了，需要在这类系统包上使用hide，隐藏系统包里的Page类
  - 关于页面的切换风格，可以在MaterialApp中的onGenerateRoute方法中，使用相应页面切换风格，这边使用ios的页面切换风格：cupertino

```dart
///需要使用hide隐藏Page
import 'package:flutter/cupertino.dart'hide Page;
import 'package:flutter/material.dart' hide Page;

void main() {
  runApp(MyApp());
}

Widget createApp() {
  ///定义路由
  final AbstractRoutes routes = PageRoutes(
    pages: <String, Page<Object, dynamic>>{
      "CountPage": CountPage(),
    },
  );

  return MaterialApp(
    title: 'FishDemo',
    home: routes.buildPage("CountPage", null), //作为默认页面
    onGenerateRoute: (RouteSettings settings) {
      //ios页面切换风格
      return CupertinoPageRoute(builder: (BuildContext context) {
        return routes.buildPage(settings.name, settings.arguments);
      })
//      Material页面切换风格
//      return MaterialPageRoute<Object>(builder: (BuildContext context) {
//        return routes.buildPage(settings.name, settings.arguments);
//      });
    },
  );
}
```

- state
  - 定义我们在页面展示的一些变量，initState中可以初始化变量；clone方法的赋值写法是必须的

```dart
class CountState implements Cloneable<CountState> {
  int count;

  @override
  CountState clone() {
    return CountState()..count = count;
  }
}

CountState initState(Map<String, dynamic> args) {
  return CountState()..count = 0;
}
```

- view：这里面就是写界面的模块，buildView里面有三个参数
  - state：这个就是我们的数据层，页面需要的变量都写在state层
  - dispatch：类似调度器，调用action层中的方法，从而去回调effect，reducer层的方法
  - viewService：这个参数，我们可以使用其中的方法：buildComponent("组件名")，调用我们封装的相关组件

```dart
Widget buildView(CountState state, Dispatch dispatch, ViewService viewService) {
  return _bodyWidget(state, dispatch);
}

Widget _bodyWidget(CountState state, Dispatch dispatch) {
  return Scaffold(
    appBar: AppBar(
      title: Text("FishRedux"),
    ),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          Text('You have pushed the button this many times:'),
          ///使用state中的变量，控住数据的变换
          Text(state.count.toString()),
        ],
      ),
    ),
    floatingActionButton: FloatingActionButton(
      onPressed: () {
        ///点击事件，调用action 计数自增方法
        dispatch(CountActionCreator.countIncrease());
      },
      child: Icon(Icons.add),
    ),
  );
}
```

- action
  - 该层是非常重要的模块，页面所有的行为都可以在本层直观的看到
  - XxxxAction中的枚举字段是必须的，一个事件对应有一个枚举字段，枚举字段是：effect，reducer层标识的入口
  - XxxxActionCreator类中的方法是中转方法，方法中可以传参数，参数类型可任意；方法中的参数放在Action类中的payload字段中，然后在effect，reducer中的action参数中拿到payload值去处理就行了
  - 这地方需要注意下，默认生成的模板代码，return的Action类加了const修饰，如果使用Action的payload字段赋值并携带数据，是会报错的；所以这里如果需要携带参数，请去掉const修饰关键字

```dart
enum CountAction { increase, updateCount }

class CountActionCreator {
  ///去effect层去处理自增数据
  static Action countIncrease() {
    return Action(CountAction.increase);
  }
  ///去reducer层更新数据，传参可以放在Action类中的payload字段中，payload是dynamic类型，可传任何类型
  static Action updateCount(int count) {
    return Action(CountAction.updateCount, payload: count);
  }
}
```

- effect
  - 如果在调用action里面的XxxxActionCreator类中的方法，相应的枚举字段，会在combineEffects中被调用，在这里，我们就能写相应的方法处理逻辑，方法中带俩个参数：action，ctx
    - action：该对象中，我们可以拿到payload字段里面，在action里面保存的值
    - ctx：该对象中，可以拿到state的参数，还可以通过ctx调用dispatch方法，调用action中的方法，在这里调用dispatch方法，一般是把处理好的数据，通过action中转到reducer层中更新数据

```dart
Effect<CountState> buildEffect() {
  return combineEffects(<Object, Effect<CountState>>{
    CountAction.increase: _onIncrease,
  });
}
///自增数
void _onIncrease(Action action, Context<CountState> ctx) {
  ///处理自增数逻辑
  int count = ctx.state.count + 1;
  ctx.dispatch(CountActionCreator.updateCount(count));
}
```

- reducer
  - 该层是更新数据的，action中调用的XxxxActionCreator类中的方法，相应的枚举字段，会在asReducer方法中回调，这里就可以写个方法，克隆state数据进行一些处理，这里面有俩个参数：state，action
  - state参数经常使用的是clone方法，clone一个新的state对象；action参数基本就是拿到其中的payload字段，将其中的值，赋值给state

```dart
Reducer<CountState> buildReducer() {
  return asReducer(
    <Object, Reducer<CountState>>{
      CountAction.updateCount: _updateCount,
    },
  );
}
///通知View层更新界面
CountState _updateCount(CountState state, Action action) {
  final CountState newState = state.clone();
  newState..count = action.payload;
  return newState;
}
```

- page模块不需要改动，这边就不贴代码了

### 优化

- 从上面的例子看到，如此简单数据变换，仅仅是个state中一个参数自增的过程，effect层就显得有些多余；所以，把流程简化成下面
  - view ---> action ---> reducer

- 注意：这边把effect层删掉，该层可以舍弃了；然后对view，action，reducer层代码进行一些小改动

**搞起来**

- view
  - 这边仅仅把点击事件的方法，微微改了下：CountActionCreator.countIncrease()改成CountActionCreator.updateCount()

```dart
Widget buildView(CountState state, Dispatch dispatch, ViewService viewService) {
  return _bodyWidget(state, dispatch);
}

Widget _bodyWidget(CountState state, Dispatch dispatch) {
  return Scaffold(
    appBar: AppBar(
      title: Text("FishRedux"),
    ),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          Text('You have pushed the button this many times:'),
          Text(state.count.toString()),
        ],
      ),
    ),
    floatingActionButton: FloatingActionButton(
      onPressed: () {
        ///点击事件，调用action 计数自增方法
        dispatch(CountActionCreator.updateCount());
      },
      child: Icon(Icons.add),
    ),
  );
}
```

- action
  - 这里只使用一个枚举字段，和一个方法就行了，也不用传啥参数了

```dart
enum CountAction { updateCount }

class CountActionCreator {
  ///去reducer层更新数据，传参可以放在Action类中的payload字段中，payload是dynamic类型，可传任何类型
  static Action updateCount() {
    return Action(CountAction.updateCount);
  }
}
```

- reducer
  - 这里直接在：_updateCount方法中处理下简单的自增逻辑

```dart
Reducer<CountState> buildReducer() {
  return asReducer(
    <Object, Reducer<CountState>>{
      CountAction.updateCount: _updateCount,
    },
  );
}
///通知View层更新界面
CountState _updateCount(CountState state, Action action) {
  final CountState newState = state.clone();
  newState..count = state.count + 1;
  return newState;
}
```

### 搞定

- 可以看见优化了后，代码量减少了很多，对待不同的业务场景，可以灵活的变动，使用框架，但不要拘泥框架；但是如果有网络请求，很复杂的业务逻辑，就万万不能写在reducer里面了，一定要写在effect中，这样才能保证一个清晰的解耦结构，保证处理数据和更新数据过程分离

![img](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/bookWeb/20200808234825.jpg)

## 列表文章

- 我们堆页面的过程中，能体会列表模块是非常重要的一部分，现在就来学学，在fish_redux中怎么使用ListView吧！

  - 废话少说，上号！

![00685430](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/bookWeb/20200808234813.png)

### 效果图

![fish_redux中list](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200812171610.gif)

- 效果图对于列表的滚动，做了俩个操作：一个是拖拽列表；另一个是滚动鼠标的滚轮。flutter对鼠标触发的相关事件也支持的越来越好了！
  - 这边我们使用的是玩Android的api，这个api有个坑的地方，没设置开启跨域，所以运行在web上，这个api使用会报错，我在玩Android的github上提了issue，哎，也不知道作者啥时候解决，，，
- 这地方只能曲线救国，关闭浏览器跨域限制，设置看这里：https://www.jianshu.com/p/56b1e01e6b6a
- 如果运行在虚拟机上，就完全不会出现这个问题！

### 准备

- 先看下文件结构

![image-20200810225418771](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200812171629.png)

### 实现

- main
  - 这边改动非常小，只在路由里，新增了：GuidePage，ListPage；同时将home字段中的默认页面，改成了：GuidePage页面；导航页面代码就不贴在文章里了，下面贴下该页面链接
    - https://github.com/CNAD666/ExampleCode/tree/master/Flutter/fish_redux_demo/lib/guide
  - ListPage才是重点，下文会详细说明

```dart
void main() {
  runApp(createApp());
}

Widget createApp() {
  ///定义路由
  final AbstractRoutes routes = PageRoutes(
    pages: <String, Page<Object, dynamic>>{
      ///导航页面
      "GuidePage": GuidePage(),
      ///计数器模块演示
      "CountPage": CountPage(),
      ///页面传值跳转模块演示
      "FirstPage": FirstPage(),
      "SecondPage": SecondPage(),
      ///列表模块演示
      "ListPage": ListPage(),
    },
  );

  return MaterialApp(
    title: 'FishRedux',
    home: routes.buildPage("GuidePage", null), //作为默认页面
    onGenerateRoute: (RouteSettings settings) {
      //ios页面切换风格
      return CupertinoPageRoute(builder: (BuildContext context) {
        return routes.buildPage(settings.name, settings.arguments);
      });
    },
  );
}
```

#### 流程

- Adapter实现的流程
  - 创建item(Component) ---> 创建adapter文件 --->  state集成相应的Source ---> page里面绑定adapter
- 通过以上四步，就能在fish_redux使用相应列表里面的adapter了，过程有点麻烦，但是熟能生巧，多用用就能很快搭建一个复杂的列表了
- 总流程：初始化列表模块 ---> item模块 ---> 列表模块逻辑完善
  - **初始化列表模块**
    - 这个就是正常的创建fish_redux模板代码和文件
  - **item模块**
    - 根据接口返回json，创建相应的bean ---> 创建item模块 ---> 编写state ---> 编写view界面 
  - **列表模块逻辑完善**：俩地方分俩步（adapter创建及其绑定，正常page页面编辑）
    - 创建adapter文件 ---> state调整 ---> page中绑定adapter
    - view模块编写 ---> action添加更新数据事件 ---> effect初始化时获取数据并处理 ---> reducer更新数据
- 整体流程确实有些多，但是咱们按照整体三步流程流程走，保证思路清晰就行了

#### 初始化列表模块

- 此处新建个文件夹，在文件夹上新建fis_redux文件就行了；这地方，我们选择page，整体的五个文件：action，effect，reducer，state，view；全部都要用到，所以默认全选，填入Module的名字，点击OK

![image-20200812140314075](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200812171706.png)

#### item模块

**按照流程走**

- 根据接口返回json，创建相应的bean ---> 创建item模块 ---> 编写state ---> 编写view界面 

**准备工作**

- 创建bean实体
  - 根据api返回的json数据，生成相应的实体
    - api：https://www.wanandroid.com/project/list/1/json
  - json转实体
    - 网站：https://javiercbk.github.io/json_to_dart/
    - 插件：AS中可以搜索：FlutterJsonBeanFactory
  - 这地方生成了：ItemDetailBean；代码俩百多行就不贴了，具体的内容，点击下面链接
    - ItemDetailBean代码：https://github.com/CNAD666/ExampleCode/blob/master/Flutter/fish_redux_demo/lib/list/bean/item_detail_bean.dart

- 创建item模块
  - 这边我们实现一个简单的列表，item仅仅做展示功能；不做点击，更新ui等操作，所以这边我们就不需要创建：effect，reducer，action文件；只选择：state和view就行了
  - 创建item，这里选择component

  ![image-20200810225523628](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200812171718.png)

**文件结构**

![image-20200812141416796](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200812171729.png)

OK，bean文件搞定了，再来看看，item文件中的文件，这里component文件不需要改动，所以这地方，我们只需要看：state.dart，view.dart

- state
  - 这地方还是常规的写法，因为json生成的bean里面，能用到的所有数据，都在Datas类里面，所以，这地方建一个Datas类的变量即可
  - 因为，没用到reducer，实际上clone实现方法都能删掉，防止后面可能需要clone对象，暂且留着

```dart
import 'package:fish_redux/fish_redux.dart';
import 'package:fish_redux_demo/list/bean/item_detail_bean.dart';

class ItemState implements Cloneable<ItemState> {
  Datas itemDetail;

  ItemState({this.itemDetail});

  @override
  ItemState clone() {
    return ItemState()
        ..itemDetail = itemDetail;
  }
}

ItemState initState(Map<String, dynamic> args) {
  return ItemState();
}
```

- view
  - 这里item布局稍稍有点麻烦，整体上采用的是：水平布局（Row），分左右俩大块
    - 左边：单纯的图片展示
    - 右边：采用了纵向布局（Column），结合Expanded形成比例布局，分别展示三块东西：标题，内容，作者和时间
  - OK，这边view只是简单用到了state提供的数据形成的布局，没有什么要特别注意的地方

```dart
Widget buildView(ItemState state, Dispatch dispatch, ViewService viewService) {
  return _bodyWidget(state);
}

Widget _bodyWidget(ItemState state) {
  return Card(
    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(20)),
    elevation: 5,
    margin: EdgeInsets.only(left: 20, right: 20, top: 20),
    child: Row(
      children: <Widget>[
        //左边图片
        Container(
          margin: EdgeInsets.all(10),
          width: 180,
          height: 100,
          child: Image.network(
            state.itemDetail.envelopePic,
            fit: BoxFit.fill,
          ),
        ),
        //右边的纵向布局
        _rightContent(state),
      ],
    ),
  );
}

///item中右边的纵向布局,比例布局
Widget _rightContent(ItemState state) {
  return Expanded(
      child: Container(
    margin: EdgeInsets.all(10),
    height: 120,
    child: Column(
      mainAxisAlignment: MainAxisAlignment.start,
      children: <Widget>[
        //标题
        Expanded(
          flex: 2,
          child: Container(
            alignment: Alignment.centerLeft,
            child: Text(
              state.itemDetail.title,
              style: TextStyle(fontSize: 16),
              maxLines: 1,
              overflow: TextOverflow.ellipsis,
            ),
          ),
        ),
        //内容
        Expanded(
            flex: 4,
            child: Container(
              alignment: Alignment.centerLeft,
              child: Text(
                state.itemDetail.desc,
                style: TextStyle(fontSize: 12),
                maxLines: 3,
                overflow: TextOverflow.ellipsis,
              ),
            )),
        Expanded(
          flex: 3,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.end,
            children: <Widget>[
              //作者
              Row(
                children: <Widget>[
                  Text("作者：", style: TextStyle(fontSize: 12)),
                  Expanded(
                    child: Text(state.itemDetail.author,
                        style: TextStyle(color: Colors.blue, fontSize: 12),
                        overflow: TextOverflow.ellipsis),
                  )
                ],
              ),
              //时间
              Row(children: <Widget>[
                Text("时间：", style: TextStyle(fontSize: 12)),
                Expanded(
                  child: Text(state.itemDetail.niceDate,
                      style: TextStyle(color: Colors.blue, fontSize: 12),
                      overflow: TextOverflow.ellipsis),
                )
              ])
            ],
          ),
        ),
      ],
    ),
  ));
}
```

item模块，就这样写完了，不需要改动什么了，接下来看看List模块

#### 列表模块逻辑完善

首先最重要的，我们需要将adapter建立起来，并和page绑定

- 创建adapter文件 ---> state调整 ---> page中绑定adapter

**adapter创建及其绑定**

- 创建adapter
  - 首先需要创建adapter文件，然后写入下面代码：这地方需要继承SourceFlowAdapter适配器，里面的泛型需要填入ListState，ListState这地方会报错，因为我们的ListState没有继承MutableSource，下面state的调整就是对这个的处理
  - ListItemAdapter的构造函数就是通用的写法了，在super里面写入我们上面写好item样式，这是个pool应该可以理解为样式池，这个key最好都提出来，因为在state模块还需要用到，可以定义多个不同的item，很容易做成多样式item的列表；目前，我们这边只需要用一个，填入：ItemComponent()

```dart
class ListItemAdapter extends SourceFlowAdapter<ListState> {
  static const String item_style = "project_tab_item";

  ListItemAdapter()
      : super(
          pool: <String, Component<Object>>{
            ///定义item的样式
            item_style: ItemComponent(),
          },
        );
}
```

- state调整
  - state文件中的代码需要做一些调整，需要继承相应的类，和adapter建立起关联
  - ListState需要继承MutableSource；还必须定义一个泛型是item的ItemState类型的List，这俩个是必须的；然后实现相应的抽象方法就行了
  - 这里只要向items里写入ItemState的数据，列表就会更新了

```dart
class ListState extends MutableSource implements Cloneable<ListState> {
  ///这地方一定要注意,List里面的泛型,需要定义为ItemState
  ///怎么更新列表数据,只需要更新这个items里面的数据,列表数据就会相应更新
  List<ItemState> items;

  @override
  ListState clone() {
    return ListState()..items = items;
  }

  ///使用上面定义的List,继承MutableSource,就把列表和item绑定起来了
  @override
  Object getItemData(int index) => items[index];

  @override
  String getItemType(int index) => ListItemAdapter.item_style;

  @override
  int get itemCount => items.length;

  @override
  void setItemData(int index, Object data) {
    items[index] = data;
  }
}

ListState initState(Map<String, dynamic> args) {
  return ListState();
}
```

-  page中绑定adapter
  - 这里就是将我们的ListSate和ListItemAdapter适配器建立起连接

```dart
class ListPage extends Page<ListState, Map<String, dynamic>> {
  ListPage()
      : super(
          initState: initState,
          effect: buildEffect(),
          reducer: buildReducer(),
          view: buildView,
          dependencies: Dependencies<ListState>(
              ///绑定Adapter
              adapter: NoneConn<ListState>() + ListItemAdapter(),
              slots: <String, Dependent<ListState>>{}),
          middleware: <Middleware<ListState>>[],
        );
}
```

**正常page页面编辑**

整体流程

- view模块编写 ---> action添加更新数据事件 ---> effect初始化时获取数据并处理 ---> reducer更新数据

- view
  - 这里面的列表使用就相当简单了，填入itemBuilder和itemCount参数就行了，这里就需要用viewService参数了哈

```dart
Widget buildView(ListState state, Dispatch dispatch, ViewService viewService) {
  return Scaffold(
    appBar: AppBar(
      title: Text("ListPage"),
    ),
    body: _itemWidget(state, viewService),
  );
}

Widget _itemWidget(ListState state, ViewService viewService) {
  if (state.items != null) {
    ///使用列表
    return ListView.builder(
      itemBuilder: viewService.buildAdapter().itemBuilder,
      itemCount: viewService.buildAdapter().itemCount,
    );
  } else {
    return Center(
      child: CircularProgressIndicator(),
    );
  }
}
```

- action
  - 只需要写个更新items的事件就ok了

```dart
enum ListAction { updateItem }

class ListActionCreator {
  static Action updateItem(var list) {
    return Action(ListAction.updateItem, payload: list);
  }
}
```

- effect
  - Lifecycle.initState是进入页面初始化的回调，这边可以直接用这个状态回调，来请求接口获取相应的数据，然后去更新列表
  - 这地方有个坑，dio必须结合json序列号和反序列的库一起用，不然Dio无法将数据源解析成Response类型

```dart
Effect<ListState> buildEffect() {
  return combineEffects(<Object, Effect<ListState>>{
    ///进入页面就执行的初始化操作
    Lifecycle.initState: _init,
  });
}

void _init(Action action, Context<ListState> ctx) async {
  String apiUrl = "https://www.wanandroid.com/project/list/1/json";
  Response response = await Dio().get(apiUrl);
  ItemDetailBean itemDetailBean =
      ItemDetailBean.fromJson(json.decode(response.toString()));
  List<Datas> itemDetails = itemDetailBean.data.datas;
  ///构建符合要求的列表数据源
  List<ItemState> items = List.generate(itemDetails.length, (index) {
    return ItemState(itemDetail: itemDetails[index]);
  });
  ///通知更新列表数据源
  ctx.dispatch(ListActionCreator.updateItem(items));
}
```

- reducer
  - 最后就是更新操作了哈，这里就是常规写法了

```dart
Reducer<ListState> buildReducer() {
  return asReducer(
    <Object, Reducer<ListState>>{
      ListAction.updateItem: _updateItem,
    },
  );
}

ListState _updateItem(ListState state, Action action) {
  return state.clone()..items = action.payload;
}
```

### 搞定

- 呼，终于将列表这块写完，说实话，这个列表的使用确实有点麻烦；实际上，如果大家用心看了的话，麻烦的地方，其实就是在这块：**adapter创建及其绑定**；只能多写写了，熟能生巧！

- 列表模块大功告成，以后就能愉快的写列表了！

![img](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200812171752.jpg)

## 最后

- 这片文章，说实话，花了不少精力去写的，也花了不少时间构思；主要是例子，必须要自己重写下，尤其在dio返回的数据不能被Response解析，那地方坑了我一下午，少引了json序列化库，他也不报错，蛋蛋

- 代码地址：https://github.com/CNAD666/ExampleCode/tree/master/Flutter/fish_redux_demo

- fish_redux版-玩Android：https://github.com/CNAD666/flutter_wan

- 大家如果觉得有收获，就给我点个赞吧！你的点赞，是我码字的最大动力！

  

![img](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200812171815.jpg)