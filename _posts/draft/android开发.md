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
* Logcat
  * Log.d(tag, msg)，但好像不支持直接打印 List/Map。println会输出到info。AI推荐用timber
  * is:crash、package:mine
* 设置 - 搜索adb - Burst Mode开启
* 捕获异常：断点选择 Java异常断点 - 任何异常 - 通知“捕获异常” - 类筛选器填`自己的包名.*`。或 代码 - 分析堆栈跟踪

## 官方示例

* https://developer.android.google.cn/courses/android-basics-compose/course?hl=zh-cn
* https://www.youtube.com/@AndroidDevelopers/courses
* https://developer.android.com/samples?hl=zh-cn
* https://github.com/android/compose-samples https://github.com/android/architecture-samples 前者是多个UI/MD，后者是一个TODO APP
* 所有安卓源码：https://cs.android.com/

## 构建

Build - Generate Signed App bundle or APK - APK。如果未创建密钥，点Create new，Key Store和密钥的密码要分别设置。创建好后勾上记住密码。最后会生成到 项目根目录/app/release/app-release.apk。可以上传到应用商店。

开发版，不用签名：Build - Assemble Project，生成到 项目根目录/app/build/outputs/apk/debug/app-debug.apk或release/app-release-unsigned.apk；NDK的在aar/lib-release.aar。如果有多个variant，会全部构建。实测miui如果不签名会报“packageinfo is null”

查看apk内容：Build - Analyze APK

hm4.2无法覆盖安装app：AS运行配置里勾上always install with package manager

## gradle配置

gradle.properties：android.useAndroidX、newDsl 等，在AGP9默认都改为了true。enableJetifier默认还是false，它重写二进制自动迁移第三方库，但会延长构建时间，构建报告可以显示是否能够移除

### build

```
namespace：java代码的包名。applicationId：上架store的唯一id，相同id能覆盖升级；一般debug版保持前者不动，后者加.debug
versionCode：标识新版本的流水号。versionName：人类可读的版本号

minSdk是app能运行的最低系统版本，影响安装。targetSdk代表在那个版本的系统上测试过了，特性最高使用此版本
行为：min(targetSdk, 实际系统版本)，即如果targetSdk比实际系统版本高，则以实际系统版本为准，反之以targetSdk为准
当minSdk小于targetSdk，要调用高版本API时区分处理：if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.Xxx) {使用高版本特性}
必须设置minSdk，否则默认为1。compileSdk不设置时报warning。targetSdkVersion现在默认等于compileSdk

buildTypes：默认有debug和release。二者公共的内容写进defaultConfig
release { isMinifyEnabled=true; isShrinkResources=true } 前者会压缩+混淆，会运行ProGuard。后者依赖前者，会删除未用资源

不同构建类型在编译期生成不同资源：buildFeatures buildConfig=true
在具体的构建类型里：buildConfigField(type="String", name="MY_VAR", value)，之后代码里BuildConfig.MY_VAR

productFlavors：如免费和收费版具有不同内容（包括源代码，在 src/变体名 里创建）。它们属于同一个dimension（是否付费），不同dimension之间以及buildType会按笛卡儿积生成构建变体。
自定义buildTypes的debuggable默认false
```

#### NDK

* 不用设置ndkVersion，构建时根据AGP自动决定版本。也不用在AS里装NDK和CMAKE，构建时自动下载
* jniDebuggable已废弃
* 不调试C++代码：上面“运行/调试配置” - Debugger - Debug type：Java only

```
// 对于lib模块：
defaultConfig {
externalNativeBuild {
    cmake {
        arguments += listOf("-DANDROID_STL=c++_shared") // 传给CMake命令行参数。此处默认static性能好，但若应用包含多个共享库则要用shared
        cppFlags += listOf("-flto") // 传给编译器的参数。此处应写在release里
    }
    ndk { // 仅构建指定ABI。其中x86_64是模拟器要用的
        abiFilters += listOf("arm64-v8a", "x86_64") // 此选项也可在app模块中使用，会过滤lib模块的库
        // 另一种选择：在cmake里指定这两个，为了开发期模拟器调试（默认x64），同时ndk只指定arm64
    }
}}

buildTypes { release {
    isDefault = true // 但好像没用

    consumerProguardFiles("proguard-rules.pro") // 当lib被app使用时，传递proguard文件；不需要proguardFiles。因为是否启用混淆由app决定
}}

externalNativeBuild {
cmake {
    path = file("src/main/cpp/CMakeLists.txt")
}}

安卓原生库：https://developer.android.com/ndk/guides/stable_apis?hl=zh-cn 要配置CMake

// 根CMakeLists.txt
cmake_minimum_required(VERSION 3.18.1)
project(jopus)

add_subdirectory(opus) // 带有CMakeLists.txt的子项目目录

add_library(${CMAKE_PROJECT_NAME}
    SHARED
    jopus.cpp
)

target_link_options(${CMAKE_PROJECT_NAME}
    PRIVATE
    -Wl,--version-script,${CMAKE_SOURCE_DIR}/libjopus.map.txt
    -Wl,--no-undefined-version
)
set_target_properties(${CMAKE_PROJECT_NAME}
    PROPERTIES
    LINK_DEPENDS ${CMAKE_SOURCE_DIR}/libjopus.map.txt
)
target_link_libraries(${CMAKE_PROJECT_NAME}
    opus
    log // <android/log.h> __android_log_print(ANDROID_LOG_ERROR, TAG, fmt, args)
)

// libjopus.map.txt
LIBJOPUS {
  global: // 导出符号列表
    JNI_OnLoad; // 动态注册，性能反而比静态查找更好。由ART自动调用
  local: // 隐藏其他符号
    *;
};

// jopus.cpp
#include <jni.h>

创建函数，签名加JNICALL，但函数名正常写、不要JNIEXPORT。只需要用JNI_OnLoad

// https://developer.android.com/training/articles/perf-jni?hl=zh-cn#native-libraries
JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved) { // 其他部分都是固定的
    JNIEnv* env;
    if (vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6) != JNI_OK) return JNI_ERR;

    jclass c = env->FindClass("ashipo/jopus/Opus"); // 此处修改为kt层声明了external函数的包名和类名
    if (c == nullptr) return JNI_ERR;

    static const JNINativeMethod methods[] = { // 此处修改为kt层声明了external的函数的方法名和签名
        {"nativeFoo", "(II)I", reinterpret_cast<void*>(nativeFoo)},
    };
    int rc = env->RegisterNatives(c, methods, sizeof(methods)/sizeof(JNINativeMethod));
    if (rc != JNI_OK) return rc;

    return JNI_VERSION_1_6;
}

// ProGuard，保护类名不被混淆，且保留native方法
-keep class ashipo.jopus.Opus {
    native <methods>;
}
```

### 依赖

```
[versions]
agp = "9.1.0"
kotlin = "2.3.10"

coreKtx = "1.17.0"
activityCompose = "1.12.4" # 给activity添加compose的桥接。现在Compose项目的Activity父类就来自于它
composeBom = "2026.02.01"

[libraries]
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "coreKtx" }
androidx-activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "activityCompose" }

androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "composeBom" }
androidx-compose-material3 = { group = "androidx.compose.material3", name = "material3" }
androidx-compose-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
androidx-compose-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
androidx-compose-material-icons = { module = "androidx.compose.material:material-icons-core" } # 可选extended

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" } // 库用.library。不再需要org.jetbrains.kotlin.android
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }

dependencies {
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.activity.compose)

    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.compose.material3)
    debugImplementation(libs.androidx.compose.ui.tooling)
    implementation(libs.androidx.compose.ui.tooling.preview)
    implementation(libs.androidx.compose.material.icons.core)

//    implementation("androidx.lifecycle:lifecycle-runtime-ktx")  // 提供lifecycleScope，自动在OnDestroy里cancel协程，避免资源泄露。Compose一般用LaunchedEffect
//    testImplementation("junit:junit")
//    androidTestImplementation("androidx.test.ext:junit")
//    androidTestImplementation(platform(libs.androidx.compose.bom))
//    androidTestImplementation("androidx.compose.ui:ui-test-junit4")  // 目前只支持junit4
//    debugImplementation("androidx.compose.ui:ui-test-manifest")
//    espressoCore 传统View的自动化UI测试
}
```

