# Vue

https://cn.vuejs.org/guide/essentials/watchers.html#eager-watchers
https://cn.vuejs.org/examples/#attribute-bindings
https://component-party.lainbo.com/#%E4%B8%8A%E4%B8%8B%E6%96%87
https://www.patterns.dev/vue/composables

## 组件

* 单文件组件SFC：在一个.vue中写html css js
* 使用者import后在template中可以像使用HTML元素一样声明。组件命名规则为AxxBxx，HTML中对应axx-bxx
* props：允许使用者传入参数给本组件。定义时用xxxYyy，使用时HTML attribute用xxx-yyy
* event：组件内emit触发，使用者添加处理程序，相当于信息由组件传出去了
* slot：使用者在HTML content里写东西，会替换掉组件中的slot元素。组件中的slot如果有content则表示默认值
  * 命名slot：组件中给slot加name attr，使用者在content里放多个template上对应加 v-slot:名称 或 #名称

```html
<script>
import SubCmp from './SubCmp.vue' // 在模板中使用时必须闭合

export default {
  data() { ... }
}
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped> @import './base.css'; 支持 k: v-bind(引用data里的变量) </style>
```

### 选项式Options API

对OOP用户友好。缺点：响应式数据依赖于组件，颗粒度太粗，如果不同组件有相同的数据处理逻辑，就不能复用；除非用vuex，但又比较重；还有mixin，但不好用，容易重名。

```html
<script>
export default { // TS加defineComponent()
  name: 'Cmp', // 需全局唯一，不声明也能用但在devtools中会显示匿名的。组合式会自动推导

  data() {  // 它返回的对象作为响应式的状态，暴露在this上
    return {
      count: 0
    }
  },

  computed: {  // 如果一个属性是由另一个计算出来的则用它。与method相比有缓存
    countplusone() { return this.count + 1 }
    // 也支持赋值修改：countplusone:{get(){返回data里的对象},set(v){修改data里}}。3.4的getter支持参数表示上一个计算的值
  },

  methods: {  // 用来更改状态与触发更新的函数，在template中作为事件监听器绑定
    increment() {  // 不能使用箭头函数，因为框架之后会绑定this，把data和method等都提取挂在组件对象上（称为注入）
      this.count++
    },

    trigger_ev1(e) { // 普通的方法。也可在生命周期钩子中触发
      this.$emit('event1', 123或props或data等数据)
      // 可在模板中行内写$emit()。若出现@ev="$emit('ev')"，前者是子组件或原生元素的事件，后者本组件发出给更父的
    }
  },

  emits: ['event1'], // 使用者 <Cmp @event1="(arg) => ..."> 接收组件内传出的

  props: {
      arg1: String, // 使用者 <Cmp arg1="xxx"> 传入参数给本组件，本组件不应修改只能使用
      arg2: {
          type: Boolean, // :arg2="true"，如果不用绑定则传了字符串
          required: true,
          default: xxx // 数组用()=>[]
          validator: (val) => 布尔值
      }
  },

  components: { 子组件对象, } // 组合式会自动推导

  mounted() {  // 生命周期钩子：在组件生命周期各个不同阶段被调用
    console.log(`The initial count is ${this.count}.`)
    this.timer=setInterval()
  },
  beforeUnmount() { clearInterval(this.timer) },
  async created() { await fetch()获取data的数据 }, // 框架不会await本函数，等给data赋值后会再渲染

  watch() { // 监听对象被重新赋值时执行副作用
    count(newCount, 可选oldCound) { ... }
  },

  template: `<button @click="increment">Count is: {{ count }}</button>` // 用于非SFC。CSS：在data里写{k:v}对象，绑定到style上
}
</script>
```

### 组合式Composition API

