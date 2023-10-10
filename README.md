# minicc: a simple compiler for a subset of C89
## 一、简介
minicc是一个基于[flex](https://github.com/westes/flex)和[bison](https://www.gnu.org/software/bison/)的编译器前端，通过词法分析、语法分析、语义分析与中间代码生成实现了附录A描述的C89语法子集。

## 二、编译环境
- OS: Ubuntu 20.04
- Compiler: gcc 12.2.0
- Flex: 2.6.4
- Bison: 3.8.2

## 三、实现
### 1. 词法分析
通过`flex`和`bison`实现了基于附录A的C语言文法的识别与未定义词法单元的报错，同时完成了对注释、对八进制数、十六进制数、指数形式的浮点数的识别与检错。

### 2. 语法分析
通过多叉树构建程序的语法树，从而实现了对表达式、语句、条件判断、注释、函数定义与函数声明的分析与报错。

### 3. 语义分析
对语法分析得到的语法树，通过语法制导翻译实现基础的语义分析。其中:
1. 语法制导翻译：
    - 基于L属性文法，对语法树进行自顶向下分析。
    - 综合属性通过函数的参数传递，继承属性通过函数的返回值传递。
2. 符号表：
    - 实验采用散列表存储分析中建立的符号表，不同作用域的同名变量或函数通过一个链表连接。hash函数采取[P.J. Weinberger提出的算法](https://en.wikipedia.org/wiki/PJW_hash_function)。
    - 除了结构体外，其他符号均采取名等价的方案。
3. 作用域：
    - 作用域简单地区分了全局变量、结构体定义变量与局部变量，但在识别符号时只简单地实现了对外层作用域的访问，没有处理作用域级数相同的符号的辨别。
作用域的定义见下：
        ```
        enum {
            GLOBAL = 1, PARAM, LOCAL
        };
        ```
4. 类型表示：
    - 符号的类型分为变量VARIABLE与函数FUNTION。
    - Function类型包含了函数的返回类型与参数类型、个数；
    - Variable类型包含了变量类型与作用域级数。

本项目的词法分析基于以下假设：
1. 假设1：不会出现注释、八进制或十六进制整型常数、浮点型常数或者变量。
测试
2. 假设2：不会出现类型为结构体或高维数组（高于1维的数组）的变量。
3. 假设3：任何函数参数都只能为简单变量，也就是说，结构体和数组都不会作为参数传
入函数中。
4. 假设4：没有全局变量的使用，并且所有变量均不重名。
5. 假设5：函数不会返回结构体或数组类型的值。
6. 假设6：函数只会进行一次定义（没有函数声明）。
7. 假设7：输入文件中不包含任何词法、语法或语义错误（函数也必有return语句）。

### 4. 中间代码生成
通过对语法树进行语法制导翻译实现中间代码的生成，其中：
1. 中间代码的组织：实验采用双向链表组织中层次中间代码。
2. 中间代码的表示：中间代码InterCode 通过代码类型与操作数组成。
3. 数组与结构体的翻译：本实验只实现了一维数组的翻译，没有完成高维数组与结构体的翻译。

## 四、程序编译
在项目所在目录下，通过`make`实现编译。编译后的程序接受一个参数，用于指定需要编译的C程序，如：
```shell
./parser tests/A-1.cmm
```
输出的中间代码文件格式为`*.ir`。
如果需要运行`tests/`目录下的所有测试文件，执行：
```shell
make test
```
编译后，可以通过`make clean`清除编译结果。