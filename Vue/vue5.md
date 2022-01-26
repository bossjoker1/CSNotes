# Vue学习笔记（五）

## `axios`

**同源策略**：协议名 、主机名、端口号必须一致

1. 通过`vue-cli`配置简易代理服务器

   ```js
   // vue.config.js
   // 开启代理服务器
   devServer: {
   proxy: 'http://localhost:5000'
   }
   ```

   存在很大局限性

   发送简单的get请求：

   ```js
   axios.get("http://localhost:8080/demo2/cars").then(
           response => {
             console.log("请求成功了", response.data)
           },
           error => {
             console.log("请求失败", error.message)
           }
         )
   ```

   

2. 完整配置：

   ```js
   devServer: {
         proxy: {
   		// 前缀名，唯一匹配
           '^/demo1': {
             target: 'http://localhost:5000',
   		// 否则显示的路由为/demo1/xxx/xxx
             pathRewrite: {'^/demo1':''},
             ws: true,
            // 是否修改host头进行欺骗
             changeOrigin: true
           },
           '^/demo2': {
             target: 'http://localhost:5001',
             pathRewrite: {'^/demo2':''},
           }
         }
       }
   ```


:exclamation:添加点击事件是对`input`

```html
<div>
    <input type="text" placeholder="enter the name you search"  @keyup.enter="searchUsers" v-model="keyWord"/>&nbsp;
    <button @click="searchUsers">Search</button>
</div>
```

## 插槽

**作用**：让父组件可以向子组件指定位置插入`html`结构，即通过`<slot>`占位，也是一种**组件间通信**的方式。

1. 默认插槽：

   子组件`Category.vue`

   ```html
   <template>
     <div class="category">
       <h3>{{title}}分类</h3>
       <!-- 定义默认插槽，占位 -->
       <slot></slot>
     </div>
   </template>
   ```

   父组件

   ```html
   <Category title="美食" :listData="foods">
   	<img src="https://s3.ax1x.com/2021/01/16/srJlq0.jpg" alt="" />
   </Category>
   ```

2. 具名插槽：

   即给每个插槽通过`name`属性取名字，便于父组件指定哪些结构插入到对应的位置

   子组件：

   ```html
   <div class="category">
       <h3>{{title}}分类</h3>
       <!-- 定义默认插槽，占位 -->
       <slot name="center"></slot>
       <slot name="footer"></slot>
   </div>
   ```

   父组件：

   ```html
   <Category title="美食" :listData="foods">
       <img slot="center" src="https://s3.ax1x.com/2021/01/16/srJlq0.jpg" alt="" />
       <a slot="footer" href="http://www.atguigu.com">更多美食推荐</a>
   </Category>
   
   <!-- vue2.6新写法 仅适合template -->
   <template v-slot:footer>
       <div class="foot">
           <a href="http://www.atguigu.com">经典</a>
           <a href="http://www.atguigu.com">热门</a>
           <a href="http://www.atguigu.com">推荐</a>
       </div>
       <h4>欢迎前来影院观看</h4>
   </template>
   ```

3. :exclamation:作用域插槽：

   数据在组件自身，而数据的结构需要调用者传递过来，可以通过此传递参数给调用者

   子组件：

   ```html
   <div class="category">
       <h3>{{ title }}分类</h3>
       <!-- 传递games参数 -->
       <slot :games="games" msg="hello"></slot>
   </div>
   ```

   父组件：

   必须要通过`<template>`标签包裹

   ```html
   <Category title="游戏">
       <!-- 得到传递过来的对象 ，可使用ES6解构赋值得到数据 -->
       <template slot-scope="dataObj">
           <ol>
               <li v-for="(game, index) in dataObj.games" :key="index">{{ game }}</li>
           </ol>
   
           <h4>{{ dataObj.msg }}</h4>
       </template>
   </Category>
   ```

   

