## formily 原理

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

相关资料：
手撕 formily https://github.com/cgfeel/formily

大佬博客：
https://fishedee.com/
