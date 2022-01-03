# Vue学习笔记（三）

<hr/>

## 组件化编程

### 非单文件组件：

Vue中使用组件的三大步骤：

1. 定义组件 `Vue.extend()`
2. 注册组件 `components` (局部/全局`Vue.component("自定义组件名", "引用组件名")`)
3. 使用组件(标签形式)

`data`配置项必须写成函数式，因为防止`内存引用`，导致一个地方修改变量影响另一个使用该组件的变量值。使用`template`配置组件模板结构

```js
        // 创建hello组件
        const hello = Vue.extend({
            template:`
                <div>
                    <h2>hello : {{name}}</h2>
                </div>
            `, 
            data(){
                return {
                    name:'Tom',
                }
            }
        })

        // 创建school组件
        const school = Vue.extend({
            // el:'#root', 不要写el配置项，因为其组件都被vm所决定服务于哪个容器
            template:`
                <div>
                    <h2>学校名称: {{schoolName}}</h2>
                    <h2>学校地址: {{addr}}</h2>
                </div>
            `, 
            data(){
                return {
                    schoolName:'google',
                    addr:'china',
                }
            }
        })

        // 创建student组件
        const student = Vue.extend({
            template:`
                <div>
                    <h2>学生姓名: {{studentName}}</h2>
                    <h2>学生年龄: {{age}}</h2>
                </div>
            `,
            data(){
                return {
                    studentName:'James',
                    age:18,
                }
            }
        })

        // 全局注册
        Vue.component("hello", hello)

        // 创建vm
        new Vue({
            el:'#root', 
            // 2. 注册组件 (局部注册)
            components:{
                school, //可以直接简写school 同名结构赋值
                student,
            },
        })
```

使用方法示例：

```html
    <div id="root">
        <hello></hello>
        <!-- 编写组件标签 -->
        <school></school>
        <hr>
        <student></student>
    </div>
```

:exclamation: 组件名：多个单词用`-`隔开(kebab-case)，且由于`js`语法，需用引号。CamelCase命名需要Vue脚手架支持。不要与HTML预留字冲突。

:exclamation: 可以在定义组件时添加`name`属性，使其不管注册时采用什么名字，开发者工具都显示`name`值。

:exclamation: 简写：

```js
const mycomponent = Vue.extend({options})
// 等价于
const mycomponent = {options}
```

#### 组件嵌套：

即`components`嵌套注册。在实际开发中存在`app`组件，它管理所有其它组件。

#### :question: 分析：

1. 组件本质是名为`VueComponent`的构造函数，由`Vue.extend`生成。
2. 我们插入的标签，Vue解析时会创建相关组件的实例对象，即执行`new VueComponent({options]})`。
3. 每次调用`Vue.extend`，返回全新的`VueComponent`
4. 组件配置中的`this`指针指向当前`VueComponent`对象

#### :exclamation: 内置关系：

:star: **实例的隐式原型属性永远指向其缔造者的原型对象**

1. `VueComponent.prototype.__proto__=== Vue.prototype`

2. 为什么？=> 使组件实例对象能够永远访问到Vue原型上的属性和方法

   ![](https://s3.bmp.ovh/imgs/2021/10/3ebc82293993be31.png)

### 单文件组件：

创建`xxx.vue`文件，借助Vue脚手架才能运行

```vue
<template>
    <div class="demo">
        <h2>学校名称: {{schoolName}}</h2>
        <h2>学校地址: {{addr}}</h2>
        <button @click="showName">点我显示学校名称</button>
    </div>
</template>

<script>
    // 组件交互相关的代码(数据方法等)
    export default {
        name:'School',
        data(){
            return {
                schoolName:'google',
                addr:'china',
            }
        },
        methods:{
            showName(){
                alert(this.schoolName)
            }
        },
    }
    // 默认暴露
    // import ??? from ???
    // export default school
</script>

<style>
    /* 组件的样式 */
    .demo{
        background-color: orange;
    }
</style>

```

整体架构：

```
|
|- App.vue
|- Component1.vue
|- Component2.vue
|- ...
|- main.js
|- index.html
```

## Vue CLI

`npm run serve` 编译启动

解读`main.js`中的代码：引用的实际上是`vue.runtime.xxx.js`只包含核心功能而没有模板解析器(webpack打包时节省空间)，所以不能使用`template`，而用`render`函数接受到的`createElement`函数代替去指定具体内容。

```js
render: h => h(App)
// 完整写法
// 用于解析模板
render(createElement){
	return createElement(App)
}
```

### `ref` 属性(标识)：

1. 被用来给元素或子组件注册引用信息
2. 对于传统`html`标签相当于替代了`id`，获取的是真实DOM元素；但是对于组件获得就是该组件的实例对象。
3. 使用方法：打标识:  `<h1 ref='tagName'>xxx</h1>`  在script中通过 `this.$refs.tagName`拿到。

### `props`属性

让组件接收外部传入的数据。

1. 传递数据

   ``` html
   <!-- 通过 : 即v-bind 获得""内的表达式值得方式来得到整型数据-->
   <Student name='张三' sex="男" :age="18"/>
   ```

2. 接收数据：

   - 简单声明接收

     ```js
      props:['name', 'age', 'sex'] // 声明属性名
     ```

   - 接收的同时对数据进行限制

     ```js
     props:{
         name:String,
         age:Number,
         sex:String,
     }
     ```

   - 是否必需和默认值

     ```js
     props:{
         name:{
             type:String,
             required:true, // 必传字段
         },
         age:{
             type:Number,
             default:99, // 默认值
         },
         sex:{
             type:String,
             required:true,
         }
     }
     ```

:exclamation: props只读，Vue底层检测对props的修改。一般如果业务需要修改的话，是在data属性中引入props的数据，然后对其进行修改。

但是`vue`只能比较浅层的监听，对于对象属性值的修改，并不会报错。

###  `mixin`属性

可以把多个组件共用的配置提取成一个混入对象。

定义混入：

```js
// mixin.js
export const mixin = {
    data(){
        return{
            x:100,
            y:200,
        }
    },
    methods:{
        showName(){
            alert(this.name)
        }
    },
    mounted() {
        console.log("你好啊")
    },
}
```

引入混入：

1. 全局混入(慎用)：`Vue.mixin(xxx)` 全部的`vm`和`vc`实例都将引入
2. 局部引入(单个`.vue`文件) 先 `import {xxx} from path` 再在`vm`实例中配置`mixins:[xxx, xxx2]`

### 插件

功能：增强Vue

本质：包含`install()`方法的一个对象，`install`的第一个参数为Vue，第二个参数是插件使用者传递的任意数据

定义插件：

```js
// plugins.js
import Vue from "vue"

export default {
    install(){
        console.log("install!")

        // 全局过滤器
        Vue.filter('mySlice', function(val){
            return val.slice(0, 4)
        })

        Vue.mixin({
            data(){
                return {
                    x:'100',
                    y:'200',
                }
            }
        })

        Vue.prototype.hello = () => {
            alert("hello!")
        }
    }
}
```

使用插件

```js
import plugins from "./plugins"

// 应用插件
Vue.use(plugins)

// Vue.use(plugins, 1, 2, 3) // 可以带其它参数
```

###  `scoped`标签 

让样式在局部生效，防止冲突

写法: `<style scoped lang='css/less'>`

