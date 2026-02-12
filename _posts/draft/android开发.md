# Android Studio

## 安装

1. https://developer.android.google.cn/studio
2. 初次运行报 Unable to access Android SDK add-on list：不用管，之后在首页的更多操作-SDK Manager（或设置-Language&Frameworks-AndroidSDK）点Edit下载，再删掉模拟器。也可能是如果点取消无法继续，就随便设置一下跳过。因为dl.google.com可以访问。不用装jdk
3. 汉化：https://github.com/sollyu/AndroidStudioChineseLanguagePack
4. 设置-工具-Google Accounts：IDE要开Proxy，单纯浏览器开不行，现象是浏览器访问localhost回调失败，实际上是IDE访问谷歌超时
5. 创建项目，删除`.gradle`和`gradle/gradle-wrapper`，**不要删gradle文件夹**。刷新build，点Change Gradle JDK location，改为本地安装的。

云端版：Android Studio on IDX但还在内测

## 功能

* live edit：调试阶段修改代码后推送更改，用Ctrl+引号；或右上角构建按钮右边的
* Running Devices - Toggle Layout Inspector
* snippet：comp、prev
* 右键 - 显示上下文操作 - 用小部件包装
* Log.d(tag, msg)，但好像不支持直接打印 List/Map。println会输出到info。AI推荐用timber

## 官方示例

* https://developer.android.google.cn/courses/android-basics-compose/course?hl=zh-cn
* https://www.youtube.com/@AndroidDevelopers/courses
* https://developer.android.com/samples?hl=zh-cn
* https://github.com/android/compose-samples https://github.com/android/architecture-samples 前者是多个UI/MD，后者是一个TODO APP
* 所有安卓源码：https://cs.android.com/

## gradle配置

build：

```
namespace：java代码的包名。applicationId：上架store的唯一id，相同id能覆盖升级；一般debug版保持前者不动，后者加.debug
versionCode：标识新版本的流水号。versionName：人类可读的版本号

minSdk是app能运行的最低系统版本。targetSdk代表在那个版本的系统上测试过了，特性最高使用此版本。
如果在低版本上使用了高版本特性，要区分处理：if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.Xxx) {使用高版本特性}

buildTypes：默认有debug和release
release的isMinifyEnabled一般打开，用于混淆

不同构建类型在编译期生成不同资源：buildFeatures buildConfig=true
在具体的构建类型里：buildConfigField(type="String", name="MY_VAR", value)，之后代码里BuildConfig.MY_VAR

productFlavors：如免费和收费版具有不同内容（包括源代码，在 src/变体名 里创建）。它们属于同一个dimension（是否付费），不同dimension之间以及buildType会按笛卡儿积生成构建变体。
```

依赖：

* androidx-activity-compose 给activity添加compose的桥接，现在Compose项目的Activity实际上就来自于它，内部再用系统的Activity
* VM：lifecycle-viewmodel-compose 提供viewModel()，lifecycle-runtime-compose 提供 collectAsStateWithLifecycle，lifecycle-viewmodel-ktx 提供 viewModelScope
* compose-ui-test 目前只支持junit4
* espressoCore 传统View的自动化UI测试，不要
* lifecycle-runtime-ktx 提供lifecycleScope，自动在OnDestroy里cancel协程，避免资源泄露。Compose一般用LaunchedEffect

Compose模块：

* compiler：处理@Composable，在插件依赖里
* runtime：管理节点树和状态。State、remember
* ui：layout、draw、input等
* foundation：Column、Image等
* material3：Button、Text
* animation：属于foundation的能力

--------------------------
# gradle

* https://mirrors.aliyun.com/gradle/distributions/v9.2.1/gradle-9.2.1-all.zip 不能下bin，否则idea又会去下src
* wrapper更换版本：./gradlew wrapper --gradle-version latest
* 版本兼容性：https://docs.gradle.org/current/userguide/compatibility.html https://blog.csdn.net/ys743276112/article/details/141501346

