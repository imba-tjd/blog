# 设计模式

* 创建者模式
  * 工厂方法
  * 抽象工厂
  * 建造者
  * 原型
  * 单例
* 结构型模式
  * 适配器
  * 桥接
  * 组合
  * 装饰
  * 外观
  * 享元
  * 代理
* 行为型模式
  * 职责链
  * 命令
  * 解释器
  * 迭代器
  * 中介者
  * 备忘录
  * 观察者
  * 状态
  * 策略
  * 模板方法
  * 访问者
* 通用缺点
  * 变复杂，难理解，难设计，出错难排查
  * 某一类职责过重
  * 具有局限性
  * 需要经验积累

## 面向对象设计原则

* 单一职责 Single Respoinsibility（提高内聚）：一个对象只应包含单一的职责，且该职责被完整地封装在一个类中
* 开闭 Open-Closed（方便变更）：对扩展开放，对修改关闭
* 里氏代换 Liskov Substitution（降低继承耦合）：用基类的地方要能用子类。如果子类抛了基类未定义的异常，或者改变了方法的语义，则违反了此原则
* 依赖倒转 Dependence Inversion（方便变更）：高层模块不应依赖低层模块，而是应依赖抽象；抽象不应依赖细节，细节应依赖抽象
* 接口隔离 Interface Segregation：使用者不应依赖不需要的接口
* 合成复用 Composite Reuse（降低继承耦合）：优先使用组合而非继承，来达到复用的目的
* 迪米特法则 Law of Demeter（降低访问耦合）：每一个软件单元对其它单元只有最少的知识，且局限于与本单元密切相关的。考试中如出现o.f1().f2()则违反了此原则
* 面向接口编程、接口分离/接口最小化（一个接口只做一件事）：降低访问耦合

## 模块化

* 设计质量的考量：可理解、易修改、易复用
* 耦合（从高到低）描述的是两个模块之间的复杂程度
  * 内容耦合：goto、不通过setter/getter访问字段、某些语言支持更改另一个模块的代码、遍历集合时未使用迭代器
  * 公共耦合：全局变量
  * 重复耦合：逻辑代码被复制到两个地方
  * 控制耦合：相当于调用函数时传递结构体。另一种说法是if或switch中调用函数
  * 印记耦合：相当于调用函数时传递结构体中的字段
  * 数据耦合：相当于传递裸参数，不使用类的属性
* 内聚（从低到高）表达的是一个模块内部的联系的紧密性
  * 偶然内聚：模块执行多个不相关操作
  * 逻辑内聚：相当于应该用策略模式但没有用的情形
  * 时间内聚：模块执行一系列与时间有关的操作。题目中如将初始化两个对象放在一个函数中
  * 过程内聚：模块执行一系列与步骤顺序有关的操作
  * 通信内聚：相当于嵌套调用函数依次传递参数，与逻辑内聚的区别是逻辑内聚的函数地位是相同的
  * 功能内聚：模块只执行一个操作，相当于单个纯函数
  * 信息内聚：相当于面向对象
* 信息和隐藏：利用抽象的方法，提炼本质（接口），消除非本质的细节（实现）。使得模块尽可能独立、易于修改、高内聚低耦合

## 简单工厂 SimpleFactory

* 不属于GoF的23种设计模式之一
* 工厂类或抽象产品类里声明一个静态方法，接受字符串参数，返回new的对象
* 缺点：添加新产品要修改工厂逻辑

## 工厂方法 FactoryMethod

* 每一种具体产品都对应建立一个工厂，抽象工厂里只有创建一种抽象产品的方法（但可以重载）
* 可通过配置XML，用反射生成工厂，用户再调用工厂。这样添加新产品时无需修改抽象工厂和其它原有工厂的内容，如无需修改代码更换数据库驱动，体现了开闭原则
* 还可以直接把产品的方法放到工厂里，工厂创建对象的过程对用户隐藏

## 抽象工厂 AbstractFactory

* 用于生产一组对象。如皮肤库有春天和秋天两种风格，每种都有按钮、对话框等控件
* “开闭原则的倾斜性”：易于增加产品族（如夏天风格），但难以增加等级结构（如新增一个控件，需要修改原来所有的工厂甚至包括抽象工厂）

## 建造者 Builder

* 用于创建复杂对象，包括对象的某些部分要按顺序创建。且不同产品之间具有较多共同点
* 抽象建造者对产品的每个部分都有buildXXX()，具体建造者实现它们
* 把具体建造者实例传给Director，按顺序调用buildXXX()。使用者最后调用Director的construct()
* “钩子方法”：抽象建造者设定一些指示某一部分是否需要创建的属性，Director在调用某个buildXXX()前先判断一下需不需要创建。另一种方式是抽象类添加空实现

## 原型 Prototype

* 原型抽象类：声明了clone()的类
* 原型管理器：储存多个原型对象
* 用于简化创建对象；深拷贝用于保存状态
* 缺点：原不支持clone的类需要修改代码、深拷贝实现复杂

