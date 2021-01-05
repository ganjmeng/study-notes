# 场景

h5页面要从cookie里面取数据，所以需要在flutter webview的cookie里面塞一些数据，设置的数据多达十几条；按照网上查的使用方式来设置，通过fiddler抓包发现，只能生效一条，来来回回试了很多次都只有一条，心态崩了

后来看到cookie设置数据也是类似键值对里面套键值对，灵机一动，变换下后就成功了，记录下正确的写法吧

# 正确姿势

## 引入

- 使用的是flutter官方维护的webview插件

```dart
webview_flutter: ^0.3.22+1
```

## 错误示例

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

## 多条cookie添加正确写法

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

# 优化写法

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

# 最后

- ok，搞定