### Manifest

https://developer.android.com/guide/topics/manifest/manifest-intro?hl=zh-cn

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">  以前在这里设置packageName

    <application
        android:name=".MyApplication"  如果没有自定义则不用写
        android:allowBackup="true"  是否允许系统备份应用数据，包括DataStore，默认true
        android:dataExtractionRules="@xml/data_extraction_rules"  安卓12规则，区分了“云端备份”和“设备间直传”。老版fullBackupContent
        android:icon="@mipmap/ic_launcher"  还有roundIcon。安卓8 Adaptive Icons 在mipmap-anydpi-v26里定义前景和背景，系统自动裁剪形状
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.GreetingCard"  用于Splash屏幕，在执行任何代码前加载
        tools:targetApi="35"  仅限本文件Lint，类似于if Build.VERSION.SDK_INT，本文件里写高版本功能、IDE标红时自动加
    >
        <activity
            android:name=".MainActivity"
            android:exported="true"  用于接收外部intent。主Activity必为true。即使为true，还有其它方式限制公开范围
            android:label、theme：不设置时会继承application的。用于被intent调用时显示
        >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />  这两项表示App入口，系统会把图标放在桌面上
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

--------------------------
# gradle

* https://mirrors.aliyun.com/gradle/distributions/v9.2.1/gradle-9.2.1-all.zip 不能下bin，否则idea又会去下src
* wrapper更换版本：./gradlew wrapper --gradle-version latest
* 版本兼容性：https://docs.gradle.org/current/userguide/compatibility.html https://blog.csdn.net/ys743276112/article/details/141501346

## 镜像

```kotlin
// ~/.gradle/init.gradle.kts
gradle.settingsEvaluated { // 下面对应settings.gradle.kts
    dependencyResolutionManagement {
        repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS)
        repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)

        repositories {
            clear()
            maven("https://maven.aliyun.com/repository/google") {
                content {
                    includeGroupByRegex("com\\.android.*")
                    includeGroupByRegex("com\\.google.*")
                    includeGroupByRegex("androidx.*")
                }
            }
            maven("https://maven.aliyun.com/repository/public")
        }
    }

    pluginManagement {
        repositories {
            clear()
            maven("https://maven.aliyun.com/repository/google") {
                content {
                    includeGroupByRegex("com\\.android.*")
                    includeGroupByRegex("com\\.google.*")
                    includeGroupByRegex("androidx.*")
                }
            }
            maven("https://maven.aliyun.com/repository/public")
            maven("https://maven.aliyun.com/repository/gradle-plugin")
        }
    }
}


// 这样会清除项目里指定的。还有一种替换的方式
// 还有一个mavenLocal()，表示mvn install的

未测试：
afterProject{
    rootProject{
        tasks.named('wrapper') {
            distributionUrl = "https://mirrors.cloud.tencent.com/gradle/gradle-" + gradleVersion + "-" + (distributionType.name().toLowerCase(Locale.ENGLISH)).toString() + ".zip"
        }
    }
}

gradle.properties中：
systemProp.org.gradle.internal.services.base.url=https\://my-artifactory-server.local/services-gradle-remote
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

# 如果AS设置了代理，而gradle没设置，会提示写入gradle的配置。但用了镜像不需要设置，点No
# systemProp.http.nonProxyHosts=*.aliyun.com
# systemProp.http.proxyHost=\:\:1
# systemProp.http.proxyPort=1080
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

升依赖版本：com.github.ben-manes.versions插件，gradle dependencyUpdates
直接指定最新版：+、1.+。但对于kotlin编译器会下到RC且又用不了。而且每次构建都会检查版本拖慢速度

菱形依赖冲突：默认选择依赖树中的最高版本。改为编译报错：resolutionStrategy{failOnVersionConflict()}
手动控制版本：
1. constraints { implementation(...) { because("...") } }
2. 声明直接依赖（在现在有1的情况下不推荐）
3. implementation(...) { exclude(group, module) }
4. 富版本约束：默认prefer语义，允许升降。require不允许降低到此版本以下。reject禁止某版本。strictly不允许升降版本
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
            MyMaterialTheme {
                MessageCard("Android")
            }
        }
    }
}

@Composable
@Preview
fun MessageCard(name = "World") {
    Column {
        Text("Hello")
        Text(name)
    }
}
```

## 理论

* UI = f(State)。UI不保存状态，UI是状态的函数映射。写的时候思考 这个UI依赖哪些状态，而不是 我要怎么修改这个控件
  * （近似）MVI架构：State不可变，V在发生事件时给VM发出单个Intent，VM将State复制一份，修改完后整体原子性更新。而MVVM可能发出多个Command
  * 单向数据流UDF：状态向下传递，事件向上传递，VM → State → UI → Event → VM
* 用@Composable函数代表组件，参数代表响应式状态
* 在函数内使用的局部自定义状态，可以被重新赋值：var state by remember { mutableStateOf("0") }
* DP 为布局使用的密度无关像素，一般以4为增量；可触摸尺寸最小48。SP 用于文本的可缩放像素，会根据用户在手机设置里的首选文本大小进行调整
* @Preview可以对同一个函数标记多次。一些属性：name、group。widthDp、fontScale。showBackground和backgroundColor用于透明内容。showSystemUI和device。apiLevel

Compose模块：

* compiler：处理@Composable，在插件依赖里
* runtime：管理节点树和状态。State、remember
* ui：layout、draw、input等
* foundation：Column、Image等
* material3：Button、Text
* animation：属于foundation的能力

### 重组

* 每一帧：组合 -> 布局 -> 渲染
* Composable必须无状态，不能在普通语句里修改局部或全局变量，不负责 数据加载、业务逻辑、状态持久化
* （不同块的）执行顺序、次数不可确定，可能并行、跳过、反复执行。动画可能每帧运行
* 乐观：如果还未重组完成，状态就变化了，会取消当前重组，只依据最新状态
* 重组范围：修改一个属性，基本就是在那个属性被用到的大括号（作用域）里重组，不直接影响子树也不影响父级（但Column除外）
  * 比较参数时，对于自定义类型，添加@Stable，可以让它的equals被认为可信，从而在相等时跳过重组。如果不加，因为引用类型，给属性赋值时对象引用不变，equals不可信，导致即使不变也会进行重组。@Immutable：不要用，被Stable替代

### CompositionLocal

* 如MaterialTheme，表面上是全局单例变量，但在那个代码块范围内是可被控制的；或理解为不用在函数上声明的函数参数，且在多个函数中通用某数据。类似于ThreadLocal
* 获取：val x = ~.current，不需要remember
* 创建：val LocalXxx = staticCompositionLocalOf { 初始值 }; 设定：CompositionLocalProvider(LocalXxx provides 新值) { 使用新值 }
  * 如果用它来做“依赖注入”，则用static的。如果值经常改变需要UI重绘，则不用static的。static的也能变但重组范围更大，不变时性能更好
* LocalContext、LocalActivity：一个Composable本身不能确定在哪个ctx里调用。如果需要具体的Activity则用后者as

## 基础组件

```
Image：
painter = painterResource(R.drawable.profile_picture),
contentDescription = "无障碍描述，可设为null",
modifier = Modifier.size(40.dp).clip(CircleShape).border(1.dp, color, CircleShape) 优先用Surface(M.size,shape)承载
保持宽高比不变但中心裁切填充满容器：contentScale = ContentScale.Crop
加载网络图片：AsyncImage https://github.com/coil-kt/coil/blob/main/README-zh.md
对图片进行处理（如压缩）：BitmapFactory.decode(bytes,0,bytes.size)，之后再写入ByteArrayOutputStream，再写入文件。显示：asImageBitmap()

Text：
文字多样式buildAnnotatedString + withStyle：在一个Text里显示不同样式的文字，如关键字标红、链接，类似于span
允许文字被选择：SelectionContainer{ Text }

TextField、OutlinedTextField：输入框
onValueChange = { textState = it } 每次用户输入都会触发。text参数接受完实参就丢弃了，必须用事件更新state
    另一种方式：rememberTextFieldState() 类似于双向绑定
label。singleLine 默认false。
leadingIcon、trailingIcon。prefix如$符号，suffix如kg。supportingText如“必填”或显示错误信息（配合isError）
设置要使用的输入法键盘：keyboardType，如Number、Password。还有设置“提交”键是换行还是下一项
visualTransformation：如将密码显示为*（自带）、每4位插入-
BasicTextField：一个单纯允许用户输入的区域，在content里用innerText()放置

Icon(Icons.Default.Favorite, tint = Color.Red) Default对应Filled实心，还有Outlined空心、Rounded圆角、Sharp尖角、TwoTone双色

Button：里面自带Row
IconButton、OutlinedButtion（一般不设置阴影高度）、TextButton（无边框，看起来像文字）
ToggleButton

Checkbox、RadioButton、Switch
```

