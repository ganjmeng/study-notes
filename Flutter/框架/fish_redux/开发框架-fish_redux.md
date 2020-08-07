# fish_redux

- Lifecycle.initState：该周期初始化，不能在该周期方法中，初始化ScreenUtil插件的ScreenUtil.init(....)方法，不然会报此类异常：

```dart
dependOnInheritedWidgetOfExactType<MediaQuery>() or dependOnInheritedElement() was called before ComponentState<HomeState>.initState() completed.
```



## 使用adapter(ListView)

