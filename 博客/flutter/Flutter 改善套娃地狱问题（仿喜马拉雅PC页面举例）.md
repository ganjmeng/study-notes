# 前言

>  这篇文章是我一直以来很想写的一篇文章，终于下定决心动笔了。

写Flutter的小伙伴可能都感受到了：掘金的一些热门的Flutter文章下，知乎的一些Flutter的话题下或者一些论坛里面，喷Flutter套娃地狱总是永不过时的一个话题。

如果你不服气，上去辩驳俩下：“嵌套是你代码习惯问题，你看我，直接一个Row，反手一个Column，在children中把widget一提，层次分明，年轻人望你耗子尾汁，莫要瞎带节奏”；然后你可能就被一群人喷成狗，大意了，这帖子没同一阵营的小伙伴，喷不过，闪了闪了；一般被喷后，不是身经百被喷，都需要一段时间来平复心情。。。

所以，终于我下定决心把这篇文章肝出来，如果你认真看完，你可能会发现：**嵌套什么的都是浮云，从此你的页面代码将变的超级好维护，交互逻辑入口，也变得层次分明。**

全篇文章，绝无教大家做事之意，这是在项目中摸爬滚打，被坑出的不得不如此规范的一种行为。

# 准备

## 改善

先说说这篇文章能帮你改善什么问题：

- **页面层的widget疯狂套娃几千行，后期维护，心态崩了等问题**
  - 套娃不划分页面，后期需求大变，让你大改页面细节甚至结构，那将是非常难受的一件事
- **逻辑交互事件入口，混杂在widget，难以寻找问题**
  - 如果你在页面层疯狂套娃，你会发现，就算用了provider，bloc中的cubit，getx之类，你想找到逻辑交互入口，也是一件很累的事情，改样式那就更方了。。。
  - 这里再哔哔一下，这些框架作者肯定是发现了这种情况，所以bloc才搞出了event层，fish_redux搞出了action层，来统一管理事件及其事件入口。

- **页面结构充斥大量细节，结构调整起来困难**

上面关于页面层的这些问题，如果多人协同开发一个大型项目，代码不规范的话，大概率都是会遇到的（改别人写的模块...）；后期改需求 ，真的是一种折磨，有种码海找针的感觉；如果改你自己写的模块，那可能还会好点，毕竟你还有点印象，整个模块的大概思路，还知道怎么改。如果是改别人写的模块，你就需要在大量widget海中，去揣摩别人写这些widget的意图，结构一下子也不能理清，十分痛苦，有可能边改边骂骂咧咧的。。。

## Demo效果

在构思文章的时候，就在想演示的Demo页面必定不能过于简单，一个简单的Demo页面，怎么能演示出套娃地狱的改善效果呢？思考了很久，想寻找一个合适demo页面，周末时在听喜马拉雅里面的盗墓小说，看了看发现页面，发现整体样式不错，咱就仿一个吧！而且整体的页面复杂度，也足够来演示了！

喜马拉雅的这个PC页面Demo，写起来真的花费了不少时间，肝痛。

### 地址

