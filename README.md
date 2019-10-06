cJSON是超轻量级的JSON解析器。
@[toc]

----------------

在介绍项目前，先说说JSON，如果你已经了解，可以跳过这一部分。


#### JSON介绍

JSON介绍网站 [json.org](http://www.json.org/)。

>JSON，即javascript对象表示法，是一种轻量级的数据交换格式，它基于JavaScript编程语言标准ECMA-262第三版（1999年12月）的子集 。


>JSON包含6种形式：
>- **对象**是一组无序`键/值对`。对象以左括号开头，以右括号结尾。每个键后面跟着冒号，键/值对之间被逗号分开
>- **数组**是值的有序集合。数组以左括号开头，以右括号结尾。值之间由逗号分开
>- **值**可以是字符串、数字、true、false或null、对象或数组。这些结构可以嵌套。
>- **字符串**是一个包含零个或多个Unicode字符的序列，用双引号括起来，并使用反斜杠转义。字符串非常像c或java字符串。
>- **数字**非常像C或Java数字，只是不使用八进制和十六进制格式。
>- **空白**

看一个json的例子：

```json
{
    "animals": {
        "dog": [
            {
                "name": "Rufus",
                "age":15
            },
            {
                "name": "Marty",
                "age": null
            }
        ]
	}
}
```
我们平时上网的时候，会从网络服务器获得数据，服务器查找到数据之后，把数据转换成 JSON 文本格式，然后网页的脚本代码就可以把此 JSON 文本解析为内部的数据结构去使用，最终将网页展示在你的面前。

json是一种树状结构，其中每一个结点的内容都只限制于下面的 6 种数据类型。
1. null: 表示为 null
2. boolean: 表示为 true 或 false
3. number: 一般的浮点数表示方式
4. string: 表示为 "..."
5. array: 表示为 [ ... ]
6. object: 表示为 { ... }

在设计 JSON 时会使用到语法规则与相应的解释如下：

```js
JSON-text = ws value ws   //JSON 文本由 3 部分组成，首先是空白（whitespace），接着是一个值，最后是空白。
ws = *(%x20 / %x09 / %x0A / %x0D)     //当中 %xhh 表示以 16 进制表示的字符；所谓空白，是由零或多个空格符（space U+0020）、制表符（tab U+0009）、换行符（LF U+000A）、回车符（CR U+000D）所组成。
value = null / false / true / number / string / array / object  // '/'表示多选一，所以值可能是 null、false 、 true，等。

null  = "null"   //null 等于 “null”字面值
false = "false"
true  = "true"


/*
number 以十进制表示，由 4 部分顺序组成：负号、整数、小数、指数。只有整数是必需部分。注意：+号是不合法的。

整数部分如果是 0 开始，只能是单个 0；而由 1-9 开始的话，可以加任意数量的数字（0-9）。也就是说，0123 不是一个合法的 JSON 数字。

小数部分比较直观，就是小数点后是一或多个数字（0-9）。

JSON 可使用科学记数法，指数部分由大写 E 或小写 e 开始，然后可有正负号，之后是一或多个数字（0-9）。
*/
number = [ "-" ] int [ frac ] [ exp ]
int = "0" / digit1-9 *digit
frac = "." 1*digit
exp = ("e" / "E") ["-" / "+"] 1*digit


/*
JSON 字符串是由前后两个双引号 quotation-mark 夹着零至多个字符，其中字符分为无转义字符 unescaped  或转义序列。转义序列有 9 种，都是以反斜线开始，比较特殊的是 \uXXXX，当中 XXXX 为 16 进位的 UTF-16 编码。
*/
string = quotation-mark *char quotation-mark
char = unescaped /
   escape (
       %x22 /          ; "    quotation mark  U+0022
       %x5C /          ; \    reverse solidus U+005C
       %x2F /          ; /    solidus         U+002F
       %x62 /          ; b    backspace       U+0008
       %x66 /          ; f    form feed       U+000C
       %x6E /          ; n    line feed       U+000A
       %x72 /          ; r    carriage return U+000D
       %x74 /          ; t    tab             U+0009
       %x75 4HEXDIG )  ; uXXXX                U+XXXX
escape = %x5C          ; \
quotation-mark = %x22  ; "
unescaped = %x20-21 / %x23-5B / %x5D-10FFFF


/*
一个数组可以包含零至多个value，而这些值可以字符，数字，也可以是数组；值与值之间以逗号分隔，。但注意 JSON数组 不接受末端额外的逗号。

%x5B 是左中括号 [
%x2C 是逗号 ,
%x5D 是右中括号 ]
ws 是空白字符
*/
array = %x5B ws [ value *( ws %x2C ws value ) ] ws %x5D


/*
一个对象可以包含零至多个 member，member是键值对，其中键是JSON字符串，值是任意JSON值。

%x7B 是左大括号 {
%x2C 是逗号 ,
%x7D 是右大括号 }
ws 是空白字符
*/
object = %x7B ws [ member *( ws %x2C ws member ) ] ws %x7D
member = string ws %x3A ws value  //%x3A 是冒号
```
----------------------------------



#### 项目cJSON介绍
我们最终设计的cJSON满足的功能有：

1. 符合标准的 JSON 解析器和生成器
2. 手写的递归下降解析器
3. 使用标准 C 语言（C89），以便支持尽可能多的平台和编译器
4. 跨平台／编译器
5. 仅支持 UTF-8 JSON 文本，仅支持以 double 存储 JSON number 类型

此外，在设计 JSON 中，我们还用到了：
- 测试驱动开发（test driven development, TDD）
- C 语言编程风格
- 数据结构
- API 设计
- 断言
- Unicode
- 浮点数
- Github、CMake、valgrind等工具

>首先说明，我们的cJSON是由单个.c文件以及头文件组成；作为一个库，CJSON的存在是为了尽可能消除繁琐的工作。

#### 源码用法
1. 复制源文件

因为整个库文件只有一个.c文件以及一个头文件，所以可以直接将文件复制到你的项目源。

2. 使用 CMake

[CMake下载链接](https://www.qqxiazai.com/down/11904.html)

首先是将 CMake 下载到自己电脑上，打开cmake-gui程序，在 "Where is the source code" 选择 json-tutorial/tutorial01，再在 "Where to build the binary" 键入上一个目录加上 /build；然后按Confuigure选择自己对应的编辑器，然后按 Generate 便会生成 VS 的 .sln 和 .vcproj 等文件；操作完成后点击 Open Project就可以在VS上看代码了。

在VS中，我们可以看到解决方案如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190909104034743.png)

右键点击leptjson_test，选择“设为启动项目”，然后编译运行后，可以看到：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190909104214884.png)

