# Day 01 面试题

> Virtual DOM 作为 Vue.js 框架的核心特性之一，你有考虑过 Vue.js、React 这类的框架为什么要用 Virtual DOM 机制吗？
> 
> 很多人都说 Virtual DOM 很快，但是它真的会“快”吗？如果快，它到底快在什么地方？
> 
> 理解 Virtual DOM，对深入学习 Vue.js 框架原理非常重要。
>
> 接下来，我们通过一个拉勾的前端面试题来考考大家对 Virtual DOM 中的一些特性的理解

## 分析以下代码，对比它们的具体差异，分析 Virtual DOM 中为什么要使用 v-key？

### 页面首次渲染时候列表中的 Key 的作用？

```html
<template>
  <div>
    <ul>
      <li v-for="value in arr" :key="value">{{value}}</li>
    </ul>
  </div>
</template>
<script>
export default {
  data: {
    arr: ['a', 'b', 'c', 'd']
  }
}
</script>
```

### 当点击按钮的时候此时列表中的 Key 的作用？

```html
<template>
  <div>
    <button @click="handler">插入</button>
    <ul>
      <li v-for="value in arr" :key="value">{{value}}</li>
    </ul>
  </div>
</template>
<script>
export default {
  data: {
    arr: ['a', 'b', 'c']
  },
  methods: {
    handler () {
      this.arr.unshift('x')
    }
  }
}
</script>
```

### 当点击按钮的时候此时列表中没有 v-key 会如何执行？

```html
<template>
  <div>
    <button @click="handler">按钮</button>
    <ul>
      <li v-for="item in list">
        <input type="checkbox"> {{item}}
      </li>
    </ul>
  </div>
</template>
<script>
export default {
  data: {
    list: ['悟空', '八戒', '沙僧']
  },
  methods: {
    handler() {
      this.list.unshift('唐僧')
    }
  }
}
</script>
```

## 解析

定义：——

首先，v-key 的作用是更新组件时，判断两个节点是否相同，相同就复用，不相同就删除旧的重新创建新的。

应用：——

如示例代码2中，当给列表中插入元素的时候，设置 Key 的情况下，Diff 的过程中会把相同的元素重用，只会在新元素的位置做一次 DOM 的插入操作，如果不适用 Key 的话会把寻找到插入元素以及之后的元素做修改 DOM 操作（通过设置 Key 和不设置 Key 对代码调试可知）

当然 Key 还有一个作用是防止副作用发生，例如示例代码3中，使用 Key 可以避免列表渲染过程中不必要的 bug。

当选中第一项后，再往第一项之前插入元素，这时候新插入的第一项会被选中，因为这个时候虚拟 DOM 在 diff 的时候重用了第一项，为了解决这个问题，可以给每一个列表项添加唯一 Key 解决。

最佳实践：——

不设置 Key 的情况下只适合不带状态的列表项，可以就地复用，但是大多数情况下，列表项都有自己的状态（数据）在 v-for 遍历列表项目的时候建议设置 Key，在 v-for 遍历列表项目如何设置 Key 应该设置为唯一值，而不是遍历时候的索引，因为索引作为 Key 和不带 Key 的作用是一样的。index 作为Key 时，每个列表项的索引在变化前后也是一样的，都是直接判断 sameVnode 然后复用。

另外在使用 transition 实现过渡的时候，当有相同标签名的元素切换，需要设置 Key 让 Vue 区分它们，否则 Vue 为了效率只会替换相同标签内部的内容。所以在使用 <transition> 的时候建议给他内部的元素都设置 Key。