## mobx

### mobx 原理

从上面的一些用法来看，很明显实现上用到了 Object.defineProperty 或者 Proxy。
**收集依赖**
当 autorun 第一次执行的时候会触发依赖属性的 getter，从而收集当前函数的依赖。

```
const person = observable({ name: 'tom' })
autorun(function func() {
    console.log(person.name)
})
```

在 autorun 里面相当于做了这么一件事情。

```
person.watches.push(func)
```

**触发依赖**
当依赖属性触发 setter 的时候，就会将所有订阅了变化的函数都执行一遍，从而实现了数据响应式。

```
person.watches.forEach(watch => watch())
```

## mobx-react/mobx-react-lite (mobx 结合 react)

### 使用 demo

```jsx
import { observable } from "mobx";
import { observer } from "mobx-react";
import React, { Component } from "react";
import ReactDOM from "react-dom";

export const store = observable({
  count: 0,
});
store.increment = function () {
  this.count++;
};
store.decrement = function () {
  this.count--;
};

@observer
class Count extends Component {
  render() {
    return (
      <div>
        Counter: {store.count} <br />
        <button onClick={this.handleInc}> + </button>
        <button onClick={this.handleDec}> - </button>
      </div>
    );
  }
  handleInc() {
    store.increment();
  }
  handleDec() {
    store.decrement();
  }
}

ReactDOM.render(<Count />, document.querySelector("#root"));
```

提供了一个高阶组件 observer，它会将组件包装成一个响应式组件。
在组件内部，会通过 autorun 来收集依赖，当依赖属性发生变化的时候，会重新执行组件的 render 方法。（劫持渲染）

### 如何收集依赖的呢？依赖长什么样？

​1. ​Proxy/Getter 拦截 ​​：  
MobX 使用 Proxy（或 Object.defineProperty）对 observable 数据进行包装。当组件访问这些数据时（如 counterStore.count），会触发 getter 拦截，记录当前正在执行的上下文（即组件的 render 函数）。  
2. ​​Reaction 注册 ​​：
每个被 observer 包裹的组件会生成一个 Reaction 实例。在组件渲染时，MobX 通过 reaction.track() 执行组件的 render 函数，并记录所有被访问的 observable 属性，将这些属性与 Reaction 建立依赖关系。  
3. 依赖触发更新 ​​：
当 observable 数据被修改（如通过 action），MobX 会通知所有依赖它的 Reaction，触发组件的重新渲染。

===> 这个 redux 区别是（从 react 的 Fiber 角度上来说）？
**​​mobx-react​​：**
基于 MobX 的 ​​ 响应式依赖追踪 ​​。在组件渲染时，通过 observer 包裹的组件会自动订阅其内部使用的 observable 状态。当状态变化时，MobX 直接触发组件的 ​​ 精准重渲染 ​​（仅依赖变更状态的组件），无需经过 Fiber 的完整协调过程。
​​ 优势 ​​：更新粒度细，跳过不必要的虚拟 DOM 对比，性能更高。
​​Fiber 影响 ​​：MobX 的更新是同步的，可能绕过 Fiber 的优先级调度（如用户输入等高优先级任务可能被阻塞）。
**​​react-redux​​：**
基于 Redux 的 ​​ 单向数据流 ​​。状态变更通过 dispatch(action) 触发，Redux 通知所有订阅的组件（通过 connect 或 useSelector），由 React Fiber 调度器统一协调更新。
​​ 优势 ​​：与 Fiber 深度集成，遵循 React 的优先级调度（如时间切片、并发模式）。
​​ 劣势 ​​：更新可能触发多个组件的重渲染，需依赖 React.memo 或手动优化。(redux 通知某个组件更新，走 Fiber 流程，仍然是要从根节点开始向下遍历)
