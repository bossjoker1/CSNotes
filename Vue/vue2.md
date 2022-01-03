# Vue学习笔记（二）

### 绑定样式

- 绑定`class`样式：

  ​	I.   字符串写法，适用于样式不确定，需要动态指定；

  ​	II.  数组写法，适用于绑定样式个数不确定，名字不确定；

  ​	III. 对象写法，适用于绑定样式的个数、名字都确定，但动态决定用或不用

- 绑定`style`样式：

  ​	I.   `:style="对象名"`

  ​	II.  ` :style="数组名"`  数组存放样式对象

### 条件渲染

`v-show`：是否显示，当前结构样式赋值`display:none`

`v-if / v-else /v-else-if` : 当前结构直接消失，与`<template></template>`配合使用，不破坏结构

3. 列表渲染

   补充`js`知识，数组用`unshift()`方法加在头部

   `v-for`:

   ```js
   // 遍历数组、对象、字符串
   // 遍历指定次数  v-for="(a, b) of 5"
   <ul>
       <li v-for="(p, i) in(of) persons" :key="p.id">
           {{p.name}} - {{p.age}}
       </li>
   </ul>
   ```

4. key原理

   将`index`和唯一`id`作为key。key是虚拟DOM对象的标识

   **虚拟DOM对比算法**：决定是否复用(效率问题)。例如用`index`作为key，并对数组进行了破坏顺序的操作，则会引起界面错误更新和重复劳动。

5. `Vue`**更新时**：

   - `vue`会监视data中所有层次的数据。

   - 通过`setter`实现监视，且在创建`Vue `对象实例时就应该传入要检测的数据。

     ​	I.   对象中后追加的属性，`Vue`默认不做响应式处理

     ​	II.  `Vue.set(target, propertyName/index, value)` 或者 `vm.$set(xxx)`来添加响应式属性，但是不能对`Vue`实例或其根属性进行添加

   - 监测数组数据：先是调用`Array.prototype.xxx`原生方法对数组元素进行更新，然后重新解析模板

   - `Vue`对数组的包装函数包括 `push/pop/shift/unshift/splice/sort/reverse` 共七个，也可用`set` API对其进行设置

6. 数据劫持：将`data`定义的数据劫持后，定义`setter/getter`方法并赋值给`_data`新属性，以此来达到响应式的模式。

7. `v-model`修饰符：`number`转换为数字，`lazy` 失去焦点后加载，`trim`去掉前后的空格

6. 过滤器`filters:` 可用于插值和`v-bind:`

   使用：

   ```html
   <h3>格式化时间：{{time | timeFormater}}</h3>
   <!-- 可接受参数，但是time作为第一个参数隐式传入 -->
   <h3>格式化时间：{{time | timeFormater("YYYY年-MM月-DD日")}}</h3>
   <!-- 串联使用过滤器 -->
   <h3>格式化时间：{{time | timeFormater("YYYY年-MM月-DD日") | mySlice }}</h3> 
   ```

   `Vue`实例定义: 配置全局过滤器: 

   ```js
   // 配置全局过滤器: 在new Vue之前 
   Vue.filter('mySlice', function(){
       return value.slice(0, 4)
   })
   // 局部过滤器
   filters:{
           timeFormater(value, str="YYYY-MM-DD HH:mm:ss"){
               return dayjs(value).format(str)
           },
           mySlice(value){
               return value.slice(0, 4)
           }
       }
   ```

###  **内置指令**

- `v-text` 向其所在的节点中渲染文本内容，正常文本解析，替换所有内容

- `v-html` 支持结构解析，注意对用户输入的恶意性检测

- `v-cloak` ：特殊属性，Vue实例创建完毕接管容器后即删除`v-cloak`属性，与`css`属性选择器配合可解决网速慢而导致出现{{xxx}}的问题

  ```css
  [v-cloak]{ // 属性选择器
  	display:none;
  }
  ```

- `v-once` :  在一次动态渲染后就被视为静态资源

- `v-pre` : 跳过所在节点的编译过程，可使用其跳过没有指令语法或者插值语法的节点来加快编译

### 自定义指令

命名采用'xxx-xxx'方式，不能使用驼峰命名法。

**函数形式**调用时机：1. 指令与元素成功绑定 2. 指令所在的整个模板被重新解析时(即不局限于其绑定元素的数据发生变化)。

```js
directives:{
	big(element, binding){
	element.innerText = binding.value * 10
	// console.log(element, binding.value)
	}
},
```

**对象形式**: 函数形式其实相当于只写了`bind`和`update`

```js
fbind:{
	// 指令与元素成功绑定
	bind(element, binding){
		console.log("bind")
         element.value = binding.value
	},
	// 指令所在元素被插入页面时调用
	inserted(element, binding){
		console.log("inserted")
         element.focus()
	},
    // 指令所在的模板被重新解析
	update(element, binding){
		console.log("update")
         element.value = binding.value
	},
},
```

注意指令中的`this`指针指向的是`Window`对象。

定义全局指令与`filters`一致:

```js
Vue.directive('fbind',{
	bind(element, binding){
		console.log("bind")
         element.value = binding.value
	},
	inserted(element, binding){
		console.log("inserted")
         element.focus()
	},
	update(element, binding){
		console.log("update")
         element.value = binding.value
	},
})
```

### 生命周期

生命周期函数(钩子hook)，指Vue在关键时刻为我们调用的特殊名称的函数，这些函数的名字不可更改，但其执行的具体内容由我们定义，函数中的`this`指针指向的是`vm`或者组件实例对象。

![](https://s3.bmp.ovh/imgs/2021/10/dc16b2a1ec024aad.png)

最重要的两个函数为`mounted() / beforeDestroy()` 。

销毁Vue实例后自定义事件会失效，但是原生DOM事件依然有效，因为已经渲染到真实DOM中了。
