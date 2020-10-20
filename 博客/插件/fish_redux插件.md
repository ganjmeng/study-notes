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