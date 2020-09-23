# Flutter Webview添加Cookie的正确姿势

## 场景

h5页面要从cookie里面取数据，所以需要在flutter webview的cookie里面塞一些数据，设置的数据多达十几条；按照网上查的使用方式来设置，通过fiddler抓包发现，只能生效一条，来来回回试了很多次都只有一条，心态崩了

后来看到cookie设置数据也是类似键值对里面套键值对，灵机一动，变换下后就成功了，记录下正确的写法吧

## 正确姿势

**引入**

- 使用的是flutter官方维护的webview插件

```dart
webview_flutter: ^0.3.22+1
```

**错误示例**

- 这是最坑的一个，widget都都没写全，就写了俩个回调，这么写只会生效一条

```dart
WebViewController _controller;
onWebViewCreated: (WebViewController wvc) {
	_controller = wvc;
}

onPageFinished: (String value) {
	_controller.evaluateJavascript( 'document.cookie = "SESSIONID=612bc4822b6996d6f335a963c20eb541fba72985; path=/"')
}
```

- 这个只写了一条cookie，这个是没问题的，和上面的区别就是，这个使用双引号包住单引号，只写了一条的使用也是让人肝痛

```dart
setSessionID() async {
  String sessionID = await LocalStorage.get("sessionID");
  if (Platform.isIOS) {
    _controller.evaluateJavascript("document.cookie = 'sessionID=${sessionID}'").then<String>((res) {
      print("webViewController.evaluateJavascript========>${res}");
      _onListCookies(_controller, context);
    });
  } else {
    _controller.evaluateJavascript('document.cookie = "sessionID=${sessionID};"').then<String>((res) {
      print("webViewController.evaluateJavascript========>${res}");
      _onListCookies(_controller, context);
    });
  }
}
```

**多条cookie添加正确写法**

- 琢磨半天试出来的正确写法，cookie的设置需要在页面加载完之后设置

```dart
///webview控制器
WebViewController _controller;
String _url = "写入你的链接";

WebView(
    initialUrl: _url,
    javascriptMode: JavascriptMode.unrestricted,
    onWebViewCreated: (controller) {
        _controller = controller;
    },
    onPageFinished: (url) {
        //页面加载结束
		String cookie =
            "document.cookie = 'name=IAmDaShuaiBi';document.cookie = 'id=233'";
        _controller.evaluateJavascript(cookie);
    },
    userAgent: "test;app/1.0.0",
)
```

- 最重要的变化就是每条cookie都要用document.cookie作为key，这是最最最关键的

**优化写法**

- 上面的写法是写成一行，写成一行是很致命的操作，让赋值操作会变得很迷惑，优化下

```dart
///webview控制器
WebViewController _controller;
String _url = "写入你的链接";

WebView(
    initialUrl: _url,
    javascriptMode: JavascriptMode.unrestricted,
    onWebViewCreated: (controller) {
        _controller = controller;
    },
    onPageFinished: (url) {
        //页面加载结束
        String cookie = '''
          document.cookie = 'nameOne=IAmDaShuaiBi';
          document.cookie = 'idOne=233';
          document.cookie = 'nameTwo=IAmDaShuaiBi';
          document.cookie = 'idTwo=233';
          document.cookie = 'nameThree=IAmDaShuaiBi';
          document.cookie = 'idThree=233';
        ''';
        _controller.evaluateJavascript(cookie);
    },
    userAgent: "test;app/1.0.0",
)
```

- ok，搞定

## 补充

- **就这么点内容就水了一片文章，显得我是个水笔，为了证明我不是个水笔，再补充个开发小技巧**

***

### 弹窗封装，优化列表数据源

### 说明

底部单列表弹窗是非常实用的弹窗，但是可能大家经常有这样的一种使用体验，传数据源，一般都是传一个List类型，内部的泛型最最最常见的就是：String和固定的实体；说下这俩种情况的问题所在

- String：显然该泛型，内部处理处理起来非常简单，直接拿String值展示就行，回调直接把选择的String返回；这种String类型存在一个非常致命的的问题，在使用前后都存在问题
  - 使用前：我们必须把数据类型转换成List<String>类型，我们数据源一般都是通过接口获取，接口里面List数据源里面的泛型五花八门的，往往需要做循环转化成List<String>类型去使用
  - 使用后：点击选择后，拿到我们选择的展示名，接口提交往往是需要该种选择类型的id，一般来说还需要去做遍历拿出选择名称对应的id，然后保存做提交操作
