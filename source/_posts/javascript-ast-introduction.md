title: javascript-ast-introduction
date: 2019-05-16 22:54:13
tags: javascript ast
---

[AST, Abstract Syntax Tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree)是一种用来描述代码抽象结构的树形表示。它包含代码的全部信息，可以认为是与代码等价的。下图是一个AST的例子(摘自wiki)

{% asset_img ast.png AST例子 %}

相比于代码，AST是结构化的，非常适合程序处理。很多过程都需要用到AST。

例如在编译过程中，经过语法分析(Syntax Analysis)我们就会得到AST，AST将会被用于后续的语义分析（Semantic Analysis）以及生成中间代码（IR Generation）。

{% asset_img compiler.jpg 编译器流程 %}

又比如在代码转换过程中，代码首先经过解析器（Parser）产生AST，然后通过对AST的遍历（AST Transform）生成新的代码。

{% asset_img babel.png Babel.js %}

AST没有统一的规范，即使是同一种编程语言，不同解析器往往产生不同的AST。

以javascript为例，能产生AST的工具有很多
- [Spidermonkey](https://developer.mozilla.org/zh-CN/docs/Mozilla/Projects/SpiderMonkey),Mozilla Javascript引擎,C++编写
- [Acorn](https://github.com/acornjs/acorn),老牌的Javascript解析器,javascript编写
- [Esprima](http://esprima.org/),使用非常广泛的Javascript解析器,拥有丰富的文档和示例
- [Babylon/Babel parser](https://babeljs.io/docs/en/babel-parser) Babel使用的解析器,基于Acorn,生态强大,丰富的preset和plugin

其实javascript的AST是有一个”名义上“的[标准](https://github.com/estree/estree)，只不过这个标准大家都不怎么遵守，总喜欢往里偷偷加点料.这个标准每年都根据EcmaScript最新确定的语法规范进行修正.

这里有一个AST在线体验工具, [AST Explore](https://astexplorer.net/)

以Babel生成的[Babel-AST](https://github.com/babel/babel/blob/master/packages/babel-parser/ast/spec.md)为例,AST中每个节点都有两个关键属性

```js
Interface Node {
  type: string,
  loc: SourceLocation | null,
}
```

其中,type表示节点的类型,loc表示当前节点在源代码中的位置

type大致分为Identifier,Function,Statements,Expression,Class,Module等几大类,每种大类下面又会有几种细分小类

了解AST能够对javascript语法有一个更清晰的概念,也能够帮助更好地理解javascript代码.例如下面两个写法一个是正确的,一个是错误的

```js
function() {} // invalid,这是一个函数定义,函数定义不能没有名字
(function() {}) // valid,这是一个函数表达式,函数表达式可以是匿名的
```

AST可以用来做静态代码分析,例如函数调用分析,数据流分析,指针分析,相关性分析;进而实现某些高级功能,比如代码优化,自动重构,Dead Code Detection,tree shaking等.

相关的工具有:

- [Closure Compiler](https://developers.google.com/closure/compiler/?hl=zh-CN), Google的开源js分析工具,Java编写
- [Flow](https://flow.org/), Facebook的开源js分析工具,Ocaml编写
- [eslint](https://eslint.org/), 这个就不用说了
- [Prettier](https://prettier.io/), 代码格式化工具

另外也有一些第三方服务,比如DeepScan,辅助评估代码质量
