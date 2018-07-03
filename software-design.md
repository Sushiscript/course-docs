# sushiscript设计与实现

## 简介

`sushiscript`是一个以bash为后端的静态类型脚本语言，目前我们完成了一个基础的语言框架，包括parsing、简单的分析和代码生成等部分。这篇文档主要介绍`sushiscript`简单的设计方向，以及目前我们完成了的特性的具体实现思路以及一些代码结构索引。

## 语言设计

**`sushiscript`的语言设计文档见：[`docs/language-specification`](https://github.com/Sushiscript/sushiscript/blob/master/docs/language-specification.md)**，上述文档目前包含了严格的词法、语法结构定义，类型系统定义以及`sushiscript`特性对应到`shell`特性的实现方式（代码生成）。文档中包含了`sushiscript`绝大部分的具体设计，在本文中会多次引用，并简称为“文档”。所以本节中仅简单介绍`sushiscript`的基本设计思路，而非严格的设计或实现。

### 语言范式

由于我们选择的语言后端没有很好的数据抽象的特性，以及个人喜好的原因，面向对象几乎不在考虑范围之内。简单的函数式的设计在语言设计上通常是一个清爽以及易于实现的选择，不过在这里我们面临一个难以规避副作用的系统，包括关于“命令”这一类型的抽象使得整个设计思维量非常的大，可能会涉及一些非常理论化的知识，这和我们的初衷是背道而驰的。简洁的过程式语言使得语言设计的难度几何级降低，而且可以保留许多拓展的可能。

### 类型系统

我们还是保守地选择了相对易于实现的，类型推导基于树后序遍历的强类型系统，内置类型包含了许多对Shell本身特性的抽象。我们对于隐式类型转换的选择非常小心，目前只有很少几个我们认为最为直观的以及符合逻辑的类型转换（见_文档4.5.3_）。因为在这一阶段还没有开始实现泛型，所以像这样简单的类型推导规则已经够用，之后加入泛型的话，为了语言展现出来的整洁程度以及一致性可能就需要一些更理论化的类型系统支持（如[`HM type system`](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system)）

### 语法风格

在语法风格上为了整体的感觉以及语言展现出来的一致性上有非常非常多细节的考虑。不过总体来说语法风格是基于缩进来识别作用域的类python风格，虽然会给前端实现带来一些麻烦，不过总的来说我认为还是值得的。

## 实现

### 总览

和简化的传统编译器架构类似，`sushiscript`实现分为三大部分：编译器前端，分析与检查，以及代码生成。但为了实现简单和分工方便的考虑：前端部分目前细分为**词法分析**（`sushi/lexer`）和**语法分析**（`sushi/parser`）两阶段，分析与检查又分为**作用域分析**（`sushi/scope`）以及**类型检查**（`sushi/type-system/type-check`）两阶段。每个阶段在实现上均假设输入的各种性质完全正确，并且各自产生自己的错误，顶层的**流水线**（`sushi/pipeline`）按顺序执行每一个阶段，并把上一阶段的输出传递给下一阶段，如果在任何一个阶段遇到错误，则将错误输出并退出。

### 抽象语法树

上述几乎所有的阶段（除了词法分析以外）都需要围绕抽象语法树来进行，所以这里简单介绍一下。语法树的完整实现见[`sushi/ast/`](https://github.com/Sushiscript/sushiscript/tree/master/include/sushi/ast)。

语法树类型是语言的语法结构在编程语言中的抽象表示，本质上是递归的数据类型（[recursive data type](https://en.wikipedia.org/wiki/Recursive_data_type))。映射到c++也就是用基于指针的链式结构来表达递归（实现上用了`std::unique_ptr`）。因此语法树整体上和[文档](https://github.com/Sushiscript/sushiscript/blob/master/docs/language-specification.md)中的语法结构完全一致（大体上分为[`sushi::ast::Statement`](https://github.com/Sushiscript/sushiscript/blob/master/include/sushi/ast/statement/statement.h)以及[`sushi::ast::Expression`](https://github.com/Sushiscript/sushiscript/blob/master/include/sushi/ast/expression/expression.h)），因此具体的结构在这里不详述。

设计上，我们用了[visitor模式](https://en.wikipedia.org/wiki/Visitor_pattern)来实现不同地方对于语法树（以及很多其它类型）的访问（visitor我们的c++元编程实现见：[`sushi/util/visitor.h`](https://github.com/Sushiscript/sushiscript/blob/master/include/sushi/util/visitor.h)）。从理论角度谈，visitor模式是[和类型](https://en.wikipedia.org/wiki/Tagged_union)的[模式匹配](https://en.wikipedia.org/wiki/Pattern_matching)在面向对象中的简单（不完整）实现方式（鉴于抽象语法树代数数据类型的本质，这个抽象是很直观的）。而从面向对象角度而言，由于可拓展性的要求，visitor模式可以解耦类定义以及对类的操作，我们不需要在每次用语法树做不同的事的时候都在类内增加新的方法，而只需要实现新的visitor。一个简单的使用的例子见：[`sushi/ast/to-string.h`](https://github.com/Sushiscript/sushiscript/blob/master/include/sushi/ast/to-string.h)

### 语法与词法分析

编译器前端的工作是以线性字符流来构造抽象语法树。其中在逻辑上首先把线性的字符流转化成线性的Token流（Token实现见：[`sushi/ast/token.h`](https://github.com/Sushiscript/sushiscript/blob/master/include/sushi/lexer/token.h)），之后再用Token流来构造语法树。

#### 常规语法分析和词法分析

`sushiscript`的词法和语法分析器是完全手写的，且基于一个[`LookaheadStream`](https://github.com/Sushiscript/sushiscript/blob/master/include/sushi/lexer/detail/lookahead-stream.h)的抽象，这个抽象使得我们可以轻易地在不消耗字符流的情况下获取字符流头部的内容，这样不管在控制字符流（[`sushi/lexer/detail/source-stream.h`](https://github.com/Sushiscript/sushiscript/blob/master/include/sushi/lexer/detail/source-stream.h)或者token流上都非常方便。

常规的词法分析部分大致是一个不严谨的自动机，通过lookahead来判断下一步需要提取的token，[`sushi/lexer/context.cpp`](#link-todo)和[`sushi/lexer/lexeme.cpp`](#link-todo)包含了一般的lookahead之后dispatch，并且在这个过程中修改[自动机状态](#link-todo)。

常规的parser部分是常规的手写parser实现：不严谨的类LL(k)的top-down parser加上[operator precedence parser](https://en.wikipedia.org/wiki/Operator_precedence_parser)来处理二元运算符优先级的情况（[`sushi/parser/parser.cpp`](#link-todo) 包含了parser的完整实现，文件比较长不过都是基于语法结构的top-down parsing）。

#### 字符串插值与context

几乎整个前端的难度都落在实现编译期的[字符串插值](https://en.wikipedia.org/wiki/String_interpolation)上。简言之，为了实现字符串插值，lexer在提取字符串的过程中还需要检测“插值”开始的信号，并且在插值开始后要产生正常状态下的token，之后需要在“合适的时机”切换回提取字符串的模式。

因此我们引入上下文（context）的概念，在lexer提取字符串，路径等字面值时，从普通上下文进入一个“可被插值”的上下文，这个上下文中允许的字符依赖字面值的类型有不同，当然这个性质也被抽象出来了（见[`sushi/lexer/detail/character-config.h`](#link-todo)）。在可被插值上下文中，lexer检测插值的开始，插值开始后，lexer再次进入普通上下文。那么这个因插值产生的普通上下文何时结束呢？这里就需要parser的配合，parser需要在插值开始后，监测插值的结束，并且在插值结束后通知lexer返回之前的可插值上下文。可插值上下文的主要实现见[`sushi/lexer/context.h`](#link-todo)。插值过程中lexer和parser之间通过token传递信息的方式见**_文档2.6: Interpolation Context_**，对于可插值的字面值，见文档中词法结构中出现`<interpolation>`的位置。

剩下的问题就是解耦上下文产生token以及lexer和上下文交互的过程，这里的实现借鉴了[状态模式](https://en.wikipedia.org/wiki/State_pattern)，只不过这里的状态不仅仅是一个状态，而是一个状态栈。不同的上下文在产生token时，同时产生控制信息，告诉lexer是应该1. 进入一个新的上下文 2. 退出当前上下文 3. 保持当前上下文。lexer对上下文产生消息的反应见[`sushi/lexer/lexer.h: Lexer::ExecuteAction()`](#link-todo)

剩下的实现部分基本上就是一些苦力活了。

### 作用域生成与检查

本阶段的主要任务是 **生成作用域** 和 **检查重定义/未定义** ，该阶段会产出 `Scope` 和 `Environment` 两个结果。

Scope 即作用域，其中记录着该作用域中定义的标识符。在遍历语法树时，遇到 **变量定义**、**函数定义**、**基于range的for语句** 则要在相应作用域插入标识符，下面将这几种情况总称为“定义”。

Environment 中提供 通过`Program`（语法上函数体/if语句主体等称为`Program`）获取`Scope`、通过`Identifier`获取“使用它的”`Scope` 和 通过语法树中的表达式获取其类型 三个接口。其中除了获取类型的接口内部的数据由下一阶段生成，另外两个接口的内部数据由本阶段完成。

本阶段实现方法是，遍历语法树：

+ 当遇到 Program 时，生成新的 Scope ，并将该 Program 与该 Scope 的对应关系存入Environment
+ 当遇到对标识符的 **使用** 时，搜索该标识符是否已经在作用域（包括父作用域）中有定义，如果有，则将 该标识符 与 当前的（即使用该标识符的）Scope 的对应关系插入到Environment中；如果没有，则是未定义错误
+ 当遇到对标识符的 **定义** 时，搜索该标识符是否已经在 **同一个** 作用域中有定义，如果有，则是重定义错误；如果没有，则向当前所在的 Scope 插入该标识符的信息

该阶段的实现难度不大，主要难度在于如何测试生成的 Environment 和 Scope 的正确性。当前的测试方法是: 按从上到下的顺序得到所有 Scope 和所有 Identifier，测试时输入每个 Scope 中所有的标识符 和 每个 Identifier 对应的“使用它的” Scope 来进行验证。

### 类型推导与检查

类型推导负责在Environment中补充`Expression`和`Type`的对应关系。具体的体现形式是`Expression`在抽象语法树中的地址与`Type`的映射。

在类型推导上，_文档4.3与4.4_详细阐述了`sushiscript`中类型的推导规则，在这里也不详述。文档中的大部分规则和定义都是为了严谨考虑，所以看上去会比较生涩。简言之，在绝大多数的情况下都是十分常规的基于语法树自下而上的类型推导。由于目前`sushiscript`中没有泛型，类型系统整体来说规则也相对简单，在明确类型的推导规则，隐式转换规则以及正确地实现了`Type`这一抽象之后，在实现上基本没有什么特别大的难度。

`sushiscript`中类型的抽象见[`sushi/type-system/type.h`](#link-todo)，对于表达式类型推导以及语句中涉及的类型检查，分别见[`sushi/type-system/expression.cpp`](#link-todo)与[`sushi/type-system/statement.cpp`](#link-todo)。整个实现的核心是`expression.cpp`中的`SolveRequirement()`和`RequireSatisfy()`，剩下的对语法树的后序遍历，以及把推导出来的类型正确放到`Environment`中，以及正确地产生错误都是基于文档的苦力活了。

### 代码生成

代码生成是在以上阶段都完成之后，通过 lexer/parse 阶段 生成的语法树 和 作用域生成/类型推导与检查阶段 生成的 Environment 来生成最终的代码 (当前只支持生成为 `bash`)。

代码生成部分的核心部分——也是难点部分在于如何将语法树的内容完整地对应到目标语言中，这要求对目标语言也有较多的了解；此外，还要考虑如何在目标代码中实现一些前端语言有而目标语言没有的特性。下面阐述的是代码生成的几个难点问题

1. Scope: 在 bash 语言中，虽有 `local` 声明符，但是其作用只是将变量作用域限制在函数中，而在 if/for 等语句中不存在作用域，那么在代码生成中就需要一些操作让他们“变得”有作用域。其实现方法是
	1. 对所有 “定义” ，都会先搜索相同标识符是否在作用域（包括父作用域）中有定义，如果有，则添加特定后缀让它们在生成的代码不同
	2. 对所有的 Program 中定义的变量，在 Program 代码生成的最后会对所有新变量进行 `unset`
2. 某些在 `sushiscript` 中的表达式，在 `bash` 中需要翻译为 多条表达式/语句 ，对于这种情况，代码生成中所有表达式的翻译结果都会包含两部分，分别是 `code_before` 和 `val` —— `code_before` 是该表达式的前置语句， `val` 是该 `sushiscript` 表达式最终形成的 `bash` 表达式，通常 `val` 的形式是 `${identifier}`；某些情况下可能上层的翻译只想要 `bash` 表达式的 `bash` 标识符，因此翻译时除了生成 `code_before` 和 `val` ，还会生成 `raw_id` 表示这只是一个标识符。
3. 为了简化翻译代码、实现某些功能，代码生成会在 `bash` 脚本中加入一些“`sushiscript`内建变量/函数”，如：
	1. `bash` 中函数的 return 返回的是表示 Exit Status 的整数，而 `sushiscript` 支持的是 return 任何类型。代码生成实现的方法是使用 `_sushi_func_ret_` 变量，存储函数中的“返回值”
	2. `sushiscript` 支持对字符串使用 `*` 运算符，表示重复该字符串多次。代码生成实现的方法是内建一个函数 `_sushi_dup_str_` 来简化翻译

关于代码生成更详细的规则，在文档第 6 章有阐述，此处不作赘述。

### 整合与命令行接口

命令行接口部分工作总体难度较低，可以划分为
1. 制定命令行参数标准。这里参考了`python`，`javac`，`dotnet`等的参数设计，最终敲定。
2. 实现命令行参数处理部分并添加测试样例。这个就是简单的字符序列的处理，测试也比较容易。
3. 调用前几层的工作转换`sushiscript`为`shell script`。这是整个程序的核心部分，中间需要对前几个层次的异常做出处理。
4. 将各个`pipeline`部分串联起来，比如使用一个`config`来传递参数，`run`和`build`公用相同的核心处理函数。

## 未来工作

### Shell功能完善

- 正则表达式内置支持
- shell expansion (glob) 支持
- 更多内置工具函数

### 前端

- （基于组合子的）parser重构

### 类型系统

- 自定义类型支持
- 泛型支持

### 代码生成

- 语法树的优化
- 多后端支持（`powershell`, `Python`，...）

### 其它语言特性

- 一等函数（闭包）正式支持
- 语法宏支持
