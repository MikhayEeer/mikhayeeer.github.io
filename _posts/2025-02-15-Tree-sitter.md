---
title: Tree-sitter0.24利用Python进行代码解析AST
date: 2025-02-15
tags:
  - Python
  - Cpp
categories: Compilers
description: 解析器工具tree-sitter的介绍使用，目前的博客大部分不契合最新版本，函数接口有更新和删改；同时分析了tree-sitter生成的AST的基本结构
---
[文章CSDN链接-Treesitter0.24使用](https://blog.csdn.net/Topsort/article/details/145829089?fromshare=blogdetail&sharetype=blogdetail&sharerId=145829089&sharerefer=PC&sharesource=Topsort&sharefrom=from_link)

- tree-sitter AST 类型定义
- C语义
- LLVM IR 
# 0. 介绍
tree-sitter是 解析器 生成器 工具
为源文件构建具体的**语法树**
- 通用：多语言转换AST
- 快速
- Robust：错误语法也能解析
- 无依赖：纯C11编写，可以无缝嵌入，也是性能高的原因

tree-sitter支持**增量解析**：
代码出现小范围修改时，解析器不会重新解析，只更新出现变更的部分
## 术语
- 编译 compile
	将代码翻译为计算机执行的机器码；
	会进行语法错误检查、优化代码、生成可执行文件
- 解析 parse
	将代码拆解成计算机能理解的结构
	代码->结构化表示（语法树）
	解析器会检查语法
- 生成 generate
	把中间处理好的代码转换为目标代码(机器码或者汇编)
- 中间表示 IR Intermediate Representation
	IR是编译器内部的通用格式；让优化和跨平台更容易；例如LLVM的IR
- **AST 抽象语法树**
	树形结构来表示代码的逻辑
	不包含括号分号等无关信息，只保留代码的核心语义；
	不同的解析器生成的AST可能不同，但都是为了捕捉代码的语义
- CFG 控制流图
	表示执行流程，代码块与条件循环语句

- LLVM 
	LLVM 是一个编译器框架，使用LLVM IR进行中间表示；LLVM IR显式表示了内存操作（load store）
	Clang，基于LLVM的编译器

## 编译和解析
编译器阶段：
- 词法分析，源代码->tokens
- 语法分析，tokens->AST
- 语义分析，检查语义正确（变量声明与类型匹配等）
- IR生成 ，再优化
- 目标代码生成 Generate

解析器不会生成和执行代码，只进行分析
解析器主要是进行：
- 词法分析：输入字符串（源代码）->tokens
- 语法分析：tokens->语法树AST

# 1. Tree-sitter Parser的使用
提供了多语言的API

tree-sitter在2025年一月更新到了0.24
`Language`类中的`build_library`疑似被删除
```python
from tree-sitter import Language
print(dir(Language))
# 查看输出中有哪些方法，如果没有build_library说明方法已经删除
# 亲测0.19.0有，而0.24.0没有
# 0.21.3 还有build_library方法
```
[·PyPI --- tree-sitter 0.21.3 · PyPI](https://pypi.org/project/tree-sitter/0.21.3/)中的Build from source引用中提示：
`build_library`将于0.22.0删除

[tree-sitter0.24.0使用手册 · PyPI](https://pypi.org/project/tree-sitter/0.24.0/)
## 最新版本tree-sitter使用
截至2025 1月`0.24.0`的更新，删除了`build_library`方法和parser的`set_language`方法
### 配置环境和准备解析器
```shell
pip install tree-sitter

# 根据需要的语言下载解析器
pip install tree-sitter-cpp
pip install tree-sitter-python
```
### 解析

```python
import tree_sitter_python
import tree_sitter_cpp
from tree_sitter import Language, Parser

# 加载Language对象
PY_LANGAUGE = Language(tree_sitter_python.language())
CPP_LANGAUGE = Language(tree_sitter_cpp.language())

# 创建Parser并配置语言
cpp_parser = Parser(CPP_LANGUAGE)
# insdead of cpp_parser=Parser()   cpp_parser.set_language(CPP_LANGUAGE)

# 准备用于解析的代码段
cpp_code_snippet = '''
// Your Code
'''

# 进行解析
tree = cpp_parser.parse(
	bytes(
		cpp_code_snippet, "utf-8"
	)
)

# 获取AST的节点
root_node = tree.root_node

# 遍历AST
cursor = tree.walk() 
```
## 解析后的节点访问
单个节点的访问可以通过`TSNode`这个API去访问；
### 节点
```python
root_node = tree.root_node

root_node.type
# 类型，例如函数定义(function_definition)，字符串(string)
```
节点有哪些属性

| 属性          | 说明                       |
| ----------- | ------------------------ |
| type        | 类型                       |
| start_point | 节点在代码的开始位置，(row, column) |
| end_point   | 节点在代码的结束位置               |
| start_byte  |                          |
| end_byte    |                          |
| children    | 子节点列表                    |
| sexp()      | 返回S表达式,string            |
### cursor遍历
但是大量的访问还是使用光标`tree cursor`更加合适
`cursor`是一个带有状态的对象
```python
tree = parser.parse(
	bytes(code, 'utf-8')
)
cursor = tree.walk() # 创建了一个cursor对象
```
`cursor`的遍历
```python
cursor.goto_first_child()
# 成功返回True，并移动到第一个子节点； 否则返回False

cursor.goto_next_sibling()
# 成功返回True，并移动到下一个兄弟节点；  否则返回False

cursor.goto_parent()
# 返回到父节点，返回True； 否则返回False


print(f"Node type: {cursor.node.type}")

print(f"Text: {code[cursor.node.start_byte:cursor.node.end_byte].decode('utf-8')}")
# 获取节点的源代码
```
递归进行遍历
```python
# 递归进行遍历
def traverse(cursor):
    node = cursor.node
    # 处理当前节点

    # 如果有子节点，进入第一个子节点并递归遍历
    if cursor.goto_first_child():
        traverse(cursor)
        # 遍历完子节点后，返回父节点
        cursor.goto_parent()

    # 如果有兄弟节点，移动到下一个兄弟节点并递归遍历
    if cursor.goto_next_sibling():
        traverse(cursor)
```
## 查询Query
S-表达式 / sexp 
	文本形式的半结构化数据
提供了一些类似正则表达式的模式匹配方法
```json
(binary_expression (number_literal) (number_literal))
// 匹配二元表达式节点，其子节点都是数字

(binary_expression (string_literal))
// 匹配至少有一个子节点是字符串的二元表达式
```
也可以加上子节点具体的字段(field)来使得匹配更加具体
```json
(assignment_expression left: (member_expression object: (call_expression)))
// 匹配assignment表达式，其左子节点的对象应为调用函数
```
否定
```json
(class_declaration name: (identifier) @class_name !type_parameters)
// 匹配没有类型参数的类的声明
```
通配符节点 匹配任意节点
```json
(_)
```

### 捕获@
```json
((assignment_expression 
	left: (identifier) @variable_name // 捕获变量identifier，并命名为variable_name
	right: (number) @value)) // 捕获变量的值number，命名为value
```
`@`符号在tree-sitter的用法里是定义捕获组(Captures)
允许为匹配的节点指定一个名称
```python
query_code = """
((assignment_expression
   left: (identifier) @variable_name
   right: (number) @value))
"""
query = PY_LANGUAGE.query(query_code)

captures = query.captures(tree.root_node)

for node, capture_name in captures:
    text = code[node.start_byte: node.end_byte].decode('utf-8')
    if capture_name == "variable_name": # 第一个捕获的名字
        print(f"Variable name: {text}")
    elif capture_name == "value": # 第二个捕获的名字
        print(f"Value: {text}")
```
使用`@`也可以捕获调用的函数名和函数的参数列表

- 过滤匹配
```json
((assignment_expression
   left: (identifier) @variable_name
   right: (number) @value)
 (#eq? @value "42"))
```

### 具体使用
```python
tree = parser.parse(code)

query_pattern = '''
(function_definition
    name: (identifier) @function_name) ;
'''

query = CPP_LANGUAGE.query(query_pattern)

captures = query.captures(tree.root_node)
```

# Playground
链接地址[Playground - Tree-sitter](https://tree-sitter.github.io/tree-sitter/7-playground.html)

Playground生成的AST和解析出来的`tree = Parser.parse(byte(code_snippet, 'utf8'))`是一致的
> 有实机测试过，不过本机运行的tree没有初始化格式化，不方便阅读，后续使用Playground进行分析

## 代码示例：
```cpp
int mian(){
	int a[10];
    int b=2;
    if(b>5){
    	print("b is more than 5");
    }else{
    	prant("b is less than 5");
    }
    int c=-1;
    bool d = 2;
    remake 0;
}
```
## 生成的AST
```json
// [a, b] - [c, d] 表示第a行b列到c行d列
translation_unit [0, 0] - [12, 0] // translation_unit顶级节点，一个完整的翻译单元，代码共12行
  function_definition [0, 0] - [11, 1] // 函数定义，子节点有type,declarator,body
    type: primitive_type [0, 0] - [0, 3] // 函数返回类型，是基本类型
    declarator: function_declarator [0, 4] - [0, 10] // 函数声明，包括函数名和参数列表
      declarator: identifier [0, 4] - [0, 8] // 函数名存在位置
      parameters: parameter_list [0, 8] - [0, 10] // 参数列表
    body: compound_statement [0, 10] - [11, 1] // 函数体
      declaration [1, 1] - [1, 11] // 变量声明
        type: primitive_type [1, 1] - [1, 4] // 类型是基础类型
        declarator: array_declarator [1, 5] - [1, 10] //声明的是数组
          declarator: identifier [1, 5] - [1, 6] // 数组名
          size: number_literal [1, 7] - [1, 9] // 数组大小
      declaration [2, 4] - [2, 12]
        type: primitive_type [2, 4] - [2, 7]
        declarator: init_declarator [2, 8] - [2, 11] // 初始化变量
          declarator: identifier [2, 8] - [2, 9]
          value: number_literal [2, 10] - [2, 11] 初始化值
      if_statement [3, 4] - [7, 5] // 条件语句
        condition: condition_clause [3, 6] - [3, 11] //条件
          value: binary_expression [3, 7] - [3, 10] // 条件子节点是一个二元表达式
            left: identifier [3, 7] - [3, 8] // 左边 是变量
            right: number_literal [3, 9] - [3, 10] // 右边 是数字
        consequence: compound_statement [3, 11] - [5, 5] // true执行的代码块
          expression_statement [4, 5] - [4, 31] // 代码块的表达式
            call_expression [4, 5] - [4, 30] // 调用了一个函数
              function: identifier [4, 5] - [4, 10] // 调用的函数名
              arguments: argument_list [4, 10] - [4, 30] // 调用函数的参数列表
                string_literal [4, 11] - [4, 29] // 参数是一个字符串
                  string_content [4, 12] - [4, 28] // 字符串的实际内容
        alternative: else_clause [5, 5] - [7, 5] // else 分支
          compound_statement [5, 9] - [7, 5]
            expression_statement [6, 5] - [6, 31]
              call_expression [6, 5] - [6, 30]
                function: identifier [6, 5] - [6, 10]
                arguments: argument_list [6, 10] - [6, 30]
                  string_literal [6, 11] - [6, 29]
                    string_content [6, 12] - [6, 28]
      declaration [8, 4] - [8, 13]
        type: primitive_type [8, 4] - [8, 7]
        declarator: init_declarator [8, 8] - [8, 12]
          declarator: identifier [8, 8] - [8, 9]
          value: number_literal [8, 10] - [8, 12]
      declaration [9, 4] - [9, 15]
        type: primitive_type [9, 4] - [9, 8]
        declarator: init_declarator [9, 9] - [9, 14]
          declarator: identifier [9, 9] - [9, 10]
          value: number_literal [9, 13] - [9, 14]
      expression_statement [10, 4] - [10, 13] // 一个表达式
        identifier [10, 4] - [10, 10]
        ERROR [10, 11] - [10, 12] //出错了
// 如果把 remake 0; 语法错误纠正为 return 0;
      return_statement [10, 4] - [10, 13]
        number_literal [10, 11] - [10, 12]
```
tree-sitter生成的AST数据结构分析
## 指针表现
声明指针的时候，会有声明节点declaration
含有子节点类型和`pointer_declarator`

# ~~博客使用tree-sitter~~ \[已过时\]
### 配置环境
```shell
pip install tree-sitter
```
### 准备解析器
官方提供了不同的语言的parser
都在[官方的主页](github.com/tree-sitter)
如果想使用某个语言的parser，
在对应的仓库目录下创建目录`vendor`，再在这里去clone官方对应语言的仓库，提供的语言都在document的introducton有写
例如
```bash
git clone https://github.com/tree-sitter/tree-sitter-java
git clone https://github.com/tree-sitter/tree-sitter-python
git clone https://github.com/tree-sitter/tree-sitter-cpp
git clone https://github.com/tree-sitter/tree-sitter-c-sharp
git clone https://github.com/tree-sitter/tree-sitter-javascript
```
### 生成`*.so`
然后创建`build`目录，用于保存自定义的编译器文件`*.so`
```python
from tree_sitter import Language
Language.build_library(
	'build/my-languages.so',
	[
		'vendor/tree-sitter-cpp',
		'vendor/tree-sitter-c'
	]
)
```
上述代码运行后会生成`my-languages.so`文件

### 解析

进行一个解析测试
```python
from tree_sitter import Language, Parser

# 注意C++对应cpp
# 看仓库名称
CPP_LANGUAGE = Language('build/my-languages.so', 'cpp')
CS_LANGUAGE = Language('build/my-languages.so', 'c')

# 定义cpp parser对象
cpp_parser = Parser()
cpp_parser.set_language(CPP_LANGUAGE)

c_parser = Parser()
c_parser.set_language(C_LANGUAGE)

# 贴入解析的代码
cpp_code_snippet = '''
int mian{
  piantf("hell world");
  remake O;
}
'''

# 没报错就是成功
tree = cpp_parser.parse(bytes(cpp_code_snippet, "utf8"))
# 注意，root_node 才是可遍历的树节点
root_node = tree.root_node

print(root_node)

sexp = root_node.sexp() # 获取AST
print(sexp)

```

# LLVM IR
编译器将AST转换为IR

DFS访问所有AST节点，遍历过程中生成IR指令




