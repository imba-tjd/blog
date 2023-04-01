# jQuery

* 中文文档3.0：https://www.jquery123.com/
* `$`是window.jQuery的别名，是个函数，可以链式调用
* `$(...)`把内部的东西变成jQuery对象，可以是CSS选择器、HTML字符串、DOM对象，之后.append()等里面也可以是一样的东西
* jQuery对象
  * 可认为是个HTML对象集合，用`[]`取出单个HTMLElement，无参get()变为真正的数组
  * 设置它们的属性会应用到每一个数组元素，空集合也不报错
  * 一些函数传一个参数是查询，传两个是修改；还有一些函数无参调用是查询，传参是修改
  * 查询时若集合中有多个元素最好不要直接用，而是用.each(function(ndx){$(this)...})
* 选择器
  * 找不到时为`[]`
  * 支持:has和:contains(相当于has-text) :input/text/password/submit/checked/selected表单元素
  * 进一步选择：find(选择器/jQuery对象) filter(选择器/回调函数) map() next(支持选择器) nextAll() prev() eq(第几个) slice(替代:eq和:lt) first() last() even() siblings() children() contents()也会选中文本节点 parent()直接父元素 parents()一直往上的每一个父元素集合，支持选择器来过滤 parentsUntil(selector, filter) closest()
* $.contains(parent, child) 检测不会包含parent
* end()：链式调用时，假设前面调用了find()和css()，调用此函数后可返回最开始的对象上
* text() html()：相当于textContent和InnerHTML，但重载接受返回string的回调，另外后者能自动执行script
* css()：是计算后的值。设置：传{k:v}
* is()：判断调用者与参数选择器是否相等
* 操作类：hasClass() addClass() removeClass() toggleClass()
* 操作属性：attr(属性名, 若有第二个参数则为设置否则为获取) removeAttr()
  * attr对应inline DOM属性，prop对应JS属性：prop('checked')->true，attr('checked')->'checked'（若有）
* data()：储存任意数据，能读取data-属性初始化，但储存时并不通过修改它
* a.append(b)等于b.appendTo(a) prepend() after() insertAfter() before() remove()返回被删除的元素 detach()也是删除但保留原来绑定的事件 empty()清空子元素
* add()：把元素添加到jQuery对象元素集合中；addBack()：链式调用中，假设调用了nextAll()和此方法，把最初的和后来选中的一起作为jQuery对象
* clone()：复制元素，传true也复制事件
* height()：是CSS计算后的值，是数字而不是'123px'这样，类似于原生的clientHeight；offset()相对于body顶端的偏离，position()相对于最近具有相对位置的父元素的距离；outerHeight(true)包含margin的高度
* index() 相比于原生，能直接用于伪数组
* 工具
  * $.parseHTML()
  * $.globaleval()
* 插件系统：`$.fn.modal = function(){ ...; return this}; $.fn.modal.defaults设定默认值 $('div').modal()`
* jQuery Tiny Pub/Sub插件：类似于C#概念上的事件
* 当前版本：$.fn.jquery
* 多版本共存时让出$和jQuery变量控制权：`const jQ = $.noConflict(true)`

## AJAX

* 注意同源策略，以及https网页无法请求http的
* JSONP：利用了script标签能跨域，但只能GET。先准备一个函数用来处理参数里的真正的内容，用JS动态创建一个script标签添加到页面里，在服务端的脚本里调用前面约定的函数
* $.get getJSON getScript加载后就执行。第二个参数为object的查询字符串，第三个参数为成功后的回调但第二个参数不存在时也能作为第二个
* $.post：第二个参数是表单body
* jqo.load()：把HTML文件加载到选择的元素中
* $.ajax('url',settings).done(data=>...).fail((xhr,status)=>...).always(()=>xxx)
* settings是个object
  * method 默认为get
  * contentType 指定POST的格式，默认是表格
  * data 发送的数据，可以是字符串、数组或object，其中字符串不会再次编码
  * headers 额外的HTTP头，object
  * dataType 接收的消息格式，不填会自动猜测
  * timeout 超时失败的毫秒数
  * success等是回调函数，参数为data
* 没有方法获得发送的头，只能用beforeSend或者headers设置

## 事件

