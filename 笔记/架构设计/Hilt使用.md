## 2. Hilt 应用

所有使用 Hilt 的应用都必须包含一个带有 `@HiltAndroidApp` 注释的 `Application` 类。
`@HiltAndroidApp` 会触发 Hilt 的代码生成操作，生成的代码包括应用的一个基类，该基类充当应用级依赖项容器。

```perl
@HiltAndroidApp
class MyApp : Application() {

}
```


## 3. 将对象注入 Android 类

在 `Application` 类中设置了 **Hilt** 且有了应用级组件后。其它的Android类可以使用`@AndroidEntryPoint` 表明该类会使用**Hilt**进行**依赖注入。**

```perl
@AndroidEntryPoint
class NormalActivity : AppCompatActivity() {}
```

Hilt 目前支持以下 Android 类：

- `Application`（通过使用 `@HiltAndroidApp`）
- `Activity`
- `Fragment`
- `View`
- `Service`
- `BroadcastReceiver`

如果使用 `@AndroidEntryPoint` 为某个 Android 类添加注解，则还必须为依赖于该类的 Android 类添加注解。例如，如果为某个 Fragment 添加注解，则还必须为使用该 Fragment 的所有 Activity 添加注解。

> Hilt 仅支持继承自 `<u>ComponentActivity</u>` 的 Activity，如 `<u>AppCompatActivity</u>`。  
> Hilt 仅支持继承自 `androidx.Fragment` 的 Fragment。  
> Hilt 不支持继承自 `android.app.Fragment` 的 Fragment。

在需要注入的目标对象的构造方法中加上`@Inject`注解。

``` kotlin
import javax.inject.Inject

class Target @Inject constructor() {

    fun print() {
       ...
    }
}
```

然后在Android类中引入目标对象，同样要添加`@Inject`注解。这里需要特别注意的是，**添加**`**@Inject**`**注解的对象，不能被**`**private**`**修饰，否则会出现编译报错**。

```kotlin
import dagger.hilt.android.AndroidEntryPoint
import javax.inject.Inject

@AndroidEntryPoint
class NormalActivity : AppCompatActivity() {

    @Inject
    lateinit var target: Target

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ...
        println(target.print())
    }
}
```


### 4. 使用 @Mdoue 设定 Hilt Module

有时候，类型不能通过构造函数注入。例如，不能通过构造函数注入接口。此外，也不能通过构造函数注入不归我们所有的类型，如来自外部库的类。在这些情况下，就可以使用 Hilt 模块向 Hilt 提供绑定信息。

Hilt module是一个带有 `@Module` 注释的类。它会告知 Hilt 如何提供某些类型的实例。 `@InstallIn` 为 Hilt 模块添加注释，以告知 Hilt 每个模块将用在或安装在哪个 Android 类中。

使用`@Module`表示这是一个Hilt Module，`@InstallIn(ActivityComponent::class)`表示当前 Module 中提供的对象在应用的所有 Activity 中都可以使用。

```kotlin
@InstallIn(ActivityComponent::class)
@Module
object AppModule {

    @Provides
    fun providerTarget3(): Target3 {
        val target = Target3.Builder()
            .setStr("str")
            .build()
        return target
    }
}
```

### 5. 使用 @Provides 注入实例

当一个来自外部库，或者必须使用建造者模式创建时，这时我们无法通过在构造方法上添加`@Inject`的形式来注入对象的实例，这时候我们就会用到`@Provides` 。

例如：存在一个类`Target3`类，它的构造方法被private修饰(`@Inject`不能注入被private修饰的构造方法)，我们只能通过类中提供的`build()`来创建`Target3`类

```kotlin
class Target3 private constructor() {

    private lateinit var string: String

    fun print() {
        println(this.javaClass.simpleName + ":" + string)
    }

    class Builder {
        private val target = Target3();

        fun setStr(string: String): Builder {
            target.string = string
            return this
        }

        fun build(): Target3 {
            return target
        }
    }
}
```

带有`@Provides`的方法会向Hilt告知以下信息：

- 方法返回类型会告知 Hilt 函数提供哪个类型的实例。
- 方法参数会告知 Hilt 相应类型的依赖项。
- 方法主体会告知 Hilt 如何提供相应类型的实例。每当需要提供该类型的实例时，Hilt 都会执行方法主体

