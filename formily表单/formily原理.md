## formily 原理

### 相关资料

相关资料：
手撕 formily https://github.com/cgfeel/formily

大佬博客：
https://fishedee.com/

### 基础表单功能现状

1. 数据管理：控件局部状态管理，局部更新。
2. 表单校验：采用 async-validator 进行校验。
3. 支持字段路径，支持嵌套字段。支持数组。
4. 支持表单联动和动态表单。

进阶的功能支持：

1. 大数据量表单 性能。
2. json 协议驱动。
3. 更方便的联动使用

### form 表单对比

【性能】
antd 表单： 采用类似 redux（订阅发布模式）来管理 Form 数据。-> 表单数据量大时性能下降，明显卡顿。
formily 表单：采用了 mobx（Proxy 响应式）来管理 Form 数据。-> 适合大数据量表单，精准更新。o(1)时间复杂度更新。

【协议驱动】
antd 表单：编程式 API。
formily 表单：支持 JSON Schema、Markup Schema 和 JSX 三种协议驱动方式。采用服务端分发 json 的方式能实现无代码构建表单。

【联动使用】

### formily 原理

### 为什么要使用 formily 构建表单？（总结）

比其他 antd 等表单的突出优势：  
1.采用了响应式数据，精准更新，高性能。  
2.支持 JSON Schema、Markup Schema 和 JSX 三种协议驱动方式。 (JSON schema 这种形式给 后端动态构建表单提供了很好的支持)  
3.非常方便的表单联动（主动联动和被动联动）。并且在联动的时候能做到精准更新！
---在 semi-ui 中，被动联动采用 useFormApi,formApi.getValue()来获取值，但是这种方式会导致整个表单重新渲染，性能很差。

其他特点：  
1.通用的领域模型 formily/core，提供动态表单能力，支持 ArrayField, ObjectField 等.强大的 路径系统 支持，支持数组和对象的嵌套
2.UI 无关，可以结合 react/vue 框架

**场景对比**
大量字段、复杂联动、json 驱动动态表单的情况下，使用 formily。
简单的业务场景，使用 antd 之类的表单。

PS. json 驱动动态表单的好处：  
需求反复变化，表单的基础性重复工作，前后端联调，导致前端工作量大且效率低。 采用 json 驱动，后端只需要提供 json 数据，前端能自动根据 json 渲染表单。

**启发**
fomily-reactive 精确更新组件。
json-schema
