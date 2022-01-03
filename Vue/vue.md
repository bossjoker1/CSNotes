# Vue学习笔记（一）

1. 创建Vue实例

   ```js
   // 创建Vue实例
   const v = new Vue({
           el: '#root', // el指定当前Vue实例为哪个容器服务，值通常为css选择器字符串
           /*data: {// 对象式
               name: '张三',
               addr: '北京'
           }*/
       	data:function(){ // 可以直接写成 data()
               console.log(this)
               return{ // 函数式
                   name: '张三'
               }
           }
       })
   v.$mount('#root') // 与el等效，挂载
   ```

   容器和实例一对一

   由Vue管理的函数不要写成`lambda`表达式，这样`this`指针会指向`Window`对象

2. 插值语法 ：  `{{xxx}}`  在标签体内用，xxx为JS表达式

3. 指令语法 ：

   - `v-bind:` 动态绑定值 可简写为 `:` ， 管理标签属性，单向绑定
   - `v-model:` 双向绑定，只能用于表单元素(或者说含value属性) `v-model:value`可以简写为`v-model`

4. `MVVM`模型：即

   - `Model` 对应data中的数据, `plaine JS objects` 
   - `View` DOM
   - `ViewModel` 对应Vue实例对象，所以用`vm`命名Vue实例 

5. `Object.defineProperty(对象, 属性名, {value: xxx,})`  默认`enumerable`属性为`false`，括号里面可以对属性性质进行修改，如`writable`, `configurable` 

   ```js
    Object.defineProperty(person, 'age', {
           // value:18, 
           // writable:true,
           // 当读取age属性时，get函数(getter)会被调用，且返回值赋给age
           // 定义get或者set后就不能有value属性
           get:function(){
               return num
           },
        	// 简写, 修改age属性时，set函数(setter)会收到，且会收到修改的具体值
        	set(value){
          		console.log("修改！")
                num = value
           }
       });
   ```

6. 数据代理：通过一个对象代理对另一个对象中属性的操作，通过5中的函数实现。

   在(1)的例子中`data`对象赋值给`vm`变量内部的`_data`，并为之里面的属性添加`getter/setter`函数，然后进行操作并实现数据代理。

7. 事件处理：`v-on:` 简写为`@`  ，`$xxx `为关键词占位符，事件回调写到`method`中，且最终会在`vm`上，直接成为其方法，而不会进行数据代理

   ```js
   <div id="root">
       <h1>欢迎来到{{name}}学习</h1>
       <button v-on:click="showInfo">点我提示信息</button>
       <button @click="showInfo($event, 66)">点我提示信息</button>
   </div>
   
   <script type="text/javascript">
       Vue.config.productionTip = false 
       new Vue({
           el:"#root",
           data:{
               name:"google"
           },
           methods:{
               showInfo(event, num){
                   console.log(event, num)
               }
           }
       })
   </script>
   ```

8. 事件修饰符：可以连写

   - `prevent` 阻止默认事件  `event.preventDefault()`
   - `stop` 阻止事件在DOM树间冒泡  `event.stopPropagation()`
   - `once` 事件只触发一次
   - `capture` 使用事件的捕获模式
   - `self` 只有`event.target`是当前操作元素才触发事件
   - `passive` 事件的默认行为立即执行，无需等待事件回调执行完毕

9. 键盘事件：可以连写如` @keyup.ctrl.y=func` 同时按下`ctrl`和`y`触发事件

   Vue常用的按键别名

   - 回车 -> `enter`
   - 删除 -> `delete`
   - 退出 -> `esc`
   - 空格 -> `space`
   - 换行 -> `tab` 会切走焦点`focus`，所以一般配合`keydown`使用
   - 上    -> `up`
   - 下    -> `down`
   - 左    -> `left`
   - 右    -> `right`
   - 大小写键 -> `caps-lock` 注意非单个单词的写法
   - 系统修饰键(特殊)：`ctrl / alt / meta` 键
     1.  当配合`keyup`事件时，需要再按下其他键，然后释放该键后才会触发
     2. `keydown`， 正常触发
   - `Vue.config.keyCodes.自定义键名 = 键码` 自定义按键别名

10. 计算属性`computed`：通过已有的属性计算而来，定义属性的`get() / set()`方法，底层实现与`Object.defineProperty()`一致，该属性计算后直接写入到Vue实例中(不是对象了，而是get后的数据直接写入)。

    `get()`函数：1. 初次读取时调用(然后缓存) 2. 依赖的数据发生变化时调用

    **简写**：该属性只读时(即之定义`get`)

    ```js
    computed:{
                fullName:{
                   get(){
                         return this.firstname + '-' + this.lastname                
                    },
                }
            }
    // 简写
    fullName(){
         return this.firstname + '-' + this.lastname  
    }
    ```

11. 监视属性：`watch`，`Vue`中的`watch`默认不检测对象内部值的改变，其自身可以检测多层结构的数据改变。配置`deep:true`开启`watch`检测对象内部值改变。可以方便的开启**异步任务**

    ```js
     watch:{
            isHot:{
                immediate:true, // 初始化时就调用handler
                // 当该属性发生改变时调用
                handler(newValue, oldValue){
                    console.log(oldValue, "被修改成", newValue)
                }
            }
         
         	// 简写，只需handler函数，不需要其它配置的场景
         	isHot(){
                //codehere
            }
        }
    // 第二种写法
     vm.$watch('info',{
            handler(newValue, oldValue){
                console.log(oldValue, "被修改成", newValue)
            }
        })
    vm.$watch('info', function(){   })
    // 监视多级结构的属性
    numbers:{
        deep:true, // 开启
        handler(){
            console.log('numbers 发生了改变')
        }                
    }
    ```

12. **注意点**：

    - 所有被Vue管理的函数尽量写成普通函数，此时`this`指针指向的是Vue实例对象
    - 不被Vue管理的函数如`setTimeout()`要写成箭头函数，因为普通函数`this`指针指向的为`Window`对象。
