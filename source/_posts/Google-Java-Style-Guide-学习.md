---
title: Google Java Style Guide 学习
date: 2021-12-21 00:42:22
tags: learning-note
---

好久没有写Java了， 由于要重新维护xWash，打算将项目按照规范来进行重构，所以选择了[Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)来进行学习。同时，本文只记录一些我个人认为比较重要或者值得记录的信息。

## 2. 文件的基础要求
2.2. 文件编码应该为UTF-8

### 2.3.3 对于非ASCII的字符处理
官方文档原文： 
> The choice depends only on which makes the code easier to read and understand, although Unicode escapes outside string literals and comments are strongly discouraged.
> 翻译：在代码编写上，表现非ASCII字符的形式应该尽量让提高代码的可读性，尽管使用\uxxxx的形式来表现Unicode很可能会让人不太容易理解。

|              代码              |                                              好坏                                              |
| :---------------------------- | :-------------------------------------------------------------------------------------------- |
|   String unitAbbrev = "μs";    |                         Best: perfectly clear even without a comment.                          |
| String unitAbbrev = "\u03bcs"; |                       // "μs"	Allowed, but there's no reason to do this.                       |
| String unitAbbrev = "\u03bcs"; |              // Greek letter mu, "s"	Allowed, but awkward and prone to mistakes.               |
| String unitAbbrev = "\u03bcs"; |                           Poor: the reader has no idea what this is.                           |
|   return '\ufeff' + content;   |  // byte order mark	Good: use escapes for non-printable characters, and comment if necessary. |

一句话：除了特殊情况，哪一种编写方式能提高代码可读性就尽量用哪种。

## 3. 源文件格式
### 3.1 版权信息和许可证
如果文件有版权信息和许可证，那么应该将版权信息和许可证写在源文件中
### 3.2 package语句
package语句没有长度限制
### 3.3 import语句
#### 3.3.1 不使用通配符
#### 3.3.2 不受长度限制

#### 3.3.3 顺序与间隔
import语句遵循下面的规则：
1. 所有静态import在一个块中
2. 非静态import在另一个块中

如果存在同时静态import和非静态import，那么就使用一行空行来分隔开这两个块

而在每个块中，import的名要按照ASCII的顺序进行排序。（不知道是不是笔者理解有误，笔者觉得这个是没有必要的事情）
附上原文：
>Within each block the imported names appear in ASCII sort order. (Note: this is not the same as the import statements being in ASCII sort order, since '.' sorts before ';'.)

#### 3.3.4 不使用静态import导入**类**
不要用static import去导入静态内部类，使用普通的import语句进行导入即可。

### 3.4 类的声明
#### 3.4.1 一个文件只存在一个顶级类
#### 3.4.2 类成员的顺序
良好的类成员顺序可以提高代码的可读性，但是实际上并没有一个固定的类成员顺序的规则。比较重要的是类成员的顺序要按照一些“**逻辑上的顺序**”的规则来进行排序，这种顺序能让维护代码的人更好地了解类。

例如一个新增的方法如果按照"添加的先后顺序"来进行排序，那么这个新增的方案就会被放置在最后或者最前，这不算逻辑上的顺序。

**一句话：以提高代码的可读性为目的对类成员进行排序，让维护者可以更清晰的了解该类。**

##### 3.4.2.1 重载
当一个类有多个构造函数或者多个相同名字的方法时，不要将这些相同名字的方法分开存放， 而是将他们连续的存放在类中。