## 镜像

```
// ~/.gradle/init.gradle.kts
gradle.settingsEvaluated { // 对应settings.gradle.kts
    dependencyResolutionManagement {
        repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS)

        repositories {
            clear()
            maven("https://maven.aliyun.com/repository/public") // mavenCentral()
            maven("https://maven.aliyun.com/repository/google")
        }
    }

    pluginManagement {
        repositories {
            clear()
            maven("https://maven.aliyun.com/repository/public")
            maven("https://maven.aliyun.com/repository/google")
            maven("https://maven.aliyun.com/repository/gradle-plugin")
        }
    }
}

// 这样会清除项目里指定的。还有一种替换的方式
// 还有一个mavenLocal()，表示mvn install的，默认不加
```

## 设置

```properties
// ~/.gradle/gradle.properties

org.gradle.java.installations.auto-download=false

// 对于无持久存储的CI不开
org.gradle.caching=true // --build-cache
org.gradle.configuration-cache=true // configuration-cache.problems=warn

// 可能有插件不兼容。不要在一个模块修改另一个模块的文件。只对多模块有效
org.gradle.parallel=true // --parallel
```

本地jar包路径：~/.gradle/caches/modules-2/files-2.1，会自动清除30天未使用的。wrapper路径：~/.gradle/wrapper/dists

## 命令

* gradle tasks --all
* 编译：gradle classes
  * 打包成jar且跳过测试：gradle assemble
  * 编译+测试+打包：gradle clean build，结果在 build/libs/demo-0.0.1-SNAPSHOT.jar
  * 使用在线服务Web界面查看构建信息：--scan
* 依赖树：gradle dependencies。构建时的版本，如AGP插件：buildEnvironment。查找某个依赖是怎么来的：dependencyInsight --dependency log4j
* gradle bootRun

## 项目配置

目录结构：

```
settings.gradle.kts 内容：rootProject.name = "root-project"对应单项目或根的artifactId。include(":sub-project-a")可多参或多次使用
根build.gradle.kts：放通用配置。某些选项，子模块能自动查找根的，而某些（如编译选项）要放进subprojects{}
sub-project-a：自动对应artifactId
 └── build.gradle.kts
gradle
 └── libs.versions.toml
```

Spring：

```
plugins {
    java  // 核心自带插件。还有java-library支持api()，application提供gradle run和自动生成启动脚本，但要配置mainClass
    id("org.springframework.boot") version "4.0.0" // 第三方插件 https://plugins.gradle.org/
    id("io.spring.dependency-management") version "1.1.7" // 曾经gradle不支持bom，后来加了，但这个留下来了，不怎么更新
    // 统一插件版本：根build里写这些但加apply false。另一种做法：settings的pluginManagement
    // 给所有子项目加：subprojects{ apply(plugin = "java") }
    // 升依赖版本：com.github.ben-manes.versions，gradle dependencyUpdates
}

// 这两可以不设置，则产生的jar只有aid；安卓也没有它们。另一种做法：放进gradle.properties
group = "com.example"
version = "0.0.1-SNAPSHOT"

// 对于多模块，此项应放入subprojects{}
java {
    toolchain {
        // 指定jdk版本。如果不设置，会用运行gradle的那个jdk构建
        // 如果设置了且与当前jdk不同，会自动搜索常见目录（如SDKMAN、IDEA）使用那个，不受JAVA_HOME影响。查看当前有的：gradle javaToolchain
        // 如果还没有，有个插件能从下载源联网下到~/.gradle/jdks，但不支持wi
        languageVersion = JavaLanguageVersion.of(21)
        // 不是--release。它用tasks.withType<JavaCompile>{ options.release = 17 }
    }
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter")
    implementation不会透传，如A使用B，B使用C，则A不能使用C。如果B的某公开API的参数或返回值用了C，则B要用api(C)，则会透传
    引用多模块项目中的另一个模块：implementation(project(":common"))
    导入bom：implementation(platform(...))

    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher") // 引入junit-jupyter后可不加版本号添加。maven无此概念，效果是test的代码里无法用到
    runtimeOnly：数据库驱动、logback-classic（编译只用slf4j-api）
    compileOnly：等于mvn的provided
}

// test：
dependencies {
    testImplementation(kotlin("test")) // IDEA纯kotlin模板中的
}
tasks.withType<Test> { useJUnitPlatform() } // spring模板中的
```