```html
<script setup lang="ts">
import { ref, computed, onMounted, nextTick, computed, watch } from 'vue'

const count = ref(0)

const countplusone = computed(()=>count.value+1) // 返回的也是ref，会自动追踪响应式依赖

function increment() {
  count.value++  // script中访问ref状态时必需用.value，访问reactive的属性不用
  nextTick(()=>访问更新后的DOM) // 改变状态后，DOM不会立即更新，用此函数等待
}

onMounted(() => { ... })

watch(count, (newCount, 可选oldCount) => {当count改变时要执行的副作用回调}, {deep:true若想监听obj属性的改变，3.5支持用数字表示深度})
// 第一个参数还可以是getter（如()=>obj.foo）、多个数据源组成的数组

interface Props { foo: string; bar?: number }
const props = defineProps<Props>() // 此函数无需导入，定义默认值要再包一个withDefaults。也可以写defineProps({选项式语法})

const emit = defineEmits<{ // 也可以写defineEmits(['ev1'])
  (e: 'event1', val: number): void
}>()
function trigger_ev1(ev: Event) { emit('event1', 123) }

// 模板引用，手动操作DOM，如 <p ref="p">hello</p> 创建一个与ref attr同名的ref对象。只能在挂载后使用
const p = useTemplateRef('ref标签的值') // 选项式语法：this.$refs.p
</script>

非setup的script用组合式语法：export default {setup(){ 组合式语法; return {状态对象} }}
```

### 响应式

* 基础值类型用ref，或如果之后需要重新赋值变量。object array Map Set用reactive()
* ref就是reactive({value:v})。reactive将引用类型参数直接变成响应式的，且是深层响应的
* tempalte中当ref对象是顶层属性或作为最终表达式时会自动解包（隐式.value）就像普通属性一样。如{{foo+1}}和{{obj.foo}}和@click="v => foo=v"，但{{obj.foo+1}}这种不会，解决可以将foo解构为顶级属性
* ref对象在reactive object里使用（作为成员）时会自动解包，但reactive数组或Map不会
* 原理：当在被监控的函数中（如模板是render函数、computed、watch等）使用的ref或reactive对象变化时，会重新运行函数
* 如果将props.xxx传进另一个函数，或解包let{x}=o，则会失去响应式。因此一般许多函数都是传props和属性名字符串，内部props['属性名']
* TODO: watchEffect

### css

* scoped将css限制于本组件。module标签将css class选择器转变为$style变量的成员，之后需手动应用
* 使用者控制组件内样式
  * 默认scoped的样式不会传到子组件中，除了：
  * 组件的根元素受使用者和本身的影响，如使用时给组件加style，会传递到组件内根元素上合并
  * 使用者在content里加class，样式写在使用者的css里
  * :deep(x)、:slotted(x)

### 生命周期

beforeCreated -> 注入 -> created -> 模板编译 -> beforeMount 无真实DOM -> 挂载 -> mounted 真实DOM已呈现 -> data变动 -> beforeUpdate -> 重新渲染 -> updated -> beforeUnmount -> 销毁 -> unmounted

## 模板template

* 只能是单根元素，因为编译成虚拟结点 `render(h=>h('根元素', [子内容]))` 后是单根的
* 使用data里的变量时，必须不能加this，会隐式加。只有在js中（和动态绑定表达式）才可用this

### 文本插值

* handlebar {{xxx}}，当xxx变动时会自动更新
* xxx可以也只能是表达式，如 {{var1 ?? 不存在时的值}}、``{{ `${user} ${pass}` }}``。变量取自组件对象的data，也可调用methods，或v-for创建的
* 内容会解释为纯文本（转义为实体）。若想插入HTML，用`<span v-html="xxx"></span>`
* 不能直接访问用户附加在window上的全局对象，要手动加到app.config.globalProperties里；常用的可以直接访问，如Math和Date

### attr绑定

* 单个attr：`<div v-bind:id="xxx"></div>` 或 :id="xxx"，xxx可以是表达式
* 多个attr：v-bind="obj"
* 布尔型attribute根据值决定是否存在于该元素上，如 :disabled="xxx.length<5" 当长度小于5时禁用。若=''算满足条件
* 动态计算绑定的attr名 :[var_attrname]="xxx"，事件同理
* 3.4同名属性 :id 表示 :id="id"
* JSX能写`<div id="{{ xxx }}">`