### Modifier

* 尺寸、间距、对齐、外观（背景、边框、阴影、形状）、可点击。对于所有组件通用的属性
* 是顺序执行的，且可重复用。如依次设定 背景、padding、背景，则第一次背景会应用于整个部分包括边距，第二次会应用于内部
* 一般定义组件时作为第一个可选参数，再传递给本组件的根布局元素，如Surface或Column
* 暴露能被点击的按钮的组件：参数里加onClick:()->Unit，组件内Modifier.clickable { onClick() }。还有toggleable、selectable
* 设定默认值再允许外部覆盖：Modifier.xxx().then(modifier参数)

## 布局

布局对象：

* 一个组件一般以布局对象为根元素，否则内部顺序被外部决定，一般没用

```
Column垂直排列多个元素。Row水平
排列子元素（容器固定大小 > 子元素大小之和）：horizontalArrangement = Arrangement.SpaceBetween 默认左排列。spacedBy固定间隔大小也支持对齐
    不保留空白：子元素用Modifier.weight(1f)
性能优化：Column{ for (x in data) key(x.id) { Text(x.msg) } } 用key指定唯一标识，区分组件，当进行非append的修改时保留前面的不重组。直接正常声明的和LazyColumn不用，但嵌套Lazy容器要用，否则会丢失状态

Box()：堆叠，如在图片上放文字，在按钮上放图标，后声明的在上层
排列元素：contentAlignment。也可对子元素单独设定在哪9个位置Modifier.align()

LazyColumn {  虚拟化。内置滚动
    items(msgs, key={it.id}) { msg ->
        MessageCard(msg)
    }
    item { 单个元素 }
    items(Int)

    还支持设定内容间的间距（如果在元素上设定，每个都要用Modifier，且间距翻倍）、容器到内容之间的间距（滚动到内容底部时还能再滚一段留白）
    stickyHeader
    响应滚动位置，如“回到顶部按钮”：rememberLazyListState()的某个属性
    Modifier.animateItem() 项发生位置交换 添加 删除时动画

    LazyVerticalGrid如相册
    分页：androidx.paging:paging-compose：将数据构建为Flow<PagingData<T>>，itmes(flow.collectAsLazyPagingItems())
}

FlowRow、FlowColumn：如果布局中有子项无法被安排在同一行，会自动换行


Scaffold：MD的标准布局，包括 顶栏、底导航栏、floating action button (FAB)、主内容区、侧边栏。如果组件是一个完整的屏幕页面，根元素通常是它
Scaffold(Modifier.fillMaxSize()){ innerPadding -> 子组件(modifier = Modifier.padding(innerPadding)) }
topBar={TopAppBar()}
bottomBar：有BottomAppBar和BottomNavigation两种，前者类似于把顶栏放在底部，不过还加了一个裁切功能方便放FAB
FloatingActionButton(FAB)：MD右下角的按钮，估计始终显示在最上面
Snackbar：一种MD规范的应用内通知，出现在底部大横条。scope.launch{rememberScaffoldState()的snackbarHostState.showSnackbar("信息")}
模态抽屉式导航栏（左右弹出）：drawerContent，scaffoldState.drawerState.open()。不用Scaffold时：ModalDrawer、BottomDrawer（底部弹出）
背景幕（前后两层）：BackdropScaffold

Surface：带样式的盒子。想象一张卡片，有背景（颜色），形状（圆角），边框，阴影（或TonalElevation色调高程，用颜色深度表示相对高度）。相比于Box+Modifier，具有更多MD效果、内部Text自动适配颜色、设置shape自动裁剪内容、可以嵌套，内部自动更改能区分的颜色
Card：Surface的语义化封装，设定了内边距等。如 列表项、网格项、信息卡片。不要嵌套

Dialog：有个全屏遮罩，中间显示的组件。点击周围一般可以取消。
AlertDialog：预定义了 标题、内容、确认/取消按钮

Spacer(Modifier.width(8.dp))
Divider(startIndent, thickness)
```

布局属性：

```
布局核心原则：约束向下传递（父->子 最大最小宽高），尺寸向上传递（子决定自己的实际大小，->父），父容器决定子组件位置
测量阶段 (Measurement)：父组件问子组件：“给你这些约束，你多大？”
决定父组件自身大小 layout(w,h)
放置阶段 (Placement)：父组件根据测量结果，决定子组件的坐标 (x, y)
Modifer本质上是修改Layout的某个值，也会经历这些阶段，返回的测量结果传给链条中的下一个节点

fillMaxSize：将自身大小设置为父元素允许的最大大小。参数可设定百分比
wrapContentSize：自身大小根据子元素内容决定大小（但不会超过父元素限制），且默认将子元素（如果有多个）居中放置（可改）
    二者结合实现自身居中（类似于margin auto）：fillMaxSize().background(Color.Red).wrapContentSize() 后者解除前者强制占满的约束
weight：Row/Column特有，用在子元素上。当父元素大小确定时，根据剩下的空白大小改变自身大小。只有一个元素使用它时，效果为占用剩下所有空间。会测量两次，先计算没有此参数的，再排列具有此参数的
matchParentSize：Box特有，用在子元素上。先测量Box（的子元素）的大小（该子元素不计入），再将该子元素设置为Box的大小
aspectRatio(1f)：确定宽高其一后（可以随父容器变化），设定另一个，保持比例，1就是正方形
此处的自身，应理解为那几个布局组件，或以它们为根元素且透传Modifier的自定义组件


最大/最小约束：
布局容器的对其子组件的最小约束默认传递为0。Box中有个propagateMinConstraints进行传递。其它情况一般是对某元素自身设置最小值
给某组件设置 size(100.dp)，但父容器传递的 maxWidth 只有 50.dp，会发生什么？
    结果：组件会被强制压缩到 50.dp。在 Compose 中，父容器的约束具有最高优先级
类似的：如果父指定了最小大小，而子组件的大小小于它，则会扩大
requiredSize()：无视父容器约束，坚持自己的尺寸
sizeIn(min, max)：设定当前元素的最大最小约束。size()和fillMaxSize本质是设置最小=最大

尺寸修饰符尽早写：fillMaxWidth().padding() 通常比 padding().fillMaxWidth() 更符合预期
Modifier.offset(x, y) 在放置阶段挪动位置，不影响测量阶段占用的空间

enableEdgeToEdge()：在setContent之前使用，将状态栏和导航栏设置为透明，允许应用内容绘制到系统栏下方
避开不可交互区：Modifier.safeXxxPadding。当用Scaffold时不用
    safeDrawingPadding：状态栏、导航栏、挖孔屏
    safeContentPadding：还包括手势区、IME键盘。如有一个输入框，当键盘弹起时，整个布局会自动向上缩，保证输入框可见


固有特性测量（Intrinsic Measurements）、涉及“二次测量”：
想让父容器“刚好包裹住最长的那个子组件”，如 一个Row里面有两个Text，中间有一个VerticalDivider。
Row(M.height(IntrinsicSize.Min)){Text(M.weight(1f)); VerticalDivider(M.fillMaxHeight()); Text(M.weight(1f))}
标准流程：测量两个Text得到大小，但测量VerticalDivider时自身大小还不知道。如果不用固有特性测量，VerticalDivider的高度会得到0。
但如果外层有确定的高度，如Column(M.height){Row{...}}，则没问题。即当父容器（比如 Box）没有设置大小，而子组件使用了 Modifier.fillMaxSize()，父容器会尝试满足子组件，从它的父级（比如屏幕）那里继承 最大约束。


自适应布局：

TODO: 实测手机横屏也满足宽度大于600

BoxWithConstraints{ val isTablet = maxHeight >= 600.dp }：想手动根据屏幕大小改变布局结构。引入“延迟重组”

androidx.compose.material3.adaptive的currentWindowAdaptiveInfo().windowSizeClass;
决定是否显示某组件的变量：val showTopAppBar = windowSizeClass.isHeightAtLeastBreakpoint(WindowSizeClass.HEIGHT_DP_MEDIUM_LOWER_BOUND)
WindowSizeClass实际上来自于Jetpack WindowManager

判断横竖屏：val isPortrait = LocalConfiguration.current.orientation == Configuration.ORIENTATION_PORTRAIT 竖屏，Landscape横屏

if (LocalWindowInfo.current.containerDpSize.width >= 600.dp) { 宽屏TabScreen() } else { PhoneScreen() }
```