## 单例 Singleton

* 具有私有构造函数、一个自身类型的静态私有变量、一个工厂方法
* “懒汉式”：getInstance()才创建对象

## 适配器 Adapter

* 用于将一个类的接口与另一个匹配起来
* 类适配器：继承Adaptee适配者，实现Target目标抽象类的接口，里面调用super().xxx
* 对象适配器：Adaptee作为成员变量，Adapter转发调用
* 缺省适配器/单接口适配器：用一个抽象类实现接口的所有方法，但是是空方法实现。用于具体类不需要实现所有接口
* 双向适配器：相当于把Adapter和Target的两个对象适配器合在一起，构造函数重载，一次只能用一种方向的

## 桥接 Bridge

* 用于分离一个类的两个独立变化的维度，两个维度可以独立继承，避免多层继承。其实就是最普通的依赖注入
* 如一个毛笔抽象类，有大中小三个子类表示型号，有一个“颜色”接口有多种实现，毛笔类的绘图函数调用接口里的着色函数。增加颜色不需要改变毛笔类

## 组合 Composite

* 又称为部分-整体(Part-Whole)，树形结构，用于忽略整体和部分的差异
* 抽象构件Component：既可表示叶子Leaf又可表示容器Composite，容器的子结点可以是叶子也可以是容器（就是抽象类集合）。使用者可以对抽象类统一处理
* 透明组合：抽象类声明叶子和容器的所有方法(取合集)，叶子也要实现那些容器的方法，不过是直接抛异常
* 安全组合：抽象类只声明叶子类的方法(取交集)
* 缺点：增加新构件时难以对容器中的构件类型进行限制

## 装饰 Decorator

* 如Java的IO流
* 给一个对象额外增加一些功能，又不使用继承
* 已有抽象构件类和多个具体构件，新增一个抽象装饰类继承抽象构件，具有抽象构件成员，实现抽象函数时就转发调用它的，具体装饰类new时传入构件，再重写抽象函数，调用super()的再添加自己的功能
* 透明装饰：使用者完全针对抽象编程，创建具体对象后要赋给抽象引用。使用者在装饰前后对对象的使用完全一致，且可以多次装饰
* 半透明装饰：原构建还是用抽象引用，装饰后用具体的

## 外观 Facade

* 又称为门面。是迪米特法则的体现
* 一个子系统的外部和内部的通信通过一个统一的外观类完成，减少使用者和子系统的耦合。类似于API Gateway
* 抽象外观：普通的如果更改子系统实现就需要改外观类，就会影响使用者；使用者用抽象外观类就不会影响。但添加子系统还是会影响

## 享元 Flyweight

* 用共享技术细粒度重用相似对象
* 储存这些对象的地方称为享元池。享元工厂类返回或创建它们，工厂一般为单例
* 能共享的关键是区分了内部状态和外部状态，内部状态作为享元对象的成员，外部状态是调用函数时传入的参数
* 单纯享元：所有具体享元类都是可复用的。复合享元：与具体单纯抽象类继承同一个抽象类，一个成员持有对抽象类的引用，其它成员表示外部状态，不可共享

## 代理 Proxy

* 当无法直接访问某个对象时，通过代理对象间接访问；真实对象和代理对象要实现相同的接口，代理对象持有真实对象的引用，转发调用
* 远程：又称为大使，真实对象位于不同地址空间。如RPC、RMI
* 虚拟：真实对象资源消耗太大时，先创建一个小的表示，真正要用到时才创建大的
* 保护：给不同用户提供不同使用权限
* 缓冲：为一个操作结果提供临时存储空间
* 智能引用：对象被引用时提供额外功能，如记录被调用次数
* 静态代理对于每一个真实类都要编写一个代理类
* 动态代理：在运行时根据需要创建，能让一个代理类代理多个真实类。从接口引用中获得Clazz，再获得接口中的方法，就能创造代理类。多用于AOP，可做到任意对象调用任意函数前后记录日志

## 职责链 Chain of Respoinsibility

* 将请求发送者和处理者(接收者)解耦，请求沿链传递直到有对象能处理为止，发送者并不知道是哪个对象处理了，最常见的是直线型
* 抽象Handler类具有本类型的抽象引用。所有具体处理者都继承它，一个具体处理者处理不了就转发给下家
* 链的创建过程由使用者负责，新增具体处理类对原有的无影响
* 纯的：要么完全处理，要么直接传给下家。不纯的：允许部分处理
* 缺点：不能保证一定会被处理，也可能没有正确配置，如循环引用

## 命令 Command

* 又称为动作或事务。将请求进行封装，还是使发送者和接收者解耦
* Invoker指控件，使用者分别创建控件和具体命令对象，再把命令传进控件，使用时控件调用抽象命令的execute()。其实就是传一个回调函数，但是Java没有函数指针，要用类承载，如Thread的Runnable
* 命令队列：抽象类含有命令集合，能添加和删除，execute()时依次调用
* 记录请求日志
* 实现撤销操作：抽象类增加一个undo()
* 宏命令：类似于组合