* click() submit()：无参调用就是触发，传回调函数就是设置
* `$(回调函数)`等于$().ready(回调函数)等于document.onload，但jQ的能用于任意对象，且可以反复绑定，会依次执行，无法取消
* 绑定：on('事件名', 选择器, 回调)：若有选择器，绑定到本父元素上，选择的子元素是实际触发的，`$(this)`是子元素
* 取消绑定：off()，无参调用会取消所有的，传名字就取消指定事件的，再传函数可取消具体的handler
* 只执行一次后自动取消绑定：one()
* 触发事件：trigger()
* 与标准不同的：dblclick双击 mouseenter mouseleave mousemove hover鼠标进入和退出都会触发
* val()：表单获取和设置value
* 查看某个元素上绑定了哪些事件，3.5之前可用$._data(elem,'events')，之后去掉了

## 动画

* 基本上第一个参数是毫秒，第二个参数是动画完成后要执行的回调函数
* show() hide() toggle()：从左上角展开/收缩；slideUp() slideDown()：向上收缩/展开；fadeIn() fadeOut()：淡入淡出
* 自定义动画：animate()
* 串行动画：中间用delay()暂停
* 有的动画没有效果，比如对非block元素设置height就无效

## 替代品

* 本体90KB。slim版无ajax和动画，72KB，不用
* https://github.com/nefe/You-Dont-Need-jQuery/blob/master/README.zh-CN.md 原生
* fabiospampinato/cash NPM上叫cash-dom，17KB
* https://umbrellajs.com/ 操作DOM和事件，有ESM版，8KB，最后更新2022年
* https://blissfuljs.com/ 最后更新2019年，12KB
* Zepto 不维护了

# svelte

* 适合编译成WebComponents，宣称没有运行时系统。项目变大后难以压缩，自定义的JS方言
* 组件内部style与使用它的模板中定义的style互不影响
* 用{@html s}渲染HTML内容，否则只会是纯文本
* 反应式更新机制基于赋值语句，如a=1会触发a的更新，o.a=1会触发o的更新；数组的push/pop、delete o.a、a=o.a;a.b=1不会触发对象的更新，后两者可再写o=o触发，数组很麻烦，要在each块中指定对象的id属性用于区分，没看懂这是什么逻辑

```js
// Nested.svelte
export var count = 1 // 导出的变量作为组件的属性。改变了export的语义
$: doubled = count * 2 // 反应式声明，count变化了就重新计算。$: 后可跟语句如console.log、{}代码块、if(){}。原本 $: 只是goto的标记的，这样改变了JS的语义

<p>{count}, {doubled}</p> // 大括号里的可以是表达式

// App.svelte？
import Nested from './Nested.svelte'
let count = 123

<Nested {count} on:click={handleClick} /> // 设置属性，{count}是count={count}的简写，不传时Nested里设置了默认值为1，{...xxx}可展开对象

import App from './App.svelte'
const app = new App({
  target: document.body,
  props: {
    answer: 42
  }
})

{#if xxx} {:else if xxx} {:else} {/if}
{#each cats as cat} <p>{cat.name}</p> {/each}，as后也可以解构
```

## tinyhttp

* 原生TS，类Express

```js
npm i @tinyhttp/app @tinyhttp/logger

import { App } from '@tinyhttp/app'
import { logger } from '@tinyhttp/logger'

const app = new App({
  noMatchHandler: (req, res) => res.status(404).end('Not found')
})
app.use(logger())

app.get('/', (_, res) => void res.send('Hello World'))

app.get('/page/:page/', (req, res) => {
    res.status(200).send(`
    ${req.url},
    ${JSON.stringify(req.params, null, 2)}
  `)
})

app.listen(3000)

Middleware: app.use([path,] handler)
path:
  同一级的path不是按顺序的，而是按最长匹配的
  参数和后缀：/:title.(mp4|mov)、可选：/:title?、通配：*、正则对象。放在req.params里
  app.route('/prefix')
handler: (req, res, next) => { ...; next(); } 支持async，但send()不是async的，只能用于readFile等
new App({
    onError: (err, req, res) => { 500 },
    settings: { // 或app.enable('xxx')
        networkExtensions: true // 添加req.protocol hostname ip
        xPoweredBy: true
        enableReqRoute: true
    }
})
app.locals、req.locals 不同生命周期的自定义属性
Mount子App：app.use(subapp)
模板引擎：https://eta.js.org/
静态文件：https://github.com/vercel/serve-handler
Session：https://github.com/expressjs/session
```

## 前端

* sweetalert2
