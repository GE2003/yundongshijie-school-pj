# Android MVC架构

![image-20220906001833755](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220906001833755.png)

## 1.Controller 逻辑处理部分——持有Model层，指向Model

## 2.Model层——通过接口来通知Controller，通过接口更新View层（UI）

# Android MVVM架构

## 1.ViewModel层

### （1）ViewModel类创建

#### ViewModelProver一共有两个构造方法

![image-20220908211615155](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220908211615155.png)

(1) 第一个构造方法的第一个参数ViewModelStoreOwner是**viewmodel的持有者**，在activity中传入this即可，如果在fragment中，传入requireActivity()即可

(2) 第二个构造方法的第一个参数ViewModelStore是viewmodel的保存容器，工厂创建viewmodel之后，将放入这个容器。

两个构造方法的第二个参数是Factory参数，用于创建viewmodel对象。如果我们需要创建的是**viewmodel**则使用new ViewModelProvider.NewInstanceFactory()即可；如果需要创建的是AndroidViewModel则使用new ViewModelProvider.AndroidViewModelFactory(this.getApplication())。

##### 1.在Acitivity中的创建方法：

![image-20220908211353618](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220908211353618.png)



##### 2.在fragment中的创建方法

![image-20220908212242051](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220908212242051.png)

### （2）ViewModel中的内容是什么？

根据官方文档中的定义，ViewModel层是UI界面（View层）和数据层（Model层）之间的桥梁。

在ViewModel层中，通过LiveData组件来实现对Model层中数据的存储，并且我们通过网络获取数据的操作也是在ViewModel层中实现，从网络中获取到数据之后通过ViewModel层中定义的getter（）和setter方法来进行赋值。

![image-20220908221909342](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220908221909342.png)

除此之外，我们也可以通过DataBinding组件将VM层与Activity绑定起来，这样子可以直接在VM中实现控件的点击事件，实现UI界面的解耦。

![image-20220908223133529](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220908223133529.png)

在Activity中获取到VM的实例之后，可以通过getxxx().observe()方法来进行对UI操作的更新。

![image-20220908223558792](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220908223558792.png)

####  Fragment之间的数据共享问题

```java
public class SharedViewModel extends ViewModel {
    private final MutableLiveData<Item> selected = new MutableLiveData<Item>();

    public void select(Item item) {
        selected.setValue(item);
    }

    public LiveData<Item> getSelected() {
        return selected;
    }
}

public class ListFragment extends Fragment {
    private SharedViewModel model;

    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        model = new ViewModelProvider(requireActivity()).get(SharedViewModel.class);
        itemSelector.setOnClickListener(item -> {
            model.select(item);
        });
    }
}

public class DetailFragment extends Fragment {

    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        SharedViewModel model = new ViewModelProvider(requireActivity()).get(SharedViewModel.class);
        model.getSelected().observe(getViewLifecycleOwner(), item -> {
           // Update the UI.
        });
    }
}
```