## 解释器 Interpreter

* 文法：expression ::= val | op; op := exp + exp  | exp - exp; val ::= an integer。字符串字面量用单引号
* 抽象表达式类：包括终结表达式和非终结表达式，只有一个interpret(context)，Context相当于全局字典
* 实现时把每一个角色创建一个Node类，构造函数获得本部分的字符串，interpret()时根据AST按树的形式递归向下创建。还需要一个InstructionHandler根据规则解析token创建Node

## 迭代器 Iterator

* 将迭代功能从聚合对象中分离

## 中介者 Mediator

* 又称为调停者。类似于QQ用户和QQ群之间的关系，将一些多对多的交互行为从各个对象(称为同事类)中分离开，封装进一个对象
* 中介者类含有同事类集合，能添加/注册同事，一些需要调用某些同事类的业务方法
* 同事类也可以含有对中介者的引用，调用想要的业务方法

## 备忘录 Memento

* 原发器类Originator有一些状态需要保存以便之后恢复，就创建一个备忘录类，选一些字段存着。又再要一个负责人类Caretaker存备忘录类的引用，可以存一个或多个，不应修改备忘录类的内容
* 唯一的意义就是设计上会尽量少改变原发器类的代码，只要新增createMemento(){return new Memento(this)}，以及restoreMemento(Memento)从备忘录中恢复，但后者的代码还是要原发器类写

## 观察者 Observer

* 又称为 发布-订阅 模型-视图 源-监听器 从属者
* 一种一对多的关系，目标类状态发生改变时通知所有观察者。可以实现表示层和数据逻辑层的分离
* 在观察目标里定义观察者集合，可以添加和删除，定义notify()在需要通知时循环调用观察者类的抽象函数update()
* 缺点：如果观察者和目标之间存在循环依赖，可能触发循环调用导致崩溃

## 状态 Status

* 使对象在其内部状态改变时改变其行为
* 环境类实际上是真正拥有状态的对象
* 状态之间的转换，可以由环境类负责，此时环境类充当状态管理器角色；也可以由具体状态类负责，此时状态类和环境类之间会存在依赖关系
* 共享状态：如果需要多个环境类共享同一个状态，定义为static的

## 策略 Strategy

* 就是为了解决Java没有函数指针，要把不同的算法实现写在不同的类里
* 使用算法的类称为环境类

## 模板方法 Template Method

* 父类定义算法的一些基本操作，再定义一个模板方法根据需要按顺序调用这些步骤，还可以定义钩子方法；子类允许重写基本操作，但不允许重写模板方法

## 访问者 Visitor

* 也称为双重分派。用于对一个集合对象结构采用不同的处理方式
* 对象结构是元素的集合，每种元素有自己的操作。访问者抽象类对每一种元素都要定义处理该元素的方法，元素接受具体访问者，只调用访问自己的那一个方法
* 增加新的访问者无需修改原有代码，但增加新的元素类很麻烦

## UML

* 类：方框。里面依次是 类名 属性 方法
  * -私有成员 +公有成员 #protected
  * 抽象类：类名用斜体
* 成员：属性名:类型、函数名():返回值
* 接口：在类名左上角画一个横向钥匙一样的东西，也可以写`<<interface>>`。还有一种圆形表示法，只写接口本身名称
* 关联：实线线箭头，起始端是使用者，将箭头端作为私有成员。如果没有箭头就是双向关联，指向自己是自关联；如果是数组，在使用者那边写1，在箭头端写`* 0..1 1..*`等
* 聚合：实线线箭头或无箭头，起始端是空心菱形，在构造函数参数里传入箭头端。表示箭头端能脱离起始端存在，如发动机和汽车
* 组合：实线线箭头或无箭头，起始端是实心菱形，在构造函数里创建箭头端。表示箭头端不能脱离起始端存在，如嘴和头，翅膀和鸟
* 依赖(参数)：虚线线箭头，箭头端对象是起始端的普通函数参数
* 泛化(继承)：实线空心箭头，箭头端是父类
* 实现(接口)：虚线空心箭头

### PlainUML

```js
@startuml
skinparam classAttributeIconSize 0 # 显示+-#而不是颜色

interface Serviceable {
   +needs_service(): bool
}

abstract class Component {
    {abstract}+needs_service(): bool
}
Serviceable <|.. Component # 实现接口，其中<|看起来像空心箭头，..是虚线

class Engine {
    +needs_service(): bool # 已实现的方法，子类不必再写
}
Component <|-- Engine # 继承

Calliope o-- CapuletEngine # 聚合
Calliope o-- SpindlerBattery

@enduml
```

## Compatibility

* backward compatibility：如程序新版本能打开老格式的文件
* forward compatibility：如程序能跳过未知格式的文件、老版本程序面对新设计不出错

## 参考

* 《Java设计模式》刘伟

### TODO

* https://blog.csdn.net/zhengzhb/category_9260995.html
* https://www.liaoxuefeng.com/wiki/1252599548343744/1264742167474528