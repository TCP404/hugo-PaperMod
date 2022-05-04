---
title: "VUE-note-day2"
tags: 
  - Vue
categories:
  - [Vue]
  - [Note]
date: 2020-04-09 11:34:38
draft: false
toc: false
images:
math: true
---

Vue 的细节是真的多.

<!--more-->

## v-text

```html
<div id="app">
        <span>{{msg}}</span> 等价于 <span v-text="msg"></span>
</div>

<script type="text/javascript">
    var app = new Vue({
        el: '#app',
        data: {
            msg: 'Hello World !',
        }
    });
</script>
```

 // 显示结果 ==>>

Hello World 等价于 Hello World.

## v-once

> 形式: v-once
>
> 值: 无
>
> 作用: 限定所在元素只被渲染一次, 完成后即使值更新了也不再渲染
>

不管是用 v-text = "msg" 还是用 {{msg}} 都会实时的更新,也就是当msg 的值改变的时候, 显示的结果也会改变.

如果只想渲染一次, 可以加上v-once

```html
<div id="app">
    <span>{{msg}}</span> 不等价于 <span v-text="msg" v-once></span>
</div>
<script type="text/javascript">
    var app = new Vue({
        el: '#app',
        data: {
            msg: 'Hello World !',
        }
    });
    
    app.msg = "Hello"
</script>
```

显示结果 ==>>

Hello World 不等价于 Hello

## v-html

