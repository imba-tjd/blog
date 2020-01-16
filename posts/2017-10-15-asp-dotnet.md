---
title: ASP.NET笔记
---

> 浙江省高等学校精品在线开放课程共享平台——Web应用程序设计（.NET）

ASP.NET网页生命周期和事件
-------------------------

* 开始：Page_Preinit
* 页初始化：Page_Init
* 加载：Page_Load
* 验证：验证控件事件
* 回发事件处理：控件事件
* 呈现：Page_PreRender
* 卸载：Page_Unload

Page类
------

* IsPostBack属性：是否是回发请求
* IsValid属性：页验证是否成功

Response对象
------------

* 是HttpResponse类的实例
* Redirect方法

Request对象
-----------

* 是HttpRequest类的实例
* Request.Url：当前请求的URL
* Request.UserHostAddress：远程客户端的IP主机地址
* Request.PhysicalApplicationPath：当前正在执行的服务器应用程序的根目录的物理路径
* Request.CurrentExecutionFilePath：当前请求的虚拟路径
* Request.PhysicalPath：当前请求的URL的物理路径
* Request.Form["TxtName"]：获取表单中名为TxtName控件的值

### Brower对象

* Request.Browser.Type
* Request.Browser.Browser
* Request.Browser.Version
* Request.Browser.Platform

### ServerVariables属性

* 是System.Collections.Specialized.NameValueCollection集合，实现了索引器
* Request.ServerVariables["URL"]：当前网页的虚拟路径
* Request.ServerVariables["PATH_TRANSLATED"]：实际路径
* Request.ServerVariables["SERVER_NAME"]：服务器名或IP
* Request.ServerVariables["SERVER_SOFTERWARE"]：软件
* Request.ServerVariables["SERVER_PORT"]：服务器连接端口
* Request.ServerVariables["SERVER_PROTOCOL"]：HTTP版本
* Request.ServerVariables["SERVER_HOST"]：客户主机名
* Request.ServerVariables["HTTP_USER_AGENT"]：浏览器

Server对象
----------

* 是HttpServerUtility类的实例
* MapPath方法：将Web服务器上的虚拟路径转换为实际路径
* Execute方法：调用其他网页，会把执行结果嵌入当前位置中
* Transfer方法：转到其他网页（但URL不变），原网页之后的代码将不再执行

<!-- -->

    string strpath = Server.MapPath(Request.ApplicationPath); // 网站根目录
    // 字符串里的波浪线也可以表示根目录？

异常处理机制
------------

### Page_Error事件

    protected void Page_Error(object sender, EventArgs e) // try-catch有未处理的异常时触发
    {
        Exception ex = Server.GetLastError();
        Response.Write(ex.Message);
        Server.ClearError();
    }

### Page.ErrorPage属性

* 可以让页面发生错误时重定向到友好的错误描述页面
* 要让ErrorPage属性发挥作用，web.config文件中`<customErrors>`配置项的mode属性必须为On

### Application_Error事件

* 位于Global.asax文件中
* 用法和Page_Error事件差不多

### web.config

* mode = RemoteOnly：对远程用户显示自定义错误页面，对本地（服务器）用户显示详细（默认）错误页面。适用于本地测试
* mode = Off：对所有用户显示详细错误页面。适用于多人测试
* mode = On：对所有用户显示自定义错误页面。适用于部署后

<!-- -->

    <customErrors mode="On" defaultRedirect=“Error.html">
        <error statusCode="403" redirect="NoAccess.html" >
        <error statusCode="404" redirect="FileNoFound.html" >
    </customErrors>

HTML服务器控件
--------------

* 位于System.Web.UI.HtmlControls
* runat="server"

### 公共属性

* InnerHtml：从对象的起始位置到终止位置的全部内容，包括Html标签
* InnerText：从对象的起始位置到终止位置的内容，不包括Html标签

<!-- -->

    例如：<span id="text"><span style="color:red;">test1</span>test2</span>

    InnerHtml：<span style="color:red;">test1</span>test2
    InnerText：test1 test2

 

* Value：获取控件的值。如input type="text"里的值和select标签的option中间的值
* Attributes：所有属性的集合。如Submit1.Attributes["Value"] ...
* Disabled
* Visible

#### 其他属性

* SelectedIndex：选中的那一项

### 事件

