# 从零开始撸一个fish_redux插件



## 运行插件项目

### 说明

- 下载别人的插件项目，才需要新增运行项，自己新建的插件项目，运行项都是设置好的

### Plugin DevKit 插件

一般来说，我们从github下载别人的插件项目，需要新建一个运行设置

- 点击三角形运行图标右边选项，选择：Edit Configurations...

![image-20201020105000665](../../../资料/图片/image-20201020105000665.png)

- 点击“+”，在“other”的折叠项下，找到Plugin，点击确定

![image-20201020110430573](../../../资料/图片/image-20201020110430573.png)

- 然后就是一些设置
  - Name：随便写个就行
  - VM Options：这个用默认的
  - Program Arguments：这个可以不填，不然运行的时候，可能老是弹一个弹窗
  - Use classpath of module：你会发现这边没办法选择，这是一个坑，下面说解决原因，这个现象只会在你下载别人的插件项目运行的时候会出现，自己新建一个插件项目，会默认有个运行设置，都是设置好的
  - JRE：选择插件的SDK就行了

![image-20201020110720259](../../../资料/图片/image-20201020110720259.png)

- 创建好，你会发现，这玩意报一个红叉，坑爹啊这是，怎么都跑不起来！

  ![image-20201020111314245](../../../资料/图片/image-20201020111314245.png)																![img](../../../资料/图片/007ADEF4.jpg)

- 解决方法，这是因为没设置对项目SDK，会导致：Use classpath of module 这项没办法选择正确的module
  - 点击：File ->  Project Structure.. -> Project Settings
  - 下载的项目都会默认项目SDK为我们设置的默认SDK，这边改成Plugin的SDK即可，点击即可，然后点击下方的：Apply，最后点击：OK

![image-20201020112204356](../../../资料/图片/image-20201020112204356.png)

- 重启下IDEA，然后再回到运行项的设置，选择：Edit Configurations...，重新对自己的插件设置下Use classpath of module即可

### Plugin Gradle 插件

- 使用Grade开发插件，运行设置就比较简单了，选择Gradle
  - Gradle project：直接点右侧文件夹图标选择即可
  - Tasks：填入  :runIde

![image-20201020113821316](../../../资料/图片/image-20201020113821316.png)

- Gradle模块开发插件，设置运行项就非常简单；OK，大功告成，可以直接运行了

## 控件使用

### dialog

- dialog这个控件在DSL中的用法，有个ok回调参数，我觉得有必要好好讲讲，坑死爷了
  - 这个用法都是我看源码边猜边试，成功蒙出来的；马丹，文档里面都没dialog的用法说明

```kotlin
private fun showDialog() {
    dialog(
        title = "Fish Redux Code Generation",
        panel = panel{
            titledRow("帅比") {
                row("Select  ShuaiBi") {
                    label(text = "我是大帅比")
                }
            }
        },
        ok = {
            //点击OK按钮会调用该回调
            //Messages.showMessageDialog("Hello World !", "Test", Messages.getInformationIcon())

            //是否满足条件关闭ok弹窗
            isCanClose()
        }
    ).show()
}

private fun isCanClose(): List<ValidationInfo> {
    val list = ArrayList<ValidationInfo>()

    val isAccess = 2
    if (isAccess == 1) {
        //点击确定按钮不会消失,且提示相关内容
        list.add(ValidationInfo("测试OK回调"))
        list.add(ValidationInfo("Please input module name"))
    }

    //list未添加元素，弹窗会直接关闭，可以进行后续逻辑操作
    return  list
}
```

- 参数说明
  - title：弹窗顶部的的标题
  - panel：整体布局样式，用panel作为最外层，然后去堆就行了
  - ok：这个回调参数的使用，真是把我猜的头疼，试了好多遍，终于对比出来了，点击OK按钮有俩种情况
    - 点击OK按钮，弹窗不消失：这种一般是检测到缺少相关数据，需要提示用户输入某些数据，不能让弹窗消失，这里初始化一个泛型为ValidationInfo的list，添加一些ValidationInfo对象，里面表明一些提示即可
    - 点击OK按钮，弹窗消失：满足条件，关闭弹窗，不填加ValidationInfo对象即可，如果添加了，点击OK时，弹窗就不会消失，切记