libs.versions.toml：统一版本

```toml
[versions]
mylib = "1.0"

[libraries]
my-lib1 = { module = "gid:aid1", version.ref = "mylib" } // 在dependencies里使用：libs.my.lib1
my-lib2 = { module = "gid:aid2", version.ref = "mylib" } // module 可改成 group, name

[bundles]
my-lib = [ "my-lib1", "my-lib2" ] // 使用：libs.bundles.my.lib

[plugins]
my-plg = { id="gid:aid", version.ref="mylib" } // 用在plugins{ alias(libs.plugins.my.plg) } 其中alias没什么实际含义，就是要这么写
```

Kotlin；

```
plugins { kotlin("jvm") version "2.3.0" }
kotlin {
    jvmToolchain(21)  只在纯kotlin中用，默认会隐式用java的
    compilerOptions {
        freeCompilerArgs.addAll("-Xjsr305=strict", "-Xannotation-default-target=param-property")  混合kotlin和java时使用？
        jvmTarget = JvmTarget.JVM_21
    }
}
dependencies {
    implementation(kotlin("stdlib"))  当纯java项目引用kotlin标准库时使用，如果用kotlin插件则不要写
    implementation(kotlin("reflect"))  一般在编译期生成而非反射，因此默认不包含
}
```

自定义任务：

```
tasks.register<Copy>("copyTask") {  // 内置任务还有：Delete、Exec、Zip
    from("source")
    into("target")
    include("*.war")

    dependsOn("xxx") // 显式任务依赖。如果不指定，可能隐式推断，也可能并发执行
}
```

---------------
# Jetpack Compose

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {
                MessageCard("Android")
            }
        }
    }
}

@Composable
fun MessageCard(name: String) {
    Column {
        Text(text = "Hello")
        Text(text = name)
    }
}

@Preview
@Composable
fun PreviewMessageCard() {
    MessageCard("Android")
}
```

## 理论

* UI = f(State)。UI不保存状态，UI是状态的函数映射。写的时候思考 这个UI依赖哪些状态，而不是 我要怎么修改这个控件
  * 如TextField的text接受完参数就丢弃了，而不会保存。没有双向绑定，如果用户能修改组件，要通过事件给属性赋值
* 用@Composable函数代表组件，参数代表响应式状态
* 在函数内使用的局部自定义状态，可以被重新赋值：var state by remember { mutableStateOf("0") }
* flow转UI状态：val state = myFlow.collectAsState(initial = 0); Text(state.value.toString())
* DP 为布局使用的密度无关像素。SP 用于文本的可缩放像素，会根据用户在手机设置里的首选文本大小进行调整
* @Preview可以对同一个函数标记多次。一些属性：name、group。widthDp、fontScale。showBackground和backgroundColor用于透明内容。showSystemUI和device。apiLevel

### 重组

* Composable必须无状态，不能在普通语句里修改局部或全局变量，不负责 数据加载、业务逻辑、状态持久化
* 执行顺序、次数不可确定，可能并行、跳过。对于动画，可能每帧运行
* 如果还未重组完成，状态就变化了，会取消当前重组
* 修改一个属性，会在那个属性被用到的大括号（作用域）里重组

## 基础组件

```
Image：
painter = painterResource(R.drawable.profile_picture),
contentDescription = "无障碍描述，可设为null",
modifier = Modifier.size(40.dp).clip(CircleShape).border(1.dp, color, CircleShape)
保持宽高比不变但中心裁切填充满容器：contentScale = ContentScale.Crop


