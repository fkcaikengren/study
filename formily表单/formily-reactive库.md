## formily/reactive

相关学习资料：

- https://fishedee.com/2021/07/13/Formily%E7%9A%84Reactive%E7%9A%84%E7%BB%8F%E9%AA%8C%E6%B1%87%E6%80%BB/

### 原理

类似 vue3 的响应式原理，通过 proxy 代理对象，监听对象的 get 和 set 操作，来实现响应式。
通过 autorun 收集依赖，当触发 setter 时，从全局的 WeakMap 中找出这个「对象.属性」对应的依赖集合，并执行这些依赖。  
依赖即组件的渲染函数，这样能实现组件的精确更新。(大致上 render 函数被称为 Reaction)

### formily/reactive-react 的 api
