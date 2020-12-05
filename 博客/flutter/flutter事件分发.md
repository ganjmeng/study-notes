# Flutter事件分发

## 关键Widget

### 阻止子树接受事件

- AbsorbPointer
  - 阻止子树接收指针事件，`AbsorbPointer`本身可以响应事件，消耗掉事件

  - `absorbing` 属性（默认true）   
    - true：拦截向子Widget传递的事件   false：不拦截   

```dart
AbsorbPointer(
    absorbing: true,
    child: Listener(
        onPointerDown: (event){
            print('+++++++++++++++++++++++++++++++++');
        },
    )
)
```

- IgnorePointer
  - 阻止子树接收指针事件，`IgnorePointer`本身无法响应事件，其下的控件可以接收到点击事件（父控件）
  - `ignoring` 属性（默认true）   
    - true：拦截向子Widget传递的事件   false：不拦截   

```dart
IgnorePointer(
    ignoring: true,
    child: Listener(
        onPointerDown: (event){
            print('----------------------------------');
        },
    )
)
```

- PlatformView
  - gestureRecognizers