### toggle class

* :class="{ cls名称: 布尔表达式 }"，当表达式为truthy时则启用此cls。常见技巧：绑定一个返回obj的computed函数
* 数组语法 :class="[b?c1:c2, c3, {c4:b2}]"。其中c可以是字符串字面量，也可以是变量
* :style="{ color: var_blue }" 名称要用js版的，如backgroundColor，而非backgournd-color，否则-会被理解为减号，或者再加单引号也行

### v-model双向绑定

* `<input v-model="xxx">`，之后用户在输入框里输入，就能自动改变xxx的值
  * 手动实现 :value="xxx" @input="e=>xxx=e.target.value"
* 控件
  * radio：value的值
  * select：v-model放在select上，值是option的value
  * checkbox：若多个绑定到了同一个vm且是个数组，则勾上时会自动添加进去。支持true-value false-value对应非bool的关联值
* 修饰符
  * v-model.lazy 丢失焦点后才更新
  * .number 将输入的值转换为数字
  * .trim
* TODO: 3.4 defineModel

### 条件

* v-if="布尔表达式" v-else-if v-else 当满足时才渲染进dom，否则移出dom
* v-show 不满足时添加display:none，频繁开关时性能更好。即使初始条件不满足也会存在于dom中只是隐藏了可能有开销，v-if若初始条件不满足则不会渲染
* 切换多个元素：v-if加在template元素上，里面放要切换的。此template元素只是包装不会渲染，相当于编程意义上的大括号

### 循环

* `<ul><li v-for="item in arr">{{item}}</li></ul>` 其中源可以是computed计算过滤出来的，item在循环作用域内就像正常变量一样能用于其他指令
  * 获得索引：(item, index) in arr
  * 解构：{id, content} in arr
  * 范围：n in 10，从1开始
  * 遍历object的属性：(val, key, ndx) in {k1:1, k2:2}
* 若item是object如 {id:稳定不重复数字, content:内容}，要加 :key="item.id"，默认是arr的index。更新时会用到，否则会重新渲染整个迭代对象
* 如要配合v-if，不应放在同一级，根据需要配合template元素

### 事件

* @等价于`v-on:`
* @click="简单js代码如count++。或调用methods中的函数，无参时可只写方法名"
  * 不支持行内直接写alert等
  * 只写方法名时handler具有event参数且会自动传递。.target访问到绑定的DOM元素
  * 主动调用函数时可用`$event`访问event参数，或创建具有event参数的箭头函数。实际上$event也能引用自定义事件的第一个参数
* 事件修饰符
  * @submit.prevent 在form里按下button或回车时触发，但并不发出请求，只是方便将事件放在form元素上。链接上的@click.prevent同理
  * .once 只触发一次 .stop 阻止继续冒泡 .capture 使用捕获模式
  * .self 只有target是自己时才触发
  * 按键修饰符：@keyup.enter="js" 按回车时触发。还有tab delete esc space等
  * 鼠标修饰符：.middle .right 右键还需配合oncontextmenu="return false"
* v-on:"object"

### 透传

* 对于组件，若只有一个根元素，使用时加在上面的非props或emits的attr，会直接合并添加到根元素上
* 若多于一个根元素，或禁用了自动透传，用$attrs，其中事件暴露为onXxx，attr若有横杠需用[]，一般直接v-bind="$attrs"全绑定到指定元素上
* 在js中访问：useAttrs()

## 安装

* 浏览器调试扩展：https://devtools.vuejs.org/
* VSC扩展：：Vue - Official(Volar)。TypeScript Vue Plugin，关闭@builtin的TS LSP。es6-string-html若使用JS字符串表示HTML模板用它可提供高亮
* 创建脚手架：npm create vue@latest，会交互式安装一些组件。然后手动cd创建的项目文件夹，npm i
* 运行：npm run dev。生产环境输出到dist中：npm run build

### 使用

