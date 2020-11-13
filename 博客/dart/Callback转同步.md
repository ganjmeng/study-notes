# Flutter&Dart  Callback转同步

## 前言

**怎么将一个Callback回调转化成Future同步方法（Callback to Future），可以配套async / await去使用呢？**

个人觉得，这是一个很常见的现象，不知道为啥，很多人在说明Future用法的时候，都没提到这个场景，奇怪+懵逼，只能自己去苟解决方案了。

## 实现

`不多哔哔，先看实现，赶时间的靓仔，可以直接忽略掉历程描述`

- 记录下Callback to Future的写法，使用Completer类即可

```dart
class ViewUtil {
  ///界面初始化完成
  static Future<Void> initFinish() async {
    Completer<Void> completer = Completer();

    WidgetsBinding.instance.addPostFrameCallback((timeStamp) {
      completer.complete();
    });

    return completer.future;
  }
}
```

- 使用
  - 使用起来，瞬间简单很多

```dart
void _init() async {
    await ViewUtil.initFinish();
    ///下面可以使用加载弹窗
}
```

**说明**

- Future<T>和Completer<T>的泛型最好保持一致
  - 例如都是String的话，complete()方法里面就可以加上相应的内容，然后await接受这个方法时候，就能拿到complete()方法里面输入的值了

```dart
class ViewUtil {
    ///界面初始化完成
    static Future<String> initFinish() async {
        Completer<String> completer = Completer();

        WidgetsBinding.instance.addPostFrameCallback((timeStamp) {
            completer.complete("测试一下...");
        });

        return completer.future;
    }
}


void _init() async {
    var s = await ViewUtil.initFinish();
    print(s);
}
```

## 历程

- 为什么我要将Callback转成Future方法？
  - 大家知道，Flutter在加载页面的时候，有个渲染的过程，在没渲染完成的时候，你去显示一些View的操作，会报错的，例如：加载loading弹窗
  - 解决方法可能大家都知道，Lifecycle.initState / iniState 生命周期里面做个延时操作或者使用WidgetsBinding

```dart
//延时操作
await Future.delayed(Duration(milliseconds: 200));
//下面可以加载弹窗

//使用WidgetsBinding
WidgetsBinding.instance.addPostFrameCallback((timeStamp) {
    //此处可以加载弹窗
});
```

- 当然，使用WidgetsBinding是更靠谱和准确的，但是这个Callback就让我很方了，而且，这名字太长，也不太记的住，这就需要将它封装了
- 封装WidgetsBinding
  - 蛋筒了，这玩意怎么封装呢

```dart
class ViewUtil {
  ///界面初始化完成
  static Future<Void> initFinish() async {
    WidgetsBinding.instance.addPostFrameCallback((timeStamp) {
      
    });
  }
}
```

- 首先我想到了：Future.delayed()
  - 进去看下他的源码
  - 有戏，可以看到，这里面明显包含了一个Timer中的Callback回调，但最后转换成了，Future方法

```dart
factory Future.delayed(Duration duration, [FutureOr<T> computation()?]) {
    if (computation == null && !typeAcceptsNull<T>()) {
        throw ArgumentError.value(
            null, "computation", "The type parameter is not nullable");
    }
    _Future<T> result = new _Future<T>();
    new Timer(duration, () {
        if (computation == null) {
            result._complete(null as T);
        } else {
            try {
                result._complete(computation());
            } catch (e, s) {
                _completeWithErrorCallback(result, e, s);
            }
        }
    });
    return result;
}
```

- 分析下

  - 首先是实例化一个_Future<T>()对象，然后返回了这个_Future<T>()对象
  - 可以看到方法的最下方是直接返回这个对象，可想而知，这地方，肯定一直处于一个阻塞状态，在等待一个条件结束这个阻塞状态
  - 然后在Timer的延迟时间到了后，其回调中使用了_complete()这个方法，这个方法应该是结束了_Future<T>()对象的阻塞状态，然后再返回_Future<T>()对象，同时这个方法也结束了

- 这不就简单了，我把这个抄出来不就欧了

![img](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20201023144244.jpg)

- 这个_Future类是个私有方法，在future_impl.dart文件，把这个文件拷出来，放在我们工具类文件同一个包下，

  - 然后。。。

  ![image-20201023140910251](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20201023144250.png)

- 这一堆报错，玩毛线啊，肯定是我打开的方式不对
  
  - 难道要一个一个去解决这些报错？要是这么麻烦，还搞毛线！

![image-20201023144223906](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20201023144227.png)

- 是不是我搜索的姿势不对，再来搜搜看
  - 我去，还自动给我提示：dart callback to future，这么神奇的吗？试试看

![image-20201023143143286](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20201023144306.png)

- 然后成功找到这个：[Dart: Turn Callback Functions into a Futures! 2018, Flutter!!](https://medium.com/@brianschardt/dart-turn-callback-function-into-a-future-95f54431936b)
  - 言简意赅，简洁明了

![img](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20201023144318.jpg)