- Web：[仿喜马拉雅页面](https://cnad666.github.io/flutter_use/web/index.html#/himalaya)
  - web无法强制设置窗口大小，可能需要你调整下web窗口的宽度，以达到最佳效果
- Windows：[Windows平台安装包](https://wwa.lanzous.com/izW8Rmwws6f)
  - 如果你的电脑开启了125%的`缩放与布局`，请打首页的`开启缩放`按钮
- 项目地址：[flutter_use](https://github.com/CNAD666/flutter_use)

> 说明

代码已经发布到Github上，web端也已经部署好了，因为使用的CanvasKit模式打包的，首次加载可能比较慢，多等一会，因为Web端部署在Github上，访问的话，要确保你的网络能访问Github；CanvasKit模式打包的web，在手机上访问效果也不错，咱在这绝对不是和前端那些牛逼轰轰的框架比，只是让自己多了一些可能，也能搞成一些小玩意

- **关于Widows安装包**
  - Window笔记本高分屏一般会开启125%的缩放，这时候，存在一个坑比的问题，开启缩放的时候，Flutter的布局都会相应的缩放，但是坑比的是，整体的窗口并不会缩放，导致内容会积压整体的窗口，这个问题我也在几台电脑上，调了好久才发现的。
  - 解决办法，写了个手动开启适配的功能。
  - 关于`开启缩放`的按钮功能，只支持放大125%窗口功能，其它的也不用折腾了，我发现window_size初始化后，第一次设置完窗口尺寸后；然后，再设置窗口时，往大了设置有效，往小了回调会无效，奇怪。。。

### 效果对比

来对比下仿制的效果吧，有个六七成相似，很多Icon和图片实在找不到相似，，，这里demo只提供一个样式演示，功能别想了，这不是一朝一夕，一个人能搞出的。。。

照片都是从喜马拉雅web端上搞下来的，数据一直在变，相应栏目的数据有对不上，但是整体样式大致还是差不多。

其中Banner模块是区别最大的一块，用的三方库只能支持搞成这样，各位靓仔将就着看看吧。

- 原版的喜马拉雅PC页面

![image-20210314165954339](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20210314225301.png)

- 仿制的喜马拉雅页面

![](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20210314225310.png)

## 总结

上面俩组图片，细节方面对比基本惨不忍睹，但是整体架构上还是比较相似。

图片因为尺寸限制，没能展示全部内容，右边的信息流模块还有一些信息没展示出来：最新精选、热门主播、经典免费榜、有声书免费榜、相声评书免费榜。

建议各位彦祖，下载下window安装包，安装体验下；MacOS的于晏们，你们可以看看web展示效果，没有苹果那一套东西，又不想折腾黑苹果，实在打包不了。

咱们马上来看看怎么搞规范代码吧！复杂的模块，让你的代码也能高度可维护！

# 开搞

## 分析

- Android的业务自定义View
  - 在Android里面有个页面分模块的开发思路，将整个页面划分成几个业务的自定义View，我们只需要关注传入数据源，和对应业务View交互事件回传信息，这明显是外观模式的思想。。。关注了：数据源和交互事件即可，其它的一切都不是我们所关心的，然后主页面里面，组合下这些业务view就OK了，彻底抛弃include坑比做法，include让xml也耦合了，如果改动了一个被多处引用的xml，可能会引发的一些影响，大家心里可以揣摩揣摩。
- Flutter的Widget
  - 然后再结合Flutter中那些众多的系统widget，系统那些Widget基本都属于功能性的Widget，需要定义巨量的字段传值，这样的好处，就是能够非常颗粒的去控制需要的字段，再配合一些定义的回到函数，就能起到：数据源和交互回调的完美组合。

结合上面的业务View和一切皆Widget的思路，我们可以得出一个结论：**搞业务Widget，然后再进行组合！**

当然，咱们在这里得出了一个不是结论的结论，一般来说，这种操作是咱们基本素养，但是具体的操作细节上，还是有很多需要注意的：

- 业务Widget，也需要划分模块
- Column，Row之类有着天然结构，怎么去利用
- 旁枝末节的Widget细节，怎么去封装

## 主模块封装

> 上面咱们一通分析猛如虎后，得出一个结论：**搞业务Widget！**

关于业务Widget的封装细节，这里说明下：

- **数据源尽量只使用一个，不要使用过多字段去划分**
  - 解释下，因为我们这是业务性widget，并不是功能性widget，过渡的细分字段输入，会导致你封装的widget过长，业务Widget很多时候，只会在你这个模块，其它模块一般都很少用的，没必要去过度的细分字段，开发多了你就会发现，你封装的那些业务Widget，百分之95的概率，只会在你自己写的那个页面吃灰一辈纸。。。
  - 如果是**比较通用的widget，那就可以细分字段了或者使用中间实体都OK！** 通用的模块开发，关于数据源输入，就需要考虑一些比较通用的数据格式，例如只需要一个list数据，就不要搞一个实体，只需要一个字段，就不需要搞一个list等等。。。
- **交互事件，必须使用回调函数，保留出来**
  - 关于交互事件，这里必须要保留出来，给业务层或者逻辑层去处理；一般来说，用户进入该页面，点击或滑动页面，就是业务事件产生的时候了，这是必须暴露出来的，切记切记。

### 仿造的喜马拉雅主模块

- 来看看仿造的喜马拉雅PC页面主模块的代码
  - 这里使用了一点Getx知识，如果你不了解，可查询此文章：[Flutter GetX使用---简洁的魅力！](https://juejin.cn/post/6924104248275763208)
  - 组装对应的业务Widget的时候：请记住，对应的业务Widget一定要写上注释

- 下面就是主模块的所有代码，大家摸着良心说说：
  - 这还死亡嵌套吗？
  - 这还俄罗斯套娃吗？
  - 这看着还恐怖吗？
- 其实按照下面的封装，基本是把View和Event做了一个结合了
  - 所有业务Widget的入口，可快速定位到需要修改的业务Widget
  - 所有的事件交互触发入口，一眼可见，能快速的定义相应业务

```dart
class HimalayaPage extends StatelessWidget {
  final HimalayaLogic logic = Get.put(HimalayaLogic());
  final HimalayaState state = Get.find<HimalayaLogic>().state;

  @override
  Widget build(BuildContext context) {
    return himalayaBuildBg(children: [
      //顶部：左边侧边导航栏 + 右边信息流
      himalayaBuildTopBg(children: [
        //左边导航栏
        HimalayaLeftNavigation(
          data: state,
          //导航栏item回调
          onTap: (Rx<HimalayaSubItemInfo> item) => logic.navigationItem(item),
        ),

        //右边信息流
        himalayaBuildInfoListBg(children: [
          //顶部搜索框及其一些个人信息设置按钮
          HimalayaPersonalInfo(
            //搜索框输入监听
            onChanged: (String msg) => logic.onSearch(msg),
            //左箭头
            onLeftArrow: () => logic.dealLeftArrow(),
            //右箭头
            onRightArrow: () => logic.dealRightArrow(),
            //刷新按钮
            onRefresh: () => logic.onRefreshData(),
            //皮肤按钮
            onSkin: () => logic.switchSkin(),
            //设置按钮
            onSetting: () => logic.onSetting(),
          ),

          //右侧信息流 - 可滑动部分
          himalayaBuildScrollInfoListBg(children: [
            //轮播图
            HimalayaBanner(
              data: state.bannerList,
              //具体banner的监听
              onTap: (int index) => logic.clickBanner(index),
            ),

            //猜你喜欢
            HimalayaGuess(
              data: state.guessList,
              //换一批
              onChange: () => logic.guessChange(),
              //猜你喜欢具体卡片
              onGuess: (HimalayaSubItemInfo item) => logic.guessDetail(item),
            ),

            //最新精选
            HimalayaNewest(
              data: state,
              //分类标题
              onSortTitle: (item) => logic.sortTitle(item),
              //具体精选卡片
              onNewest: (HimalayaSubItemInfo item) => logic.onNewest(item),
            ),

            //热门主播
            HimalayaAnchor(
              data: state.anchorList,
              onAnchor: (HimalayaSubItemInfo item) => logic.hotAnchor(item),
            ),

            //各类榜单
            HimalayaRankList(
              data: state.rankList,
              //标题
              onTitle: (String title) => logic.rankTitle(title),
              //榜单上具体item
              onItem: (HimalayaSubItemInfo item) => logic.rankItem(item),
            ),
          ]),
        ]),
      ]),

      //底部：音频播放控制台
      HimalayaAudioConsole(
        data: state.audioPlayInfo,
        //左切换
        onLeftArrow: () => logic.onLeftArrow(),
        //播放
        onPlay: () => logic.onPlay(),
        //右切换
        onRightArrow: () => logic.onRightArrow(),
        //喜欢
        onLove: () => logic.onLove(),
        //播放模式
        onPlayModel: () => logic.onPlayModel(),
        //封面
        onCover: () => logic.onCover(),
        //进度
        onProgress: () => logic.onProgress(),
        //音量
        onVolume: () => logic.onVolume(),
        //标题
        onSubtitle: () => logic.onSubtitle(),
        //倍速
        onSpeed: () => logic.onSpeed(),
        //定时
        onTiming: () => logic.onTiming(),
        //目录
        onCatalog: () => logic.onCatalog(),
      ),
    ]);
  }
}
```

### 主体细节封装

>  一般来说，一个页面整体基本上是横向（Row）或者纵向（Column）的结构

咱们仿造的喜马拉雅模块也是属于纵向结构：上下俩大模块

- 上模块：导航栏 + 信息流  =>  又分左右模块
  - 左模块：左边的侧面导航栏  =>  很明显的纵向布局
  - 右模块：信息流  =>  这就是简单的纵向结构，从上到下了

- 下模块：音频播放栏  =>  完全就是横向布局了

通过上面的说明，很明显，Row和Column中children属性才是我们所关注的，其它的细节描述封装起来即可

主模块的很多主体细节，是完全可以封装起来，新建一个（模块名_function）文件：

- **himalaya_function.dart**：主体部分有很多无需关注的细节，统一放在这个模块，此处，只需要暴露一个`children`属性即可
  - 请勿将这些无关的细节写在主模块中，会干扰到我们需要关注的信息，这些主体样式写完后，基本就很少去修改了

```dart
import 'package:flutter/material.dart';
import 'package:flutter_use/app/base/base_scaffold.dart';
import 'package:flutter_use/app/utils/ui/auto_ui.dart';

///喜马拉雅整体外层布局设置
Widget himalayaBuildBg({List<Widget> children}) {
  return BaseScaffold(
    backgroundColor: Colors.white,
    body: Column(children: children),
  );
}

///播放控制栏上面的外层布局设置
Widget himalayaBuildTopBg({List<Widget> children}) {
  return Expanded(
    child: Row(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: children,
    ),
  );
}

///顶部右侧信息流外层布局设置
Widget himalayaBuildInfoListBg({List<Widget> children}) {
  return Expanded(
    child: Column(children: children),
  );
}

///顶部右侧信息流外层布局设置 - 可滑动部分
Widget himalayaBuildScrollInfoListBg({List<Widget> children}) {
  return Expanded(
    child: Scrollbar(
      child: SingleChildScrollView(
        child: Container(
          width: 860.dp,
          child: Column(children: children),
        ),
      ),
    ),
  );
}
```

## 业务Widget封装

> 主模块的封装还是比较简单的，只需将主体模块的细节封装起来，暴露children属性即可，然后进行组装即可
>
> **接下来业务Widget封装，这就是核心所在了**

**几个要点**

- 尽量只暴露一个数据源（非通用业务Widget）
- 所有的事件交互必须暴露出来
- 主体细节封装起来
- children中的widget全部提成方法

### children中封装

先来看看第一种情况，最常见的情况，children的widget，从上到下排列下来，非列表类数据

- 来看看这个顶部一些功能按钮的布局，这块涉及到很多事件交互，所以单独提成了一个业务Widget

![image-20210314212412718](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20210314225335.png)

- 实现代码：关于业务Widget，这是基石，规范写好后，后期修改，异常简单
  - 结合上面的效果图，再结合下面的代码，大家应该一眼看出来，就知道是哪个widget方法，对应界面上的哪个控件；如果你想修改哪个控件样式，直接点进对应的widget方法里修改即可
  - **children里面的每个widget方法上面，请一定一定记得写上注释**，因为此处才是业务Widget最主要的入口，具体的widget方法写不写注释无所谓了

```dart
///搜索框 个人信息 设置等按钮
class HimalayaPersonalInfo extends StatelessWidget {
  HimalayaPersonalInfo({
    Key key,
    this.onRefresh,
    this.onLeftArrow,
    this.onRightArrow,
    this.onSetting,
    this.onSkin,
    this.onChanged,
  }) : super(key: key);

  .............

  @override
  Widget build(BuildContext context) {
    return _buildBg(children: [
      //左图标
      _buildLeftArrow(),

      //右图标
      _buildRightArrow(),

      //刷新图标
      _buildRefresh(),

      //搜索框
      _buildSearch(),

      //头像
      _buildHeadImg(),

      //皮肤
      _buildSkin(),

      //设置
      _buildSetting(),
    ]);
  }

  ..........
}
```

- **来看下其中的`_buildBg`方法**
  - 可以发现`_buildBg`主体的这些细节描述，真的是无关紧要的代码，这个写完后，基本上，后面都很少去改，所以把它提取出来后，放在墙角吃灰就行了

```dart
///搜索框 个人信息 设置等按钮
class HimalayaPersonalInfo extends StatelessWidget {
  ........

  Widget _buildBg({List<Widget> children}) {
    return Container(
      margin: EdgeInsets.symmetric(vertical: 10.dp, horizontal: 18.dp),
      width: 800.dp,
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: children,
      ),
    );
  }
}
```

- 关于方法提取
  - 选中你需要提取的Widget代码
  - 打开 Flutter Outline 选择`右箭头`图片

![image-20210314214406466](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20210314225342.png)

- 填上方法命后，就能自动生成一个widget方法
- 如果你提取的Widget块中，还含有一些数据，自动生成的方法都会带上相应参数，非常方便

![image-20210314214520198](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20210314225347.png)

### 列表类样式封装

类列表样式的封装也是比较关键的，直接从头莽尾式的提取是不行，这边有一丝调整

这里就以`猜你喜欢`模块举例

- 猜你喜欢模块

![image-20210314220037075](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20210314225354.png)

- 代码分析：总体是Column布局，分上下俩模块
  - 上模块使用Row搞定即可
  - 下模块是四个卡片，这边是直接用的写死List数据源

```dart
///猜你喜欢
class HimalayaGuess extends StatelessWidget {
  HimalayaGuess({
    Key key,
    this.onChange,
    this.data,
    this.onGuess,
  }) : super(key: key);

  ..........

  @override
  Widget build(BuildContext context) {
    return _buildBg(children: [
      //标题 + 换一批
      Row(mainAxisAlignment: MainAxisAlignment.spaceBetween, children: [
        //标题
        _buildTitle(),

        //换一批
        _buildGuessChange()
      ]),

      //显示具体信息流
      _buildItemBg(itemBuilder: (item) {
        return Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
          //图片卡片
          _buildPicCard(item),

          //文字描述
          Text(item.title, style: TextStyle(fontSize: 15.sp)),

          //作者
          Text(item.subTitle, style: TextStyle(fontSize: 13.sp, color: Colors.grey)),
        ]);
      })
    ]);
  }

  ..........
}
```

- 上述children代码，整体上还是比较清晰，有点迷糊的，可能就是`_buildItemBg`，来看看其中代码
  - 此方法对面暴露了一个`itemBuilder`参数，这其实是一个回调方法
  - 因为列表类样式，必须要遍历整个列表数据，然后，需要把列表遍历的具体数据，反向传给Widget，所以必须使用此类的回调方法

```dart
///猜你喜欢
typedef HimalayaSubBuilder = Widget Function(HimalayaSubItemInfo item);
class HimalayaGuess extends StatelessWidget {
  
  ...............

  Widget _buildItemBg({HimalayaSubBuilder itemBuilder}) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceBetween,
      children: data.map((e) {
        return itemBuilder(e);
      }).toList(),
    );
  }
}
```

**关于双层列表数据源（List的每个具体数据源，又含有List）又该怎么封装呢？**

- 俩层List数据源封装是比较麻烦，这边以侧边栏举例
  - 整个布局是一个Column：标题 + 栏目（List数据控制）
  - 栏目：可划分具体的Item
    - Item：标题 + 栏目（List数据控制）

![image-20210314221811228](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20210314225402.png)

- 代码实现
  - 上面的布局整体是由数据源驱动页面，数据能控制页面item生成

```dart
///数据源：侧边导航栏目初始数据,简化了下，数据源太长了
///该数据源都放在state层维护，此处放在这里，让大家有个对比
leftItemList = [
    HimalayaItemInfo(title: '推荐', subItemList: [
        HimalayaSubItemInfo(
            title: '发现',
            icon: CupertinoIcons.compass,
            tag: TagHimalayaConfig.find,
            isSelected: true,
        ).obs,
        ..............
    ]),
    HimalayaItemInfo(title: '我听', subItemList: [
        HimalayaSubItemInfo(
            title: '我的订阅',
            icon: Icons.star_border,
            tag: TagHimalayaConfig.subscription,
        ).obs,
        .........
    ]),
    HimalayaItemInfo(title: '我创建的听单', subItemList: [
        HimalayaSubItemInfo(
            title: '我喜欢的声音',
            icon: Icons.favorite_border,
            tag: TagHimalayaConfig.sound,
        ).obs,
        ............
    ]),
];

///左边导航栏
class HimalayaLeftNavigation extends StatelessWidget {
  HimalayaLeftNavigation({
    Key key,
    this.data,
    this.onTap,
  }) : super(key: key);

  ........

  @override
  Widget build(BuildContext context) {
    return _buildBg(children: [
      //喜马拉雅logo图标
      _buildLogo(),

      //遍历俩层循环：不同item栏目 - 可点击,可滑动
      //第一层：标题 + 子item列表
      //第二层：子item详细布局
      _buildItemListBg(itemBuilder: (item) {
        return [
          //最外层item - 大标题
          _buildTitle(item.title),

          //子栏目 - 列表
          _buildSubItemListBg(
            data: item,
            subBuilder: (subItem) => _buildSubItemBg(data: subItem, children: [
              //选中红色长方形条块
              _buildRedTag(subItem),

              //图标
              _buildItemIcon(subItem),

              //描述
              _buildItemDesc(subItem),
            ]),
          ),
        ];
      }),
    ]);
  }
    
  ..........
}
```

- 第一层：来看下第一层`_buildItemListBg`方法
  - 这玩意不得不套了，需要的属性太多（滚动，滚动条等），这玩意要是不提出来，放在children，那简直就是毒瘤了

```dart
typedef HimalayaItemBuilder = List<Widget> Function(HimalayaItemInfo item);
class HimalayaLeftNavigation extends StatelessWidget {
  ..........

  Widget _buildItemListBg({HimalayaItemBuilder itemBuilder}) {
    return Expanded(
      child: Scrollbar(
        child: SingleChildScrollView(
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: data.leftItemList.map((e) {
              return Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: itemBuilder(e),
              );
            }).toList(),
          ),
        ),
      ),
    );
  }
}
```

- 第二层
  - 这里面必须需要第一层遍历的具体数据源，所以必须增加一个输入参数
  - 这里就是常规提取，需要注意的就是传入的数据源

```dart
typedef HimalayaRxSubBuilder = Widget Function(Rx<HimalayaSubItemInfo> item);
class HimalayaLeftNavigation extends StatelessWidget {
    ..........

    Widget _buildSubItemListBg({HimalayaItemInfo data, HimalayaRxSubBuilder subBuilder}){
        return Column(
            children: data.subItemList.map((e) {
                return subBuilder(e);
            }).toList(),
        );
    }
}
```

## 总结

> 经过上面的一通操作，业务Widget立马变的清爽N倍

大家在写Flutter的时候，应该能明显的感觉到，写页面拥有高度的自由，样式、页面结构及其逻辑全都能耦合在一起；所以在实际开发中，更要注意自己代码规范。

假设一种情况：你开发完一个模块，过了几月之后，需求调整，你要去改这个模块，看到几千行的套娃页面代码，然后一边改一边骂骂咧咧，然后喷是哪个睿智的人写的，最后打开文件的git注释（annotate）记录，如果上面写满了你的名字，那岂不是很尴尬。。。

# 题外话

> 说一点题外话

实际上写html也是无限套娃，不同的是，它从根本上做到的样式结构分离，控件的细节描述，全部交给了css去做，所以页面整体看上去还是满清爽的：

- 但是有一点让我很蛋筒，写小程序的时候，查看具体控件的描述样式，需要跨文件去找
  - uniapp则是直接把这些东西放在一个文件里（19年写的时候是这样的，不知道现在有没有改），算是一种改善，查找起来方便，但是单个文件代码量有点爆炸
- 样式因为是交给css去处理，层级描述也放在css中，有时候看代码看的有点懵逼（是我太菜了）

Flutter直接从根本上样式结构不分离，结构上直接从上往上下一套到底

- 优点：修改样式简单（方便定位）；结构清晰（从上往下看就行了）
- 缺点：代码阅读，观感爆炸；不做模块划分，后期代码维护困难

所以，哪里有十全十美的框架，总是有舍有得。。。

> 新的事物发展，必然会迎来相应的阻力

这里假设一种场景：

- 你已经写了俩三年Flutter了，各种控件，框架玩的牛的飞起
- 然后，你听说：又来了一种神奇的，跨时代的前端框架，甚至能无缝调用所有平台的底层硬件api，omg，反正就是各种6

- 然后你看到，关于这种跨时代框架的文章，在各个技术论坛中，疯狂涌现
- 此时，你心中会不会有丝丝异样，心想：杂家，这几年Flutter白写了？又得去学这个新框架了？我踏马岂不是又变成萌新了！又要天天去群里抱大佬大腿了！
- 然后你看到那一片片热点文章，文章下满是捧上天的评论，，，
- 此时，你的心中会不会有丝波澜，想当一当这技术界的清醒者，情不自禁吟诵：众人皆醉我独醒.....
- 然后，拿起键盘，化身一个大喷子，以一敌百，不落下风
- 一瞬间，让你觉得：这个论坛，现在叫lbw论坛！我就是这论坛的王！

> 角色互换

其实，对于很多言论，我们没必要在意；角色互换，说不定，对方此刻的行为，就是我们自己以后可能会做的事。

![小丑竟是我自己是什么梗小丑竟是我自己是什么意思出处在哪-站长之家](https://pic.chinaz.com/thumb/2020/1223/6374431204931358119476526.png)

**其实，我们都是打工人，又何必撕来撕去呢？**

# 最后

文中DEMO地址：[flutter_use](https://github.com/CNAD666/flutter_use)

**系列文章**

通过上面一些代码规范操作后，再配合上GetX的状态管理，相信一般的项目，你都能hold的住了

> 加油，我们都是这条街，最靓的仔

- 状态管理：[Flutter GetX使用—简洁的魅力！](https://juejin.cn/post/6924104248275763208)

- Dialog解决方案，墙裂推荐：[一种更优雅的Flutter Dialog解决方案](https://juejin.cn/post/6902331428072390663)