## 主题

```
app.ui.theme：
Color.kt 存放颜色常量：val Purple80 = Color(0xFFD0BCFF)其中最高位的代表透明度，预定义了Color.Blue等。动态主题颜色：从用户的壁纸生成一套颜色。默认未经定制的不支持暗色模式
Type.kt 存放文字设置（排版）：val Typography = Typography(display、headline、title、body、lable的大中小=TextStyle(fontFamily等))
    要考虑的TextStyle：颜色、字体（字号、斜体、粗体、family）、文字间距、行高、下划线/删除线、对齐、溢出处理、行数限制
    TextAlign.Justify：先按正常铺文本，如果换行了，再把那一行的空白均匀利用
Shape.kt
Theme.kt 创建MaterialTheme单例
motionScheme：速度（标准、快、慢）。空间型：移动、放缩、旋转。效果型：颜色渐变。创建：standard、expressive。但无法赋值，MaterialTheme的公开API没有这个参数的重载。根据文档所说，要1.5才在正式版有

使用：
颜色：MaterialTheme.colorScheme.primary。背景颜色：Modifer.background()；但Surface和MD组件是自己的color参数。内部元素的透明度：alpha()
    渐变色：Brush.verticalGradient(listOf(Color.Xxx)) 线性（水平、垂直、某个方向）、放射（从圆心向外扩散）、扫描（从一个角度开始做圆周运动）
字体：Text(color=xxx, style = MaterialTheme.typography.titleSmall)、h5。自定义大小：xx.sp。以作用域方式使用：ProvideTextStyle(style){ 影响所有Text。行内样式不用它 }
形状：Surface(shape = MaterialTheme.shapes.medium, shadowElevation = 1.dp)、Modifier.clip（影响整体绘制，后面调用的就在它里面），background(shape)仅限背景、border(1.dp, color, shape)
    Shapes.各种Shape。一般是设置“角”的形状

这个MaterialTheme（不是同名函数）是个CompositionLocal
Theme嵌套：默认不是CSS的层叠效果，对于内层未设置的内容，会用MD的默认值。实现叠加：先获取当前方案，然后用copy(修改单个值)或merge(style)保留旧值，再传给MaterialTheme(修改的scheme, content)，未修改的scheme会自动读取当前的
扩展、添加Assets：创建一个类，声明需要的资源。两个单例对象继承它，分别放亮/暗主题。创建CompositionLocal，亮色 as 父类。创建扩展属性：val MaterialTheme.myAssets @Composable @ReadonlyComposable get() = LocalMyAssets.current

添加资源：AS左边工具栏的Resouce Manager
Qualifier type限定类型：根据设备的某些属性（如语言、屏幕尺寸和方向）选择最合适的资源。一般选density中的no density
使用资源：painterResource(R.drawable.filename)、stringResource、dimension（统一dp）、color

primary：整个应用最常使用的主色。primaryVariant：主色的变种，用于与主色调区分
secondary：次选色。用于悬浮按钮、checkbox、radiobutton、链接标题
background：Scaffold的背景色
surface：用于Surface、Sheet、Menu
error
onPrimary：当某组件用primary作为背景色时，上面的文字用本色
```

* 设计：https://stitch.withgoogle.com/
* 官方图标：androidx.compose.material:material-icons-extended。导入：`import androidx.compose.material.icons.filled.*`。core包比较小，只有icons.Default，实际上也是filled。现在不更新了，推荐手动下载：https://fonts.google.com/icons

## 状态

* 简单的UI交互状态，如展开/折叠按钮、动画进度：remember
  * remember表示只会运行一次而不会重复初始化。但不是在整个进程范围内持久缓存，而是当组件被不同调用者使用时就会不同。如果声明在if里，某次没执行到，则下次会恢复初始值，相当于组件被移除
  * 在class或VM里不用remember
  * 从参数进行计算：=remember(ndx){arr[ndx]}。其实by对应的是state
  * 状态提升State Hoisting：把它放到组件的参数里，本组件变得无状态。如果某状态被多个组件使用，则提升到“最小公共父组件”，复杂的放到VM
  * 底层：数据放在UI树调用点的Slot Table里，remember变量相当于索引
* 关键的UI恢复状态，如滚动位置、输入框里的长文草稿、当前翻到的页码：rememberSaveable
  * 会将状态保存在Bundle中，生命周期长于VM，配置变化和当进程因为内存不够被系统杀死(杀后台)存活。当用户主动退出后消失
  * 需要类型可被序列化。保存非基本类型：给data class加@Parcelize且继承Parcelable。还要加 kotlin-parcelize 插件。第三方类型：定义Saver
  * Bundle有1MB总容量限制
  * Nav会自动打包参数进Bundle
* 业务数据，从网络获取的，如用户信息；Screen之外的状态：ViewModel（配置变化存活）、SavedStateHandle（VM中使用，就是自动同步到Bundle的Map）、数据库（DataStore）
* retain（BOM 25.12+）：配置变化时存活。原理与VM一样
* State Holder：先创建一个普通data class MyState放各种状态（如ScaffoldState）及更新方法，构造函数无默认参数。再创建一个rememberMyState，参数具有默认值，如rememberScaffoldState()，再用remember(各参数){ MyState(各参数) }。如果一次要更新多个属性，用copy再赋值，可避免多次重组。一般整合多个UI层的状态，非UI层的放VM

### ViewModel

* 存活原理：挂在在ComponentActivity或Fragment的ViewModelStore上
* 2.11：允许给子Composable创建VM

```kotlin
class CounterViewModel(private val savedStateHandle: SavedStateHandle) : ViewModel() {
    // 简单的非持久属性，见kotlin笔记
    private val _state = MutableStateFlow(0);
    val state = _state.asStateFlow();

    // 存入Bundle的属性
    private val KEY_COUNT = "count_state"
    val countState = savedStateHandle.getStateFlow(KEY_COUNT, 0)
    fun increment() { // 直接设置Bundle，会触发更新
        val currentState = savedStateHandle.get<Int>(KEY_COUNT) ?: 0
        savedStateHandle[KEY_COUNT] = currentState + 1
    }

    // 与Compose耦合的简单属性：
    var cnt by mutableStateOf(0)
        private set
    fun incr() = cnt++

    override fun onCleared() {
        super.onCleared()
        // 清理协程等资源
    }

    fun loadData() {
        viewModelScope.launch { // 调用挂起函数，调度在主线程。应创建协程，更新内部状态，而不是公开挂起函数
            val result = repository.readFile("config.txt") // 里面切IO线程，VM无感知
            fileContent = result
        }
    }
}

使用：
@Composable
fun CounterScreen(modifier: Modifier, vm: CounterViewModel = viewModel()) { // 一般只在Screen上用，内部用状态提升
    val count by vm.countState.collectAsStateWithLifecycle(initial) // flow转UI状态。最小化时不收集，一般都用这个。只有短期局部的才用collectAsState，或非安卓；相当于LaunchedEffect+collect
}

作为Activity的成员变量的创建方法：val vm by viewModels<MyVM>()，它就是去ViewModelStore里找，而不是每次都创建

如果VM有构造函数参数：viewModel(factory = viewModelFactory { initializer { MyVM(args) } })。其中如果只有SavedStateHandle会自动注入（手动获取用createSavedStateHandle()），而Repo则不行。在作用域里创建完后，viewModel()可以获取到已有的

全局生命周期的VM，或需要用到context：MyViewModel(application: Application) : AndroidViewModel(application) { val ctx = getApplication<Application>() }
不要在里面持有Activity的ctx，否则会内存泄漏

库：
lifecycle-viewmodel-compose 提供 viewModel()
lifecycle-runtime-compose 提供 collectAsStateWithLifecycle
lifecycle-viewmodel-ktx 提供 viewModelScope

Repository：
针对一次性调用公开挂起函数：suspend fun readFile(name: String): String = withContext(Dispatchers.IO) { ... }
    推荐不硬编码Dispatchers，而是放进默认参数，方便测试
返回Flow的不用加挂起：fun getXxx(): Flow<Xxx>
页面关了（用户返回手势），任务也要在后台偷偷跑完，如 发消息、存数据库：类构造函数加externalScope: CoroutineScope，挂起函数用externalScope.launch { ... }.join()
```

