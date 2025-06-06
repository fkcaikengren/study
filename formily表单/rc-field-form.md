## antd 的 rc-field-form 库

### 基础结构和概念

**FormContext**
整体上 FormContext 通过分发方法，传递给子表单，能实现对多个表单联动。

**Form**  
form 元素。通过参数 component 指定自定义 form 类型的组件

**FieldContext**
分发了一系列获取和设置 Field 的 value、error 等方法给子组件 Field。

registerField : 注册 FieldEntity，用于 notifyObservers 通知回调。时机：Field 挂载后调用。
registerWatch : 注册 watch，用于 notifyWatch 通知回调。时机：开发这使用 useWatch 时调用。

**Field**
用来包裹表单控件的字段组件。管理字段名、验证规则、值等。

**formInstance**
位于 Form 层。  
formInstance 是 Form API 引用，提供了一些方法，如 getFieldValue, setFieldValue, validateFields, resetFields 等。  
获取代码如下：

```jsx
const forceReRender = () => {
  forceUpdate({});
};
const formStore: FormStore = new FormStore(forceReRender);
formRef.current = formStore.getForm(); // 获取 FormInstance 实例
```

### FormStore

【数据中心】
[store] : 存放表单数据（普通对象）
[updateSotre] : 更新 store。 (通过 setValue 对指定路径上的值进行修改，并进行一次深拷贝，返回一个新的 store)
[fieldEntities]: 用于管理所有的 Field 实例。

---

【表单项联动】
[registerWatch] : 用于 useWatch 钩子 -> 内部维护了一个 state, registerWatch 注册的函数会从 store 中获取 namePath 对应的最新的数据来修改这个 state。 【实现被动联动！】

[notifyWatch] : 用于 registerField/setField/resetField 等方法。

---

【暴露给外部使用，挂在 formInstance 上】
[getFieldValue] : 获取指定路径上的值。
[setFieldValue] : 设置指定路径上的值。 -> 通知 UI 更新：1.notifyObservers，2.notifyWatch。

### 两个 hooks

[useForm] : 获取 formInstance .
[useWatch] : 其作用是 实现被动联动。当依赖的 namePath 的值发生变化，触发更新，获取该 namePath 的最新值。

### Field

1.didMount 后，registerField.  
`registerField(this)`, 本身就是一个 FieldEntity 实例，包含 onStoreChange 方法。
onStoreChange 方法会在 store 更新时调用。store 更新后会调用 notifyObservers 方法，通知所有的 FieldEntity 实例。

2.onStoreChange 处理.  
store 上更新，会通知所有的 FieldEntity 的 onStoreChange 方法， 方法里会判断本次更新的 namePath 是否匹配当前的 fieldEntity 的 namePath，匹配情况下更新（组件局部更新，不影响这个表单）。

**Field 是如何接管控件的？**
当 children 时 ReactNode 时，获取 children 的 props，经过 getControlled 处理后得到新的 props，传递给 children（拦截）。

拦截 onChange 事件，获取新的值更新到 store 上。 通过自定义的 dipatch 方法：

```jsx
dispatch({
  type: "updateValue",
  namePath,
  value: newValue,
});
```

执行路径：dispatch -> setValue -> updateStore -> notifyObservers -> FieldEntity.onStoreChange

### List

**本质上是：实现用一个 Field 管理数组数据的能力。**

children 是一个函数 (fields) => ReactNode。用 store 中的数据数组来渲染重复的 ReactNode。  
并且提供了一些操作方法： add, remove。这些方法可以控制 store 数据数组的增删。
