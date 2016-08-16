---
title: 组件
type: guide
order: 11
---

## 什么是组件？

组件（Component）是 Vue.js 最强大的功能之一。组件可以扩展 HTML 元素，封装可重用的代码。在较高层面上，组件是自定义元素，Vue.js 的编译器为它添加特殊功能。在有些情况下，组件也可以是原生 HTML 元素的形式，以 `is` 特性扩展。

## 使用组件

### 注册

之前说过，我们可以用 像这样创建一个Vue实例：

``` js
new Vue({
  el: '#some-element',
  // 选项...
})
```

为了注册全局组件，可以用`Vue.component(tagName, options)` **注册** 。

示例：

``` js
Vue.component('my-component', {
  // 选项...
})
```

<p class="tip">对于自定义标签名字，Vue.js 不强制要求遵循 [W3C 规则](http://www.w3.org/TR/custom-elements/#concepts)（小写，并且包含一个短杠），尽管遵循这个规则比较好。</p>

组件在注册之后，便可以在父实例的模块中以自定义元素 `<my-component></my-component>` 的形式使用。要确保在初始化根实例**之前**注册了组件。

示例：

``` html
<div id="example">
  <my-component></my-component>
</div>
```

``` js
// 注册
Vue.component('my-component', {
  template: '<div>A custom component!</div>'
})

// 创建根实例
new Vue({
  el: '#example'
})
```

渲染为：

``` html
<div id="example">
  <div>A custom component!</div>
</div>
```

{% raw %}
<div id="example" class="demo">
  <my-component></my-component>
</div>
<script>
Vue.component('my-component', {
  template: '<div>A custom component!</div>'
})
new Vue({ el: '#example' })
</script>
{% endraw %}

### 局部注册

不需要全局注册每个组件。可以让组件只能用在其它组件内，用实例选项 `components` 注册：

``` js
var Child = {
  template: '<div>A custom component!</div>'
}

new Vue({
  // ...
  components: {
    // <my-component> 只能用在父组件模板内
    'my-component': Child
  }
})
```

这种封装也适用于可注册的Vue特性，如指令。

### 组件选项问题

传入 Vue 构造器的多数选项可以用在组件中，不过有两个特例： `data` 和 `el`必须是函数。事实上，你可以尝试这样:

``` js
Vue.component('my-component', {
  template: '<span>{{ message }}</span>',
  data: {
    message: 'hello'
  }
})
```

Vue将在控制台显示警告，告诉你`data`必须是组件实例的一个函数。虽然很显然存在这个规则，不过我们可以绕过它。

``` html
<div id="example-2">
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
</div>
```

``` js
var data = { counter: 0 }

Vue.component('simple-counter', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  // data在技术上是个函数，所以vue不报错,但我返回一样的对象给每一个组件实例
  data: function () {
    return data
  }
})

new Vue({
  el: '#example-2'
})
```

{% raw %}
<div id="example-2" class="demo">
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
</div>
<script>
var data = { counter: 0 }
Vue.component('simple-counter', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  data: function () {
    return data
  }
})
new Vue({
  el: '#example-2'
})
</script>
{% endraw %}

所有的实例将共享同一个 `data` 对象！这基本不是我们想要的，因此我们应当使用一个函数作为 `data` 选项，让这个函数返回一个新对象：

``` js
data: function () {
  return {
    counter: 0
  }
}
```

现在所有的计数器都有自己内部状态。

{% raw %}
<div id="example-2-5" class="demo">
  <my-component></my-component>
  <my-component></my-component>
  <my-component></my-component>
</div>
<script>
Vue.component('my-component', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  }
})
new Vue({
  el: '#example-2-5'
})
</script>
{% endraw %}


同理，`el` 选项用在组件实例中时也须是一个函数。

### 模板解析问题