- 规定实体：使用规定的泛型实体，有一个好处，可以避免在使用后去遍历对比，然后去取id的操作，所以他只存在使用前的麻烦
  - 使用前：这里我们必须把拿到的数据源转换符合要求的List规定实体泛型，这步操作就可以把id等信息一起放在规定的实体里面，选择之后也可以直接从实体里面拿了

**吐槽**：虽然上面俩种情况，第二种明显优于第一种，但是实际开发中，往往不假思索的用第一种，因为第一种不用去新建一个class啊！然后用起来就蛋痛了；有没有一种方法可以一劳永逸的方法呢，数据的list泛型随便用，只用指定泛型实体用于展示的key；然后在回调里面又能拿到我们传的实体呢？通过dart强大的动态类型，加一些巧妙转换是完全可以的

### 实现思路

java里可以通过泛型+反射；然后指定一个展示的key，理论上也是可以实现，我没试过，大家可以试试

**实现思路**

实现方法还是很简单的，这里说下思路就行了

**实现步骤**

- 数据源定义为List<E>类型，然后指定一个展示的key
  - 泛型不指定的话，会默认为动态类型，这样的话，便有了很大的操作空间
- 遍历这个List，然后拿到遍历列表时的实体
- **将实体转换成Map类型，通过指定的key，获取展示内容就行了**
- 回调时，传出来选的实体就ok，似不似很简单

**举例**

- 引入：flutter_picker

```dart
flutter_picker: ^1.1.5
```

- 实现

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:flutter_picker/Picker.dart';

typedef ParamSingleCallback<E> = Void Function(E data);

void showBottomSingleDialog<E>(
    BuildContext context, {
        @required List<E> list,
        @required ParamSingleCallback<E> callback,
        String title = '请选择',
        String showKey = '',
    }) {
    List<PickerItem<E>> pickList = [];
    for (E item in list) {
        String showContent;
        if (showKey == '') {
            //兼容泛型为String的情况
            showContent = item as String;
        } else {
            //将实体转成map，通过设置的key指定展示的字段
            var map = json.decode(jsonEncode(item));
            showContent = map[showKey];
        }

        pickList.add(
            PickerItem(
                text: Text(showContent),
                value: item,
            ),
        );
    }

    Picker(
        adapter: PickerDataAdapter<E>(data: pickList),
        hideHeader: false,
        title: Text(title),
        cancelText: "取消",
        confirmText: "确定",
        onConfirm: (Picker picker, List value) async {
            //必须做一个延时操作,先执行回调内部的pop,不然pop页面无法回传值
            await Future.delayed(Duration(milliseconds: 10));
            callback(picker.getSelectedValues()[0]);
        },
    ).showModal(context);
}
```

**使用**

- 数据源

```dart
class InfoBean {
    String name;
    int id;

    InfoBean({this.id, this.name});

    /// jsonDecode(jsonStr) 方法中会调用实体类的这个方法。如果实体类中没有这个方法，会报错。
    Map toJson() {
        Map map = Map();
        map["name"] = this.name;
        map["id"] = this.id;
        return map;
    }
}

///创建数据源
List<InfoBean> list = [];
for (var i = 0; i < 10; i++) {
    list.add(InfoBean(name: "姓名-$i", id: i));
}
```

- 使用
  - 这地方加不加泛型都是可以的，不加泛型默认为动态类型，item点的时候没有提示，可以凭着自己的记忆写key值，只要是key是对，完全能拿到数据

```dart
///加泛型
showBottomSingleDialog<InfoBean>(
    context,
    list: list,
    showKey: 'name',
    callback: (item) {
        print(item.name);
        print(item.id);
    },
);

///不加泛型
showBottomSingleDialog(
    context,
    list: list,
    showKey: 'name',
    callback: (item) {
        print(item.name);
        print(item.id);
    },
);
```

- 效果图

![image-20200922194205445](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200922194702.png)

![image-20200922194356028](https://cdn.jsdelivr.net/gh/CNAD666/MyData/pic/flutter/blog/20200922194711.png)