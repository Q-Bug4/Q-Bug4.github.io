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
3. 在所有`{`之前都不加空格，除了下面两种情况：
   1. @SomeAnnotation({a, b}) (no space is used)
   2. String[][] x = {{"foo"}}; (no space is required between {{, 见下面第8项)
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

