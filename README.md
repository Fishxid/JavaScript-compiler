## JavaScript-compiler项目简介：
&emsp;&emsp;编译原理在编程世界中无处不在，是我们向高级或底层开发路上不得不要逾越的一道坎。编译原理比较复杂，我们不求写出一个完整的编译器，但掌握基本原理还是很有必要的。
<br/>
&emsp;&emsp;核心内容：自动机、上下文无关文法、自顶向下语法分析、中序转换为后序算法解决语法优先级问题、中间代码生成、内存分配、运行时分析、opcode生成等。
<br/>
&emsp;&emsp;理解不到位的地方还望斧正。


## 目录
### [01 词法分析](./01_词法分析/README.md)
### [02 语法分析](./02_语法分析/README.md)
### [03 中间代码生成](./03_中间代码生成/README.md)
### [04 运行时刻环境](./04_运行时刻环境/README.md)
### [05 目标代码生成](./05_目标代码生成/README.md)
### 源码目录结构：
```
src
├─common 公共库
├─parse  语法分析
│   ├─expression.ts  表达式
│   ├─exprParser.ts  表达式解析器
│   │─parser.ts   语法解析器
│   ├─statement.ts  陈述语句
│   └─terminal.ts  终结符
├─tokenizer 词法分析
│   └─tokenizer.ts  词法解析器        
└─tsconfig.json # ts项目配置 
``` 

## 编译器：
### 什么是编译器：
&emsp;&emsp;编译器就是将一种编程语言转换为另一种编程语言的程序
### 编译器的使用场景:
* 将高级代码编译成浏览器能识别的代码：例如vue中的.vue文件是无法被浏览器识别的，这时需要编译器将其编译成html文件才能正常显示。又如typescript编译成javascript，类似的还有Babel、ESLint、Stylus等等
* 热更新：接触过小程序开发的同学应该知道，小程序运行的环境禁止new Function，eval等方法的使用，导致我们无法直接执行字符串形式的动态代码。此外，许多平台也对这些JS自带的可执行动态代码的方法进行了限制，那么我们是没有任何办法了吗？既然如此，我们便可以用JS写一个解析器，让JS自己去运行自己。
* 开发跨平台工具：例如京东开源框架Taro，可以只书写一套代码，再通过Taro的编译工具，将源代码分别编译出可以在不同端（微信小程序、H5、App 端等）运行的代码。类似的还有Egret、Weex等等
* 其他常用工具：代码压缩、混淆等
* 用 JavaScript 写成的 JavaScript 解释器，意义是什么？ - caoglish的回答 - 知乎
https://www.zhihu.com/question/20004379/answer/20123641
### 编译流程：
* 常规编译过程：
  源码（source code） → 词法分析器（Lexical Analyzer） → 符号流（tokens） → 语法分析器（Syntax Analyzer） → 抽象语法树 → 语义分析（Semantics Analyzer） → 抽象语法树 → 中间代码生成（Intermediate Code/Language Generator） → 中间表现形式 → 代码优化器（Code Optimizer） → 中间表现形式 → 代码生成（Code Generator） → 目标机器语言
* C语言的编译过程：
  .c文件（源代码） → 预处理器（preprocessor） → .i文件 → 编译器（compiler） → .s文件（汇编码） → 汇编程序（assembler） → .o文件（可重定位机器码） → 链接器（Linker）/加载器  → .exe文件
* 早期Java的编译和解释执行过程：
  .java文件（源代码） → javac.exe进行编译 → .class字节码文件（中间码） → java.exe加载到虚拟机中 → 在虚拟机中解释执行字节码
* babel的编译过程：
  es6代码 → Babylon.parse → AST  → babel-traverse  → 新的AST → es5代码
* TypeScript的编译过程：
  .ts文件（源程序） → ts编译器 → .js文件*（目标程序）
* vue模板编译过程：
  <template></template> → parse(template.trim()) → AST → Optimize(ast) → 新的AST → generate(ast) → render函数
### 编译器和解释器：
* 编译器：
  <br/>
  ~&emsp;源程序 → 编译器 → 目标程序
  <br/>
  ~&emsp;编译器是将一种语言一次性转换为另一种语言，不包括执行过程。
* 解释器：
  <br/>
  ~&emsp;源程序 + 数据 → 解释器 → 输出
  <br/>
  ~&emsp;解释器是将一种语言一行一行的编译成机器语言，并执行。解释器是编译一行并且执行一行的,不生成可存储的目标代码。