### 处理副作用

* LaunchedEffect(key){ 协程环境 }
  * 进入组合或key改变时触发，会cancel老的；当key用Unit单例或true就只会触发一次
  * 用于网络请求、动画、定时任务（里面while(true)）
* rememberCoroutineScope()
  * 在Composable中 val scope = rememberCoroutineScope(); 在onClick等用户触发的事件中 scope.launch{ 调用挂起函数 }
  * onClick里是非Composable环境，无法使用LaunchedEffect；使用它后仍然是非Composable的，而是协程环境
  * 一般只用于修改UI状态。VM的用VM的scope
* SideEffect{}：每次成功重组后执行，不受Composable中断影响。用于把Compose状态同步给非Compose对象如Analytics、日志记录
* DisposableEffect(key) { 进入组合或key改变时执行，一般是注册监听器; onDispose { key改变或组件从UI树中移除时执行，一般是反注册监听器 } }
  * RetainedEffect { onRetire { 清理 } } 功能类似，但配置变化时它不清理。见retain
* rememberUpdatedState(param)
  * 解决 “开启了长耗时协程任务，过了一阵要用到某参数，但传入参数已经变了，不想（作为LE的key）每次变都重开” 的问题
  * 用它包装参数，之后在LaunchedEffect(Unit)里使用能拿到最新值，而不会因为参数改变而触发重启
  * 本质就是普通remember，不过会按参数更新
* produceState(initial) { 多次改变value } 返回State：提供一个协程环境，将非Compose的异步数据源封装为State；与SideEffect数据流向正好相反。相当于 remember + LE{改变它}，或类似于 flow{emit} + collectAsState。awaitDispose{处理非协程的资源清理}
* remember { derivedStateOf { 表达式，一般是bool，但必须用到State } }
  * 从多个状态计算一个派生状态，且当派生状态真正改变（同StateFlow）时通知。如监听列表滚动（每秒触发几十次），当滚动超过100像素时显示“回顶”按钮
  * remember(k1,k2)是输入驱动，而它是输出驱动；remember的key不变时不会进行计算，而它看计算结果是否不变；remember的key支持非State
* snapshotFlow：监控状态变化（不变时不发射），但不是修改UI，而是产生副作用。LE(pagerState) { snapshotFlow { pagerState.currentPage或表达式 }.可选debounce().collect { currentPage发生变化 } }

## 动画和手势

### 动画

* AnimatedVisibility(visible) { ... } 改变visible时为content添加 出现/消失 效果。会改变占位，代替if控制组件是否存在
  * 动画支持淡入fadeIn、滑动slideInVertically、放大scaleIn、展开expendVertically等。用+同时使用多种，生效顺序有优先级
  * MutableTransitionState：用来做入场动画、动画完成回调（isIdle）
* Modifier.animateContentSize()：加在容器上，用于 展开/折叠面板、输入框自动伸长。子项一直存在，但改变了大小，想让容器的大小动画改变
* AnimatedContent(state) { state->when(state)... }：用于 子项内容增减/替换，state就是一个Int，当它变化时动画，when里根据需求自己写content。如果因内容切换而改变的大小，不用animateContentSize也能动画，否则需要配合使用
* 属性动画：`animate*AsState`，其中星可以是Dp Int Color Size Offset等。先用remember创建一个变量，再由它计算animateDpAsState的目标值（如if (isBig) 200.dp else 100.dp），组件使用后者，修改前者
  * Animatable：上者的底层。允许初始值与目标值不同，提供更多操作。需协程环境
  * 只执行一次：创建一个变量played，对应目标属性值；为false时对应初始值。用LaunchedEffect把它设为true
* 多动画管理器：updateTransition，当切换状态要对多个属性动画时使用，也能管理AnimatedVisibility和Content
* rememberInfiniteTransition 循环动画
* Crossfade：布局切换时淡入淡出（仅透明度），大小跳变因此一般用于大小不变的。是AnimatedContent的简化版
* 其它底层：需对动画时间精细控制 - Animation。动画不是唯一可信来源：AnimationState / animate
* AnimationSpec 动画风格：spring弹簧、tween渐变、keyframes、repeatable（设置前两者迭代次数）、infiniteRepeatable、snap（延迟后立即切换）
  * Easing：对于基于时长的动画，调整加速度，如快进慢出
* 手势和动画同时执行：一般手势优先级更高，且要获取动画中断时的状态
* CircularProgressIndicator：圆圈加载动画，一般用Box叠加
* 动画矢量图资源：androidx.compose.animation:animatin-graphics，AnimatedImageVector.animatedVectorResource(id)、rememberAnimatedVectorPainter

### 手势

* 点击：Modifier.clickable。复合点击（长按、双击）：combinedClickable。一般尽量放到M的末尾
* 滚动：Modifier.verticalScroll(rememberScrollState()) 可加在Text上、具有单个很高的子元素的父元素上（如Box(m){Column{很多}}）
  * 完全自定义：scrollable() 仅检测手势
  * 嵌套滚动：优先滚动子元素（称为传播），当子元素无法滚动时才滚父元素。Compose的某些组件内置了此支持。手动添加用nestedScroll
  * Fling惯性滑动：用户迅速拖动某元素，抬手，元素继续滚动
* 拖动（进度条）：Modifier.offset{IntOffset}.draggable(rememberDraggableState { delta -> ... }) 本身只检测手势的拖动距离且只能水平/垂直
* 滑动Swipeable：用于吸附开关、下拉刷新（现在有封装了的pullrefresh）
* 多点触控：transformable

### Canvas

```
Canvas(onDraw= {
    drawPoints、Line、Circle、Rect、Oval（椭圆）、Arc（弧形，参数在椭圆的基础上增加角度）
})

混合模式BlendMode：当两个图形重叠时，如何显示。如区域包括重叠的和分别两个不重叠的，可以选择Xor、减去等。而不仅仅只是覆盖

点：pointMode 选择是单个实心、每两个连成线、所有点连一起。cap 首尾两端的样式如圆形/方形。pathEffect 如虚线

矩形：是否填充、圆角

Path：由直线、曲线构成的图形，类似于多边形

默认是实心的。设成空心后要再设笔触厚度。
```

在放置组件时顺便进行绘制，如小红点：

```
Card(..., Modifier.drawWithContent {
    drawContent()  // 绘制组件本身
    drawCircle(...)  // 相当于在组件上方绘制
})
在组件后方绘制：drawBehind，无需调用drawContent

如绘制时依赖某状态，而此状态又是动画会不断更新：drawWithCache{ val 加载资源; onDrawBehind{...} } 其实用remember也可以，前者作用域更小
```

## Nav3

* 回退栈 + 参数传递 + 转场动画
* 重点：可以创建多个（嵌套）
* 教程：https://github.com/android/nav3-recipes https://medium.com/@dayanand1531/navigation-3-jetpack-compose-android-a14beef1c239 https://www.youtube.com/playlist?list=PLQkwcJG4YTCRjyfVKB8vcK7zeC1BNmBz4
* Deep Link：用户从应用外直接跳转到应用内特定页面的连接。Nav3没有原生支持