Text：
文字多样式buildAnnotatedString：在一个Text里显示不同样式的文字，如关键字标红
允许文字被选择：SelectionContainer{ Text }

TextField：输入框
onValueChange = { text = it } 每次输入都会触发



Scaffold：MD的标准布局，包括 顶栏、底导航栏、floating action button (FAB)、主内容区、侧边栏。
    一般是Theme下的根元素，用法：Scaffold(modifier = Modifier.fillMaxSize(), topBar={TopAppBar}, bottomBar=xxx){ innerPadding -> 子组件(modifier = Modifier.padding(innerPadding)) }

Surface：想象一张卡片，有背景（颜色），形状（圆角），边框，阴影（或TonalElevation色调高程，用颜色深度表示相对高度）。代替某些Modifier。如果只设置背景，另一种选择是Box

Icon(Icons.Default.Favorite, contentDescription = "Favorite", tint = Color.Red)

```

### Modifier

* 尺寸、间距、对齐、外观（背景、边框、阴影、形状）、可点击。对于所有组件通用的属性
* 是顺序执行的，且可重复用。如依次设定 背景、padding、背景，则第一次背景会应用于整个部分包括边距，第二次会应用于内部
* 一般定义组件时作为第一个可选参数，再传递给本组件的根布局元素
* 暴露能被点击的按钮的组件：参数里加onClick:()->Unit，组件内Modifier.clickable { onClick() }

## 布局

```
Column垂直排列多个元素。Row水平
排列子元素（固定大小>元素大小之和）：horizontalArrangement = Arrangement.SpaceBetween。或元素用Modifier.weight(1f)
性能优化：Column{ for (x in data) key(x.id) { Text(x.msg) } } 用key指定唯一标识，区分组件；直接正常声明的和LazyColumn不用

Box()：堆叠，如在图片上放文字，在按钮上放图标，后声明的在上层
排列元素：contentAlignment。也可单独设定内容在哪9个位置Modifier.align()

LazyColumn {  虚拟化
    items(msgs) { msg ->
        MessageCard(msg)
    }
}

Modifier.fillMaxWidth()
offset(x,y)

Spacer(modifier = Modifier.width(8.dp))
Divider()



padding：推荐以4.dp为增量

如果一个父项指定了明确的大小（如fillMaxSize()），它默认会要求子项填充满（否则留了空地方），子项指定的size会失效。除非父项指定wrapContentSize()，或对于容器有contentAlignment

一个组件一般以布局对象为根元素，否则内部顺序被外部决定，一般没用

HorizontalPager(rememberPagerState{有几页})
```

## 主题

```
app.ui.theme：
Color.kt 存放颜色常量：val Purple80 = Color(0xFFD0BCFF)，预定义了Color.Blue等。动态主题颜色：从用户的壁纸生成一套颜色
Type.kt 存放文字设置（排版）：val Typography = Typography(display、headline、title、body、lable的大中小=TextStyle(fontFamily等))
    要考虑的TextStyle：颜色、字体（字号、斜体、粗体、family）、文字间距、行高、下划线/删除线、对齐、溢出处理、行数限制
Shape.kt
Theme.kt 创建MaterialTheme单例
motionScheme：速度（标准、快、慢）。空间型：移动、放缩、旋转。效果型：颜色渐变。创建：standard、expressive
copy()

使用：
颜色：MaterialTheme.colorScheme.primary。背景颜色：Modifer.background()；但Surface的是自己的参数。内部元素的透明度：alpha()
    渐变色：Brush.verticalGradient(listOf(Color.Xxx)) 线性（水平、垂直、某个方向）、放射（从圆心向外扩散）、扫描（从一个角度开始做圆周运动）
字体：Text(color=xxx, style = MaterialTheme.typography.titleSmall)、h5。自定义大小：xx.sp
形状：Surface(shape = MaterialTheme.shapes.medium, shadowElevation = 1.dp)、Modifier.clip（影响整体绘制，后面调用的就在它里面），background(shape)仅限背景、border(1.dp, color, shape)
    Shapes.各种Shape。一般是设置“角”的形状

