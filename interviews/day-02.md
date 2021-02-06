# Day 02 面试题

> 无论是 BATJ 还是其他的一线大厂，只要是使用 Vue.js 框架的前端职位，Vue.js 的响应式实现原理都是必问的，因为只有你搞懂了响应式的实现原理，才有可能彻底理解这个框架，面对复杂问题或者需求才不会无从下手，也可以避免项目开发过程中响应式数据带来的影响。
> 
> 通过今天课程的介绍，我们知道了 Vue.js 2.x 的响应式是基于 ES5 中的 Object.defineProperty (getters/setters) 实现的。而在即将发布的 Vue.js 3.0 中，数据响应这块换为了 ES2015 中新增的 Proxy。
> 
> 那今天的面试题，我们就来尝试一下 Vue.js 3.0 数据响应式实现

## 请实现下面代码中的 reactive 与 watch 函数，具体要求如下：

```javascript
// 将给定对象包装为 `Proxy` 实现数据响应
function reactive(obj) {
  // ... 内部实现
}

// 添加一个数据状态监视函数
function watch(effect) {
  // ... 内部实现
}

const state = reactive({
  foo: 100,
  bar: 200,
});

watch(() => {
  console.log("foo changed: ", state.foo);
});

watch(() => {
  console.log("bar changed: ", state.bar);
});

state.foo++; // => foo changed:  101
state.bar++; // => bar changed:  201
```

假设现有 `reactive` 和 `watch` 这两个函数
最终效果是 `state.foo` 或 `state.bar` 的值一旦改变，对应的 `watch` 函数自动执行

例如：

- `state.foo` 的值改变，执行第一个 `watch` 中的回调函数
- `state.bar` 的值改变，执行第二个 `watch` 中的回调函数

**要求：实现这个 `reactive` 和 `watch` 这两个函数！！！**

思路：

1. 利用 `Proxy` 实现数据响应
2. `reactive` 函数将给定对象包装为 `Proxy` 实现数据响应
3. `watch` 函数的作用是添加一个数据状态监视函数，只有这个函数内使用到的成员发生变化时，这个函数才会被执行

难点：

1. 如何收集属性对应的依赖函数

## 参考答案

```javascript
const effects = new Map();

const proxyHandler = {
  get(target, property) {
    // 收集依赖

    if (effects.last) {
      // 确保当前这个对象下这个属性所对应依赖列表
      effects.set(target, { [property]: [], ...effects.get(target) });
      // 记录依赖
      effects.get(target)[property].push(effects.last);
    }

    // 正常返回结果
    return Reflect.get(target, property);
  },
  set(target, property, value) {
    // 触发 watch

    // 先执行设置操作
    const succeed = Reflect.set(target, property, value);

    // 找到当前这个对象下这个属性所对应依赖列表
    const deps = effects.get(target)[property];
    // 依次调用
    deps.forEach((e) => e());

    // 正常返回
    return succeed;
  },
};

// 将给定对象包装为 `Proxy` 实现数据响应
function reactive(obj) {
  // 为每个对象记录映射一个依赖列表对象，结构为 { prop: [ ...deps ] }
  effects.set(obj, {});

  // proxyHandler 可以复用，减少内存消耗
  return new Proxy(obj, proxyHandler);
}

// 添加一个数据状态监视函数
function watch(effect) {
  effects.last = effect;
  effect();
  // 注意一定要清空
  effects.last = null;
}

const state = reactive({
  foo: 100,
  bar: 200,
});

watch(() => {
  console.log("foo changed: ", state.foo);
});

watch(() => {
  console.log("bar changed: ", state.bar);
});

watch(() => {
  console.log("both changed: ", state.foo, state.bar);
});

state.foo++; // => foo changed:  101
state.bar++; // => bar changed:  201
```
