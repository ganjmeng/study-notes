# Flutter坑点记录（干货）

# Pub包引用

- 依赖本地包：本地路径

```dart
dependencies:
  pkg1:
    path: ../../code/pkg1
```

- 依赖git仓库包：在git仓库根目录

```
dependencies:	
  package_name:
    git:
      url: git://github.com/xxx_user_name/xxx_repository_name.git
```

- 依赖git仓库包：不在git仓库的根目录

```
dependencies:
  package_name:
    git:
      url: git://github.com/xxx_user_name/xxx_repository_name.git
      path: package_name/path 
```



# 屏幕适配

**移动端的屏幕适配整起来还是蛮简单的，一般来说是整个方法返回的，咱们来个简单的写法！**

- 正常版

```

```