​    请注意，这两个 Fragment 都会检索包含它们的 Activity。这样，当这两个 Fragment 各自获取 [`ViewModelProvider`](https://developer.android.google.cn/reference/androidx/lifecycle/ViewModelProvider?hl=zh-cn) 时，它们会收到相同的 `SharedViewModel` 实例（其范围限定为该 Activity）。

此方法具有以下优势：

- Activity 不需要执行任何操作，也不需要对此通信有任何了解。
- 除了 `SharedViewModel` 约定之外，Fragment 不需要相互了解。如果其中一个 Fragment 消失，另一个 Fragment 将继续照常工作。
- 每个 fragment 都有自己的生命周期，而不受另一个 fragment 的生命周期的影响。如果一个 fragment 替换另一个 fragment，界面将继续工作而没有任何问题。

### （3）Databinding

——根据布局文件的名字生成对应名字的Databinding类，生成的Databinding类可以根据控件的id直接获取到该控件

如ListView类：

布局文件：          ![image-20220907103814136](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220907103814136.png)

在ViewModel中：![image-20220907103837290](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220907103837290.png)



（2）可以通过布局文件中所定义的方法来直接对控件的值进行更新和设置

如我需要在activity中实现这个点击方法：则命名应为：data标签中所定义的类的名字.方法名

![image-20220908112008349](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220908112008349.png)

![image-20220908112018749](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220908112018749.png)

![image-20220908112041030](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220908112041030.png)

同时需要先在activity中用binding.setActivity（），此控件的点击方法才会生效

![image-20220908112139478](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220908112139478.png)

（2）单向数据绑定

实现数据变化自动驱动UI刷新的方式有三种：BaseObservable、ObservableField、ObservableCollection

### BaseObservable

一个纯净的 ViewModel 类被更新后，并不会让 UI 自动更新。而数据绑定后，我们自然会希望数据变更后 UI 会即时刷新，Observable 就是为此而生的概念

**BaseObservable** 提供了 **notifyChange()** 和 **notifyPropertyChanged()** 两个方法，前者会刷新所有的值域，后者则只更新对应 **BR** 的 **flag**，该 BR 的生成通过注释 **@Bindable** 生成，可以通过 **BR notify** 特定属性关联的视图

```java
public class Goods extends BaseObservable {

    //如果是 public 修饰符，则可以直接在成员变量上方加上 @Bindable 注解
    @Bindable
    public String name;

    //如果是 private 修饰符，则在成员变量的 get 方法上添加 @Bindable 注解
    private String details;

    private float price;

    public Goods(String name, String details, float price) {
        this.name = name;
        this.details = details;
        this.price = price;
    }

    public void setName(String name) {
        this.name = name;
        //只更新本字段
        notifyPropertyChanged(com.leavesc.databinding_demo.BR.name);
    }

    @Bindable
    public String getDetails() {
        return details;
    }

    public void setDetails(String details) {
        this.details = details;
        //更新所有字段
        notifyChange();
    }

    public float getPrice() {
        return price;
    }

    public void setPrice(float price) {
        this.price = price;
    }

}
```

实现了 **Observable** 接口的类允许注册一个监听器，当可观察对象的属性更改时就会通知这个监听器，此时就需要用到 `OnPropertyChangedCallback`

当中 `propertyId` 就用于标识特定的字段

```java
goods.addOnPropertyChangedCallback(new Observable.OnPropertyChangedCallback() {
            @Override
            public void onPropertyChanged(Observable sender, int propertyId) {
                if (propertyId == com.leavesc.databinding_demo.BR.name) {
                    Log.e(TAG, "BR.name");
                } else if (propertyId == com.leavesc.databinding_demo.BR.details) {
                    Log.e(TAG, "BR.details");
                } else if (propertyId == com.leavesc.databinding_demo.BR._all) {
                    Log.e(TAG, "BR._all");
                } else {
                    Log.e(TAG, "未知");
                }
            }
        });
```

### （4）LiveData

### （5）ViewModel的保存界面状态

![image-20220911152631843](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220911152631843.png)

## 2.Navigation组件

使用该组件首先要在build.gradle中添加依赖

**![image-20220912204442812](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220912204442812.png)**

（1）创建一个主Activity，用来装填NavHostFragment，而NavHostFragment是其他Fragment的容器，其余的Fragment都会显示在这个容器上面

主Activity的布局文件：

```java
<androidx.constraintlayout.widget.ConstraintLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">
    <fragment
            android:id="@+id/fragment_main"
            android:name="androidx.navigation.fragment.NavHostFragment"
            android:layout_width="0dp"
            android:layout_height="0dp"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintBottom_toBottomOf="parent"
            app:defaultNavHost="true"
            app:navGraph="@navigation/nav_graph" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

(2)创建Navigation的资源文件

必须给startDestination设置一个起始的Fragment

![image-20220912211501715](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220912211501715.png)

在Design视图中设置各Fragment之间的关系更加方便，在左上角手机小图标添加导航视图，小屋子图标设置起始的导航Fragment

![image-20220912211618631](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220912211618631.png)

点击箭头后，在右上角可以给跳转至此fragment设置进入与退出的动画，并设置传递参数等。

![image-20220912211734774](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220912211734774.png)

在NavHostFragment中，程序会根据你的导航Fragment的名字生成一个类，比如NavHostFragment生成的类就是NavHostFragmentDirections。每在xml中设置一个路线，就会生成相应的action。然后获取到controller，用controller.navigate()来进行界面的跳转

![image-20220912212710162](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220912212710162.png)

（3）SafeArgs安全传递参数

首先在build.gradle下添加依赖

```cpp
 classpath "android.arch.navigation:navigation-safe-args-gradle-plugin:1.0.0-alpha04"
```

在 app 下的 build.gradle 里 apply, 同步一下 gradle

```bash
apply plugin: 'androidx.navigation.safeargs'
```

# CustomView

# Activity

## ①Activity的横竖屏声明周期

在Manifest文件中添加以下声明 横竖屏切换时activity不会被销毁 防止数据丢失

![image-20220930100408033](C:\Users\GE gie gie\AppData\Roaming\Typora\typora-user-images\image-20220930100408033.png)