主题叠加：跟直觉一样

添加资源：AS左边工具栏的Resouce Manager
Qualifier type限定类型：根据设备的某些属性（如语言、屏幕尺寸和方向）选择最合适的资源。一般选density中的no density
使用资源：painterResource(R.drawable.filename)、stringResource
```

* 设计：https://stitch.withgoogle.com/
* 官方图标，默认只有很少几个：androidx.compose.material:material-icons-extended。导入：`import androidx.compose.material.icons.filled.*`

## 动画和手势

### 手势

* 滚动：Modifier.verticalScroll(rememberScrollState()) 可加在Text上、具有单个很高的子元素的父元素上（如Box{Column{很多}}在Box上）
  * 完全自定义的滚动行为：scrollable() 仅检测手势
  * 嵌套滚动：优先滚动子元素（称为传播），当子元素无法滚动时才滚父元素。Compose的某些组件内置了此支持。手动添加用nestedScroll
* 拖动：Modifier.draggable(rememberDraggableState { delta -> ... }) 本身只检测手势的拖动距离，一般配合加到offset上。可选限定方向
* 滑动：用户迅速拖动某元素，抬手，元素继续滚动
* 多点触控：transformable

## 处理副作用

* LaunchedEffect(key){}：进入组合或key改变时触发，会cancel老的；当key用Unit单例或true就只会触发一次。用于网络请求、动画、定时任务（LaunchedEffect(Unit){while(true) delay()}）
* rememberUpdatedState：解决 “开启了长耗时协程任务，过了一阵要用到某参数，但传入参数已经变了” 的问题。用它包装参数，之后在LaunchedEffect里使用能拿到最新值，而不会因为参数改变而触发重启
* rememberCoroutineScope：在Composable中 val scope = rememberCoroutineScope(); 在onClick等用户触发的事件中 scope.launch{ 修改或展示UI }。因为LaunchedEffect本身是composable，而onClick回调是非composable环境，用不了前者
* SideEffect{}：每次成功重组后执行。用于把Compose状态同步给非Compose对象如Analytics、日志记录
* DisposableEffect(key) { 进入组合或key改变时执行，一般是注册监听器; onDispose { key改变或组件从UI树中移除时执行，一般是反注册监听器 } }
* produceState：提供一个协程环境，用于将非Compose的异步数据源封装为State，与SideEffect数据流向正好相反。相当于 remember { mutableStateOf(initial) } + LaunchedEffect
* derivedStateOf：从多个状态计算一个派生状态，且当派生状态真正改变（同StateFlow）时，重组。如监听列表滚动（每秒触发几十次），当滚动超过100像素时显示“回到顶部”按钮。=remember(k1,k2)是输入驱动，而它是输出驱动；remember的key不变时不会进行计算，而它看计算结果是否不变从而触发重组。它本身也要放在remember里

## 状态

* 简单的UI交互状态，如展开/折叠按钮、动画进度：remember
  * remember表示只会运行一次而不会重复初始化。但不是在整个进程范围内持久缓存，而是当组件被不同调用者使用时就会不同
  * 在class或VM里不用remember
  * 状态提升State Hoisting：把它放到组件的参数里，本组件变得无状态。单向数据流UDF：状态向下传递，事件向上传递，VM → State → UI → Event → VM。如果某状态被多个组件使用，则提升到“最小公共父组件”，复杂的放到VM
* 关键的UI恢复状态，如滚动位置、输入框里的长文草稿、当前翻到的页码：rememberSaveable
  * 会将状态保存在Bundle中，生命周期长于VM，配置变化和当进程因为内存不够被系统杀死(杀后台)存活。当用户主动退出后消失
  * 保存非基本类型：给data class加@Parcelize且继承Parcelable。AI说还需要启用 kotlin-parcelize 插件
  * Bundle有1MB总容量限制
  * Nav会自动打包参数进Bundle
* 业务数据，从网络获取，如用户信息：ViewModel（配置变化存活）、SavedStateHandle（就是VM中自动同步到Bundle的Map）、数据库（DataStore）
* 存活原理：VM挂在在ComponentActivity或Fragment的ViewModelStore上
* State：是Observable的。MutableIntStateOf()

### ViewModel

```kotlin
class CounterViewModel(private val savedStateHandle: SavedStateHandle) : ViewModel() {
    private val KEY_COUNT = "count_state"