> 形式: v-html = "variable"
>
> 值: variable -> string, 取自于data{} 里的属性
>
> ! WARN:  容易导致[XSS攻击](https://en.wikipedia.org/wiki/Cross-site_scripting). 所以, 只在可信内容上使用. **永不**用在用户提交的内容上。
>
> 作用: 更新元素的innerHTML. 内容按普通的HTML插入, 不会被编译

```html
<div id="app">
    <span v-html="dodo"></span>
</div>

<script type="text/javascript">
    var app = new Vue({
        el: '#app',
        data: {
            dodo: '<h2>Hello</h2>'
        }
    });
</script>

<!-- 渲染结果 =>> -->
<div id="app">
    <span v-html="dodo">
        <h2>Hello</h2>
    </span>
</div>

<!--显示结果 ==>> -->
Hello
```

## v-on

> 形式： v-on:event[.qualifier] = "Function | Inline statements | Object"
>
> 缩写： @event[.qualifier] = "Function | Inline statements | Object"
>
> 值： 
>
> - event -> 要监听的事件， 如click， keyup等
> - qualifier -> 修饰符，监听事件做一些限定
> - methods -> 当所监听的事件触发时的响应方法
> - Inline statements -> 内联语句
> - Object -> [2.4.0]新增，使用键值对对象作为响应事件，但是不支持任何修饰器
>
> 作用： 绑定监听事件。事件类型由event参数指定，
>
> 修饰符：
>
> - `.stop` - 调用 `event.stopPropagation()`。
> - `.prevent` - 调用 `event.preventDefault()`。
> - `.capture` - 添加事件侦听器时使用 capture 模式。
> - `.self` - 只当事件是从侦听器绑定的元素本身触发时才触发回调。
> - `.{keyCode | keyAlias}` - 只当事件是从特定键触发时才触发回调。
> - `.native` - 监听组件根元素的原生事件。
> - `.once` - 只触发一次回调。
> - `.left` - (2.2.0) 只当点击鼠标左键时触发。
> - `.right` - (2.2.0) 只当点击鼠标右键时触发。
> - `.middle` - (2.2.0) 只当点击鼠标中键时触发。
> - `.passive` - (2.3.0) 以 `{ passive: true }` 模式添加侦听器

```html
<!--方法处理器-->
<button v-on:click="Function1"></button>

<!-- 动态事件（2.6.0+） -->
<button v-on:[event]="dosth"></button>
<!-- 内联语句 -->
<button v-on:click="dosth('hello', $event)"></button>
<!-- 缩写 -->
<button @click="dosth"></button>
<!-- 停止冒泡 -->
<button @click.stop="dosth"></button>
<!-- 阻止默认行为 -->
<button @click.prevent="dosth"></button>
<!-- 没有表达式的阻止默认行为 -->
<form @click.prevent></form>

<!-- 串联修饰符 -->
<button @click.stop.prevent="dosth"></button>
<!-- 键修饰符.键名 -->
<button @keyup.enter="onEnter">
<!-- 键修饰符.键码 -->
<button @keyup.13="onEnter">
<!-- 点击回调只触发一次 -->
<button @click.once="dosth"></button>
<!-- 对象语法（2.4.0+） -->
<button v-on="{keyup: dosthA, keydown: dosthB"></button>
```



## v-bind

> 形式： v-bind:AttributeOrProperty[.qualifier] = "value"
>
> 缩写： :AttributeOrProperty="value"
>
> 值：
>
> - AttributeOrProperty -> 标签的原生属性或特性
> - value -> 标签原生属性所对应的值
>
> 作用：将属性或特性与变量绑定在一起，实现动态修改
>
> 修饰符：
>
> - `.prop` - 作为一个 DOM property 绑定而不是作为 attribute 绑定。([差别在哪里？](https://stackoverflow.com/questions/6003819/properties-and-attributes-in-html#answer-6004028))
> - `.camel` - (2.1.0+) 将 kebab-case attribute 名转换为 camelCase。(从 2.1.0 开始支持)
> - `.sync` (2.3.0+) 语法糖，会扩展成一个更新父组件绑定值的 `v-on` 侦听器。

```html
<!-- 绑定一个属性 -->
<img v-bind:src="imageSrc">

<!-- 动态 attribute 名 (2.6.0+) -->
<button v-bind:[key]="value"></button>

<!-- 缩写 -->
<img :src="imageSrc">

<!-- 动态 attribute 名缩写 (2.6.0+) -->
<button :[key]="value"></button>

<!-- 内联字符串拼接 -->
<img :src="'/path/to/images/' + fileName">

<!-- class 绑定 -->
<div :class="{ red: isRed }"></div>
<div :class="[classA, classB]"></div>
<div :class="[classA, { classB: isB, classC: isC }]">

<!-- style 绑定 -->
<div :style="{ fontSize: size + 'px' }"></div>
<div :style="[styleObjectA, styleObjectB]"></div>

<!-- 绑定一个有属性的对象 -->
<div v-bind="{ id: someProp, 'other-attr': otherProp }"></div>

<!-- 通过 prop 修饰符绑定 DOM 属性 -->
<div v-bind:text-content.prop="text"></div>

<!-- prop 绑定。“prop”必须在 my-component 中声明。-->
<my-component :prop="someThing"></my-component>

<!-- 通过 $props 将父组件的 props 一起传给子组件 -->
<child-component v-bind="$props"></child-component>

<!-- XLink -->
<svg><a :xlink:special="foo"></a></svg>
```

## v-model

> 形式： v-model[.qualifier] = "variable"
>
> 值：variable -> 双向数据绑定的变量，通过这个变量实现数据与视图之间的绑定
>
> ! WARN: 只能在 `<input>`、`<select>`、`<textarea>` 和组件上使用
>
> 作用： 实现双向数据绑定
>
> 修饰符：
>
> - [`.lazy`](https://cn.vuejs.org/v2/guide/forms.html#lazy) - 取代 `input` 监听 `change` 事件,  懒加载，会等到失焦才更新
> - [`.number`](https://cn.vuejs.org/v2/guide/forms.html#number) - 输入字符串转为有效的数字，限制只有数字有效， 但是如果先输入字符串则该修饰符无效
> - [`.trim`](https://cn.vuejs.org/v2/guide/forms.html#trim) - 输入首尾空格过滤

```html
<p>{{msg}}</p>
<p>v-model.lazy<input type="text" v-model.lazy="msg"></p>

<script type="text/javascript">
    var app = new Vue({
        el: '#app',
        data: {
            msg: 'Hello World !'
        }
    });
</script>
```

加了lazy修饰符， 所以会等到input输入框失去焦点才渲染更新

