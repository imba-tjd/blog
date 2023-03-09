# Vue

https://cn.vuejs.org/guide/essentials/list.html#maintaining-state-with-key
https://docs.microsoft.com/zh-cn/learn/paths/vue-first-steps/
https://cn.vuejs.org/examples/
https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA3NTc4Njk1NQ==&action=getalbum&album_id=1978608648645902339

https://vueschool.io/lessons/vue-3-component-template
https://vueschool.io/courses/reusable-vuejs-components-with-slots
https://vueschool.io/courses/vue-router-4-for-everyone
https://www.vuemastery.com/courses/touring-vue-router/vue-router-introduction/

## 单文件组件SFC

* 在一个.vue中写html css js
* 使用者：import Cmp from './Comp.vue'，之后在template中就可以像使用HTML元素一样声明
* props：允许使用者传入参数给本组件
* event：组件内发出，使用者添加处理程序后相当于信息由组件传出去了。无冒泡机制
* slot：使用者在content里的东西，会替换掉组件中的slot元素，slot如果有content则为默认值

```html
<script>
export default {
  name: 'Cmp', // 需全局唯一，不声明也能用但在devtools中会显示匿名的。组合式会自动推导
  data() { ... },
}
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped> @import './base.css'; </style> scoped将css限制于本组件。还有module
```

### 选项式API：对OOP用户友好

```html
<script>
export default {
  data() {  // 它返回的对象作为响应式的状态，暴露在this上
    return {
      count: 0
    }
  },

  computed: {  // 相当于C#的get property。不应修改数据，只应转换
	  countplusone(){ return this.count + 1 } // 也可以写countplusone:{get(){},set(v){}}
  },

  methods: {  // 用来更改状态与触发更新的函数，在template中作为事件监听器绑定
    increment() {  // 不能使用箭头函数
      this.count++
    },

    trigger_ev1(e) {
      this.$emit('event1', 123) // 普通的方法，也可以在模板中行内写$emit()，参数也可以由使用者传attr进来；也可放在生命周期钩子中
    }
  },

  emits: ['event1'], // 使用者<Cmp @event1="(arg) => ...">

  props: {
      arg1: {
          type: Boolean,
          required: true,
          default: xxx
          validator: (val) => 布尔值
      },
      arg2: String
  },

  mounted() {  // 生命周期钩子会在组件生命周期的各个不同阶段被调用。一般还有created
    console.log(`The initial count is ${this.count}.`)
  },

  watch() { ... } // 执行副作用
}
</script>

<template> <button @click="increment">Count is: {{ count }}</button> </template>
```

### 组合式API

* 响应式状态
  * 基础类型用ref；object array Map Set用reactive()，且是深层响应的
  * 感觉上ref就是(reactive({value:v}))
  * tempalte中当ref对象是顶层属性或作为最终表达式时会自动解包（相当于.value），如{{foo+1}}和{{obj.foo}}和@click="v => foo=v"，但{{obj.foo+1}}这种不会
  * ref对象在reactive object里使用时会自动解包，但reactive数组或Map不会

```html
<script setup lang="ts">
import { ref, computed, onMounted, nextTick, computed, watch } from 'vue'

const count = ref(0)

const countplusone = computed(()=>count.value+1) // 返回的也是ref，会自动追踪响应式依赖

function increment() {
  count.value++  // script中访问ref状态时必需用.value，访问reactive的属性不用
  nextTick(()=>访问更新后的DOM) // 改变状态后，DOM不会立即更新，此函数等待
}

const p = ref(null) // 模板引用手动操作DOM：<p ref="p">hello</p> 创建一个与ref attr同名的ref对象，必须这么声明
onMounted(() => { ... })  // 生命周期钩子

watch(count, (newCount) => 当count改变时要执行的副作用回调)

interface Props {
  foo: string
  bar?: number
}
const props = defineProps<Props>() // 此函数无需导入。js版用defineProps({foo: String})

const emit = defineEmits(['event1'])
function trigger_ev1(ev: Event) { emit('event1', 123) }
</script>
```

## 模板template

* 文本插值
  * {{xxx}}，当xxx变动时会自动更新；可以也只能是表达式，如 {{xxx || 不存在时的值}}
  * 内容会解释为纯文本（转义为实体）。若想插入HTML，用`<span v-html="xxx"></span>`
  * 不能直接访问用户附加在window上的全局对象，要手动加到app.config.globalProperties里；常用的可以直接访问，如Math和Date
* attr绑定
  * 单个attr：<div v-bind:id="xxx"></div> 或 :id="xxx"
  * 多个attr：v-bind="xxx"，其中xxx是个object
  * 还支持表达式，如 :disabled="xxx.length<5" 当长度小于5时禁用。布尔型attribute根据值决定是否存在于该元素上
  * 动态计算绑定的attr名：:[var_attrname]="xxx"，事件同理。在DOM内嵌模板中使用时，变量名不能大写，因为浏览器转换成小写，SFC不受此限制
