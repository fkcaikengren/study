## Redux

#### 1.Redux 的概念：

**订阅发布模式**
store 对象维护一个状态树（对象），reducers 负责计算状态树逻辑，action 是修改所需的数据。
store.dispatch(action); //触发，会调用所有的监听函数。  
store.subscribe(fn); //注册监听函数，这个订阅会订阅 store 的整个状态

**dispatch(action)里发生了什么？**
（reducers 是一个对象，每个属性是一个函数——子 reducer, reducer 会被合成一个总的 reducer 挂在 store 上）
用 reducer 计算返回新的 state, 然后调用所有的监听函数。  
调用所有的 listeners，那么 react-redux 中是如何更新指定组件的呢？

**中间件-洋葱模型**

#### 2.react-redux 是什么？

react-redux 是一个用于 React 应用和 Redux 状态容器之间进行连接的库。它提供了一组高阶组件和自定义钩子，用于将 React 组件与 Redux 状态树连接起来。

##### react-redux 核心实现

**分发 store**
基于 Context API，通过 Provider 将 store 对象分发给所有的子组件。
通过 connect HOC 或 useSelector/useDispatch，消费 store。
PS.并不是通过 context 来更新 UI 的，因为 context 只分发 store，store 几乎是不会变的。

**调用所有的 listeners，那么 react-redux 中是如何更新指定组件的呢？**

关键在于 useSelect 的实现：
参考： https://www.51cto.com/article/809759.html

```jsx
import{useSyncExternalStore } from"react";
function useSelector(selector）{
    const store = useContext(StoreContext);
    return useSyncExternalStore(
        store.subscribe,//订阅 Redux store
        ()=>selector(store.getState) //获取最新状态
    )
}
```

**useSyncExternalStore 如何工作？**
useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);

React 使用 subscribe 函数， 注册一个 callback:会调用 getSnapshot 并在需要的时候重新渲染组件。当 getSnapshot 返回的数据引用发生变化（浅比较），就会重新渲染组件。

#### react 可以局部 手动 更新组件？

```jsx
<App>
  <A />
  <B />
</App>
```

结构如上，在 B 组件中执行 setState，那么 App 和 A 组件会重新渲染吗？
答：不会！但是会有如下过程：  
在 React 18 中，setState（或函数组件的状态更新函数如 useState 的 setX）的核心原理和步骤如下：

- 更新任务的创建与入队。当调用 setState 时，React 会创建一个更新对象（Update），包含新状态值或更新函数（如 prev => prev + 1）以及优先级标记。
- 标记更新范围与优先级调度。React 从触发更新的 Fiber 节点向上回溯到根节点（FiberRoot），标记路径上所有受影响的节点为“待更新”（dirty）。
- React 18 的调度器（Scheduler）根据任务优先级（如用户交互、数据加载）动态安排更新时机：
- 协调（Reconciliation）与渲染。增量遍历 Fiber 树：从根节点开始，React 通过深度优先遍历跳过未受影响的节点，仅处理标记为 dirty 的组件及其子树。
- commit 阶段。提交更新到 DOM
- 在渲染阶段完成后，React 将变更同步提交到真实 DOM，触发浏览器重绘。
- 回调执行：若 setState 传入了回调函数（第二个参数），会在 DOM 更新后执行。

**总结流程**

setState → 创建更新对象 → 加入队列 → 标记优先级 → 批量合并 → 协调（Diff）→ 提交 DOM → 执行回调
