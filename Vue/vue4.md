# Vue学习笔记（四）

## 总结TodoList案例


1. 组件化编码流程：

    (1).拆分静态组件：组件要按照功能点拆分，命名不要与html元素冲突。

    (2).实现动态组件：考虑好数据的存放位置，数据是一个组件在用，还是一些组件在用：

    ​		1).一个组件在用：放在组件自身即可。

    ​		2). 一些组件在用：放在他们共同的父组件上（<span style="color:red">状态提升</span>）。

    (3).实现交互：从绑定事件开始。

2. props适用于：

    (1).父组件 ==> 子组件 通信

    (2).子组件 ==> 父组件 通信（要求父先给子一个**函数**）

3. 使用v-model时要切记：v-model绑定的值不能是props传过来的值，因为props是不可以修改的！

4. props传过来的若是对象类型的值，修改对象中的属性时Vue不会报错，但**不推荐**这样做。

## web Storage

1. 存储内容大小一般支持5MB
2. 浏览器端通过`Window.sessionStorage`和`Window.localStorage`属性来实现本地存储机制
3. 相关API
   - `setItem('key', 'value')`
   - `getItem('key')`
   - `removeItem('key')`
   - `clear()` 清空所有数据
4. 备注:exclamation:
   - `sessionStorage`存储的内容会随着浏览器窗口关闭而消失，即只存在一次"会话"期间
   - `localStorage`存储的内容需要手动清除
   - `getItem('key')`若对应val获取不到，则返回空值
   - 可结合`JSON.stringify(className)`和`JSON.parse(val)`来实现对象的存取

## 组件自定义事件

1. 适用于 **子组件=>父组件**

2. 当A是父，B是子，B给A传递数据，需要在A中给B绑定自定义事件，同时A中包含事件的回调，这样当B调用回调函数时，A才能得到数据

3. 绑定自定义事件

   ```vue
   1. <Demo @事件名="回调函数名"/> 或者 <Demo v-on:事件名="回调函数名"/> 
   2. 
   	<Demo ref="tag_name"/>  //即给子组件一个标记
   	...
   	mounted(){
   		this.$ref.demo.tag_name.$on('事件名', this.回调函数)
   	}
   ```

4. 触发自定义事件

   `this.$emit('事件名', data)`

5. 解绑自定义事件

   `this.$off('事件名')`

6. 组件上绑定原生DOM事件，需要用`native`修饰符，例如`@click.native='回调'`

7. 通过`this.$ref.xxx.$on('事件名', 回调函数)`绑定自定义事件，要么在methods配置好，这里只用`this.函数名`，要么用箭头函数，否则`this`指向的触发事件的子组件而不是父组件本身。

## 全局事件总线

:exclamation: 任意组件间通信

1. 安装全局事件总线:

   ```js
   // main.js
   new Vue({
   	...
   	beforeCreate(){
   		Vue.prototype.$bus = this
   	},
   	...
   })
   ```

2. 使用事件总线，接收数据的组件需要给`$bus`绑定一个事件，并且写好回调函数(可以是箭头函数)

   ```js
   methods(){
   	demo(data){...}
   }
   ...
   mounted(){
   	this.$bus.$on('xxx', this.demo)
   }
   ```

   提供数据，则需要在组件中触发该事件 `this.$bus.$emit('xxx', data)`

3. 最好在`beforeDestroy()`钩子中，解绑当前组件用到的事件

## 消息订阅与发布

库：`pubsub-js` 安装 `npm i pubsub-js`

引入 `import pubsub from pubsub-js`，可以使用于任意组件间通信

- 接收数据

  ```js
      methods: {
          demo(msgName, data){
              console.log('hello消息收到了, ', data)
          }
      },
      mounted() {
          // 订阅hello消息
          this.pubId = pubsub.subscribe('hello', function(msgName, data){
              console.log('有人发布了hello，执行回调', data)
          })
          // 方法二
          // pubsub.subscribe('hello', this.demo)
      },
      beforeDestroy(){
          // 解绑
          pubsub.unsubscribe(this.pubId)
      },
  ```

- 发送数据 `pubsub.publish('xxx', 数据)`

同样要注意解绑。

## `nextTick`

语法：`this.$nextTick(回调函数)`

作用：在下一次DOM更新结束后执行其指定的回调，即一般用在改变数据后，需要基于更新后的DOM进行某些操作时，要在`nextTick`指定的回调函数中执行。

## Vue动画效果

自定义动画帧：

```js
@keyframes anim {
        from{
            transform: translateX(-100%);
        }
        to{
            transform: translateX(0);
        }
    }

// 使用
// eg:
.v-leave-active{
        animation: anim 1s reverse;
 }

```

将需要用到动画的DOM元素通过`transaction`标签包裹：

```css
<!-- appear表示一加载就显示进场动画，name取名 -->
<transition name = 'xxx' appear>
<h1 v-show="isShow">你好啊</h1>
</transition>
```

使用:

```js
.v-enter-active{
    animation: anim 1s;
}

.v-leave-active{
    animation: anim 1s reverse;
}
```

如果取名了，则需要将v修改为对应名xxx。

完整方法：

```js
/* 进入的起点、离开的终点 */
.hello-enter,.hello-leave-to{
    transform: translateX(-100%);
                          }
.hello-enter-active,.hello-leave-active{
    transition: 0.5s linear;
}
/* 进入的终点、离开的起点 */
.hello-enter-to,.hello-leave{
    transform: translateX(0);
}
```

如果需要多个元素进行过渡动画，需要`transaction-group`包裹，并且每个DOM元素需要指定唯一的key值

```css
<transition-group name="hello" appear>
<h1 v-show="!isShow" key="1">你好啊！</h1>
<h1 v-show="isShow" key="2">尚硅谷！</h1>
</transition-group>
```

:exclamation: 可以借用封装好的`css`样式。

