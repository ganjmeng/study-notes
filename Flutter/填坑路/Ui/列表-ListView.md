# ListView

- 注意：ListView头部有一段空白区域，是因为当ListView没有和AppBar一起使用时，头部会有一个padding，为了去掉padding，可以使用MediaQuery.removePadding

```
Widget _listView(BuildContext context){
    return MediaQuery.removePadding(
      removeTop: true,
      context: context,
      child: ListView.builder(
        itemCount: 10,
        itemBuilder: (context,index){
          return _item(context,index);
        },

      ),
    );
  }

```

- 报错：RenderBox was not laid out:***
  - SingleChildScrollView组件不能和ListView组件一起使用

- ListView嵌套ListView注意点

  - 解决子控件必须设置高度的问题
  - 必须禁用子ListView滑动事件

  ```
  ListView.builder(
      shrinkWrap: true, //为true可以解决子控件必须设置高度的问题
      physics:NeverScrollableScrollPhysics(),//禁用滑动事件
    )
  ```



