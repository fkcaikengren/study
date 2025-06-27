## 前置知识

### AST

var answer = 6 \* 7;

1.解析为 token 列表

```
[
    {
        type: 'var',
        value: 'var'
    },
    {
        type: 'Identifier',
        value: 'a'
    },
    {
        type: '=',
        value: '='
    },
    {
        type: 'Number',
        value: '1'
    },
    {
        type: '*',
        value: '*'
    },
    {
        type: 'Number',
        value: '2'
    },
    {
        type: ';',
        value: ';'
    }
]
```

2.词法分析，将 token 列表转换为 AST

```
[
    {
        "type": "Keyword",
        "value": "var"
    },
    {
        "type": "Identifier",
        "value": "answer"
    },
    {
        "type": "Punctuator",
        "value": "="
    },
    {
        "type": "Numeric",
        "value": "6"
    },
    {
        "type": "Punctuator",
        "value": "*"
    },
    {
        "type": "Numeric",
        "value": "7"
    },
    {
        "type": "Punctuator",
        "value": ";"
    }
]
```

## webpack 原理

### 初始化

加载配置文件，创建 compiler 对象，加载所有插件和 loader.

### 构建（编译）阶段

**概述：**
创建 compilation 对象（一次编译过程的实例，每次编译生成一个新的 compilation），包含所有模块和依赖关系，遍历所有模块，调用 loader 处理模块，生成 AST，递归处理 AST 中的依赖。

webpack 构建的模块依赖图结构如下：（本质是有向图）

```js
class ModuleGraph {
  constructor() {
    this._modules = new Map(); // 模块映射表：key=模块ID，value=模块实例（如NormalModule）
    this._dependencyMap = new Map(); // 正向依赖：key=源模块，value=依赖模块集合
    this._reverseDependencies = new Map(); // 反向依赖：key=被依赖模块，value=依赖者集合
  }

  //   SplitChunksPlugin 在优化 chunks 处理中，需要使用反向依赖关系
}
```

模块的结构：

```js
class Module {
  constructor() {
    this.id; // 模块ID
    this.dependencies = []; // 遍历AST后，收集的依赖
    this._source; // 模块源代码
    this._ast; // 模块 AST
  }
}
```

从 addEntry()方法进入流程： (本质：触发 EntryPlugin)  
1.创建 module
1.1.找到入口文件，创建 module.
1.2.分析入口文件的 import/require 依赖，递归创建 module.

2.在 module 创建后：调用 loader 转换资源文件为 js 代码文件，然后（acorn）解析 js 成 AST，然后遍历 AST 分析依赖，构建模块依赖图。
2.1 如何构建模块依赖图

**compiler 的 hooks 是什么**
compiler.hooks 是什么

compiler.hooks.compilation.tap()

compiler.hooks.make.tapAsync()

### 打包（生成代码）阶段

包含两个子阶段：
1.seal 阶段
根据 splitChunk 的规则，分析依赖图，生成 chunk.

2.emitAssets 阶段
把 chunk 输出到文件系统。