```kotlin
库：
androidx.navigation3:navigation3-ui 传递依赖了runtime和kt的serialization库
androidx-navigation3-runtime
androidx.lifecycle:lifecycle-viewmodel-navigation3 版本与lifecycle一致，不用再加其它几个lifecycle库，可加lifecycle-runtime-ktx。让VM与Screen绑定，原理：NavEntry提供的LocalViewModelStoreOwner
插件：org.jetbrains.kotlin.plugin.serialization 用于配置改变时保持参数

val backstack = rememberNavBackStack(HomeRoute初始状态) // 可简单理解为list<T>，但保存到了Bundle
NavDisplay(backstack, entryProvider = entryProvider { // 原始方法：用when区分路由参数，创建NavEntry
    entry<Route.HomeRoute> {
        HomeScreen(onUserClick = {
            backstack.add(UserDetailRoute(123)) // 切换新页面
        })
    }
    entry<Route.UserDetailRoute> { entry -> // 转换为了强类型
        UserDetailScreen(entry.userId)
    }
})

返回按钮：backstack.pop()。也可以不做，用系统手势
完成登录后要清理栈，否则用户用系统手势返回又会回到登录页

定义页面，以下分别代表无参和有参：
sealed interface Route: NavKey { // NavKey区分同类页面
    @Serializable data object HomeRoute: Route
    @Serializable data class UserDetailRoute(val userId: Int): Route
}
```

### Tab

NavigationBar，底部的切换，用手势返回时不会回到上一个Tab。

自适应手机（下面Bar）平板（左边Rail）桌面（Drawer）：NavigationSuiteScaffold。androidx.compose.material3:material3-adaptive-navigation-suite，不知道是否包含adaptive-layout和adaptive-navigation3

```kotlin
@Composable
fun MainScreen() {
    val tabs = listOf(MainTabRoute.Chat, MainTabRoute.Contacts, MainTabRoute.Discover, MainTabRoute.Me)
    var selectedTab by rememberSaveable { mutableStateOf<MainTabRoute>(MainTabRoute.Chat) }

    Scaffold(
        bottomBar = {
            NavigationBar {
                tabs.forEach { tab ->
                    NavigationBarItem(
                        selected = selectedTab == tab,
                        onClick = { selectedTab = tab },
                        icon = { Icon(...) },
                        label = { Text(tab.toString()) }
                    )
                }
            }
        }
    ) { innerPadding ->
        Box(Modifier.padding(innerPadding)) {
            TabContent(selectedTab)
        }
    }
}

@Composable
fun TabContent(selectedTab: MainTabRoute) {
    Crossfade(selectedTab) { tab -> // Crossfade会保留内部组件的状态
        when (tab) {
            is MainTabRoute.Chat -> ChatNavStack() // 每个Tab内部可以有自己的NavWrapper，实现“详情页 -> 返回”
            ...
        }
    }
}
```

顶部大量平级分类（如新闻、体育、娱乐）：ScrollableTabRow。如果数量少且固定：PrimaryTabRow。

```kotlin
val pagerState = rememberPagerState(有几页)

Column{
PrimaryTabRow(selectedTabIndex = pagerState.currentPage) {
    tabs.forEachIndexed { index, title ->
        Tab(
            selected = pagerState.currentPage == index,
            onClick = {
                coroutineScope.launch {
                    pagerState.animateScrollToPage(index)
                }
            },
            text = { Text(title) }
        )
    }
}

HorizontalPager(pagerState) { page -> // 让用户在主区域左右滑动时能切换Tab，且懒加载，只保留左右两个；且与上方标签联动
    TabContent(page)
}
}
```

# 安卓系统

## Context

* Android四大应用组件：Activity、Service、BroadcastReceiver、ContentProvider，都间接继承了Context。一个它们的实例会附加在一个ctx上，具有对应相同的生命周期。都要在manifest中声明
* 在单例中持有Activity的Context会导致资源泄漏。使用ctx前一般要检测有效性
* 使用安卓系统功能时用到
  * 请求权限：ActivityCompat.requestPermissions要使用Activity的Context
  * 使用系统服务（此处ktx隐式处理了）：`getSystemService<NotificationManager>()`
  * 访问应用资源：context.getString(R.string.app_name)
  * 读取内部储存
* Application的Context与UI无关
* ContextCompat：兼容工具类

## Activity

* 生命周期：create -> start（有界面，对用户可见，但不可交互） -> resume（完全可见且活跃，相当于running） -> pause（前台被其他东西覆盖，unfocus；-> resume） -> stop（最小化；-> restart -> start） -> destroy
* 配置变化：旋转屏幕、切换黑暗模式、切换语言。会杀掉activity重建。在用compose时，可以在manifest里设置停用activity重建，改为读取某些State从而自动重组
* Fragment：轻量Activity，有自己的生命周期，但必须依附于Activity存在。一般在非Compose中根据导航切换多个Fragment，或平板上左右同时显示
* Compose一般单Activity。生命周期：Enter、重组、Leave（组件从UI树中移除），最小化时就自动销毁了。大部分原本在onStart做的，用LaunchedEffect(key)监听数据变化；如果真的需要监听Activity的onResume，用DisposableEffect + LocalLifecycleOwner + LifecycleEventObserver
* Jetpack LifeCycle库：传统app会在Activity的各个生命周期方法里初始化和销毁组件，容易出错。此库让需要处理生命周期的组件（LifecycleObserver）内部写逻辑，Activity（LifecycleOwner）只要“注册”一下

### Task

* 一个screen栈。screen可以是Activity和Dialog等，可以含有不同APP的。按多任务按键看到的就是。一个APP可以有多个Task
* 但已经结束的程序也可能出现在多任务列表里，只是一种预览
* 不同Task但相同的Task Affinity，多任务列表里只会显示出一个，其他的可能活着但没显示
  * Task具体的Affinity值会按第一个启动的Activity，之后再在里面启动Activity不会管Affinity

### Launch Mode

有两个APP，有一个A，用户在用B，点击链接跳到A，当A的Launch Mode分别是以下四种时，行为是什么。

* Standard 标准模式：在B的Task里新建A的Activity，对A的Task无影响。返回时回到B
* Single Top 栈顶复用：这种情形，行为与标准模式完全一致
* Single Task 栈内复用：先把A的Task切换到前台，如果A的栈内有目标Activity，则清除在其之上的其它Activity。返回时回到A的上一个页面，最后回桌面
* Single Instance：ActivityA运行在单独的Task里，且上下都没有其它Activity。返回时回到B。一般用于支付界面

对于前两者，B的Task里都会出现A的Activity。如果此时再创建A，且A在栈顶：标准默认会新建，而Single Top会复用。

只有标准模式才一定会创建，其它几种都可能复用。复用时不重新调用onCreate，而是调用onNewIntent(intent)。接收者override它，实现一般为调用super、setIntent(intent)、更新状态。

一般App内部用标准和SingleTop。SingleInstance用于外部的。SingleTask内外部都用得到。

Launch Mode是被调用者设置的。调用者有一些FLAG可以覆盖。

## Intent

一种IPC机制。启动另一个应用组件时用到。第一个参数是action: String，实际可以任意传参，要看双方约定。

发送显式intent：精确跳转到另一个Activity。

```kotlin
// 本App的Activity
val intent = Intent(applicationContext, SecondActivity::class.java).apply {
    action = ...
    putExtra(k,v) // 对应Bundle。其它创建方式：Bundle().apply{putString ...}; putExtras(b)
}
ctx.startActivity(intent)

// 其它包的Activity
val i = Intent(Intent.ACTION_MAIN).apply {
    package = "另一个程序的包名"
}
try { ctx.startActivity(i) } catch (e: ActivityNotFoundException)
```

发送隐式intent：如分享、编辑文件，会显示应用列表给用户选择，不预先确定要打开哪个Activity。

