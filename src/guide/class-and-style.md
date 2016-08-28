---
title: Class 与 Style 绑定
type: guide
order: 6
---

数据绑定一个常见需求是操作元素的 class 列表和它的内联样式。因为它们都是 attribute，我们可以用 `v-bind` 处理它们：只需要计算出表达式最终的字符串。不过，字符串拼接麻烦又易错。因此，在 `v-bind` 用于 `class` 和 `style` 时，Vue.js 专门增强了它。表达式的结果类型除了字符串之外，还可以是对象或数组。

# 绑定 HTML Class

## 对象语法

我们可以传给 `v-bind:class` 一个对象，以动态地切换 class。

```html
<div v-bind:class="{ active: isActive, 'text-danger': hasError }"></div>
```

```javascript
data: {
  isActive: true,
  hasEror: false
}
```

The `v-bind:class` directive can also co-exist with the plain `class` attribute:

`v-bind:class` 指令可以与普通的 `class` 特性共存：

```html
<div class="static" v-bind:class="{ active: isActive, 'text-danger': hasError }"></div>
```

渲染为：

```html
<div class="static active"></div>
```

当 `isActive` 和 `hasError` 变化时，class 列表将相应地更新。例如，如果 `hasError` 变为 `true`，class 列表将变为 `"static active text-danger"`。

你也可以直接绑定数据里的一个对象：

```html
<div v-bind:class="classObject"></div>
```

```javascript
data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```

我们也可以在这里绑定一个返回对象的[计算属性](computed.html)。这是一个常用且强大的模式。

```html
<div v-bind:class="classObject"></div>
```

```javascript
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal',
    }
  }
}
```

## 数组语法

我们可以把一个数组传给 `v-bind:class`，以应用一个 class 列表：

```html
<div v-bind:class="[activeClass, errorClass]">
```

```javascript
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

渲染为：

```html
<div class="active text-danger"></div>
```

如果你也想根据条件切换列表中的 class，可以用三元表达式：

```html
<div v-bind:class="[isActive ? activeClass : '', errorClass]">
```

此例始终添加 `errorClass`，但是只有在 `isActive` 是 `true` 时添加 `activeClass` 。

不过，当有多个条件 class 时这样写有些繁琐。这就是为什么可以在数组语法中使用对象语法：

```html
<div v-bind:class="[{ active: isActive }, errorClass]">
```

# 绑定内联样式

## 对象语法

`v-bind:style` 的对象语法十分直观----看着非常像 CSS，其实它是一个 JavaScript 对象。CSS 属性名可以用驼峰式（camelCase）或短横分隔命名（kebab-case）：

```html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

```javascript
data: {
  activeColor: 'red',
  fontSize: 30
}
```

直接绑定到一个样式对象通常更好，让模板更清晰：

```html
<div v-bind:style="styleObject"></div>
```

```javascript
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```

同样的，对象语法常常结合返回对象的计算属性使用。

## 数组语法

`v-bind:style` 的数组语法可以将多个样式对象应用到一个元素上：

```html
<div v-bind:style="[baseStyles, overridingStyles]">
```

## 自动添加前缀

当 `v-bind:style` 使用需要厂商前缀的 CSS 属性时，如 `transform`，Vue.js 会自动侦测并添加相应的前缀。

--------------------------------------------------------------------------------

> 原文: http://rc.vuejs.org/guide/class-and-style.html

***

# Class and Style Bindings

A common need for data binding is manipulating an element's class list and its inline styles. Since they are both attributes, we can use `v-bind` to handle them: we just need to calculate a final string with our expressions. However, meddling with string concatenation is annoying and error-prone. For this reason, Vue provides special enhancements when `v-bind` is used with `class` and `style`. In addition to strings, the expressions can also evaluate to objects or arrays.

## Binding HTML Classes

### Object Syntax

We can pass an object to `v-bind:class` to dynamically toggle classes:

``` html
<div v-bind:class="{ active: isActive, 'text-danger': hasError }"></div>
```
``` js
data: {
  isActive: true,
  hasError: false
}
```

The `v-bind:class` directive can also co-exist with the plain `class` attribute:

``` html
<div class="static" v-bind:class="{ active: isActive, 'text-danger': hasError }"></div>
```

This will render:

``` html
<div class="static active"></div>
```

When `isActive` or `hasError` changes, the class list will be updated accordingly. For example, if `hasError` becomes `true`, the class list will become `"static active text-danger"`.

You can also directly bind to an object:

``` html
<div v-bind:class="classObject"></div>
```
``` js
data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```

This will render the same result. We can also bind to a [computed property](computed.html) that returns an object. This is a common and powerful pattern:

``` html
<div v-bind:class="classObject"></div>
```
``` js
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal',
    }
  }
}
```

### Array Syntax

We can pass an array to `v-bind:class` to apply a list of classes:

``` html
<div v-bind:class="[activeClass, errorClass]">
```
``` js
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

Which will render:

``` html
<div class="active text-danger"></div>
```

If you would like to also toggle a class in the list conditionally, you can do it with a ternary expression:

``` html
<div v-bind:class="[isActive ? activeClass : '', errorClass]">
```

This will always apply `errorClass`, but will only apply `activeClass` when `isActive` is `true`.

However, this can be a bit verbose if you have multiple conditional classes. That's why it's also possible to use the object syntax inside array syntax:

``` html
<div v-bind:class="[{ active: isActive }, errorClass]">
```

## Binding Inline Styles

### Object Syntax

The object syntax for `v-bind:style` is pretty straightforward - it looks almost like CSS, except it's a JavaScript object. You can use either camelCase or kebab-case for the CSS property names:

``` html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```
``` js
data: {
  activeColor: 'red',
  fontSize: 30
}
```

It is often a good idea to bind to a style object directly so that the template is cleaner:

``` html
<div v-bind:style="styleObject"></div>
```
``` js
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```

Again, the object syntax is often used in conjunction with computed properties that return objects.

### Array Syntax

The array syntax for `v-bind:style` allows you to apply multiple style objects to the same element:

``` html
<div v-bind:style="[baseStyles, overridingStyles]">
```

### Auto-prefixing

When you use a CSS property that requires vendor prefixes in `v-bind:style`, for example `transform`, Vue will automatically detect and add appropriate prefixes to the applied styles.