说明已成功搭建编译环境。

#### 需求分析

我们主要是完成 3 个需求：

1. 把服务器提供的杂乱无章的 JSON 文本解析为一个 JSON 树状数据结构
2. 提供接口访问该数据结构
3. 将 JSON 树状数据结构 变回 杂乱无章的 JSON 文本

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190911203008183.png)

最终我们将实现一个库，这个 JSON 库名为 leptjson；代码文件只有 3 个：
- leptjson.h：含有对外的类型和 API 函数声明。
- leptjson.c：含有内部的类型声明和函数实现。此文件会编译成库。
- test.c：我们使用测试驱动开发（test driven development, `TDD`）。此文件包含测试程序，需要链接 leptjson 库。

项目命名规则设计：使用项目的简写作为标识符的前缀lept。枚举值用全大写（如 LEPT_NULL），而类型及函数则用小写（如 lept_type）

#### 设计数据结构
JSON 数据 使用 struct 数据类型：
```c
typedef struct lept_value lept_value;

struct lept_value {
    union {
        struct { lept_member* m; size_t size; }o;   /* object: members, member count */
        struct { lept_value* e; size_t size; }a;    /* array:  elements, element count */
        struct { char* s; size_t len; }s;           /* string: null-terminated string, string length */
        double n;                                   /* number */
    }u;
    lept_type type;
};
```
其中  type 代表的是 JSON的类型，我们用一个枚举变量来存储它。
```c
typedef enum { LEPT_NULL, LEPT_FALSE, LEPT_TRUE, LEPT_NUMBER, LEPT_STRING, LEPT_ARRAY, 
LEPT_OBJECT } lept_type;
```
其中：
 LEPT_NULL：表示一个null值，存储它时只改变 type 的类型
 LEPT_FALSE：表示一个false布尔值，存储它时只改变 type 的类型
 LEPT_TRUE：表示一个true布尔值，存储它时只改变 type 的类型
 LEPT_NUMBER：表示一个数字值，它存储在 double n 里面。
 LEPT_STRING：表示一个字符串值。它以零终止字符串的形式存储在 struct s中。
 LEPT_ARRAY：表示一个数组值。它存储在 struct a 中。
LEPT_OBJECT：表示一个对象值。它存储在 struct o 中，其中lept_member又是一个结构体：

```c
typedef struct lept_member lept_member;

struct lept_member {
    char* k; size_t klen;   /* member key string, key string length */
    lept_value v;           /* member value */
};
```
#### API 设计

在设计API之前，我们先明确我们的需求：
1. 将杂乱无章的 JSON 文本解析为一个 JSON 树状数据结构
2. 提供接口访问 这个数据结构
3. 将 JSON 树状数据结构 变回 杂乱无章的 JSON 文本

所以我们设计的API如下：

