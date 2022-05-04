---
title: "VUE-note-day1"
tags: 
  - Vue
categories:
  - [Vue]
  - [Note]
date: 2020-04-08 12:05:50
draft: false
toc: false
images:
math: true
---

疫情在家真的无聊 T_T，学点Vue吧

<!--more-->


## v-if & v-else-if & v-else

### v-if

> 形式: v-if = "express | variable"
> 
> 值:  true -> 渲染[^1] | false -> 不渲染
> 
> 作用: vue的一个内部指令, 用在html标签中, 作为标签的一个属性. 用来判断是否渲染所在的标签.

其作用相当于C语言中的 `if判断语句`

```vue
<div id="app">
    <div v-if="isLogin">欢迎来到XXX.</div>
</div>
```

```javascript
<script type="text/javascript">
    new Vue({
        el: '#app',
        data: {
            isLogin: true
        }
    });
</script>
```

### v-else-if [2.1.0新增]

> 形式: v-else-if = "express | variable"
> 
> 值: true -> 渲染 | false -> 不渲染
> 
> ! WARN:  必须紧跟在 v-if 或 v-else-if后面, 否则将不被识别.
> 
> 作用: 其作用相当于C语言中的 `else if 判断语句`.

### v-else

> 形式: v-else
> 
> 值: 无
> 
> 作用: 同 v-if 一样, vue的一个内部指令, 用在html标签中.
> 
> ! WARN: 必须紧跟在 v-if 或 v-else-if后面, 否则将不被识别.

### 综合示例

```javascript
<div v-if="type === 'A'">  A  </div>

<div v-else-if="type === 'B'">  B  </div>

<div v-else-if="type === 'C'">  C  </div>

<div v-else>  Not A/B/C  </div>
```

### key 管理可复用元素

> 形式: key = "unique-value"
> 
> 值: 不固定, 只要是全局唯一即可
> 
> 作用: vue为了高效渲染使得加载速度变快, 会复用已有元素,  有时候有的元素虽然相同但是我们不想被复用, 可以在元素中添加key属性来避免被vue复用. 使用了key属性的元素会被重新渲染而不是复用

#### [官网示例]([https://cn.vuejs.org/v2/guide/conditional.html#%E7%94%A8-key-%E7%AE%A1%E7%90%86%E5%8F%AF%E5%A4%8D%E7%94%A8%E7%9A%84%E5%85%83%E7%B4%A0](https://cn.vuejs.org/v2/guide/conditional.html#%E7%94%A8-key-%E7%AE%A1%E7%90%86%E5%8F%AF%E5%A4%8D%E7%94%A8%E7%9A%84%E5%85%83%E7%B4%A0))

```html
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address">
</template>
```

这个例子, 如果input 输入框内有内容, 在切换的时候不会被清空, 因为被vue复用了.

```html
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>
```

而这个例子, 如果`<input>`输入框内有内容, 在切换时会被清空, 因为不会被复用而是重新渲染. 但是 `<label>`依然会被高效复用, 因为没有key属性.

> key属性作用: 避免被复用, 使之重新渲染

### v-show

> 形式: v-show = "express | variable"
> 
> 值: true -> 显示[^4] | false -> 不显示
> 
> 作用: vue的一个内部指令, 用在html标签中,  用于判断所在标签是否显示, 而不是是否渲染

```html
<div id="app">
    <h1 v-show="right">Hello!</h1>  <!--当right为true时, 该标签会被显示, 为false时不显示-->
    <h2 v-show="status === 1"> 当status为1时显示 </h2>
</div>
<script type="text/javascript">
    new Vue({
        el: '#app',
        data:{
            right: true,
            status: 2
        }
    });
</script>

渲染后的结果:
<h1 style="display:block;">Hello!</h1>
<h2 style="display:none;"> 当status为1时显示 </h2>
```

### v-if   VS   v-show

渲染:

- `v-if` 是**“真正”的条件渲染**，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。

- `v-show`只是简单地基于 **CSS 进行切换**。

渲染时机:

