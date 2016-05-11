# Android官方托管在Github上的MVP示例学习笔记

## TODO-MVP
项目的托管地址：[Github](https://github.com/googlesamples/android-architecture/blob/todo-mvp/README.md)
### 主要内容翻译：
#### 摘要

这个示例可以作为很多项目的基础，并且该示例简单演示了没有任何工程框架的“Model-View-Presenter“模式是如何组织的。该项目使用手动依赖注入来提供一个用于本地和远程数据源的仓库。异步操作使用回调完成。


![Summary](https://github.com/googlesamples/android-architecture/wiki/images/mvp.png)

注意：当我们讨论MVP的时候，`View`这个包含了太多意思，因此我们需要区分一下：

* 称`android.view.View`为`Android View`
* 称`从presenter接收指令的view`为`View`.

#### Fragment

该项目之所以使用了Framgent主要有两个原因：

* Activity和Fragment之间的区隔恰好适合本例的MVP实现：`Activity`刚好可以作为生成和连接`View`和`Presenter`控制器。
* 适合平板设备的布局或者有多个视图的屏幕可以更好的利用Fragment框架。

#### 关键概念

这个app有四个功能

* `任务(Tasks)`
* `任务详情(TaskDetail)`
* `添加或编辑任务(AddEditTask)`
* `统计(Staticstics)`

每个功能有：

* 一个协议接口类用来定义`View`和`Presenter`
* 一个`Activity`用来创建`Fragemnt`和`presenter`
* 一个`Fragment`用来实现`View`接口
* 一个`presenter`用来实现`Presenter`接口

<mark>简单来说，就是业务逻辑由presenter处理，再通过View来展现Android的UI变化。</mark>

View几乎没有包含任何逻辑：View的职能仅仅是把presenter的命令转化成对应的UI变化，并且将用户的行为传递给presenter。

协议类是用来定义View和Presenter之间联系的接口。

#### 依赖

* Android support library(例如`com.android.support.*`)
* Android Testing Support Library(例如`Espresso`,`AndroidJUnitRunner`等)
* Mockito(用于单元测试)
* Guava(用于判断null对象)

## 项目特点
### 复杂度 - 理解难度
####使用的框架/库/工具:
无

####概念复杂度
低，如你所见，这是一个单纯为安卓打造的MVP架构实现

####可测试性
#####单元测试
高，`presenter`和`仓库`以及`数据源`都经过了单元测试

#####UI测试
高，伪模块的注入使得UI得以用伪数据进行测试

####代码矩阵

相比于没有架构的传统工程，这个示例展示了很多新增的类和接口：比如`presenter`,`repository`,`contract(协议类)`等等。所以在MVP中，工程所拥有的行数以及类的数量要相对高一些。

```
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Java                            46           1075           1451           3451
XML                             34             97            337            601
-------------------------------------------------------------------------------
SUM:                            80           1172           1788           4052
-------------------------------------------------------------------------------
```

####可掌握性

#####易于修改或增加功能
高。

#####学习成本
低。功能点很容易找到并且各个类的分工也很清晰。开发者不需要对第三方类库很熟悉就能理解这个工程。
## 学习心得

待添加，暂时还没有太多有用的东西搬上来，尝试之后会更新。