```kotlin
@InstallIn(ActivityComponent::class)
@Module
object AppModule {

    @Provides
    fun providerTarget3(): Target3 {
        val target = Target3.Builder()
            .setStr("str")
            .build()
        return target
    }
}
```

然后我们就可以在Activity中使用Target3对象

```kotlin
@AndroidEntryPoint
class ProviderActivity : AppCompatActivity() {

    @Inject
    lateinit var target3: Target3

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ...
        target3.print()
    }
}
```

### 6. 注入 Context

`Target2`类在逻辑实现上需要使用到Context和Activity还有`Target3`

```kotlin
class Target2 constructor(
    val context: Context,
    val activity: Activity,
    val target: Target3
) {

    fun print() {
        println(this.javaClass.simpleName + "\n" + context + "\n" + activity + "\n")
        target.print()
    }
}
```

对于context我们可以使用`@ActivityContext`和`@ApplicationContext`来指定我们需要的是**Activity**的context还是**Application**的context。

Activity不需要添加任何注解，因为`@InstallIn(ActivityComponent::class)`存在Hilt会为我们自定注入`target2`作用域对应的Activity。

`Target3`对象同样由`providerTarget3()`为我们自动注入。

```kotlin
@InstallIn(ActivityComponent::class)
@Module
object AppModule {

    @Provides
    fun providerTarget3(): Target3 {
        val target = Target3.Builder()
            .setStr("str")
            .build()
        return target
    }

    @Provides
    fun providerTarget2(
        @ActivityContext context: Context,
        activity: Activity,
        target3: Target3
    ): Target2 {
        return Target2(context, activity, target3)
    }
}
```

`Target2`的使用就非常简单了，如下所示。

```kotlin
@AndroidEntryPoint
class ProviderActivity : AppCompatActivity() {

    @Inject
    lateinit var target2: Target2

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ...
        target2.print()
    }
}
```

### 7. 使用 @Binds 注入接口实例

实际开发中，我们经常会使用一个接口的实例，接口是没有构造方法的，这时可以通过`@Binds`来注入一个接口。`@Binds` 注释会告知 Hilt 在需要提供接口的实例时要使用哪种实现。

```kotlin
interface ISimple {

    fun print(string: String)

}
```

定一个抽象的Module，在其中注入接口的实现类，如下所示

```less
@InstallIn(ActivityComponent::class)
@Module
abstract class SimpleModule {

    @Binds
    abstract fun providerISimple(impl: ISimpleImpl): ISimple

}
```

带有`@Binds`注解的方法会向 Hilt 提供以下信息：

- 方法返回类型会告知 Hilt 函数提供哪个接口的实例。
- 方法参数会告知 Hilt 要提供哪种实现。

在接口的实现类的构造方法上添加@Inject注解：

```kotlin
class ISimpleImpl @Inject constructor() : ISimple {

    override fun print(string: String) {
        println(this::class.simpleName + ";" + string)
    }

}
```

使用起来同样非常的简单。

```kotlin
@AndroidEntryPoint
class AbsActivity : AppCompatActivity() {

    @Inject
    lateinit var simple: ISimple

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ...
        simple.print("ISimple!!")
    }
}
```

抽象类的注入与接口的注入完全相同。

### 8. 注入同一类型的不同实例

有的时候，在项目我们会经常使用同一个类的不同的实例，Module中返回同样类型的方法，还需要做些额外的操作。

首先我们需要使用`@Qualifier`来定义一个用来区分不同`Target`对象实例的注解。

```less
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class TargetType1()

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class TargetType2()
```

然后在Module中用定义好的注解，来区分不同的Target注入方式

```kotlin
@InstallIn(ActivityComponent::class)
@Module
object AppModule2 {

    @TargetType1
    @Provides
    fun providerTarget4Type1(): Target4 {
        return Target4();
    }

    @TargetType2
    @Provides
    fun providerTarget4Type2(): Target4 {
        return Target4();
    }
}
```

最后，在使用时也要添加注解来区分不同对象实例。

```less
@AndroidEntryPoint
class MultiActivity : AppCompatActivity() {

    @TargetType1
    @Inject
    lateinit var target4Type1: Target4

    @TargetType2
    @Inject
    lateinit var target4Type2: Target4

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_multi)
        println(target4Type1)
        println(target4Type2)
    }
}
```