- `v-if` 也是**惰性的**：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

- `v-show` 就简单得多——不管初始条件是什么，元素总是会被渲染，

开销:

- `v-if` 有更高的切换开销，而 `v-show` 有更高的初始渲染开销。

因此:

- 如果需要非常**频繁地切换**，则使用 `v-show` 较好；

- 如果在运行时条件**很少改变**，则使用 `v-if` 较好。



## v-for

### 数组作为数据源

> 形式: 渲染一个数组
>
> 1. v-for = "alias in source" 或 v-for = "alias of source"
> 2. v-for = "(alias, index) in source"
>
> 值: 
>
> - source -> 数据源, 是一个数组
> - alias -> 别名, 遍历时的临时变量
> - index -> 索引(下标), 默认的, 可以取别的名, 但是约定俗成是 index
>
> ! WARN: 
>
> - 别名和索引的位置不能换. 即使换了解释器也默认按照 `第一个参数是临时变量, 第二个参数是索引, in 后面的参数是数据源` 的规则来解释
> - 哪个元素要被循环渲染就写在哪个元素里作为它的属性.
>
> 作用: 将数组里的每一个值渲染到标签的插值[^7]中. 

```html
<div id="app">
    <ul>
        <li v-for="age in DemoArray">{{age}}</li>
    </ul>
</div>

<script type="text/javascript">
	new Vue({
        el: '#app',
        data: {
            DemoArray: [20, 30, 44, 10, 33]
        }
    });
</script>

// 渲染结果=>>
<div id="app">
    <ul>
        <li>20</li>
        <li>30</li>
        <li>44</li>
        <li>10</li>
        <li>33</li>
    </ul>
</div>
```

> // 显示结果 ==>>
>
> - 20
> - 30
> - 44
> - 10
> - 33

#### 排序

> 计算的工作都在computed里完成

排序实现在vue对象中的computed, 但是computed里的键名不能和data里的键名相同. 而我们要按排序后的数组渲染, 所以 html 里要改成 v-for = "age in sortArray"



```html
<div id="app">
    <ul>
        <li v-for="age in DemoArray">{{age}}</li>
    </ul>
</div>

<script type="text/javascript">
	...
</script>

```

```html
<script type="text/javascript">
	new Vue({
   	 	el: '#app',
    	data: {
        	DemoArray: [20, 30, 44, 10, 33]
    	},
    	computed: {
            /** 错误写法. 键名重复
            DemoArray: function(){
            	return this.DemoArray.sort((a, b) => a - b);
        	}
        	*/
        	sortItems: function(){
            	return this.DemoArray.sort((a, b) => a - b);
        	}
    	}
	});
</script>
```

因为 `javascript` 自带的 bug, 对数组排序 DemoArray.sort()是把每个数组元素的最前面的一位[^5], 所以排序出来是有问题的.

导致这个 bug 的原因我猜想是因为 js 是弱类型语言, 解释器也不知道你这个数组里到底是字符串还是数字还是什么, 又得给你排序, 所以干脆统统按 `给字符串排序` 的方法处理.