    val countState = savedStateHandle.getStateFlow(KEY_COUNT, 0)
    // 非持久属性：private val _state = MutableStateFlow(0); val state = _state.asStateFlow();

    fun increment() { // 直接设置Bundle，会触发更新
        val currentState = savedStateHandle.get<Int>(KEY_COUNT) ?: 0
        savedStateHandle[KEY_COUNT] = currentState + 1
    }

    // 与Compose耦合的简单方式：
    var cnt by mutableStateOf(0)
        private set
    fun incr() = cnt++

    override fun onCleared() {
        super.onCleared()
        // 清理协程等资源
    }
}

使用：
@Composable
fun CounterScreen(modifier = Modifier, vm: CounterViewModel = viewModel()) {
    val count by vm.countState.collectAsStateWithLifecycle() // 最小化时不收集。99%的情况用它而不是collectAsState
}

传统用法，作为Activity的成员变量：private val vm by ViewModels<MyVM>()

如果VM有构造函数参数：viewModel(factory = viewModelFactory { initializer { MyVM(args) } })
```

## Canvas

```
Canvas(onDraw= {
    drawPoints、Line、Circle、Rect、Oval（椭圆）、Arc（弧形，参数在椭圆的基础上增加角度）
})

混合模式BlendMode：当两个图形重叠时，如何显示。如区域包括重叠的和分别两个不重叠的，可以选择Xor、减去等。而不仅仅只是覆盖

点：pointMode 选择是单个实心、每两个连成线、所有点连一起。cap 首尾两端的样式如圆形/方形。pathEffect 如虚线

矩形：是否填充、圆角

Path：由直线、曲线构成的图形，类似于多边形
```

## 库

* https://google.github.io/accompanist/ 官方未转正但有用的组件

# 安卓系统

## Activity

* 生命周期：create -> start（有界面，对用户可见，但不可交互） -> resume（完全可见且活跃，相当于running） -> pause（前台被其他东西覆盖，unfocus；-> resume） -> stop（最小化；-> restart -> start） -> destroy
  * Compose只区分以下状态：CREATED、STARTED、RESUMED、DESTROYED
* 配置变化：旋转屏幕、切换黑暗模式、切换语言。会杀掉activity重建
* Fragment：轻量Activity，有自己的生命周期，但必须依附于Activity存在。一般在非Compose中根据导航切换多个Fragment，或平板上左右同时显示
* Compose一般单Activity。导航：NavHost内部切换，而不是Intent。生命周期：onRemembered, onAbandoned

### Context

* 使用安卓系统功能时用到
* 是Activity、Service、Application的父类，一个它们的实例会附加在一个ctx上，具有对应相同的生命周期
* 在单例中持有Activity的Context会导致资源泄漏。使用ctx前一般要检测有效性
* 用途
  * 请求权限：ActivityCompat.requestPermissions要使用Activity的Context
  * 使用系统服务（此处ktx隐式处理了）：`getSystemService<NotificationManager>()`
  * 访问应用资源：context.getString(R.string.app_name)
  * 读取内部储存
* Application的Context与UI无关
* ContextCompat：兼容工具类

### Task

* 一个screen栈。screen可以是Activity和Dialog等。按多任务按键看到的就是
* 但已经结束的程序也可能出现在多任务列表里，只是一种预览
* 不同Task但相同的Task Affinity，多任务列表里只会显示出一个，其他的可能活着但没显示
  * Task具体的Affinity值会按第一个启动的Activity，之后再在里面启动Activity不会管Affinity

### Launch Mode

有一个A，用户在用B，点击链接跳到A，行为是什么。

* 标准模式：再开一个Activity，且在B的Task里开，对A的Task无影响
* Single Top：类似于标准模式。但如果已经有了Activity，不会再开。接收者override onNewIntent
* Single Task：在A的task里（可能已有一些Activity）新建Activity。如果那个Activity已存在，不会创建，而是调用onNewIntent()；如果那个Activity上有东西，会被清掉。用户返回时会先在A里回退
* Single Instance：类似于Single Task，但上下都不允许有Activity；如果现存task没有符合的，会新建task（即一个程序多个task）。隔离性更强，如用于支付界面

一般App内部用标准和SingleTop。SingleInstance用于外部的。SingleTask内外部都用得到。

## Intent

启动另一个activity（显式intent）：

```kotlin
val intent = intentFor<SecondActivity> { // 自身的Activity。传统创建方式：Intent(applicationContex, SecondActivity::class.java)
    putExtra(k,v) // 放Bundle。其它创建方式：bundleOf(k to v)、putString、putExtras(b)
}
context.startActivity(intent)