* 对象.Attributes.Add(键, 值);
* 客户端事件：onclick属性 = "return Submit1_onclick()"
* 服务端事件：onserverclick属性 = "Submit1_onserverclicklick"
* OnClick、OnClientClick、OnServerClick之间的关系：[http://blog.csdn.net/candyzha/article/details/6713413](http://blog.csdn.net/candyzha/article/details/6713413)

#### 常见事件

* onclick
* onchange
* ondbclick：双击
* onfocus：控件必须能获得焦点才会触发
* onkeydown
* onkeypress
* onkeyup
* onmousedown
* onmouseup
* onmousemove
* onmouseover：滑过？
* onmouseout：移出

Web控件
-------

* 动态添加控件：div1.Control.Add(new Button)

### 常用属性

* AccessKey：如果设为A，快捷键即为ctrl + A
* TabIndex
* Attributes：控件属性集合，只能在编程时指定
* BackColor、ForeColor
* Enabled
* Font
* Height、Width
* ToolTip：悬浮时显示的文本
* Visible

### 事件

* Click
* TextChanged
* CheckedChanged
* SelectedIndexChanged

### TextBox

* TextMode属性：SingleLine、MultiLine、Password、Color、Date
* TextChanged事件：内容改变且焦点离开且AutoPostBack为true时会自动回传并触发。只有回传时才会触发。

### HyperLink

* ImageURL
* NavigateURL
* Target

### Image

* AlternateText
* ImageUrl
* ToolTip
* 不响应用户事件，但是它可以根据其他控件的输入动态地显示图片

### Button

* CommandArgument：用于指定传给Command事件的参数
* CommandName：指定传给Command事件的名称
* OnClientClick
* PostBackUrl：单击按钮时发送的URL
* UseSubmitBehavior：指示按钮是否呈现为提交按钮
* Focus()

### DropDownList（ListItem）

* WinForm叫ComboBox
* AutoPostBack：默认为false
* Items
* SelectedIndex
* SelectedItem
* SelectedIndexChange：AutoPostBack为true时会立即触发

### ListBox

* Rows
* SelectionMode

### CheckBoxList、RadioButtonList（ListItem）

* RepeatDirection：表示横向（horizontal）还是纵向（vertical）排列
* RepeatColumn：一行排几列
* TextAlign
* Selected
* CheckedChanged

* * * * *

* GroupName
* SelectedItem

### Panel

* BackImageUrl
* HorizontalAlign
* Visible
* Scroll

### MultiView(View)

* ActiveViewIndex：读取或设置当前显示的View控件的索引值。默认值为-1，表示没有被显示。
* SetActiveView：参数为View
* ActiveViewChanged

### Calendar

* SelectionMode：None-不能选择，只能显示、Day、DayWeek、DayWeekMonth
* SelectedDate、SelectedDates
* SelectionChanged

验证控件
--------

### WebForms UnobtrusiveValidationMode异常

#### 原因

> .net framework4.5开发中， Unobtrusive ValidationMode是一种隐式的验证方式，需要前端调用jquery来进行身份验证。且默认启用。
>
> 如果用到了\<asp:RequiredFieldValidator/\>这样的验证控件，就会出现上述问题。

#### 解决方法

> 添加jQuery或者禁用客户端验证。
>  https://code.jquery.com/
>  https://www.cnblogs.com/maxiaofeng/p/3149920.html

    script
     src="https://code.jquery.com/jquery-3.2.1.min.js"
     integrity="sha256-hwg4gsxgFZhOsEEamdOYGBf13FyQuiTwlAQgxVSNgt4="
     crossorigin="anonymous"></script>

    // 放于Global.asax的Application_Start事件中
    ScriptManager.ScriptResourceMapping.AddDefinition("jquery", new ScriptResourceDefinition
    {
        Path = "~/scripts/jquery-3.2.1.min.js",
        DebugPath = "~/scripts/jquery-3.2.1.js",
        CdnPath = "https://ajax.microsoft.com/ajax/jQuery/jquery-3.2.1.js",
        CdnDebugPath = "https://ajax.microsoft.com/ajax/jQuery/jquery-3.2.1.js"
    });

 

### 达到的效果

* 验证用户输入
* 减少错误处理的等待时间（客户端验证）
* 避免非法输入导致错误结果（手机号11位）或服务器崩溃（长度太长）
* 避免欺骗或恶意代码（sql注入；客户端验证办不到）
* 阻止窗体下一步执行，直到验证成功
* 客户端验证可以即时反馈、不能访问服务器资源、安全性较低
* 服务端验证要重复一遍客户端验证，因为客户端验证可以被禁用

### 公共属性

* Display：None、Static、Dynamic
* Text：验证失败时，在验证控件中显示的文字内容
* ErrorMessage：也是显示失败时的内容，但Text会覆盖它
* ControlToValidate：指示要验证的控件
* EnableClientScript：指示是否启用客户端验证
* SetFocusOnError：验证失败时把焦点改为指定的控件上
* ValidationGroup：此验证控件所属的验证组的名称
* IsValid：是否通过验证。Page的实例（this）可显示是否所有的验证都通过了

### RequiredFieldValidator

* 验证一个必填字段
* InitialValue：当设置为某个值（value）时，认为它无效（比如下拉列表第一项提示性的文字）

### CompareValidator

* 将用户输入和其他值进行比较，可使用大于小于等于等运算符
* 可以与给定的公式、数据库数据、COM、调用Web服务进行比较
* ControlToCompare：跟另一个控件进行比较（可以是另一个输入）
* ValueToCompare：跟常数比较
* Operator
* Type

### RangeValidator

* 检查用户的输入是否在指定的上下限内
* 可以检查数字对、字母对和日期对
* 空输入是有效验证
* MaximumValue、MinimumValue
* Type

### RegularExpressionValidator

* ValidationExpression：正则表达式，有预定义的，但不一定符合中国的

### CustomValidator

* 可以进行服务端验证
* ClientValidationFunction：设置客户端验证脚本函数的名称
* function *FunctionName*(source, arg)
* ServerValidate：设置服务端验证事件的名称
* void *FunctionName*(object source, ServerValidateEventArgs arg)
* arg.Value：控件的值
* arg.IsValid：验证通过

### ValidationSummary

* 不执行验证，显示其他验证控件的错误信息
* HeaderText：设置标题
* DisplayMode：BulletList项目符号列表、list无符号列表、SingleParagraph单个段落
* ShowMessageBox：是否弹出一个错误列表对话框

### 禁用验证

* 把验证控件的Enable设为false
* 要使单个按钮（或其他控件）不产生验证，把它的的CausesValidation设为false
* 只执行服务器验证，将EnableClientScript设为false

状态管理
--------

* 服务端状态管理：应用程序状态（Application）、会话状态（Session）、配置文件、数据库
* 客户端状态管理：视图状态（ViewState）、控件状态、隐藏域、Cookie、查询字符串

### 视图状态

* EnableViewState：@Page指令或控件标记的属性或对象的属性。默认为true
* Web.config的system.web下的Pages元素的EnableViewState控制所有页面
* MaxPageStateFieldLength对ViewState储存的数据进行分块，单位为字节视图状态变量
* 只适用于某一页
* 用法：this.ViewState["key"] = value;

### 查询字符串

* 不需要任何服务器资源、实现简单、支持广泛；有安全性风险
* 发送：Respose.Redirect(“\~/Hello.aspx?key=” + TextBox1.Text.Trim() + "&...");
* 接收（Page_Load）：var value = Request.QueryString["key"];
* 还有HttpRequest对象的Params属性可以读取查询字符串

### Cookie

#### 创建

    HttpCookie aCookie = new HttpCookie("LastVisit");
    aCookie.Value = DateTime.Now.ToString();
    aCookie.Expires = DateTime.Now.AddDays(1);
    Response.Cookies.Add(aCookie);

    // 多值
    HttpCookie aCookie = new HttpCookie("UserInfo");
    aCookie.Values["userName"] = "123";
    aCookie.Values["pw"] = "456";

#### 读取

* cookie的Name属性为Key

<!-- -->

    string userName = "123",pw;
    if (Request.Cookies[userName] != null)
        pw = Request.Cookies[userName].Value;

#### 修改和删除

* 修改Cookie貌似只能再添加一次同Key的Cookie，Request获取或者自己创建后要Response回去。貌似每次都要把东西重新设置一遍。直接`Response.Cookies["Key"].Value=...`好像也要全部设置一遍。
* `aCookie.Expires = DateTime.Now.AddDays(-1)`可以让浏览器删除Cookie，还是要Response回去
* `aCookie.Values.Remove(subKey)`可以删除多值Cookie的子键

#### 控制范围

    aCookie.Domain = "www.baidu.com" // 域范围
    aCookie.Path = "/App1" // 某个文件夹或应用程序

### 会话

* Web.Config的system.web下的sessionState项
* timeout属性：过期时间，单位为分钟

#### 储存会话数据

* Session技术是服务端技术，用户可以把各自的数据保存在各自的Session中
* 每一次会话都是用SessionID进行唯一标识
* SessionID通过Cookie传输，如果指定了无Cookie会话（cookieless属性为true），则通过URL传输

#### 会话状态模式

* sessionState的mode属性（SessionStateMode枚举）：InProc、StateServer、Custom
* InProc：进程内模式。储存在内存中，唯一支持Session_End的模式
* StateServer：状态服务器模式，可以在重启App时保留应用状态。需要设置stateConnectionString="tcpip=服务器：端口"

#### 使用Session

* Session["Key"] = Value
* Session.Remove("Key")和Session.RemoveAt()
* Session.Clear()和Session.RemoveAll()
* Session.Abandon()

### 应用程序状态

* 可以被任何客户端访问，保存在内存中
* Application["Key"] = Value或Application.Add("Key", Value)
* Application.Set("Key", Value)
* 修改前需要Application.Lock()，之后UnLock()
* Clear、RemoveAll、Remove、RemoveAt

### 主题

* page元素的Theme和StylesheetTheme属性，前者主题的外观优先于本身的外观
* 写在web.config里就是全局的
* EnableTheme=false可以禁用主题
* 去掉ID和Text等内容，其他的放到skin里
* 默认外观只能有一个，否则加SkinID
* 在Page_PreInit事件中可以动态指定主题，但该事件先于其他事件（button_click）发生，所以必须加一句redirect

母版页
------

* @ Master指令
* asp:ContentPlaceHold外部是母版，内部是内容页的内容
* 请求时，母版页和内容页会合并成结果页发送给用户
* 母版页和内容页的事件独立，某些相同的事件母版页先触发
* 母版页可以嵌套（父母版页）

### 内容页

* asp:Content控件的ContentPlaceHoldID属性对应母版页中的那个控件的ID
* `@ Page MasterPageFile`属性引用母版页
* 如果要访问母版页，需要添加`@ MasterType VirtualPath`，对象名为Master。但无法直接访问控件，控件的属性可以用public的属性读取或修改。

Menu(TreeNode)
--------------

### 创建方式

* 设计时手动添加
* 以编程方式添加：Menu.Items.Add(MenuItem); TreeView.Nodes.Add(TreeNode)
* 绑定站点地图（sitemap——SiteMapDataSource——Menu）
* 绑定XML文件（XML——XMLDataSource——Menu），需要手动进行数据绑定

### 属性

* Orientation：竖排还是水平
* Static/Dynamic DisplayLevel：静态时展示多少级菜单
* Static/Dynamic MenuStyle/MenuItemStyle/SelectedStyle/HoverStyle

### MenuItem(TreeNode)

* Text
* ToolTip
* NavigateUrl
* ChildItems.Add(MenuItem); ChildNodes.Nodes.Add(TreeNode)

### 带复选框的TreeView控件

* CheckedNodes集合

#### ShowCheckBoxes属性（TreeNodeTypes枚举）

* All：所有节点
* Leaf：所有叶节点
* None：不显示，默认值
* Parent：所有父节点
* Root：所有根节点

### SiteMapPath

* 根据Web.config的sitemap定义的数据自动显示当前页面的位置
* ParentLevelDisplay：默认-1，显示所有节点；为0就只显示当前节点
* PathDirection：路径方向，RootToCurrent和CurrentToRoot
* PathSeparator：分隔符
* RenderCurrentNodeAsLink：是否显示为超链接
* ShowToolTips：是否显示工具提示
* SiteMapProvider：指定其他站点地图提供程序

#### SiteMapDataSource

* StartFormCurrentNode为false且未设置StartingNodeUrl：起始节点为层次结构的根节点（默认设置）
* StartFormCurrentNode为true且未设置StartingNodeUrl：当前正在查看的页的节点
* StartFormCurrentNode为false且已设置StartingNodeUrl：层次结构的特定节点

加密（System.Web.Security）
---------------------------

* FromsAuthentication.HashPasswordForStoringInConfigFile(string,"md5")

数据控件
--------

### 数据绑定控件

* GridView
* DetailsView：以表格形式显示单条记录
* FormView：比DetailsView可以自由编辑模板
* ListView：不带有分页功能，要配合DataPager

### GridView

* DataSource
* DataMember：如果DataSource是数据库的话，这个是表名
* DataBind方法：修改后需要重新绑定一下
* SelectedIndexChanged事件：点击“选择”后触发
* DataKeyName：主键，把列名赋给它。不知道可不可以用非主属性
* SelectedValue：选中某一行后可以根据DataKeyName获取主键的值

#### 图形化配置

* 自动生成字段会把所有列显示出来，而且名字直接是列名
* BoundField：手动指定列
* DataField：列名
* HeaderText：显示的东西

### SqlDataSource

* 可以图形化配置：高级生成选项里可以自动生成下面的那些Command
* ConnectionString
* Delete/Insert/Select/Update Command
* Delete/Insert/Select/Update CommandType：Text、StoredProcedure
* Delete/Insert/Select/Update Parameters
* DataSourceMode：DataSet、DataReader
* EnableCaching
* ProviderName

### ObjectDataSource

* 不能直接访问数据库，需要配置，但与数据库结合相对少
* 需要编写一个类，里面写好几种方法，然后和这个控件绑定（TypeName)
* Delete/Insert/Select/Update Method/Parameters
* Filter Expression/Parameters：SelectMethod时使用的Where语句
* EnableCaching
* SelectCountMethod：检索行数所用的方法

杂项
----

* 404：https://jingyan.baidu.com/article/925f8cb8f0b624c0dde0569a.html

 

 

 