```kotlin
val i = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_EMAIL, arrayOf("xxx"))
}
ctx.startActivity(i)

打开网页：Intent.ACTION_VIEW, Uri.parse("https://网址")
打电话：Intent.ACTION_DIAL, Uri.parse("tel:10086")
强制弹出程序选择列表，忽略默认应用直接跳转：Intent.createChooser(原始intent, title)

检查是否有应用能处理某Intent：i.resolveActivity(ctx.packageManager) != null。还必须先在manifest里加queries，否则永远返回null；无论加不加都允许直接startActivity，目的是保护隐私，因此一般try-catch而不检查

处理返回结果：ctx.startActivityForResult(i, REQUEST_CODE)已废弃。见下方的Uri

注册作为隐式intent的接收者：manifest里的activity的intent-filter，里面写action、category（一般是DEFAULT，还有个BROWSABLE从浏览器启动）、data（MIME、URI）
```

被调用时拿传入的intent：Activity的onCreate里有intent对象（标准LaunchMode）。

## Uri、IO、存储

```kotlin
val uri = Uri.parse("scheme://authority/path?queryk=v")、Uri.Builder().xxx().build()
val xxxBytes = ctx.contentResolver.openInputStream(uri)?.use { it.readBytes() }

启动相册app，选择内容。那个app有权限访问外部存储，本应用无需特别权限：
val pickImage = rememberLauncherForActivityResult(
    ActivityResultContracts.GetContent()
) { uri: （content://授权式临时的） -> 读取另一个APP返回的文件，或一般只是给某个State赋值 }
btn(onClick = {pickImage.launch("image/*")})
Image(rememberImagePainter(uri)) // Coil库
拍照：TakePicturePreview

ContentProvider：即上面的相册app的角色。contentResolver就是其使用者。
把一个APP的数据提供给另一个，必须用它们。不能用File。


存储位置：
内部存储 ctx.filesDir 在 /data/user/0/包名/files 或逻辑路径 /data/data/包名
外部私有（用户可见） ctx.getExternalFilesDir(null) 在 /storage/emulated/0/Android/data/包名/files
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/> 新版不需要write权限
缓存 ctx.cacheDir、externalCacheDir
任意位置（UI提升用户选择授权，包括文件和目录）：Storage Access Framework (SAF)。默认仍然非永久；获得永久后每次使用前仍然要检查，因为可能失效/撤销
直接管理任意位置：提交到商店时会被审核是否真的需要，且用户有开关
传统：虽然内部是私有的，但外部可以任意访问。现在外部虽然对用户可见，但各APP只能操作自己那部分，且不需要特别的权限

读写文件。一般要用scope.launch(Dispatchers.IO)：
File(context.filesDir, "data.txt").readText() // kotlin扩展了Java的File
ctx.openFileInput(fileName)：不建议用，只能操作filesDir的根目录，不支持子目录


MediaStore API：读写图片视频公告目录时，不再操作文件，而是用它“订阅”或“提交”数据，类似于一个数据库。如果是本应用生成的，虽然与其它混在同一个目录里，却能自由操作
写入：
val imgCollection = MediaStore.Images.Media.getContentUri(MediaStore.VOLUME_EXTERNAL_PRIMARY)
val contents = ContentValues.apply { put(MediaStore.Images.Media.DISPALY_NAME, "xxx.jpg")、MIME_TYPE、WIDTH、HEIGHT }
val uri = contentResolver.insert(imgCollection, contents); contentResolver.openOutputStream(uri).use{os->...}
读取：
val projection = arrayOf(MediaStore.Images.Media._ID) 相当于SELECT。一个图像有很多种属性，根据需要从数据库里取一部分
selection：相当于WHERE/filter。不设置时获取所有内容。当想获取昨天拍的就设置
contentResolver.query(imgCollection, projection, selection, selectionArgs, sortOrder)?.use { cursor -> while(cursor.moveToNext()){根据列取数据} }


与res平行的assets目录：InputStreamReader(ctx.assets.open("file.txt"))
```

## 权限

```kotlin
无论是否需要动态申请，都必须先在manifest里声明：
<uses-permission android:name="android.permission.INTERNET" />

简单版：
val launcher = rememberLauncherForActivityResult(ActivityResultContracts.RequestPermission()) {}
LaunchedEffect(Unit) {
    if (PermissionChecker.checkSelfPermission(context, Manifest.permission.Xxx) != PermissionChecker.PERMISSION_GRANTED) {
        launcher.launch(Manifest.permission.Xxx)
    }
}

要用时临时申请：

val launcher = rememberLauncherForActivityResult(
    ActivityResultContracts.RequestPermission() 或 RequestMultiplePermissions()
) { isGranted -> 一次性申请多个权限时是Map<String权限名, Boolean>
    if (isGranted) { 权限已授予，可以执行扫码等逻辑 } else { 权限被拒绝，可能需要显示提示弹窗 }
}
Button(onClick = {
    when (PermissionChecker.PERMISSION_GRANTED) {
        PermissionChecker.checkSelfPermission(context, Manifest.permission.CAMERA) -> {
            // 已有权限，直接执行
        }
        else -> launcher.launch(Manifest.permission.CAMERA) 或 arrayof(permissions)
    }
})

老版：ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.POST_NOTIFICATIONS), 0)
```

## Broadcast

Intent广播内容，如ACTION_BATTERY_LOW。Sender通常是系统。Receiver过滤并处理特定Intent。

应用内通信：以前用Custom Broadcast，现在用 SharedFlow（EventBus也退役了）。系统状态监听：以前监听网络变化广播，现在推荐 ConnectivityManager.NetworkCallback。后台任务（也包括监听系统状态触发逻辑）：以前监听开机广播启动后台任务，现在推荐 WorkManager。\
仍然有用的：耳机拔插/蓝牙状态、电量状态。系统开机（唯一的自启动入口）。自定义通知交互，点击通知按钮发送一个BroadcastIntent。

显式广播：指定目标Receiver的包名和类名 intent.setComponent() 或 setClass()。隐式广播：只设置Action。

普通广播：异步，所有接收者几乎同时收到。有序广播：按优先级依次同步执行前面的接收者可修改Intent或abort截断。

静态注册：在manifest里写receiver，app不运行也可以收到。但限制多，隐式广播只有一小部分允许静态注册。

动态注册：

```kotlin
class MyReceiver: BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        if (intent?.action == Intent.ACTION_BATTERY_LOW) {
                ...
            }
        }
}
再在Activity里，val rcv = BroadcastReceiver(); ctx.registerReceiver(rcv, IntentFilter(Intent.ACTION_BATTERY_LOW))
再在onDestroy里ctx.unregisterReceiver(rcv)
```

## WorkManager 后台任务

* 满足某些条件时执行任务，且App退出、手机重启也能恢复。如 后台同步数据库、上传日志、压缩视频、定期更新天气
* 非即时性，秒级准确用AlarmManager。普通Worker有10分钟的时间限制
* 底层为JobScheduler
* androidx.work:work-runtime-ktx

```kotlin
class UploadWorker(appContext: Context, params: WorkerParameters):
    CoroutineWorker(appContext, params) {

    override suspend fun doWork(): Result {
        params.id
        params.inputData.getString("key", "default") // inputData类似于弱类型字典

        withContext(Dispatchers.IO) {
            val success = uploadLogs()
        }

        return if (success) Result.success(workDataOf(不超过10kb的数据)) else Result.retry()
    }
}

val constraints = Constraints.Builder() // 约束条件，满足后即使App退出了也会拉起进程；但某些ROM清理后台后，必须用户下次主动运行App才会检查
    .setRequiredNetworkType(NetworkType.UNMETERED) // 仅限 Wi-Fi
    .setRequiresCharging(true) // 仅限充电
    .build()

val uploadRequest = OneTimeWorkRequestBuilder<UploadWorker>()
    .setConstraints(constraints)
    .setBackoffCriteria(...) // 失败后默认指数级退避重试
    .setInputData(workDataOf(k to v))
    .build()

WorkManager.getInstance(context).enqueue(uploadRequest) // 向系统注册Job。还支持设置一系列步骤，如果中间失败，后续会自动取消
UI获得执行进度：workManager.getWorkInfoByIdFlow(requestId).collect { info ->?.state } + LaunchedEffect(result?.outputData)
```

## Foreground Service 和 通知