// 其它包的Activity
Intent(Intent.ACTION_MAIN).also {
    it.package = "另一个程序的包名"
    try { startActivity(it) } catch (e: ActivityNotFoundException)
}

// 如果context是Activity，推荐传REQUEST_CODE：if (context is Activity) { context.startActivityForResult(intent, REQUEST_CODE) }
```

隐式intent，如分享、编辑文件，会显示应用列表给用户选择：

```kotlin
val i = Intent(Intent.ACTION_SEND).also {
    type = "text/plain"
    putExtra(Intent.EXTRA_EMAIL, arrayOf("xxx"))
}
if (i.resolveActivity(packageManager) != null) startActivity(i) 还要先在manifest里加queries
注册作为intent的接收者：manifest里的activity的intent-filter
```

## Uri

```kotlin
val uri = Uri.parse("android.resource://&$packageName/drawable/xxx")
val xxxBytes = contentResolver.openInputStream(uri)?.use { it.readBytes() }

File(filesDir, "filename")  app私有存储
file.toUri()

// 启动相册app，选择内容。那个app有权限访问外部存储。本应用无需特别权限。此Uri是临时的
val pickImage = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.GetContent(),
    onResult = { contentUri -> ... }
)
pickImage.launch("image/*")

ContentProvider：即上面的相册app的角色。contentResolver就是其使用者。


val projection = arrayOf(MediaStore.Images.Media._ID) 相当于SELECT。一个图像有很多种属性，根据需要从数据库里取一部分
selection：相当于WHERE、filter。如果不设置，就是获取所有内容。当想获取昨天拍的就设置


contentResolver.query(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,)
```

## Broadcast

接收~：如切换飞行模式时做什么、用户接到电话时做什么

```
class MyReceiver: BroadcastReceiver() {
    重写 onReceive：if (intent?.action == Intent.ACTION_AIRPLAIN_MODE_CHANGED)
}
再在Activity里，val rcv = BroadcastReceiver(); registerReceiver(rcv, IntentFilter(Intent.ACTION_AIRPLAIN_MODE_CHANGED))
再在onDestroy里unregisterReceiver(rcv)

这称为动态receiver，只有app运行的时候才行。还有一个静态的，app不运行也可以，但限制多
```

## 相关开发软件

* LibChecker
* 快捷方式 github.com/sdex/ActivityManager

# 参考

* https://space.bilibili.com/27559447/
* https://www.youtube.com/@PhilippLackner

## 待看

* https://docs.gradle.org/current/userguide/best_practices.html
* https://jetpackcompose.cn/docs/
* https://juejin.cn/user/2384195547303688/columns

## 其他项目示例

* https://github.com/FunnySaltyFish/Transtation-KMP