* 例如：
  <br/>
  &emsp;&emsp;早期java，通过javac.exe编译器进行编译，将.java文件编译成 .class字节码文件，这个过程是编译，但没有执行代码，这是编译过程。<br/>
  &emsp;&emsp;.class+数据放入jvm虚拟机中，虚拟机将字节码再一行一行的编译成机器码并执行，这个过程是解释。
  <br/>
  &emsp;&emsp;由于一行一行的解释执行较慢，jvm后面还引入了jit，可以将常用的机器码直接保存下来，提升效率,这里就包含和编译和解释两个过程。
### 编译流程解析：
* 词法分析：词法分析的目标是将文本分割成一个个的“token”，例如：init、main、init、x、;、x、=、3、;、}等等。同时它可以去掉一些注释、空格、回车等等无效字符
* 语法分析：将tokens解析分配在解析树的层次结构中，生成AST。它使tokens组合结构更清晰、层次更明了
* 语法分析的本质：程序本质是一串字符串，但给字符串添加语法规则后，便有可能产生token，以及token之间产生关系。
* 语义分析：检查每个节点含义对每个节点进行修饰，例如cpu不能做整数乘以浮点数操作，需要将整数转换成浮点数，以满足计算机能够进行的浮点数乘以浮点数的操作
* 编译器前端：词法分析、语法分析和语义分析统称为编译器的前端
* 编译器前端的目标是生成AST(抽象语法树)
* AST：
  <br/>
  ~&emsp;不依赖源语言文法：如果按bnf文法解析源代码，解析为一个自定义的结构，那在解释这个自定义结构的时候，肯定是为bnf文法量身定制的。一旦这个语言有了升级，bnf文法发生变动，相应的，后端解释器也会做相应的改动，十分麻烦。抽象语法树，相当于一个后端解释器给前端定制的一个规范，无论语言规范怎么变动，只需要改变将源代码解析为抽象语法树的解析器就行了，解析抽象语法树的解释器并不用改动。
  <br/>
  ~&emsp;不依赖语言的细节：比如说，嵌套括号被隐含在树的结构中，并没有以节点的形式呈现。
* 中间代码生成：将AST中的表达式转换为三地址表达式
* 中间代码的意义：
  <br/>
  &emsp;&emsp;既然已经拿到AST，机器运行需要的又是二进制。为什么不直接翻译成二进制呢？其实到目前为止从技术上来说已经完全没有问题了。但是，我们有各种各样的操作系统，有不同的CPU类型，每一种的位数可能不同；寄存器能够使用的指令也不同，像是复杂指令集与精简指令集等；在进行各个平台的兼容之前，我们还需要替换一些底层函数。因为不同平台的汇编处理都是不一样的，AST并不能完美运行在各个硬件平台上，于是便在 AST 和多个平台的汇编代码中间，抽象出了一个中间码（Intermediate Representation），在中间码的设计里抹平了硬件平台造成的差异。中间码的强大之处在于跨平台，与语言无关。
  <br/>
  &emsp;&emsp;中间码存在的另外一个价值是提升后端编译的重用，比如我们定义好了一套中间码应该是长什么样子，那么后端机器码生成就是相对固定的。每一种语言只需要完成自己的编译器前端工作即可。这也是大家可以看到现在开发一门新语言速度比较快的原因。编译是绝大部分都可以重复使用的。
  <br/>
  &emsp;&emsp;而且为了接下来的优化工作，中间代码存在具有非凡的意义。因为有那么多的平台，如果有中间码我们可以把一些共性的优化都放到这里，可使程序的结构在逻辑上更为简单明确，特别是可使目标代码的优化比较容易实现中间代码，即为中间语言程序，中间语言的复杂性介于源程序语言和机器语言之间。
* 编译器后端：即三地址表达式运行时环境（虚拟机）
* 编译器执行流程简述：（Java、Node、C、C++）
  <br/>
  ~&emsp;AST → 中间代码 → 代码生成（二进制或汇编指令）
  <br/>
  ~&emsp;AST → 中间代码 → 字节码
* 代码优化：对三地表达式进行优化，提高用户的执行速度。例如：
  ```
  t1=inttofloat(60)
  t2=id3*t1
  t3=id2+t2
  id1=t3
  ```
  &emsp;&emsp;可以优化为：
  ```
  t1=id3*60.0
  di1=id2+t1
  ```