## 4 格式
> 名称解释： block-like constructs, 指代类、方法或者构造函数的类体或者方法体（即被花括号包围起来的部分），下称块结构。
> 原文：Terminology Note: block-like construct refers to the body of a class, method or constructor. Note that, by Section 4.8.3.1 on array initializers, any [array initializer](https://google.github.io/styleguide/javaguide.html#s4.8.3.1-array-initializers) may optionally be treated as if it were a block-like construct.


### 4.1 花括号
#### 4.1.1 可选的花括号
在使用`if` `else` `for` `do` `while`的时候，就算执行的代码块为空，或者只有一行代码，也要使用花括号。

#### 4.1.2 K&R style
这一节主要是说花括号的格式问题，对于非空的代码块和块结构，有如下规则：
1. 在`{`之前不要出现换行符
2. 在`{`之后接换行符
3. 在`}`之前接换行符
4. 当花括号表示当前的语句，方法，构造函数或者有名的类的结束时，在`}`后接换行符。例如下面的例子中`}`后跟着`else`，说明`if`语句还没有结束，所以`}`后面不会接换行符。

例子：
```java
return () -> {
  while (condition()) {
    method();
  }
};

return new MyClass() {
  @Override public void method() {
    if (condition()) {
      try {
        something();
      } catch (ProblemException e) {
        recover();
      }
    } else if (otherCondition()) {
      somethingElse();
    } else {
      lastThing();
    }
  }
};
```

#### 4.1.4 空块：简单格式
一个空块或者块结构可以遵守4.1.2中提到的`K&R style`。需要注意的是，对于空块来说，可以在`{`之后紧跟`}`而不是换行符，**除非**这个空块属于某一个带有多个块的语句(`if/else`或者`try/catch.finally`).

例子：
```java
  // ✅ 合格的格式
  void doNothing() {}

  // ✅ 也是合格的格式
  void doNothingElse() {
  }

  // ❌ 不合格的格式，在带有多个块的语句中，块不能出现{}的简单的格式
  try {
    doSomething();
  } catch (Exception e) {}
```

### 4.2 块缩进
缩进为2个空格

### 4.3 每行一个语句
原文：
>Each statement is followed by a line break.
笔者在这里有个疑问：怎样才算是一个statement?
例如:`new List<Integer>().stream().forEach(System.out::println)`这种应该要分行写还是一行？

### 4.4 每行字符数量限制：100
每行的字符数量应该少于100，这里的字符指的是**任何Unicode字符**

以下情况是例外，不需要遵守长度限制：
1. 一些长度无法限制在100字符以内的行，例如：Javadoc中的长链接，长JSNI方法引用
2. package和import语句
3. 注释中可能被直接复制到终端执行的命令行语句

### 4.5 分行
术语解释：**分行**，指当代码被分割称几行的行为。

原文如下：
> Terminology Note: When code that might otherwise legally occupy a single line is divided into multiple lines, this activity is called line-wrapping.


#### 4.5.1 换行的位置
换行的主要观点是：在更高的语法等级上分行。
1. 在非赋值语句中，应该在符号前换行
   1. 小数点`.`
   2. 方法作用域冒号`::`
   3. 类型限制中的`&`符号
   4. catch方法中的管道`|`符号
2. 在赋值语句中，应该在`=`后换行（在`=`前换行也可以接受）
3. 方法或者构造方法中的`(`需要紧跟在方法后
4. 逗号`,`紧跟在逗号之前的token
5. 关于lambda表达式，原文为：A line is never broken adjacent to the arrow in a lambda, except that a break may come immediately after the arrow if the body of the lambda consists of a single unbraced expression.

#### 4.5.2 持续缩进：至少4个空格
当出现分行时，换行之后的代码需要和源句子保持4个空格以上的缩进距离。

关于缩进问题，这里笔者认为还是保持所有地方缩进一致可能比较好，保持一致在编写代码时就不需要去过分关注缩进问题，不会出现偶尔忘记哪里哪里需要缩进多少，哪里哪里又是缩进多少。

### 4.6 空白
#### 4.6.1 垂直方向上的空白行
空行可以出现在：
1. 在类的字段，构造器，方法，内部类，静态和实例初始化器之间
   1. 在连续的代码中是否要使用空行来分割是可选的，如果要使用空行，那么需要做到将代码块分割成逻辑上的多个部分
2. 第三节提到的import语句中，静态和非静态的import需要使用空行分隔。

空行的出现是为了提高代码的可读性，例如将代码从逻辑层面上分开。鼓励在进行初始化的块之前添加空行；鼓励在初始化之后，或者类最后一个成员定义之后添加空行。

添加连续的空行是允许的，但是不建议这样做。
#### 4.6.2 水平方向上的空白
除了语言本身需要的空格，以及字面量，注释和Javadoc之外，代码中的空格只会出现在下面的几种情况中
1. 分隔带有`(`的保留字：`if` `for` `catch`
2. 分隔前面可能带有`{`的保留字：`else` `catch`
3. 在所有`{`之前都加空格，除了下面两种情况：
   1. @SomeAnnotation({a, b}) (no space is used)
   2. String[][] x = \{\{"foo"\}\}; (两个`{`之间不需要加空格，见下面第8项)
4. 在任何二元或者三元运算符的左右两边都需要加入空格
   1. `<T extends Foo & Bar>`
   2. `catch (FooException | BarException e)`
   3. `foreach`语法中的`:`
   4. lambda中的箭头`->`
   5. 以下情况除外：
      1. `::`
      2. `.`
5. 在`,:;`之后或者`)`之后加入空格
6. 注释行符号`//`的两边
7. 声明变量时，类型和变量之间需要空格
8. 初始化数组时，下面两种格式都是可以的
   1. `new int[] {5, 6}
   2. `new int[] { 5, 6 }
9. 在类型注解和`[]`之间需要加入空格

#### 4.6.3 水平对齐：非必需
这个不是很重要，举一个例子：
```java
private int x; // this is fine
private Color color; // this too

private int   x;      // permitted, but future edits
private Color color;  // may leave it unaligned
```

### 4.7 分组括号
例子： `a = (tmp == true) ? 1 : 0`

除了以下2种情况下之外，建议使用括号来提高代码可读性。
1. 不加入括号不会误导阅读代码的人
2. 加入括号不会提高代码可读性

同时，不能假设每个人都对Java中的计算优先级了如指掌。

### 4.8 特殊结构
#### 4.8.1 枚举类
每个逗号之后都会跟着一个枚举实例，每个枚举实例之间可以有空行。
例子：
```java
private enum Answer {
  YES {
    @Override public String toString() {
      return "yes";
    }
  },

  NO,
  MAYBE
}
```
如果一个枚举类中没有方法或者实例没有文档，那么可以按照有以下格式:`private enum Suit { CLUBS, HEARTS, SPADES, DIAMONDS }`

#### 4.8.2 变量声明
##### 4.8.2.1 一个变量一次声明
每次声明只能声明一个变量, 所以不能这样声明变量：`int a, b;`。

例外：在`for`循环的头部中，可以一次声明多个变量

##### 4.8.2.2 需要时再声明
局部变量在声明时应该尽量靠近第一次使用的地方。

#### 4.8.3 数组
##### 4.8.3.1 数组初始化：可以用块的形式初始化
当初始化数组时，可以按照下面的例子来进行初始化：
```java
new int[] {           new int[] {
  0, 1, 2, 3            0,
}                       1,
                        2,
new int[] {             3,
  0, 1,               }
  2, 3
}                     new int[]
                          {0, 1, 2, 3}
```
##### 4.8.3.2 不使用C语言格式的数组声明
`[]`紧跟在类型后而不是变量后

`String[] args`✅

`String args[]`❌

#### 4.8.4 switch语句
##### 4.8.4.1 缩进
缩进2个空格
##### 4.8.4.2 fall-through: 加注释
fall-through指的是`switch`语句中某个`case`不加`break`等中断语句, 从而进入下一个`case`执行下一个`case`的代码的过程。

允许使用fall-throught, 但是在使用的时候需要使用注释`// fall through`来注明存在fall-throught

例子：
```java
switch (input) {
  case 1:
  case 2:
    prepareOneOrTwo();
    // fall through
  case 3:
    handleOneTwoOrThree();
    break;
  default:
    handleLargeNumber(input);
}
```
需要注意的是`case 1:`并没有加注释。因为`case 1:`紧跟着`case 2:`, 不会造成误解。

##### 4.8.4.3 必需包含`default`
每个switch语句必需包含`default:`, 无论`default`中是否有代码

例外：用`switch`语句用在枚举变量上，在举例出所有可能性之后可以不使用`default`。

#### 4.8.5 注解
应用在类，方法，构造方法的注解后面应该紧跟着修饰的对象，如果有多个注解那么每个注解占一行，每个注解都是同级的。
```java
@Override
@Nullable
public String getNameIfPresent() { ... }
```
例外：只有一个没有参数的注解可以跟修饰对象出现在同一行：
```java
@Override public int hashCode() { ... }
```
如果注解用于修饰成员变量，即使注解带有参数，多个注解也可以和被成员变量处于同一行。
例如：
```java
@Partial @Mock DataLoader loader;
```
对于局部变量，类型和参数而言，没有固定的规则来规定格式。笔者认为，按照整个Google style的风格来说，只要能提高可读性即可。

#### 4.8.6 注释

##### 4.8.6.1 块注释
块注释有和代码一样的缩进等级： 2. 在使用`/* ... */`进行注释时，中间包括的每一行都需要以`*`开始，并且要和上一个带`*`的行对齐。
例如：
```java
/*
 * This is          // And so           /* Or you can
 * okay.            // is this.          * even do this. */
 */
```

#### 4.8.7 修饰符
修饰符的顺序如下：`public protected private abstract default static final transient volatile synchronized native strictfp`

#### 4.8.8 数字字面量
`long`类型的整数使用大写字母`L`作为后缀，不要使用小写字母`l`，防止和数字`1`搞混。例如：`3000000000L`而不是`3000000000l`

## 5 命名
### 5.1 变量命名
变量只使用ASCII字母以及数字，下划线。所以所有的变量名都可以用正则表达式`\w+`来进行匹配。

在Google Style里，不会使用任何特殊前缀或者后缀，例如下面的几个命名都不属于Google Style: `name_` `mName` `s_name` `kName`

### 5.2 变量类型规则
#### 5.2.1 包名
包名只使用小写字母，在连接多个单词时直接连接。例如：`com.example.deepspace`而不是`com.example.deepSpace`或者`com.example.depp_space`

#### 5.2.2 类命名
类使用首字母大写的驼峰式命名。

类名往往是名称或者名词短语。例如`Character` `ImmutableList`. 接口名同样也可以是名词或者名词短语，不过有时候也会是形容词或者形容词性短语（如：`Runnable`）

对于注解，没有一个固定的命名规范。

测试类的命名为：`被测试的类名` + `Test`. 例如：`HashTest` `HashIntegrationTest`

#### 5.2.3 方法名
方法名采用小写字母开头的驼峰式写法。

方法名通常是名词或者名词短语，例如`sendMessage` `stop`

下划线可以出现在Junit测试方法中，用于区分不同的测试用例，一个常用的格式为：`<methodUnderTest>_<state>`，一个例子：`pop_emptyStack`. 对于测试方法，没有一个唯一正确的命名规则。

#### 5.2.4 常量命名
常量使用大写字母和下划线进行命名，下划线用于分割不同的单词。

当一个字段的内容永远不会发生改变以及设置为常量后不会带来副作用时，该字段应该是被`static final`修饰的常量。例如基本类型变量，字符串，不变量，类型固定的不变集合。

如果一个实例中能被观测到的内容会发生改变，那么就不应该设置为常量。仅仅预计一个对象是不可变是不够的。例如：
```java
// Constants
static final int NUMBER = 5;
static final ImmutableList<String> NAMES = ImmutableList.of("Ed", "Ann");
static final ImmutableMap<String, Integer> AGES = ImmutableMap.of("Ed", 35, "Ann", 32);
static final Joiner COMMA_JOINER = Joiner.on(','); // because Joiner is immutable
static final SomeMutableType[] EMPTY_ARRAY = {};
enum SomeEnum { ENUM_CONSTANT }

// Not constants
static String nonFinal = "non-final";
final String nonStatic = "non-static";
static final Set<String> mutableCollection = new HashSet<String>();
static final ImmutableSet<SomeMutableType> mutableElements = ImmutableSet.of(mutable);
static final ImmutableMap<String, SomeMutableType> mutableValues =
    ImmutableMap.of("Ed", mutableInstance, "Ann", mutableInstance2);
static final Logger logger = Logger.getLogger(MyClass.getName());
static final String[] nonEmptyArray = {"these", "can", "change"};
```
同时，常量一般为名称或者名词短语

#### 5.2.5 变量命名
变量使用小写开头的驼峰式写法，一般为名词或名词短语。

#### 5.2.6 参数命名
参数使用小写开头的驼峰式写法，**不允许在`public`方法中使用只有一个字母的参数名。**
> 笔者注：public方法是用来暴露给外界查看的，如果使用一个字母的参数名那么很容易让使用该方法的人误解。同时笔者觉得私有方法可能尽量也不用一个字母为好。

#### 5.2.7 局部变量
局部变量使用小写开头的驼峰式写法。如果局部变量可以是常量，是不可变的，也不应该将其设置为常量，不应该将其按照常量的格式来命名。

#### 5.2.8 类型变量命名
每个类型变量应该遵循下面的其中一个规则：
1. 一个大写字母，可以在字母后带有一个数字`T` `E` `X` `T2`
2. 使用类命名的规范，在后面紧跟一个大写字母`T`. 例如：`RequestT` `FooBarT`

#### 5.3 驼峰命名的定义
有时我们需要将英语短语转化为驼峰式，例如一些缩写和一些比较不一样的结构：`IPv6` `iOS`. 为了提高可读性，在命名时我们采用下面的确定性的方案：

首先从名字的散文格式开始: 
> 笔者不是很理解这里的意思，放上原文：Beginning with the prose form of the name

1. 将短语转化为纯ASCII字符，同时去掉`'`号，例如：`Müller's algorithm`会被转化为`Muellers algorithm`
2. 以空格和其他表达符号为分隔符，将第1步的结果进行分隔。
   1. 推荐：如果一个单词在日常使用时已经含有驼峰式写法了，对其进行拆分。（例如：`AdWords` -> `ad words`）需要注意的是，例如`iOS`并不是一种驼峰式的写法，所以不建议拆分形如`iOS`的单词。
3. 将第2步中所有单词都变为小写，同时将除了第一个单词之外的每个单词的首字母变为大写
4. 按顺序连接所有第3步中的单词，就得到了一个符合驼峰式的命名

需要注意的是，有一些词是比较容易出错和被忽视的。
> 原文：Note that the casing of the original words is almost entirely disregarded.

例如：
| Prose form |	Correct |	Incorrect |
| ----------  | ------- | -------- |
| "XML HTTP request" |	XmlHttpRequest	| XMLHTTPRequest|
| "new customer ID" |	newCustomerId	| newCustomerID|
| "inner stopwatch" |	innerStopwatch	| innerStopWatch|
| "supports IPv6 on iOS?" |	supportsIpv6OnIos	| supportsIPv6OnIOS|
| "YouTube importer" |	YouTubeImporter 或  YoutubeImporter	| |

其中`YoutubeImporter`虽然是可以的，但是不推荐这种格式

> 注：在英文中有一些含糊不清的连字符单词，例如`nonempty`和`non-empty`都是正确的，所以方法名`checkNonempty`和`checkNonEmpty`都是正确的

## 6 编程习惯
### 6.1 `@Override`: 必须使用
必须给重写父类的方法加上`@Override`注解

### 6.2 异常捕获：不要什么都不做
一般来说，我们捕获到某个异常之后，至少都会将异常打印出来，或者重新将异常抛出，而不是什么都不做。如果确定在异常处理中要什么都不做的话，那么就加上一句注释来解释
例如：
```java
try {
  int i = Integer.parseInt(response);
  return handleNumericResponse(i);
} catch (NumberFormatException ok) {
  // it's not numeric; that's fine, just continue
}
return handleTextResponse(response);
```

在编写测试的时候，如果异常名是`expected`或者以`expected`开头，那么就不需要加注释。例如：
```java
try {
  emptyStack.pop();
  fail();
} catch (NoSuchElementException expected) {
}
```
### 6.3 静态成员：使用类调用静态成员
当我们使用静态成员时，必须使用类来调用静态成员而不是使用实例来调用静态成员：
```java
Foo aFoo = ...;
Foo.aStaticMethod(); // good
aFoo.aStaticMethod(); // bad
somethingThatYieldsAFoo().aStaticMethod(); // very bad
```

### 6.4 Finalizes: 不使用
一般情况下不要使用`Object.finalize`

>Tips: 如果一定要使用`finalize`方法，那么首先阅读一下[Effective Java Item 7](http://books.google.com/books?isbn=8131726592)的"Avoid Finalizers"，小心使用，不过还是建议别用。
>原文：Don't do it. If you absolutely must, first read and understand Effective Java Item 7, "Avoid Finalizers," very carefully, and then don't do it.

## 7 Javadoc
### 7.1 格式
#### 7.1.1 一般格式
基本的Javadoc块格式如下例子：
```java
/**
 * Multiple lines of Javadoc text are written here,
 * wrapped normally...
 */
public int method(String str) { ... }
```
或者也可以是一行：`/** An especially short bit of Javadoc. */`

一般来说使用Javadoc块格式都没有错。在Javadoc确实只需要一行就可以完成的情况下才使用一行的Javadoc. 需要注意的是使用一行的Javadoc需要在没有类似于`@return`的标签的时候才能使用。

#### 7.1.2 段落
使用一行以`*`开头的空行来隔开不同的段，每个段的开头都要有`<p>`标签，`<p>`标签和随后的单词之间没有空格。

#### 7.1.3 块标签
标准的块标签和出现的顺序如下：`@param` `@return` `@throws` `@deprecated`, 同时这四个标签一定要带有描述。

### 7.2 总结
每个Javadoc块都会以一个简短的总结开头。这个总结非常重要，当我们使用`IDEA`等IDE时，鼠标悬停在某个方法或者类上就会显示这个总结来对方法或者类进行描述。同时在生成文档时可以更好地对方法和类进行描述。
> 此处笔者还没有很清楚，可以查阅[原文](https://google.github.io/styleguide/javaguide.html#s7.2-summary-fragment)

### 7.3 Javadoc使用位置
至少要做到将Javadoc应用在所有`public`类和该类中所有`public` `protected`方法上。

#### 7.3.1 例外：自我解释型方法
Javadoc可以不用在某些非常**简单直观**的方法上，例如`getFoo`, 因为只看方法名就能知道方法做了什么，而且也没有必要在这么简单的方法上加上"Returns the foo".

> 特别注意：在某些带有相关信息的例子中不适用上述自我解释型方法的例外，例如：有一个方法`getCanonicalName`，这里就不能忽略Javadoc（如果忽略了Javadoc，那么阅读的人根本不知道这里的"canonical name"指代的是什么东西。

#### 7.3.2 例外：重写
Javadoc在重写的方法上不是必须的。

#### 7.3.4 不需要Javadoc的情景
原文：
Other classes and members have Javadoc as needed or desired.

Whenever an implementation comment would be used to define the overall purpose or behavior of a class or member, that comment is written as Javadoc instead (using /**).

Non-required Javadoc is not strictly required to follow the formatting rules of Sections 7.1.2, 7.1.3, and 7.2, though it is of course recommended.