* toggle class
  * :class="{ cls名称: 布尔表达式 }"，当表达式为truthy时则启用此cls，可绑定计算出的obj
  * 数组语法 :class="[b?c1:c2, c3, {c4:b2}]"。单纯的[c1,c2]不如用原生的静态class
  * style类似，但名称要用js版的，如backgroundColor，而不是backgournd-color，否则横杠会被理解为减号，或者再加单引号也行；也支持绑定object
* v-model双向绑定
  * `<input v-model="xxx">`，之后用户在输入框里输入，就能自动改变xxx的值
  * 对于radio，为value的值。对于select，只需指定在select上，无需指定在option上。对于checkbox，若多个绑定到了同一个vm且是个数组，则勾上时会自动添加进去
  * 修饰符：v-model.lazy丢失焦点后才更新，.number将输入的值转换为数字，.trim略
* 条件
  * v-if="布尔表达式" v-else-if v-else 当满足时才渲染进dom，否则移出dom
  * v-show不满足时添加display:none，频繁开关时性能更好，但初始必会渲染。v-if若初始条件不满足则不会渲染
  * 切换不只一个元素：v-if加在template元素上，里面放要切换的，此元素只是包装不会渲染
* 循环
  * `<ul><li v-for="item in arr">{{arr}}</li></ul>` 其中源可以是computed计算过滤出来的
  * 获得索引：(item, index) in arr
  * 解构：{id, content} in arr
  * 范围：n in 10，从1开始
  * 若item是object如 {id:不重复数字, content:内容}，要加 :key="item.id"，默认是arr的index
  * 遍历object的属性：(val, key, ndx) in {k1:1, k2:2}
  * 如要配合v-if，不应放在同一级，可配合在template元素
* 事件
  * @等价于v-on:
  * @click="js代码，或调用methods中的函数，无参时可不加括号"
  * @keyup.enter="js" 按回车时触发
  * @submit.prevent 在form里按下button或回车时触发，但并不发出请求，只是方便将事件放在form元素上
  * .once 只触发一次
* 透传
  * 对于组件，若只有一个根元素，使用时加在上面的非props或emits的attr，会直接合并添加到根元素上
  * 若多于一个根元素，或禁用了自动透传，用$attrs，其中事件暴露为onXxx，attr若有横杠需用[]，一般直接v-bind="$attrs"全绑定到指定元素上
  * 在js中访问：useAttrs()

## 安装

* 浏览器调试扩展：https://devtools.vuejs.org/
* 创建脚手架：npm init vue@latest，会交互式安装一些组件。然后手动进入创建的项目文件夹，npm install
* 运行：npm run dev。生产环境输出到dist中：npm run build

### 使用

在浏览器中直接使用，不支持SFC。

```html
<div id="app">{{ message }}</div>  // DOM内嵌模板，仅限根

ESM版：
<script type="module">
  import { createApp } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js' // 生产版.prod.js

  createApp({
    data() { ...}
  }).mount('#app')
</script>
```

### app

```js
const app = createApp(根组件)
app.config 能配置一些应用级选项，如.errorHandler捕获所有由子组件上抛而未被处理的错误
console中用$vm0也能访问到第一个app
```

### TS

* 对单文件组件类型检查：vue-tsc --noEmit

## vite

https://cn.vitejs.dev/
npm install vite@latest myapp -- --template vue-ts; cd myapp; npm install; npm run devf
npm init @vitejs/app 交互式创建项目
自动重载。可以直接在html的script中引入module和ts。内置postcss支持。可以在js中import css
https://github.com/fi3ework/vite-plugin-checker

## 其它组件

* https://vueuse.org/ 一些组合式API的增强
* VSC：Volar、TypeScript Vue Plugin，关闭@builtin的TS LSP
* pinia：状态管理
* nuxt：ssr全栈
* petite-vue

### UI

* NaiveUI
* element plus
* vuetify MD
* https://quasar.dev/ 用同一套代码同时开发桌面端和移动端应用
* primefaces/primevue
* balmjs/balm-ui 虽然贡献值少但提交数多
* antoniandre/wave-ui 虽然贡献值少但提交数多
* https://vuestic.dev/ 虽然贡献值少但提交数多

## 暂时不看的

https://vueschool.io/courses/application-monitoring-in-vue-js-with-sentry
https://vueschool.io/courses/javascript-testing-fundamentals
https://www.vuemastery.com/courses/vue3-forms/base-input https://www.vuemastery.com/courses/validating-vue3-forms/why-vee-validate/