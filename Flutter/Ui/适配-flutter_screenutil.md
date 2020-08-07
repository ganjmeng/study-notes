# 适配-flutter_screenutil

- pub库地址：https://pub.dev/packages/flutter_screenutil
- 中文文档：https://github.com/OpenFlutter/flutter_screenutil/blob/master/README_CN.md

### 在每个使用的地方都要导入包：

```
import 'package:flutter_screenutil/flutter_screenutil.dart';
```

### 属性

| 属性             | 类型   | 默认值 | 描述                                                   |
| :--------------- | :----- | :----- | :----------------------------------------------------- |
| width            | double | 1080px | 根据设计稿中设备的宽度适配，单位px                     |
| height           | double | 1920px | 根据设计稿中设备的高度适配，单位px                     |
| allowFontScaling | bool   | false  | 设置字体大小是否根据系统的“字体大小”辅助选项来进行缩放 |