* 通知：构建渠道（Channel）、配置内容（Builder）、执行发送（Notify）
* 前台服务：创建一个不可移除的通知。用于 音乐播放、导航、正在进行的录音
* Svc运行时，整个程序都在运行。Svc一般只放与通知交互的逻辑，业务逻辑放在别的类里

```kotlin
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_DATA_SYNC" />
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
类型：https://developer.android.com/develop/background-work/services/fgs/service-types?hl=zh-cn

<application android:name=".RunningApp">
    <service
        android:name=".MyForegroundService"
        android:foregroundServiceType="dataSync" // 多种用|
        android:exported="false" />
</application>


class MyForegroundService: Service {
    val channelId = "my_service_channel" // 一组通知的类别
    val notificationId = 100 // 与某具体通知关联，调用时传同一个id就会更新内容。至少为1

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val notification = NotificationCompat.Builder(this, channelId)
            .setContentTitle("正在同步数据")
            .setContentText("请勿关闭应用...")
            .setSmallIcon(R.drawable.ic_sync) // 必须是 24x24dp 的纯白色图形（带透明度）
            .setContentIntent( // 点击行为
                PendingIntent.getActivity(this, 0,
                    Intent(this, MainActivity::class.java).apply { // 点击后去哪
                            flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_SINGLE_TOP // 前者必用，后者还有CLEAR_TOP
                            // 传参给MainActivity
                        }
                    , PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE) // 后者当用户不用输入时必用
            )
            .addAction(R.drawable.ic_stop, "停止", stopPendingIntent) // 添加按钮
            .setAutoCancel(false) // 如果为true，点击通知后自动取消
            .setOngoing(true) // 不让用户侧滑移除
            .build()
            其它功能：
            .setProgress(最大值，当前值，是否不确定模式)
            .setStyle(NotificationCompat.BigTextStyle().bigText("这里是很长很长的内容..."))

        ServiceCompat.startForeground(
            this,
            notificationId,
            notification,
            ServiceInfo.FOREGROUND_SERVICE_TYPE_DATA_SYNC
        )

        // 执行后台耗时逻辑...

        return START_STICKY // 如果服务被杀，尽量重启它
    }

    // 如果需要在UI树中使用，则定义以下内容，否则onBind=null即可
    inner class LocalBinder : Binder() {
        fun getService(): MyForegroundService = this@MyForegroundService // 也可以只定义细粒度的提取状态的方法，而不直接暴露整个Service
    }
    private val binder = LocalBinder()
    override fun onBind(intent: Intent?): IBinder? = binder
}

// 创建Channel。重复创建相同的chid是no-op
class RunningApp: Application() {
    override fun onCreate() {
        super.onCreate()
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(chid, name用户友好文本, NotificationManager.IMPORTANCE_DEFAULT有声音/HIGH弹出横幅/LOW无声但显示/MIN无声折叠)
            val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
            notificationManager.createNotificationChannel(channel)
        }
    }
}

请求权限：POST_NOTIFICATION
启动：val intent = Intent(ctx, MyForegroundService::class.java); ctx.startForegroundService(intent)
    这个intent实际上只是启动那个Serivce组件调用onStartCommand，里面要用ServiceCompat.startForeground从“后台”提升为前台
停止：外部 ctx.stopService(intent)，或 内部stopSelf()

UI中启动并交互状态：
var boundService by remember { mutableStateOf<MyForegroundService?>(null) }
val progress by boundService?.progress?.collectAsState(initial = 0) ?: remember { mutableStateOf(0) }
val connection = remember {
    object : ServiceConnection { // 也可放在VM里
        override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
            val binder = service as MyForegroundService.LocalBinder
            boundService = binder.getService()
        }

        override fun onServiceDisconnected(name: ComponentName?) {
            boundService = null
        }
    }
}

DisposableEffect(ctx) {
    val intent = Intent(ctx, MyForegroundService::class.java)
    ctx.bindService(intent, connection, Context.BIND_AUTO_CREATE)

    onDispose {
        ctx.unbindService(connection) // 自动销毁Svc
        boundService = null
    }
}

Text(text = "服务运行进度: $progress")
```

发送普通通知：

```kotlin
with(NotificationManagerCompat.from(context)) {
    if (ActivityCompat.checkSelfPermission(context, Manifest.permission.POST_NOTIFICATIONS) == PackageManager.PERMISSION_GRANTED) {
        notify(notificationId, builder.build()) // 同一个notificationId会更新内容，不同notificationId会堆叠
    }
}
```

应用内显示在屏幕下方气泡：Toast.makeText(ctx, 信息, Toast.LENGTH_SHORT).show()

## DataStore

* 键值对存储，替代 SharedPreferences
* 默认用Dispatchers.IO

```kotlin
androidx.datastore:datastore-preferences // 还有基于Protobuf的，要写Schema、Serializer

// 定义在文件顶部，不在类内部。会自动绑定到Application上
val Context.dataStore: DataStore<Preferences> by preferencesDataStore("user_settings")
使用：context.dataStore，读data，写edit

object PreferenceKeys {
    val USER_NAME = stringPreferencesKey("user_name")
    val LOGIN_COUNT = intPreferencesKey("login_count")
    val IS_NIGHT_MODE = booleanPreferencesKey("is_night_mode")
}

class UserRepository(private val dataStore: DataStore<Preferences>) {
    val userName: Flow<String> = dataStore.data.map { // it是一个preference/setting Map，每当修改就会产生一整个全新的
            it[PreferenceKeys.USER_NAME] ?: "Guest"
        }
        .stateIn( // 转换为StateFlow，具有内存缓存不会重复读取，一般在VM类里运行stateIn
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000), // 如果没有UI订阅了，5秒后切断与底层磁盘流的联系
            initialValue = "Loading..."
        )

    suspend fun saveName(name: String) { // 要在scope里运行
        dataStore.edit { // 具有原子性
            it[PreferenceKeys.USER_NAME] = newName
        }
    }
}
```

## Hilt

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DataModule {
    @Provides
    @Singleton
    fun provideDataStore(@ApplicationContext context: Context): DataStore<Preferences> {
        return context.dataStore
    }
}

class UserRepository @Inject constructor(val dataStore: DataStore<Preferences>)

@HiltViewModel class UserViewModel @Inject constructor(val repository: UserRepository)

@HiltAndroidApp class MyApplication

@AndroidEntryPoint class MyActivity


---
[plugins]
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
[libraries]
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
androidx-hilt-navigation-compose = { module = "androidx.hilt:hilt-navigation-compose", version.ref = "hiltNavigationCompose" }

implementation(libs.hilt.android)
ksp(libs.hilt.compiler)
implementation(libs.androidx.hilt.navigation.compose)
implementation(androidx.hilt:hilt-lifecycle-viewmodel)
```

## 性能监视

```
application onCreate：
if (BuildConfig.DEBUG) {
    StrictMode.setThreadPolicy(StrictMode.ThreadPolicy.Builder().detectAll().penaltyLog().build())
    StrictMode.setVmPolicy(StrictMode.VmPolicy.Builder().detectAll().penaltyLog().build())
}
```

## 其它官方组件

* Splash Screen API
* https://google.github.io/accompanist/ 官方未转正但有用的组件
* 布局适应窗口可用空间：Jetpack WindowManager。因为未来要支持多窗口模式，不能根据物理屏幕
* Initializer（androidx.startup:startup-runtime）：代替Application.onCreate，把逻辑拆分成多个类。注册为provider但大部分模板不用改动

## 其它库

* square/leakcanary 检测内存泄漏

## 相关开发软件

* LibChecker
* 快捷方式 github.com/sdex/ActivityManager
* https://github.com/skylot/jadx 反编译apk

# 参考

* https://space.bilibili.com/27559447/
* https://www.youtube.com/@PhilippLackner
* 《Jetpack Compose 从入门到实战》

## 待看

* https://docs.gradle.org/current/userguide/best_practices.html
* https://jetpackcompose.cn/docs/
* https://juejin.cn/user/2384195547303688/columns

## 其他项目示例

* https://github.com/FunnySaltyFish/Transtation-KMP
* https://github.com/samolego/Canta
* https://github.com/you-apps
