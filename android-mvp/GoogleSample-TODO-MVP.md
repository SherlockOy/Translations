# Android官方托管在Github上的MVP示例学习笔记


项目的托管地址：[Github](https://github.com/googlesamples/android-architecture/blob/todo-mvp/README.md)

转载请注明出处。
## 主要内容翻译：
### TODO-MVP
#### 摘要

这个示例可以作为很多项目的基础，并且该示例简单演示了没有任何工程框架的“Model-View-Presenter“模式是如何组织的。该项目使用手动依赖注入来提供一个用于本地和远程数据源的仓库。异步操作使用回调完成。


![Summary](https://github.com/googlesamples/android-architecture/wiki/images/mvp.png)

注意：当我们讨论MVP的时候，`View`这个单词包含了太多意思，因此我们需要区分一下：

* 称`android.view.View`为`Android View`
* 称`从presenter接收指令的view`为`View`.

#### Fragment

该项目之所以使用了Framgent主要有两个原因：

* Activity和Fragment之间的区隔恰好适合本例的MVP实现：`Activity`刚好可以作为生成和连接`View`和`Presenter`的控制器。
* 适合平板设备的布局或者有多个视图的屏幕都可以更好的利用Fragment框架。

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

### 项目特点：
#### 复杂度 - 理解难度
#####使用的框架/库/工具:
无

#####概念复杂度
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


`以上就是项目README.md的翻译，下面贴上自己对这个项目的分析`

## 项目分析

这个项目用功能模块来组织结构，个人也比较倾向于这种结构，在定位代码的时候相对便捷，不过还有一种流行的组织方式，是将相同组件归类在一起，比如按`adapter`, `activity`, `fragment`分类，其实也挺好的，萝卜白菜的问题，不再赘述。

在这里抽出`添加和编辑任务的模块`作为例子进行分析。

#### addedittask功能模块分析

进入工程目录，在`addedittask`文件夹下，分别有：

* AddEditTaskContract.java
* AddEditTaskFragment.java	
* AddEditTaskPresenter.java
* AddEditTaskActivity.java

这四个文件。

##### AddEditTaskContract.java

`AddEditTaskContract`是一个接口类，在此我们把它翻译成契约类，方便理解。值的注意的是，这个类是整个工程与我之前所看见过的其他MVP示例工程最不同的地方，Google工程师们将`View`和`Presenter`对应要实现的方法定义在了这个契约类中，维护或者阅读时一目了然，这确实是一个很棒的设计。我在这里贴出契约类的全部代码，代码不长，却短小精悍。

```

/**
 * This specifies the contract between the view and the presenter.
 */
public interface AddEditTaskContract {

    interface View extends BaseView<Presenter> {

        void showEmptyTaskError();

        void showTasksList();

        void setTitle(String title);

        void setDescription(String description);

        boolean isActive();
    }

    interface Presenter extends BasePresenter {

        void createTask(String title, String description);

        void updateTask( String title, String description);

        void populateTask();
    }
}


```

贴完代码，忽然觉得都不需要把`AddEditTaskFragment`和`AddEditTaskPresenter`的代码再贴上来了，但是为了理解结构和功能这一点，单纯的阅读契约类就足够了。

##### AddEditTaskFragment.java

`AddEditTaskFragment`作为`AddEditTaskContract.View`接口的具体实现，具体实现了定义在上述契约类的方法。如果你阅读了源码，会发现在这个Fragment里完全找不到和业务逻辑相关的方法，这不正是我们所追求的分离界面和业务逻辑的样子吗？

这里要说一下，由于`AddEditTaskContract.View`继承了`BaseView<Presenter>`，因此`AddEditTaskFragment `实现了setPresenter方法。

```
/**
 * Main UI for the add task screen. Users can enter a task title and description.
 */
public class AddEditTaskFragment extends Fragment implements AddEditTaskContract.View {

	...
	
	@Override
	public void setPresenter(@NonNull AddEditTaskContract.Presenter presenter) {
        mPresenter = checkNotNull(presenter);
   	}
   	
   	...  	
   	
}
```
正是这个方法，将`Presenter`注入了`View`中。

##### AddEditTaskPresenter.java

`AddEditTaskPresenter`作为`AddEditTaskContract.Presenter`的具体实现，完成业务逻辑实现。这里很巧妙的使用自身的构造方法将自己作为`Presenter`注入到了`View`中。

```
public class AddEditTaskPresenter implements AddEditTaskContract.Presenter,
       TasksDataSource.GetTaskCallback {
       
       ...
       
    /**
     * Creates a presenter for the add/edit view.
     *
     * @param taskId ID of the task to edit or null for a new task
     * @param tasksRepository a repository of data for tasks
     * @param addTaskView the add/edit view
     */
    public AddEditTaskPresenter(@Nullable String taskId, @NonNull TasksDataSource tasksRepository,
            @NonNull AddEditTaskContract.View addTaskView) {
        mTaskId = taskId;
        mTasksRepository = checkNotNull(tasksRepository);
        mAddTaskView = checkNotNull(addTaskView);

        mAddTaskView.setPresenter(this);
    }
       ...  
}

```

##### AddEditTaskActivity.java

`AddEditTaskActivity`作为控制器(Controller)，关联`AddEditTaskFragment`和`AddEditTaskPresenter`，这里用到了上面提到的AddEditTaskPresenter的构造方法，将`AddEditTaskFragment`作为`AddEditTaskPresenter`构造方法的第三个参数。

```
/**
 * Displays an add or edit task screen.
 */
public class AddEditTaskActivity extends AppCompatActivity {

    public static final int REQUEST_ADD_TASK = 1;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.addtask_act);        
        ...
        // Create the presenter
        new AddEditTaskPresenter(
                taskId,
                Injection.provideTasksRepository(getApplicationContext()),
                addEditTaskFragment);            
    }
    
    ...
                
}        

```

#### 小结

至此，整个MVP示例的框架就已经很明了了，正如项目本身README文档所提到的，“业务逻辑由presenter处理，再通过View来展现Android的UI变化”。
简单小结一下，`Contract`类也就是刚才提到的契约类，用来定义`View`和`Presenter`的接口以及其各自要实现的方法，使得整个功能模块一目了然。这里的`AddEditTaskActivity`仅仅作为控制器，关联`View`和`Presenter`，而真正实现`View`接口的是`AddEditTaskFragment`。


## 一点感慨
读完代码并且尝试实践完，不吐不快的是，面向接口编程确实使得工程的设计感更强了，所需要的抽象能力也更强，整体上代码的可拓展性和可维护性，还有易删除性都有所提升。不过也要尽可能多的避免过度设计，减法永远比加法难做。以笔者现在的水平，还差得远呢，继续努力吧。