> 修复方法就是自己实现一个函数. 上述代码中用了[箭头函数](https://www.liaoxuefeng.com/wiki/1022910821149312/1031549578462080)使得更加简洁. 关键代码即 `(a, b) => a - b`

#### 数组更新

##### 会修改原数组的方法: 变异方法

- push()
- pop()
- shift()
- unshift()
- splice()
- sort()
- reverse()

通过这些方法修改数组, **会触发视图的更新**. 

##### 不会修改原数组的方法: 非变异方法

- filter()
- concat()
- slice()

它们不会改变原始数组，而**返回一个新数组**。

当使用非变异方法时，可以用新数组替换旧数组：

```js
var app = new Vue ({
    el: '#app',
    data:{
        items: [
            {msg: "m1"},
            {msg: "m2"}
        ]
    }
});
// 替换              
app.items = app.items.filter( item => item.items.match(/mmm/) ) ;
```

> ? 这里不清楚怎么实现的, 但是[官网]([https://cn.vuejs.org/v2/guide/list.html#%E6%9B%BF%E6%8D%A2%E6%95%B0%E7%BB%84](https://cn.vuejs.org/v2/guide/list.html#替换数组))的说法是: 并非丢弃现有DOM而重新渲染整个列表, 因为Vue里有些智能的方法, 所以, 数组在改动不大的情况下去替换原有数组的非常高效的, 不必担心.
>
> 至于怎么个智能法没说, 有待深挖. 

#### 数组更新的注意事项

##### 问题: 

由于不靠谱的JavaScript的限制, 以下两种方法的更新Vue是无法检测到的

1. 利用索引直接设置一个数组项时，例如：`vm.items[indexOfItem] = newValue`
2. 修改数组的长度时，例如：`vm.items.length = newLength`

```js
var app = new Vue({
    ...
    data: {
        array: [0,1,2,3,4,5,6,7,8]
    }
});
app.array[3] = 10		//Vue 检测不到更新, 也不会触发视图更新
app.array.length = 8	//Vue 检测不到更新				
```

##### 解决: 问题1

```js
Vue.set(vm.items, indexOfItem, newValue), 
// 例如: Vue.set(app.array, 2, 10)
// ==>> [0,1,10,3,4,5,6,7,8]
或
vm.items.splice(indexOfItem, 1, newValue), 
// 例如: app.array.splice(3, 1, 50)
// ==>> [0,1,2,50,4,5,6,7,8]
    
// 关于这个1,可以是任何数, 
// 0则不吃掉任何元素, 
// 1则吃掉array[indexOfItem]那个元素,
// 2则吃掉array[indexOfItem] 和 array[indexOfItem + 1]两个元素
// 以此类推

// 例如: app.array.splice(2,2,30)
// ==>> [0,1,30,5,6,7,8]
    
或
vm.$set(vm.items, indexOfItem, newValue)
// 例如: app.$set(app.array, 2, 10)
// ==>> [0,1,10,3,4,5,6,7,8]
```



##### 解决: 问题2

```js
app.array.splice(新长度值)
```



### 对象作为数据源

> 形式: 渲染一个对象
>
> 1. v-for = "value in object"
> 2. v-for = "(value, key) in object"
> 3. v-for = "(value, key, index) in object"
>
> 值:
>
> - object -> 数据源, 是一个对象, 要遍历的是它的所有属性
> - value -> 别名, 遍历时的临时变量, 输出的是每一个属性的值
> - key -> 索引(键名), 默认的, 可以取别的名, 但是约定俗成是 key, 输出的是每一个属性(键值对)的键名
> - index -> 索引(下标), 默认的, 可以取别的名, 约定俗成是index, 输出的是每一个属性的下标[^6].
>
> ! WARN:
>
> - 和遍历数组一样, 键名索引和下标索引的位置不要换. 即使换了解释器也是按照 `第一个参数是临时变量,第二个参数是键名索引, 第三个参数是下标索引, in 后面是数据源`的规则解释.
> - 哪个元素要被循环渲染就写在哪个元素里作为它的属性.
>
> 作用: 将对象里的每一个属性(键值对)的值渲染到标签的插值[^7]中.
>



```html
<div id="app">
    <ul>
        <li v-for="(val, key, idx) in obj">{{idx}}-{{key}}-{{val}}</li>
    </ul>
</div>

<script type="text/javascript">
	new Vue({
      el: '#app',
      data: {
        obj:{
      		  prop1: 'key1',
      		  prop2: 20,
      		  prop3: true
         }
      }
    });
</script>

// 渲染结果 =>>
<div id="app">
    <ul>
        <li>0-prop1-key1</li>
        <li>1-prop2-20</li>
        <li>2-prop3-true</li>
    </ul>
</div>
```



> // 显示结果 ==>>
>
> - 0-prop1-key1
> - 1-prop2-20
> - 2-prop3-true

#### 对象更新的注意事项

##### 问题: 

由于不靠谱的JavaScript的限制, 对象属性的添加或删除 Vue是检测不到的

```js
var app = new Vue({
    el: '#app',
    data: {
        a: 1
    }
});

app.b = 2 	// Vue检测不到, 不会更新视图
```

##### 解决方法

app.a   app.b    这里a和b叫做根级别响应式属性, 是不允许动态添加的, 但是根级别属性的属性是可以动态添加的.

```js
var app = new Vue({
    el: '#app',
    data: {
        root:{ child: 1 }
    }
});

// app.root 不允许动态添加, app.root.child允许动态添加
```

添加单个属性的方法

```js
Vue.set(rootAttribute, childKey, childValue)
// 例如: Vue.set(app.root, 'age', 10)
// ==>> data: {
//			root: { child: 1, age: 10}
//		}
或
vm.$set(rootAttribute, childKey, childValue)
// 例如: app.$set(app.root, 'age', 10)
```

添加多个属性的方法: 使用 `Object.assign()` 或 `_.extend()`

```js
app.root = Object.assign({}, app.root, { age: 27, favoriteColor: 'Vue Green' })
```











### 数组对象作为数据源的排序

> 计算处理同样是在computed中.
>
> 重新定义一个函数, 并作出处理
>
> 在v-for调用时 把数据源换成刚刚定义的函数名

```js
 computed: {
 	...
 	//数组对象方法排序:
 	sortStudents: function () {
 	    var key = "age";
 	    return this.students.sort((a, b) => a[key] < b[key] ? -1 : (a[key] > b[key] ? 1 : 0))
    }
 }
```



### 范围作为数据源

形式: 渲染一个对象

1. v-for = "value in object"
2. v-for = "(value, key) in object"
3. v-for = "(value, key, index) in object"

值:

- object -> 数据源, 是一个对象, 要遍历的是它的所有属性
- value -> 别名, 遍历时的临时变量, 输出的是每一个属性的值
- key -> 索引(键名), 默认的, 可以取别的名, 但是约定俗成是 key, 输出的是每一个属性(键值对)的键名
- index -> 索引(下标), 默认的, 可以取别的名, 约定俗成是index, 输出的是每一个属性的下标[^6].

! WARN:

- 和遍历数组一样, 键名索引和下标索引的位置不要换. 即使换了解释器也是按照 `第一个参数是临时变量,第二个参数是键名索引, 第三个参数是下标索引, in 后面是数据源`的规则解释.
- 哪个元素要被循环渲染就写在哪个元素里作为它的属性.

作用: 将对象里的每一个属性(键值对)的值渲染到标签的插值[^7]中.

> 形式:  v-for=" n in range"
>
> 值: 
>
> - n -> 别名, 遍历时的临时变量
> - range -> 数据源, 遍历的范围

```html
<div>
  <span v-for="n in 10">{{ n }} </span>
</div>
```

显示结果 ==>>

1 2 3 4 5 6 7 8 9 10

### v-for 和 v-if 一起用时

尽量不要把v-for 和 v-if 放在同一个标签里

> 但他们处在同一个标签内时, v-for 的优先级比 v-if 高

```html
<!--尽量不要-->
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo }}
</li>
```

```html
<!--上面是特殊情况下的写法,  下面是正常情况下的规范写法-->
<ul v-if="todos.length">
  <li v-for="todo in todos">
    {{ todo }}
  </li>
</ul>
<p v-else>No todos left!</p>
```







[^1]:渲染, 加载完DOM树之后就开始在页面上加载, 这个过程叫做渲染
[^2]:加载, 当浏览器接收到服务器返回的html文件时, 会读取所有html标签形成一颗DOM树[^3]
[^3]:DOM树, 全部html标签的树状结构
[^4]:display: block
[^5]:如果是字符串就,第一位就是第一个字符; 如果是数字,第一位就是最大位的那个数字, 比如39的第一位是3
[^6]:对象中的属性都是键值对, 属性的下标从0开始, 先定义的属性(键值对)下标就靠前
[^7]:插值, HTML标签中间用 `{{ 插值 }}` 包起来的地方

