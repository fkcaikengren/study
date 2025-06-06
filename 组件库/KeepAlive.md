## KeepAlive 的实现原理

- 不销毁的父组件: KeepAlive
- 包裹待渲染元素的子组件: CacheComponent

KeepAlive 用状态缓存了组件的「实例」（包括 cacheKey:string, children:reactNode）. 每次渲染经过 useLayoutEffect 时不断把新组件加入状态数组中。然后所有的状态数组都渲染对应一个子组件 CacheComponent。

在 CacheComponent 中，判断当前的 cacheKey 和 activeCacheKey 是否一致，一致则渲染缓存的 children，否则不渲染任何东西。 （内部保存了一个 cacheDiv 的空节点，把 children 挂到 cacheDiv，真正的渲染交给了 appendChild 直接操作）  
「渲染缓存的组件」： 先将 inactive 的组件移除， 然后将 active 的组件插入到父组件 ref 里，containerDiv.appendChild(cacheDiv);

为什么不直接走 react 的渲染，而是用 appendChild?
不是：`activatedRef.current ? children : null`，  
而是：`activatedRef.current? createPortal(<ErrorBoundary>{children}</ErrorBoundary>, cacheDiv) : null`，`containerDiv.appendChild(cacheDiv);`。
答：为了缓存，createPortal 并不会把 div 节点挂载到页面上，而直接 return children 会挂载到页面上。 createPortal 把 children 的 dom 结构缓存到了 cacheDiv 上，用 useMemo 缓存这，不会因为 cacheDiv 的卸载而导致 children dom 结构销毁。

PS.createPortal 返回的 portalNode 被渲染=>是将元素插入到指定的 DOM 节点中。
