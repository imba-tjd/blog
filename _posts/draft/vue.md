# Vue


https://vueschool.io/lessons/vue-3-component-template
https://vueschool.io/courses/reusable-vuejs-components-with-slots
https://vueschool.io/courses/vue-router-4-for-everyone
https://www.vuemastery.com/courses/touring-vue-router/vue-router-introduction/

https://docs.microsoft.com/zh-cn/learn/paths/vue-first-steps/
https://cn.vuejs.org/guide/essentials/template-syntax.html#using-javascript-expressions

暂时不看的：
https://vueschool.io/courses/application-monitoring-in-vue-js-with-sentry
https://vueschool.io/courses/javascript-testing-fundamentals
https://www.vuemastery.com/courses/vue3-forms/base-input https://www.vuemastery.com/courses/validating-vue3-forms/why-vee-validate/


单文件组件SFC：在一个.vue中写html css js

```
<script>
export default {
  data() { ... }
}
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped> @import './base.css' </style> // scoped将css限制于本组件
```

选项式 API：对OOP用户友好

```
<script>
export default {
  data() {  // 它返回的属性将会成为响应式的状态，暴露在 this 上
    return {
      count: 0
    }
  },

  computed: {  // 相当于C#的get property。不应修改数据，只应转换
	countplusone(){ return this.count + 1 }
  },

  methods: {  // 用来更改状态与触发更新的函数，在模板中作为事件监听器绑定
    increment() {  // 不能使用箭头函数
      this.count++
    }
  },

  mounted() {  // 生命周期钩子会在组件生命周期的各个不同阶段被调用
    console.log(`The initial count is ${this.count}.`)
  }
}
</script>

<template> <button @click="increment">Count is: {{ count }}</button> </template>
```

组合式 API：

```
<script setup>
import { ref, computed, onMounted } from 'vue'

const count = ref(0)  // 响应式状态。基础类型用ref，object和array用reactive()

const countplusone = computed(()=>count.value+1)

function increment() { count.value++ }  // script中访问ref状态时必需用.value，访问reactive不用

onMounted(() => { ... })  // 生命周期钩子
</script>

<template> ... </template>
```

安装

https://devtools.vuejs.org/
创建脚手架：npm init vue@latest，会交互式安装一些组件。然后进入创建的项目文件夹，npm install。
运行：npm run dev。生产环境：npm run build，会输出到./dist中
在浏览器中直接使用，不支持SFC：

```
<div id="app">{{ message }}</div>

<script type="module">
  import { createApp } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'

  createApp({
    data() { ...}
  }).mount('#app')
</script>

使用ImportMaps，目前只有Chrome原生支持，需要polyfill：
<script async src="https://ga.jspm.io/npm:es-module-shims@latest/dist/es-module-shims.js"></script>
<script type="importmap">{"imports": {"vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"}}</script>
import { createApp } from 'vue'
import App from './App.vue'
```

const app = createApp(根组件)。app.config 能配置一些应用级选项，如.errorHandler捕获所有由子组件上抛而未被处理的错误。console中用$vm0也能访问到第一个app


## 模板

* 文本插值
  * {{xxx}}，当xxx变动时会自动更新；可以是表达式
  * 内容会解释为纯文本（转义为实体）。若想插入HTML，用`<span v-html="xxx"></span>`
* 属性绑定
  * 单个属性：<div v-bind:id="xxx"></div> 或 :id="xxx"
  * 多个属性：v-bind="xxx"，其中xxx是个object
  * 还支持表达式，如 :disabled="xxx.length<5" 当长度小于5时禁用
* toggle class
  * :class="{ cls名称: 布尔表达式 }"，当表达式为truthy时则启用此cls
  * 数组语法 :class="[b?c1:c2, c3]"。单纯的[c1,c2]不如用原生的静态class
  * style类似，但名称要用js版的，如backgroundColor，而不是backgournd-color，否则横杠会被理解为减号，或者再加单引号也行；也支持绑定object
* vm
  * 双向绑定：`<input v-model="xxx">`，之后用户在输入框里输入，就能自动改变xxx的值
  * 对于radio，为value的值。对于select，只需指定在select上，无需指定在option上。对于checkbox，若多个绑定到了同一个vm且是个数组，则勾上时会自动添加进去
  * 修饰符：v-model.lazy丢失焦点后才更新，.number将输入的值转换为数字，.trim略
* 条件
  * v-if="布尔表达式" v-else-if v-else 当满足时才渲染进dom，否则移出dom
  * v-show不满足时添加display:none，频繁开关时性能更好
* 循环
  * `<ul><li v-for="item in arr">{{arr}}</li></ul>`
  * 获得索引 v-for="(item, index) in arr
  * 解构 v-for="{id, content} in arr"
  * 若item是object如 {id:不重复数字, content:内容}，要加 :key="item.id"，默认是arr的index
  * 也可以遍历object，如 {"k1":1, "k2":2}，用 (val, key) in obj
  * 如要配合v-if，应把if放在v-for外面，否则产生的每个元素都有v-if了
* 事件
  * @等价于v-on:
  * @click="js代码，或调用methods中的函数，无参时可不加括号"
  * @keyup.enter="js" 按回车时触发
  * @submit.prevent 在form里按下button或回车时触发，但并不发出请求，只是方便将事件放在form元素上



component（自定义元素）：
```
app.component('comp', {
    props: {  // 允许使用者传入参数给本组件
        arg1: {
            type: Boolean,
            required: true,
            default: xxx
            validator: (val) => 布尔值
        },
        arg2: String
    },
    template: `html模板`,
    methods: {
        event1() {
            this.$emit('event1', arg) // 向使用者发出事件，使用者添加处理程序后相当于信息由组件传出去了
        }
    }
})
使用：script标签引入js文件。然后就能用<comp arg1="true" @event1="..."></comp>，参数也可以动态绑定
```

## vite

npm install vite@latest myapp -- --template vue-ts; cd myapp; npm install; npm run devf
npm init @vitejs/app 交互式创建项目。或npx create-vue
自动重载。可以直接在html的script中引入module和ts。内置postcss支持。可以在js中import css