Vue 的模板是 DOM 模板，使用浏览器原生的解析器而不是自己实现一个。相比字符串模板，DOM 模板有一些好处，但是也有问题，它必须是有效的 HTML 片段。一些 HTML 元素对什么元素可以放在它里面有限制。
只要你使用字符串模板 ([`<script type="text/x-template">`](#X-Templates),内联模板字符串,或[`.vue` components](single-file-components.html)),可以不受许多固有HTML元素的限制，不过用`el`在已存在内容插入模板时有一些常见的限制:

- `a` 不能包含其它的交互元素（如按钮，链接）
- `ul` 和 `ol` 只能直接包含 `li`
- `select` 只能包含 `option` 和 `optgroup`
- `table` 只能直接包含 `thead`, `tbody`, `tfoot`, `tr`, `caption`, `col`, `colgroup`
- `tr` 只能直接包含 `th` 和 `td`


在实际中，这些限制会导致意外的结果。尽管在简单的情况下它可能可以工作，但是你不能依赖自定义组件在浏览器验证之前的展开结果。例如 `<my-select><option>...</option></my-select>` 不是有效的模板，即使 `my-select` 组件最终展开为 `<select>...</select>`。

另一个结果是，自定义标签（包括自定义元素和特殊标签，如 `<component>`、`<template>`、 `<partial>` ）不能用在 `ul`, `select`, `table` 等对内部元素有限制的标签内。放在这些元素内部的自定义标签将被提到元素的外面，因而渲染不正确。

对于自定义元素，应当使用 `is` 特性：

``` html
<table>
  <tr is="my-component"></tr>
</table>
```

`<template>` 不能用在 `<table>` 内，这时应使用 `<tbody>`，`<table>` 可以有多个 `<tbody>`：

``` html
<table>
  <tbody v-for="item in items">
    <tr>Even row</tr>
    <tr>Odd row</tr>
  </tbody>
</table>
```

重申一遍，这些限制一定不能用在字符串模板中:

- `<script type="text/x-template">`
- 内联模板字符串
- `.vue` 组件

## Props

### 用Props 传递数据

**组件实例的作用域是孤立的**。这意味着不能并且不应该在子组件的模板内直接引用父组件的数据。可以使用 **props** 把数据传给子组件。

"prop" 是组件数据的一个字段，期望从父组件传下来。子组件需要显式地用 [`props` 选项](/api/#props) 声明 props：

``` js
Vue.component('child', {
  // 声明 props
  props: ['message'],
  // prop 可以用在模板内
  // 可以用 `this.message` 设置
  template: '<span>{{ message }}</span>'
})
```

然后向它传入一个普通字符串：

``` html
<child message="hello!"></child>
```

结果:

{% raw %}
<div id="prop-example-1" class="demo">
  <child message="hello!"></child>
</div>
<script>
new Vue({
  el: '#prop-example-1',
  components: {
    child: {
      props: ['message'],
      template: '<span>{{ message }}</span>'
    }
  }
})
</script>
{% endraw %}

### camelCase vs. kebab-case

HTML 特性不区分大小写。名字形式为 camelCase 的 prop 用作特性时，需要转为 kebab-case（短横线隔开）：

``` js
Vue.component('child', {
  // camelCase in JavaScript
  props: ['myMessage'],
  template: '<span>{{ myMessage }}</span>'
})
```

``` html
<!-- kebab-case in HTML -->
<child my-message="hello!"></child>
```

如果用字符串模板，这个限制不适用。

### 动态 Props

类似于用 `v-bind` 绑定 HTML 特性到一个表达式，也可以用 `v-bind` 绑定动态 Props 到父组件的数据。每当父组件的数据变化时，也会传导给子组件：

``` html
<div>
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>
```

使用 `v-bind` 的缩写语法通常更简单：

``` html
<child :my-message="parentMsg"></child>
```

结果:

{% raw %}
<div id="demo-2" class="demo">
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>
<script>
new Vue({
  el: '#demo-2',
  data: {
    parentMsg: 'Message from parent'
  },
  components: {
    child: {
      props: ['myMessage'],
      template: '<span>{{myMessage}}</span>'
    }
  }
})
</script>
{% endraw %}

### 字面量语法 vs. 动态语法

初学者常犯的一个错误是使用字面量语法传递数值：

``` html
<!-- 传递了一个字符串 "1" -->
<comp some-prop="1"></comp>
```

因为它是一个字面 prop，它的值以字符串 `"1"` 而不是以实际的数字传下去。如果想传递一个实际的 JavaScript 数字，需要使用动态语法，从而让它的值被当作 JavaScript 表达式计算：

``` html
<!-- 传递实际的数字  -->
<comp v-bind:some-prop="1"></comp>
```

### One-Way Data Flow

All props form a **one-way-down** binding between the child property and the parent one: when the parent property updates, it will flow down to the child, but not the other way around. This prevents child components from accidentally mutating the parent's state, which can make your app's data flow harder to reason about.

This means you should never mutate props. If you do, Vue will slap your hand with a warning in the console. If you feel the need to mutate a prop, you can instead use it in a computed property or to define the initial value for a data property.

<p class="tip">Note that objects and arrays in JavaScript are passed by reference, so if the prop is an array or object, mutating the object or array itself inside the child **will** affect parent state.</p>

Here's an example:

### 单向数据流

所有的props

``` html
<div id="parent-object-mutation-demo">
  <p>Parent: {{ message.text }}</p>
  <p>Child: <my-component v-bind:parent-message="message"></my-component></p>
</div>
```

``` js
Vue.component('my-component', {
  template: '<input v-model="message.text">',
  props: ['parentMessage'],
  data: function () {
    return {
      message: this.parentMessage
    }
  }
})

new Vue({
  el: '#parent-object-mutation-demo',
  data: {
    message: {
      text: 'hello'
    }
  }
})
```

{% raw %}
<div id="parent-object-mutation-demo" class="demo">
  <p>Parent: {{ message.text }}</p>
  <p>Child: <my-component v-bind:parent-message="message"></my-component></p>
</div>
<script>
Vue.component('my-component', {
  template: '<input v-model="message.text">',
  props: ['parentMessage'],
  data: function () {
    return {
      message: this.parentMessage
    }
  }
})
new Vue({
  el: '#parent-object-mutation-demo',
  data: {
    message: {
      text: 'hello'
    }
  }
})
</script>
{% endraw %}

To fix this, you can clone the object with:

``` js
data: function () {
  return {
    message: JSON.parse(JSON.stringify(this.parentMessage))
  }
}
```

Or if you're using Babel with ES2015, the much simpler:

``` js
data () {
  return {
    message: { ...this.parentMessage }
  }
}
```

### Prop Validation

It is possible for a component to specify requirements for the props it is receiving. If a requirement is not meant, Vue will emit warnings. This is especially useful when you are authoring a component that is intended to be used by others.

Instead of defining the props as an array of strings, you can use an object with validation requirements:

``` js
Vue.component('example', {
  props: {
    // basic type check (`null` means accept any type)
    propA: Number,
    // multiple possible types
    propB: [String, Number],
    // a required string
    propC: {
      type: String,
      required: true
    },
    // a number with default value
    propD: {
      type: Number,
      default: 100
    },
    // object/array defaults should be returned from a
    // factory function
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    },
    // custom validator function
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```

The `type` can be one of the following native constructors:

`type` 可以是下面原生构造器：

- String
- Number
- Boolean
- Function
- Object
- Array

In addition, `type` can also be a custom constructor function and the assertion will be made with an `instanceof` check.

When a prop validation fails, Vue will refuse to set the value on the child component and throw a warning if using the development build.

`type` 也可以是一个自定义构造器，使用 `instanceof` 检测。

当 prop 验证失败了，Vue 将拒绝在子组件上设置此值，如果使用的是开发版本会抛出一条警告。

## Parent-Child Communication

### Parent Chain

A child component holds access to its parent component as `this.$parent`. A root Vue instance will be available to all of its descendants as `this.$root`. Each parent component has an array, `this.$children`, which contains all its child components.

<p class="tip">These properties are made available as an escape hatch to accomodate extreme edge cases. They are not the correct way to access and mutate components in the vast majority of circumstances and if abused, can make your components much more difficult to reason about.</p>

Instead, prefer passing data down explicitly using props. Where data must be shared and mutated by multiple components, a parent component can be used to manage state in a single location. To mutate parent state, custom events can be emitted that parents may choose to listen to or mutation methods can be passed to child components.

For managing state in more complex applications, [vuex](https://github.com/vuejs/vuex/) is recommended as an officially supported companion library.

## 父子组件通信

### 父链

子组件可以用 `this.$parent` 访问它的父组件。根实例的后代可以用 `this.$root` 访问它。父组件有一个数组 `this.$children`，包含它所有的子元素。

尽管可以访问父链上任意的实例，不过子组件应当避免直接依赖父组件的数据，尽量显式地使用 props 传递数据。另外，在子组件中修改父组件的状态是非常糟糕的做法，因为：

1. 这让父组件与子组件紧密地耦合；

2. 只看父组件，很难理解父组件的状态。因为它可能被任意子组件修改！理想情况下，只有组件自己能修改它的状态。


### Custom Events

You can use a pattern similar to the EventEmitter in Node.js: a centralized event hub that allows components to communicate, no matter where they are in the component tree. Because Vue instances implement the event emitter interface, you can actually use an empty Vue instance for this purpose:


### 自定义事件

Vue 实例实现了一个自定义事件接口，用于在组件树中通信。这个事件系统独立于原生 DOM 事件，用法也不同。

``` js
var bus = new Vue()
```
``` js
// in component A's method
bus.$emit('id-selected', 1)
```
``` js
// in component B's created hook
bus.$on('id-selected', function (id) {
  // ...
})
```

As you can see, the event system can:

- Listen to events using `$on()`
- Trigger events using `$emit()`

The example above looks pretty simple, but when looking at component B's code, it's not so obvious where the `"id-selected"` event comes from. That's why when possible, it's better to use custom events for explicit communication between parent and children using `v-on`.

Here's a simple example:

每个 Vue 实例都是一个事件触发器：

- 使用 `$on()` 监听事件；

- 使用 `$emit()` 在它上面触发事件；

- 使用 `$dispatch()` 派发事件，事件沿着父链冒泡；

- 使用 `$broadcast()` 广播事件，事件向下传导给所有的后代。

<p class="tip">不同于 DOM 事件，Vue 事件在冒泡过程中第一次触发回调之后自动停止冒泡，除非回调明确返回 `true`。</p>

简单例子：

``` html
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```

``` js
Vue.component('button-counter', {
  template: '<button v-on:click="increment">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    increment: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})

new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```

{% raw %}
<div id="counter-event-example" class="demo">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
<script>
Vue.component('button-counter', {
  template: '<button v-on:click="increment">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    increment: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
</script>
{% endraw %}

In this example, it's important to note that the child component is still completely decoupled from what happens outside of it. All it does is report information about its own activity, just in case a parent component might care.

#### Binding Native Events to Components

There may be times when you want to listen for a native event on the root element of a component. In these cases, you can use the `.native` modifier for `v-on`. For example:

``` html
<my-component v-on:click.native="doTheThing"></my-component>
```

### Form Input Components using Custom Events

This strategy can also be used to create custom form inputs that work with `v-model`. Remember:

``` html
<input v-model="something">
```

is just syntactic sugar for:

``` html
<input v-bind:value="something" v-on:input="something = $event.target.value">
```

When used with a component, this simplifies to:

``` html
<input v-bind:value="something" v-on:input="something = arguments[0]">
```

So for a component to work with `v-model`, it must:

- accept a `value` prop
- emit an `input` event with the new value

Let's see it in action:

``` html
<div id="v-model-example">
  <p>{{ message }}</p>
  <my-input
    label="Message"
    v-model="message"
  ></my-input>
</div>
````

``` js
Vue.component('my-input', {
  template: '\
    <div class="form-group">\
      <label v-bind:for="randomId">{{ label }}:</label>\
      <input v-bind:id="randomId" v-bind:value="value" v-on:input="onInput">\
    </div>\
  ',
  props: ['value', 'label'],
  data: function () {
    return {
      randomId: 'input-' + Math.random()
    }
  },
  methods: {
    onInput: function (event) {
      this.$emit('input', event.target.value)
    }
  },
})

new Vue({
  el: '#v-model-example',
  data: {
    message: 'hello'
  }
})
```

{% raw %}
<div id="v-model-example" class="demo">
  <p>{{ message }}</p>
  <my-input
    label="Message"
    v-model="message"
  ></my-input>
</div>
<script>
Vue.component('my-input', {
  template: '\
    <div class="form-group">\
      <label v-bind:for="randomId">{{ label }}:</label>\
      <input v-bind:id="randomId" v-bind:value="value" v-on:input="onInput">\
    </div>\
  ',
  props: ['value', 'label'],
  data: function () {
    return {
      randomId: 'input-' + Math.random()
    }
  },
  methods: {
    onInput: function (event) {
      this.$emit('input', event.target.value)
    }
  },
})
new Vue({
  el: '#v-model-example',
  data: {
    message: 'hello'
  }
})
</script>
{% endraw %}

This interface can be used not only to connect with form inputs inside a component, but also to easily integrate input types that you invent yourself. Imagine these possibilities:

``` html
<voice-recognizer v-model="question"></voice-recognizer>
<webcam-gesture-reader v-model="gesture"></webcam-gesture-reader>
<webcam-retinal-scanner v-model="retinalImage"></webcam-retinal-scanner>
```

### Child Component Reference

Despite the existence of props and events, sometimes you might still need to directly access a child component in JavaScript. To achieve this you have to assign a reference ID to the child component using `ref`. For example:

``` html
<div id="parent">
  <user-profile ref="profile"></user-profile>
</div>
```

``` js
var parent = new Vue({ el: '#parent' })
// access child component instance
var child = parent.$refs.profile
```

When `ref` is used together with `v-for`, the ref you get will be an array or an object containing the child components mirroring the data source.

## Content Distribution with Slots

When using components, it is often desired to compose them like this:

``` html
<app>
  <app-header></app-header>
  <app-footer></app-footer>
</app>
```

There are two things to note here:

1. The `<app>` component does not know what content may be present inside its mount target. It is decided by whatever parent component that is using `<app>`.

2. The `<app>` component very likely has its own template.

To make the composition work, we need a way to interweave the parent "content" and the component's own template. This is a process called **content distribution** (or "transclusion" if you are familiar with Angular). Vue.js implements a content distribution API that is modeled after the current [Web Components spec draft](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Slots-Proposal.md), using the special `<slot>` element to serve as distribution outlets for the original content.

### Compilation Scope

Before we dig into the API, let's first clarify which scope the contents are compiled in. Imagine a template like this:

``` html
<child-component>
  {{ message }}
</child-component>
```

Should the `message` be bound to the parent's data or the child data? The answer is the parent. A simple rule of thumb for component scope is:

> Everything in the parent template is compiled in parent scope; everything in the child template is compiled in child scope.

A common mistake is trying to bind a directive to a child property/method in the parent template:

``` html
<!-- does NOT work -->
<child-component v-show="someChildProperty"></child-component>
```

Assuming `someChildProperty` is a property on the child component, the example above would not work. The parent's template is not aware of the state of a child component.

If you need to bind child-scope directives on a component root node, you should do so in the child component's own template:

``` js
Vue.component('child-component', {
  // this does work, because we are in the right scope
  template: '<div v-show="someChildProperty">Child</div>',
  data: function () {
    return {
      someChildProperty: true
    }
  }
})
```

Similarly, distributed content will be compiled in the parent scope.

### Single Slot

Parent content will be **discarded** unless the child component template contains at least one `<slot>` outlet. When there is only one slot with no attributes, the entire content fragment will be inserted at its position in the DOM, replacing the slot itself.

Anything originally inside the `<slot>` tags is considered **fallback content**. Fallback content is compiled in the child scope and will only be displayed if the hosting element is empty and has no content to be inserted.

Suppose we have a component called `my-component` with the following template:

``` html
<div>
  <h2>I'm the child title</h2>
  <slot>
    This will only be displayed if there is no content
    to be distributed.
  </slot>
</div>
```

And a parent that uses the component:

``` html
<div>
  <h1>I'm the parent title</h1>
  <my-component>
    <p>This is some original content</p>
    <p>This is some more original content</p>
  </my-component>
</div>
```

The rendered result will be:

``` html
<div>
  <h1>I'm the parent title</h1>
  <div>
    <h2>I'm the child title</h2>
    <p>This is some original content</p>
    <p>This is some more original content</p>
  </div>
</div>
```

### Named Slots

`<slot>` elements have a special attribute, `name`, which can be used to further customize how content should be distributed. You can have multiple slots with different names. A named slot will match any element that has a corresponding `slot` attribute in the content fragment.

There can still be one unnamed slot, which is the **default slot** that serves as a catch-all outlet for any unmatched content. If there is no default slot, unmatched content will be discarded.

For example, suppose we have an `app-layout` component with the following template:

``` html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

Parent markup:

``` html
<app-layout>
  <h1 slot="header">Here might be a page title</h1>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <p slot="footer">Here's some contact info</p>
</app-layout>
```

The rendered result will be:

``` html
<div class="container">
  <header>
    <h1>Here might be a page title</h1>
  </header>
  <main>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </main>
  <footer>
    <p>Here's some contact info</p>
  </footer>
</div>
```

The content distribution API is a very useful mechanism when designing components that are meant to be composed together.

## Dynamic Components

You can use the same mount point and dynamically switch between multiple components using the reserved `<component>` element and dynamically bind to its `is` attribute:

``` js
new Vue({
  el: 'body',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})
```

``` html
<component v-bind:is="currentView">
  <!-- component changes when vm.currentView changes! -->
</component>
```

If you prefer, you can also bind directly to component objects:

``` js
var Home = {
  template: '<p>Welcome home!</p>'
}

new Vue({
  el: 'body',
  data: {
    currentView: Home
  }
})
```

### `keep-alive`

If you want to keep the switched-out components in memory so that you can preserve their state or avoid re-rendering, you can wrap a dynamic component in a `keep-alive` element:

``` html
<keep-alive>
  <component :is="currentView">
    <!-- inactive components will be cached! -->
  </component>
</keep-alive>
```

## Misc

### Components and v-for

You can directly use `v-for` on a custom component, like any normal element:

``` html
<my-component v-for="item in items"></my-component>
```

However, this won't automatically pass any data to the component, because components have isolated scopes of their own. In order to pass the iterated data into the component, we should also use props:

``` html
<my-component
  v-for="(item, index) in items"
  v-bind:item="item"
  v-bind:index="index">
</my-component>
```

The reason for not automatically injecting `item` into the component is because that makes the component tightly coupled to how `v-for` works. Being explicit about where its data comes from makes the component reusable in other situations.

Here's a complete example of a simple todo list:

``` html
<div id="todo-list-example">
  <input
    v-model="newTodoText"
    v-on:keyup.enter="addNewTodo"
    placeholder="Add a todo"
  >
  <ul>
    <li
      is="todo-item"
      v-for="(todo, index) in todos"
      v-bind:title="todo"
      v-on:remove="todos.splice(index, 1)"
    ></li>
  </ul>
</div>
```

``` js
Vue.component('todo-item', {
  template: '\
    <li>\
      {{ title }}\
      <button v-on:click="$emit(\'remove\')">X</button>\
    <\li>\
  ',
  props: ['title']
})

new Vue({
  el: '#todo-list-example',
  data: {
    newTodoText: '',
    todos: [
      'Do the dishes',
      'Take out the trash',
      'Mow the lawn'
    ]
  },
  methods: {
    addNewTodo: function () {
      this.todos.push(this.newTodoText)
      this.newTodoText = ''
    }
  }
})
```

{% raw %}
<div id="todo-list-example" class="demo">
  <input
    v-model="newTodoText" v
    v-on:keyup.enter="addNewTodo"
    placeholder="Add a todo"
  >
  <ul>
    <li
      is="todo-item"
      v-for="(todo, index) in todos"
      v-bind:title="todo"
      v-on:remove="todos.splice(index, 1)"
    ></li>
  </ul>
</div>
<script>
Vue.component('todo-item', {
  template: '\
    <li>\
      {{ title }}\
      <button v-on:click="$emit(\'remove\')">X</button>\
    <\li>\
  ',
  props: ['title']
})
new Vue({
  el: '#todo-list-example',
  data: {
    newTodoText: '',
    todos: [
      'Do the dishes',
      'Take out the trash',
      'Mow the lawn'
    ]
  },
  methods: {
    addNewTodo: function () {
      this.todos.push(this.newTodoText)
      this.newTodoText = ''
    }
  }
})
</script>
{% endraw %}


### Authoring Reusable Components

When authoring components, it's good to keep in mind whether you intend to reuse it somewhere else later. It's OK for one-off components to be tightly coupled, but reusable components should define a clean public interface and make no assumptions about the context it's used in.

The API for a Vue component comes in three parts - props, events, and slots:

- **Props** allow the external environment to feed data to the component

- **Events** allow the component to trigger actions in the external environment

- **Slots** allow the external environment to insert content into the component

With the dedicated shorthand syntaxes for `v-bind` and `v-on`, the intents can be clearly and succinctly conveyed in the template:

``` html
<my-component
  :foo="baz"
  :bar="qux"
  @event-a="doThis"
  @event-b="doThat"
>
  <img slot="icon" src="...">
  <p slot="main-text">Hello!</p>
</my-component>
```

### Async Components

In large applications, we may need to divide the app into smaller chunks and only load a component from the server when it's actually needed. To make that easier, Vue allows you to define your component as a factory function that asynchronously resolves your component definition. Vue will only trigger the factory function when the component actually needs to be rendered and will cache the result for future re-renders. For example:

``` js
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```

The factory function receives a `resolve` callback, which should be called when you have retrieved your component definition from the server. You can also call `reject(reason)` to indicate the load has failed. The `setTimeout` here is simply for demonstration; How to retrieve the component is entirely up to you. One recommended approach is to use async components together with [Webpack's code-splitting feature](http://webpack.github.io/docs/code-splitting.html):

``` js
Vue.component('async-webpack-example', function (resolve) {
  // this special require syntax will instruct webpack to
  // automatically split your built code into bundles which
  // are automatically loaded over ajax requests.
  require(['./my-async-component'], resolve)
})
```

### Component Naming Conventions

When registering components (or props), you can use kebab-case, camelCase, or TitleCase. Vue doesn't care.

``` js
// in a component definition
components: {
  // register using camelCase
  'kebab-cased-component': { /* ... */ },
  'camelCasedComponent': { /* ... */ },
  'TitleCasedComponent': { /* ... */ }
}
```

Within HTML templates though, you have to use the kebab-case equivalents:

``` html
<!-- alway use kebab-case in HTML templates -->
<kebab-cased-component></kebab-cased-component>
<camel-cased-component></camel-cased-component>
<title-cased-component></title-cased-component>
```

When using _string_ templates however, we're not bound by HTML's case-insensitive restrictions. That means even in the template, you reference your components and props using camelCase, PascalCase, or kebab-case:

``` html
<!-- use whatever you want in string templates! -->
<my-component></my-component>
<myComponent></myComponent>
<MyComponent></MyComponent>
```

If your component isn't passed content via `slot` elements, you can even make it self-closing with a `/` after the name:

``` html
<my-component/>
```

Again, this _only_ works within string templates, as self-closing custom elements are not valid HTML and your browser's native parser will not understand them.

### Recursive Component

Components can recursively invoke themselves in their own template. However, they can only do so with the `name` option:

``` js
name: 'stack-overflow',
template: '<div><stack-overflow></stack-overflow></div>'
```

A component like the above will result in a "max stack size exceeded" error, so make sure recursive invocation is conditional (i.e. uses a `v-if` that will eventually be false). When you register a component globally using `Vue.component`, the global ID is automatically set as the component's `name` option.

### Inline Templates

When the `inline-template` special attribute is present on a child component, the component will use its inner content as its template, rather than treating it as distributed content. This allows more flexible template-authoring.

``` html
<my-component inline-template>
  <p>These are compiled as the component's own template.</p>
  <p>Not parent's transclusion content.</p>
</my-component>
```

However, `inline-template` makes the scope of your templates harder to reason about. As a best practice, prefer defining templates inside the component using the `template` option or in a `template` element in a `.vue` file.

### X-Templates

Another way to define templates is inside of a script element with the type `text/x-template`, then referencing the template by an id. For example:

``` html
<script type="text/x-template" id="hello-world-template">
  <p>Hello hello hello</p>
</script>
```

``` js
Vue.component('hello-world', {
  template: '#hello-world-template'
})
```

These can be useful for demos with large templates or in extremely small applications, but should otherwise be avoided, because they separate templates from the rest of the component definition.

### Cheap Static Components with `v-once`

Rendering plain HTML elements is very fast in Vue, but sometimes you might have a component that contains **a lot** of static content. In these cases, you can ensure that it's only evaluated once and then cached by adding the `v-once` directive to the root element, like this:

``` js
Vue.component('terms-of-service', {
  template: '\
    <div v-once>\
      <h1>Terms of Service</h1>\
      ... a lot of static content ...\
    </div>\
  '
})
```