* 浏览器中不编译直接使用：不支持SFC，因为module强制检查MIME只能是JS类型，而SFC算text/html。import本地JS也有此问题，file协议和py的http.server不行（TODO: 好像是BUG）
  * v-cloak：当所在元素被挂载后移除，配合`[v-cloak] { display: none; }`避免“未编译模板闪现”。另一种普通处理白屏方式：在#app里写骨架屏，vue加载后会替换内容
  * 一种on-the-fly使用SFC的方式：vue3-sfc-loader
  * 手动编译：npm i -D vue-tsc，-m esnext App.vue。仅类型检查（IDE中不需要用）：--noEmit。之后可用runtime版vue。esm必须用importmap

```html
<script type="importmap">{"imports":{"vue":"https://registry.npmmirror.com/vue/latest/files/dist/vue.esm-browser.js"}}</script>  非debug版：.prod.js

<div id="app">{{ msg }}</div>  DOM内嵌模板，仅限根组件，且要用aaa-bbb使用AaaBbb组件

<script type="module">  // 一般内容写在main.js中，html里用src
  import { createApp } from 'vue'
  import './assets/main.css' // TODO: 看看非vite是否可行

  createApp({
    data() { ... }   // 根组件
  }).mount('#app')
</script>

另一种模板来源：<script type="text/x-template" id="xxx">模板内容</script> 或用template元素。组件属性指定template:"#xxx"。无法加载自外部js
```

### app

```js
const app = createApp(根组件)
app.config  配置一些应用级选项，如.errorHandler捕获所有子组件上抛而未被处理的错误
app.component('MyCmp', MyCmp或{定义})  在app范围内注册组件，使得它在任何地方都可用
app.mount(可传DOM对象)  返回值不是app
console中用$vm0也能访问到第一个app
```

### 运行时渲染原理

数据变化 -> 派发更新（运行依赖函数） -> render（全量重新生成组件中的所有UI节点） -> 生成虚拟DOM -> diff + patch -> 运行渲染函数（抽象层）

vue未来（3.6+）不会全量生成，而是更精细化地生成响应式代码（Vapor模式），就不需要虚拟dom和diff了。但必须用编译器，没有渐进式

## vue2

* 若data的属性是数组，修改时要用splice，而不能直接赋值

## vite

* https://cn.vitejs.dev/
* 特性：自动重载。可以直接在html的script中引入module和ts。内置postcss支持。可以在js中import css
* https://github.com/fi3ework/vite-plugin-checker
* 编译打包：index在项目根目录。public目录下的会原封不动复制，src下的会被处理。

## 其它组件

* 在线playground：https://play.vueuse.org/
* 代码规范：https://eslint.vuejs.org 加ESLint VSC扩展
* https://vueuse.org/ 一些组合式API的增强
* pinia：状态管理。组件的data其实就是它们的状态，但有些是全局或app级别的，如果项目太大太复杂就要本工具，如是否登录
* nuxt：SSR全栈，避免纯客户端SPA白屏
* Vitesse：单测
* VitePress：SSG

### UI

* 官方各UI列表：https://ui-libs.vercel.app/
* 星数较多：Vuetify MD风格、daisyUI 基于Tailwind
* 国产：NaiveUI、ElementPlus、AntDesignVue
* quasar：用同一套代码同时开发桌面端和移动端应用
* primefaces/primevue
* 小众：balmjs/balm-ui antoniandre/wave-ui epicmaxco/vuestic-ui varletjs/varlet MD风格

## 暂时不看的

* router：https://router.vuejs.org/zh/ https://vueschool.io/courses/vue-router-4-for-everyone https://www.vuemastery.com/courses/touring-vue-router/vue-router-introduction/
* 例子：https://cn.vuejs.org/ecosystem/themes
* https://cn-vuejs-challenges.netlify.app/
* https://vueschool.io/courses/application-monitoring-in-vue-js-with-sentry
* https://www.vuemastery.com/courses/vue3-forms/base-input https://www.vuemastery.com/courses/validating-vue3-forms/why-vee-validate/
* https://vueschool.io/lessons/fake-scoped-slots-with-functions