* 代码生成：将优化后的中间码生成汇编或二进制指令，cpu可以直接执行指令
  ```
  t1=id3*60.0
  di1=id2+t1
  ```
  &emsp;&emsp;生成汇编指令为：
  ```
  LD   R2,   id3
  MUL  R2,   R2,   #60.0
  LD   R1,   id2
  ADD  R1,   R1,   R2
  ST   id1,  R1
  ```
* 源码转换成汇编指令的实例过程：
  ```
  源码：                    中间码：           汇编码：
  var x=1                  0:x=1              0:set sp #1
  function(foo(y)){    foo 1:r=nil            1:inc sp #4
    return x+y             2:r=x+y            2:set sp #nil
  }                        3:return           3:inc sp #4
  foo(100)                 4:pass 100         4:add x y R
                           5:call foo 1       5:load R (sp-4)
                                              6:goto (sp-8)
                                              7:set sp #100
                                              8:inc sp #4
                                              9:set sp #11 (sp+4)
                                             10:goto #1
  ```


## 编译器后端学习前置知识：
### cpu:
* cpu主要由运算器、控制器、寄存器组成
* 运算器：信息处理
* 控制器：控制各种器件进行工作
* 寄存器：信息存储，比内存更快
### 机器码：
* cpu只能识别由0和1组成机器码
* 机器码由cpu直接运行，是执行速度最快的代码
* 机器码分2种：
  <br/>
  ~&emsp;数据：存放图片、数字、音频等数据类型等的数据最终编译链接成的机器码
  <br/>
  ~&emsp;[指令](https://www.zhihu.com/question/65385471/answer/231486020)：告诉计算机执行何种操作
  <br/>
  &emsp;&emsp;~~ 操作码(opcode)
  <br/>
  &emsp;&emsp;~~ 操作数 (operand)
* 什么是指令和数据？ 指令和数据是应用上的概念，同一串二进制代码，可以是指令，也可以是数据，这决定于我们的程序设计。例：1000100111011000，当被应用为数据时，它等于 89D8H，H 表示是十六进制。当被应用为指令时，它指的是 MOV AX, BX。不难看出，同一串二进制代码，应用不同，既可以作为指令使用，也可以作为数据来使用。在存储器中，指令和数据是没有任何区别的，它们都是二进制信息。唯一区分它们的方式就是，这些信息是通过哪种类型的总线来传输的，使用地址总线传输的信息是地址信息，使用数据总线传输的信息是数据信息，使用控制总线传输的信息是控制信息。
* 机器码难编写、难读懂、易出错
* cup不同，对二进制码的要求也不同。一套机器码在不同cpu中运行结果不同
* c、c++经由编译器直接生成机器码，但不能跨机器运行
### cpu与指令集的关系：
* 每种型号的CPU，都有自己特有的指令集。
* cpu依靠（机器）指令来计算和控制系统，每款CPU在设计时就规定了一些列与其硬件电路相配合的指令系统，或者说某款cpu的硬件设计其实就是针对某个指令集的一种硬件实现。
* 指令集也就是所谓的目标代码（或称为机器代码，是可以直接在CPU上运行的代码）可以看作是要求cpu对外提供的功能，某款CPU的设计肯定是朝着某个指令集来的。所以，不同的cpu架构，也表示这他的指令集要么较之前的指令集有所拓展或者就是实现了一种全新的指令集。指令集中的一条指令，就是让cpu完成一系列的动作，而该动作的完成则表明了某种运算的完成。一个功能可能需要一条或几条指令来实现。比如汇编的MOV或者LD语句就可能对应着几条cpu指令。
### 汇编码：
* 机器码是0和1组成的二进制序列，可读性极差，而汇编码是比机器码更容易被人理解的代码
* 汇编语言由以下3类指令组成：
    <br/>
  ~&emsp;汇编指令：就是把特定的0和1序列，简化成对应的指令（一般为英文简写，如mov，inc等），可读性稍好
    <br/>
  ~&emsp;伪指令：为了编程方便，对部分汇编指令做的封装就是伪指令
    <br/>
  ~&emsp;其它符号
* 汇编指令栗子：
    <br/>
  ~&emsp;move指令：move ax,19;    //将数字19送入寄存器ax中，即ax=19
    <br/>
  ~&emsp;add指令：add ax,18;    //将寄存器ax的只加18，即ax=19+18
* 下面的机器指令和汇编指令是一一对应的，它们操作的含义都是：把寄存器 BX 中的内容送到 AX 中。
  <br/>
  机器指令：  1000100111011000
  <br/>
  汇编指令：  MOV  AX, BX
* [更多汇编>>](./专题/汇编.txt)
### 操作系统与编译的关系：
* 程序编译成机器码可以不需要依赖操作系统：https://www.zhihu.com/question/49580321
* 只要是运行在某操作系统之上的程序都会烙上该操作系统的印，对操作系统有依赖，包括编译程序。不过这些程序对操作系统的依赖程度和依赖的内容确实有很多区别。例如一支最简单的【Hello world程序】都会对【操作系统的C库】产生依赖，如果去掉【Hello world程序】的输入输出功能，只作加减或逻辑运算，【Hello world程序】依然会对操作系统有少量依赖，因为【Hello world程序】由运行在该【操作系统上的编译程序】编译的，有特定的目标文件格式，并由该【操作系统的载入程序】载入内存运行。这种只【在形式上】对OS存在依赖的“无用”程序可谓是最独立于OS的程序。在此基础之上，其它程序都对OS有不同程度的依赖，依赖表现在对OS内的各种程序库的依赖，比如C标准库，POSIX系统库，线程库、网络库和其它基于这些基础库的第三方应用代码库。
* 例如：java是通过虚拟机实现的跨平台的，也就是只编写一个程序，但是它去不同平台上运行的时候，其实带了对应的jvm，不同的操作系统由对应不同的jvm，去帮这一个程序去解释，而不同的jvm由sum公司去实现和更新，而且它会把对不同操作系统的底层调用（API）进行封装，对开发者却提供了统一的Java API，这样就减少程序员去了解操作系统API的差异性
* 《JavaScript的功能是不是都是靠C或者C++这种编译语言提供的？》：https://www.zhihu.com/question/49176184/answer/116675413
* 编译依赖操作系统发生在链接阶段？？
### 链接器（Linker）：
* 链接器是一个程序，将一个或多个由编译器或汇编器生成的目标文件外加库链接为一个可执行文件。
* 例如，hello程序中调用了printf函数，它是每个C编译器都会提供的标准C库中的一个函数。printf函数存在于一个名为printf.o的单独的编译好了的目标文件中，而这个文件必须以某种方式合并到我们的hello.o程序中。链接器（ld）就负责处理这种合并。结果就得到hello文件，它是一个可执行目标文件，可以被加载到内存中。由系统执行。


## 其他补充：
### LLVM：
* 广义的LLVM其实就是指整个LLVM编译器架构，包括了前端、后端、优化器、众多的库函数以及很多的模块
* 狭义的LLVM其实就是聚焦于编译器后端功能（代码生成、代码优化、JIT等）的一系列模块和库。
* LLVM IR:假如有N种语言（C、OC、C++、Swift...）的前端，同时也有M个架构（模拟器、arm64、x86...）的target，是否就需要N*M个编译器？LLVM编译器架构的好处是将所有语言转换为同一种IR，即LLVM IR，这样后端共用一种即可，只需要N个编译器了
* Clang：LLVM项目下的一个高性能前段工具，它能把C/C++代码转为LLVM IR，它可以取代GCC编译器
* 完全需要我们手工，或者依靠其他工具如lex, yacc来做的事情，是从源代码到token的词法分析和从token到AST的语法分析。AST生成LLVM IR也可以简单（见：https://hacpai.com/article/1570000872211）。 LLVM有完善的后端将IR转换为机器码
* 《利用LLVM实现JS的编译器，创造属于自己的语言》：https://juejin.im/post/5b88d5ef51882542d733765c
* 《LLVM架构-编译原理》：https://www.jianshu.com/p/b51345a323e2
### V8引擎：
* 老的JS引擎, 生成的是字节码, 通过字节码编译器来运行
* 字节码的执行效率要低于直接在CPU上运行的机器码。于是Java做出了改进, 将字节码编译成机器码执行
* v8直接将js代码编译成机器码，不产生中间代码。然而, 这样会拉长编译时间, 并且生成的机器码占用了更多的内存，不利于缓存。于是在2017年4月改进了编译方法, Ignition新架构, 是支持混合模式的。混合的方式是将第一层调用编译成机器码, 第一层以上的调用编译成字节码。
* 其实，Ignition + TurboFan 的组合，就是字节码解释器 + JIT 编译器的黄金组合。这一黄金组合在很多 JS 引擎中都有所使用，例如微软的 Chakra，它首先解释执行字节码，然后观察执行情况，如果发现热点代码，那么后台的 JIT 就把字节码编译成高效代码，之后便只执行高效代码而不再解释执行字节码。
* 对于常见编译型语言（例如：Java）来说，编译步骤分为：词法分析->语法分析->语义检查->代码优化和字节码生成。
* 编译过程包含了非常重要的一步，就是代码优化；而且编译过程完成了对代码的解析，所以会比解释型语言快。而jit是对代码不断编译优化的过程，例如提前确定数据类型，所以jit对于重复执行代码速度更快
* 《JavaScript 语法解析、AST、V8、JIT》：https://cheogo.github.io/learn-javascript/201709/runtime.html
* V8 是一个用C++编写的开源运行时引擎。
* JavaScript => V8（C ++）=> 机器码
* V8 实现了 ECMA-262 中指定的名为 ECMAScript 的脚本。 ECMAScript 由 Ecma International 创建，用于标准化JavaScript。
* V8 可以独立运行，也可以嵌入到任何 C++ 程序中。它有一些钩子，允许你编写自己的C++代码供 JavaScript 使用。这实际上允许你通过将 V8 嵌入到 C++ 代码中来向 JavaScript 添加功能，以便使你的 C++ 代码实现比 ECMAScript 标准更多的功能。
* [V8引擎介绍](./专题/V8引擎介绍（httpszhuanlan.zhihu.comp27628685）.png)
* [V8如何生成机器码](./专题/V8如何生成机器码（httpswww.zhihu.comquestion57532509）.png)
### 字节码：
* Bytecode（字节码）是一种IR（中间表示）的形式。
* 通常说的“字节码”其实就是一种线性代码，它最重要的特征是：
  <br/>
  ~&emsp;为存储、传输或直接解释执行而设计，因而指令格式一般比较紧凑
    <br/>
  ~&emsp;只使用1字节或者2字节来编码指令的操作码（opcode）。“字节码”因此而得名。
    <br/>
* LLVM IR的二进制序列化形式叫做bitcode（比特码），原因是这种序列化格式更倾向最大限度的紧凑，里面可能会有窄于1字节的数据类型，所以特意不叫字节码而叫比特码。本质上并没啥特别的。
* 字节码中常常涉及的、但并非本质的一些点：
  <br/>
  ~&emsp;指令既可能是固定长度的（例如Android的dex字节码、Lua的字节码），也可能是可变长度的（例如JVM的Java字节码、CPython的字节码）
    <br/>
  ~&emsp;指令既可能是“基于寄存器”形式的（上面说的四地址、三地址代码），也可能是“基于栈”形式的（上面说的零地址代码）
* java字节码：
  <br/>
  ~&emsp;java文件通过编译器编译成.class文件，就是java字节码文件
    <br/>
  ~&emsp;java字节码的运行和软件环境、硬件环境无关
    <br/>
  ~&emsp;java字节码是一种抹平了不同cpu架构的机器码，字节码不能直接在任何一种cpu架构上运行，但由于非常接近机器码，可以非常快的被翻译为对应架构的机器码
    <br/>
  ~&emsp;.class字节码文件不能被cpu直接运行，需要java虚拟机直译成机器码才能被cpu运行
    <br/>
  ~&emsp;java一次编写，到处运行的原理：将java文件编译成字节码文件，再通过不同平台的不同java虚拟机（JVM）可以快速的转换为各个机器所需要的机器码。
    <br/>
  ~&emsp;因为java需要经过一次JVM转机器码才能运行，所以比c、c++直接编译机器码多一道流程，所以更慢一些


## 参考文档：
* 《编译原理》：Alfred V.Aho，机械工业出版社
* 《编译原理》：哈工大·陈鄞，https://www.bilibili.com/video/BV1zW411t7YE
* 《编译器实现Category》：https://www.hashcoding.net/categories/%E7%BC%96%E8%AF%91%E5%99%A8%E5%AE%9E%E7%8E%B0/
* 《博客园·dejavudwh的博客》：https://www.cnblogs.com/secoding/p/11193700.html
* 《RednaxelaFX写的文章/回答的导航帖》：https://zhuanlan.zhihu.com/p/25042028
* 《kyjm/compiler-in-js》：https://github.com/kyjm/compiler-in-js
* 《jquery/esprima》：https://github.com/jquery/esprima
* 《estools/escodegen》：https://github.com/estools/escodegen