```c
/*
函数目的： 初始化 JSON树状数据结构
参数：1. lept_value* v ：存储JSON树状结构的根节点指针 
返回值：void

为什么要用  do { ... } while(0) ？是为了把表达式转为语句，这样可以模仿无返回值的函数，所以就相当于：
void lept_init(ept_value* v){
    (v)->type = LEPT_NULL;
}
*/
#define lept_init(v) do { (v)->type = LEPT_NULL; } while(0)


/* 
函数目的：解析杂乱无章的 JSON 文本
参数：1. lept_value* v ：存储JSON树状结构的根节点指针 2. const char* json ：传入的 JSON 文本
返回值：int 数值，不同的数值表示了解析过程中的不同情况
*/
int lept_parse(lept_value* v, const char* json); 

//为了方便了解解析时发生的错误类型，我们对上面函数的返回值使用了一个枚举结构：

enum {
    LEPT_PARSE_OK = 0,  //解析过程正确
    
    //解析数字时可能会产生的错误码
    LEPT_PARSE_EXPECT_VALUE,
    LEPT_PARSE_INVALID_VALUE,
    LEPT_PARSE_ROOT_NOT_SINGULAR,
    LEPT_PARSE_NUMBER_TOO_BIG,
    
     //解析字符串时可能会产生的错误码 
    LEPT_PARSE_MISS_QUOTATION_MARK,  
    LEPT_PARSE_INVALID_STRING_ESCAPE,    
    LEPT_PARSE_INVALID_STRING_CHAR,     
    LEPT_PARSE_INVALID_UNICODE_HEX,
    LEPT_PARSE_INVALID_UNICODE_SURROGATE,
    
    //解析数组时可能会产生的错误码 
    LEPT_PARSE_MISS_COMMA_OR_SQUARE_BRACKET,
    
     //解析对象时可能会产生的错误码 
    LEPT_PARSE_MISS_KEY,
    LEPT_PARSE_MISS_COLON,
    LEPT_PARSE_MISS_COMMA_OR_CURLY_BRACKET
};

/* 
函数目的： 解析JSON树状数据结构
参数：1. lept_value* v ：存储JSON树状结构的根节点指针 2. size_t* length ：
返回值：char* ，指向解析后的 JSON 文本
*/
char* lept_stringify(const lept_value* v, size_t* length); 


/* 
函数目的： 释放空间
参数：1. lept_value* v ：存储JSON树状结构的根节点指针 
返回值：void
*/
void lept_free(lept_value* v);

/* 
函数目的： 获得JSON的类型
参数：1. lept_value* v ：存储JSON树状结构的根节点指针 
返回值：lept_type 表示 JSON的类型
*/
lept_type lept_get_type(const lept_value* v);

/* 
函数目的： 
参数：1. lept_value* v ：存储JSON树状结构的根节点指针 
*/
#define lept_set_null(v) lept_free(v)

/* 
函数目的： 获取布尔值
参数：1. lept_value* v ：存储JSON树状结构的根节点指针 
返回值：int
*/
int lept_get_boolean(const lept_value* v);

/* 
函数目的： 设置布尔值
参数：1. lept_value* v ：存储JSON树状结构的根节点指针 2. int b
返回值：void
*/
void lept_set_boolean(lept_value* v, int b);

/* 
函数目的： 获取对应JSON里面的数值
参数：1. lept_value* v ：存储JSON树状结构的根节点指针 
返回值：double 存储获得的数值
*/
double lept_get_number(const lept_value* v);
void lept_set_number(lept_value* v, double n);  //设置

/* 
函数目的： 获取对应JSON里面的字符串
参数：1. lept_value* v ：存储JSON树状结构的根节点指针 
返回值：char* 存储获得的字符串
*/
const char* lept_get_string(const lept_value* v);
size_t lept_get_string_length(const lept_value* v);  //获取对应JSON里面的字符串长度
void lept_set_string(lept_value* v, const char* s, size_t len);  //设置

/* 
函数目的： 获取对应JSON里面的数组长度
参数：1. lept_value* v ：存储JSON树状结构的根节点指针 
返回值：size_t 存储获得的数组长度
*/
size_t lept_get_array_size(const lept_value* v);
lept_value* lept_get_array_element(const lept_value* v, size_t index);  //获取指定数组某一项的内容


/* 
函数目的： 获取对应JSON里面的对象个数
参数：1. lept_value* v ：存储JSON树状结构的根节点指针 
返回值：size_t 存储获得的对象个数
*/
size_t lept_get_object_size(const lept_value* v);
const char* lept_get_object_key(const lept_value* v, size_t index);//获取某个对象的键
size_t lept_get_object_key_length(const lept_value* v, size_t index);//获取某个对象的键的长度
lept_value* lept_get_object_value(const lept_value* v, size_t index);//获取某个对象的值
```