### 9. Hilt 与 ViewModel 联合使用

Hilt 支持多种对象类型的注入，比如我们最常使用的**ViewModel**，在注入**ViewModel**之前，我们需要引入额外的依赖。

```clean
dependencies {
    ...
    implementation "androidx.hilt:hilt-lifecycle-viewmodel:1.0.0-alpha03"
    kapt "androidx.hilt:hilt-compiler:1.0.0"
    // 不是必须的，但是会方便我们使用 ViewModel 的KTX方法
    implementation 'androidx.activity:activity:1.1.0'
    implementation 'androidx.fragment:fragment-ktx:1.2.5'
}
```

在ViewModel中使用`@HiltViewModel`标记ViewModel，并在构造方法上添加`@Inject`。

```kotlin
@HiltViewModel
class SimpleViewModel @Inject constructor(
    val repository: SimpleRepository
) : ViewModel() {

    fun getData() {
        repository.getData()
    }

}
```

由于ViewModel的构造方法中引入`SimpleRepository`，所以我们需要在`SimpleRepository`的构造方法上添加`@``Inject`注解。

```kotlin
class SimpleRepository @Inject constructor() {

    fun getData() {
        println("getData")
    }
}
```

最后，在Activity中引入`SimpleViewModel`。

```kotlin
@AndroidEntryPoint
class ViewModelActivity : AppCompatActivity() {

    private val viewModel: SimpleViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_view_model)
        viewModel.getData()
    }
}
```

## 四. 组件的作用域

默认情况下，Hilt 中的所有绑定都未限定作用域。这意味着，每当应用请求绑定时，Hilt 都会创建所需类型的一个新实例。Hilt 也允许将绑定的作用域限定为特定组件。Hilt 只为绑定作用域限定到的组件的每个实例创建一次限定作用域的绑定，对该绑定的所有请求共享同一实例。

|Android 类|生成的组件|作用域|
|---|---|---|
|Application|ApplicationComponent SingletonComponent|@Singleton|
|View Model|ActivityRetainedComponent|@ActivityRetainedScope|
|Activity|ActivityComponent|@ActivityScoped|
|Fragment|FragmentComponent|@FragmentScoped|
|View|ViewComponent|@ViewScoped|
|带有 @WithFragmentBindings 注释的 View|ViewWithFragmentComponent|@ViewScoped|
|Service|ServiceComponent|@ServiceScoped||

举个例子：如果我们希望对象的实例是一个单例。首先需要指定Module安装到`SingletonComponent`上，然后在provider方法中指定其作用域为`@Singleton`，这样`Target5`在App的生命周期内，只会存在一个实例。

```less
@InstallIn(SingletonComponent::class)
@Module
object AppModule3 {

    @Singleton
    @Provides
    fun providerTarget():Target5{
        return Target5()
    }
}
```

在举个例子，如果我们希望对象的实例，在Activity的生命周期中只存在一个实例，我们可以指定它的作用域为`@``ActivityScoped`。

```less
@InstallIn(ActivityComponent::class)
@Module
object AppModule3 {

    @ActivityScoped
    @Provides
    fun providerTarget():Target5{
        return Target5()
    }
}
```

## 五. 组件的生命周期

Hilt 会按照相应 Android 类的生命周期自动创建和销毁生成的组件类的实例。

| 生成的组件                     | 创建时机                   | 销毁时机                    |
| ------------------------- | ---------------------- | ----------------------- |
| ApplicationComponent      | Application#onCreate() | Application#onDestroy() |
| ActivityRetainedComponent | Activity#onCreate()    | Activity#onDestroy()    |
| ActivityComponent         | Activity#onCreate()    | Activity#onDestroy()    |
| FragmentComponent         | Fragment#onAttach()    | Fragment#onDestroy()    |
| ViewComponent             | View#super()           | 视图销毁时                   |
| ViewWithFragmentComponent | View#super()           | 视图销毁时                   |
| ServiceComponent          | Service#onCreate()     | Service#onDestroy()     |
|                           |                        |                         |

> **注意**：  
> `ActivityRetainedComponent` 在配置更改后仍然存在，因此它在第一次调用 `Activity#onCreate()` 时创建，在最后一次调用 `Activity#onDestroy()` 时销毁。