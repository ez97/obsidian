	Generic programming centers around the idea of abstracting from concrete, efficient algorithms to obtain generic algorithms that can be combined with different data representations to produce a wide variety of useful software.

## 1. 应用的角度

从应用的角度，模板编程的最直接解释是参数化类型（parameterized types ），即用类型（type）作为参数的类型。模板编程将算法与处理的数据类型解耦，以得到对广泛情形下的通用解决方案，提升了代码复用率并减少了开发成本。

不过如果愿意从更广泛的角度思考，模板编程是一个很有趣的思考编程语言设计理念的切入点。

## 2.  ”概念”的本质

正如`concept`这个关键词暗示的那样，模板函数操作的对象不局限于编程语言所规定的内建类型(built-in types)，而是一系列更抽象的概念。初学者可能会语法的用法，将其更直观地理解为对模板参数的约束（constraint）。

但其实这是一种上下颠倒。模板参数（typename）在没有约束的情况下可以接受任何类型，它可以是任何东西，但同时它也“什么都不是”。正是约束让参数变成概念，因而与其说是程序员用约束制造了只有目标类型才能通过的通道，不如说这个约束才是概念的“本质”（或者退一步讲，定义），BS在书中也提到了“鸭子类型”（ducktype）这个例子。

有学习过一点群论的人应该熟悉，在定义群的概念时我们是直接讲“群是满足XXX条件的集合”，而不是像高中那样联系一些直观的概念，再去谈它有什么“性质”。从最还原的角度来讲，计算机中我们对值（value）进行操作（operation），而这个操作的意义完全取决于操作的结果——这里的操作的结果不仅包括计算的规则，还包括是否能够操作。

因此C++中对其最直接的表达就是`require`表达式：

```C++
template<typename T>
concept Addable = requires(T a, T b) { 
	{ a + b } -> std::convertible_to<int>; 
};
// 只有操作有意义（valid）才能通过编译
```

注意这里`requires`是用来描述`concept Addable`的，`typename T`并不是对应的模板函数。后面如果要使用这个约束，则为：

```C++
template<Addable T>
void foo(T t){
    // do something
}
```

或者：

```C++
template<typename T>
requires 
void foo(T t){
    // do something
}
```

不想分步实现，嵌套使用也可以，类似于原地构造对象然后使用，但不推荐：

```C++
template<typename T>
requires requires(T a, T b) { 
	{ a + b } -> std::convertible_to<int>; 
}
void foo(T t){
    // do something
}
```

`requires`类似模板编程中的汇编，其实更多时候我们是去调库。

	TODO : 常用的concept

## 3. C++20之前的实现：SFINAE

`concept`是C++20提供的特性，在此之前程序员只能使用名为SFINAE（Substitution Failure Is Not An Error）的语言特性，即“代换失败不是错误”，当模板形参在替换成显式指定的类型或推导出的类型失败时，从重载集中丢弃这个特化，而非导致编译失败。

从编译的视角来看，作为静态多态（polymorphism）的一种，模板是生成代码的代码，（有可能）会根据程序的具体需求生成多份实现，编译器是无法或者很难一开始就为模板生成适用于全部场合的代码的。

因而编译器只会在模板被实例化的时候，去一个个“试”可能对应的模板。当对模板实参进行推导的过程，以及实参内部嵌套推导的过程中，只要有一处匹配不上（即非良构，ill-formed）时，就去尝试其它的模板，而不会直接报错（ERROR）。

需要说明的是，由于实例化本身具有成本，因此尽量在第一步，也就是模板本身实参推导的时候就触发SFINAE，而不要等到计算副作用出错（硬错误），可以节省性能并让编译器报的错更可读（这个技巧以后会不会被硬件和编译器的进步取消呢。。。），比如：

```C++
template<typename T> auto add(const T& t1, const T& t2) -> decltype(t1 + t2){ 
	std::puts("SFINAE +");
	return t1 + t2;
}
```

这里通过规定返回类型，保证了`t1`和`t2`是可加的，在第一步就完成了检查。

模块（`module`）也是C++20引入的特性，模块对模板编程的支持在于编译器生成模块时，是将其生成一个类似于抽象语法书（AST）的文件格式，包含全文件的全部信息，在模板实例化的时候可以直接通过模块文件快速获取模板的信息，而不需要访问原始代码。

## 4. 常用的语法和功能

### 4.1 auto推导与模板特化

用`auto`进行函数参数推导（C++14），和`template`相似：

```C++
int f(auto x) { 
	return x + 1; 
}

auto lambda = [](int x, int y) { return x + y; };
```

不过`auto`无法实现特化（specialization），比如：

```C++
template<>
auto f(const double& a, const int& b){
    return a - b;
}
// 全特化，函数/类/变量模板均可
template<typename T>
const char* s<T*> = "pointer"; 
// 偏特化，对指针这一类类型
template<typename T,typename T2>
const char* s = "?";
template<typename T2> 
const char* s<int, T2> = "T == int";
// 偏特化，指定某一类型
// 函数模板目前无法偏特化（C++23）
```

### 4.2 变长参数模板

变长参数模板从C++11开始，可以接受任意个模板参数：

```C++
template<typename... Ts>  
void magic(Ts... args) {  
    std::cout << sizeof...(args) << std::endl;  //sizeof... 是一个整体
}
// 任意多个Ts
template<typename T0, typename... Ts>  
void magic(T0 t0, Ts... args) {  
    std::cout << sizeof...(args) << std::endl;  //sizeof... 是一个整体
}
// 至少一个T0
```

解决方案有两种，一种是递归，另一种是C++17开始支持的折叠表达式：

```C++
template<typename...Args>
void print(const Args&...args) {
    ((std::cout << args << ' '), ...);
}
print("test", 1, 3.14); // test 1 3.14
```

一元右折叠 `(E 运算符 ...)` 成为 `(E1 运算符 (... 运算符 (EN-1 运算符 EN)))`
这里结合逗号运算符的性质，理解为对每个args依次调用就行了
除此之外还有一元左折叠（全部相反）和二元左右折叠，需要的时候再查

## 4. 从泛型编程到类型系统

`template`是生成`type`的机制，从这个角度来讲，`int`、`double`等编程语言自带的类型也没有那么特殊的地方（至少不是非它不可），对于用C入门的人来说，很容易把C当成天经地义的标准（甚至连一些历史遗留的强制转换问题都认为），这种错觉来自于C和底层实现的紧密联系。

按Bjarne Stroustrup的区分标准，`type`（从C继承而来的类型）指定了对象在内存上的存放方式（Specifies how an object is laid out in memory），对于关注底层优化的人来说这当然十分重要（比如缓存命中、并行运算等），但并不代表全部。现代编译器的优化在很多方面已经超过了绝大多数人类，比起教编译器怎么做事（比如`register`），告诉编译器更多信息，让编译器去放开手优化是更有效的选择。

而从上而下看，根据具体的场景选择合适的抽象方式和程度，可以给予程序员更强大的表达能力。引用SICP中的话就是：

	In general, we can think of data as defined by some collection of selectors and constructors, together with specified conditions that these procedures must fulfill in order to be a valid representation

而像领域特定语言（Domain-Specific Languages，DSL）正是一精神的最直接体现。

## 参考

1. A tour of C++ Third Edition
2. cppreference
3. 现代 C++ 模板教程 2024（mq白）
4. pl-docs : C & C++广义类型系统缺陷（FrankHB）
5. 现代C++教程：高速上手C++ 11/14/17/20（欧长坤）

2024/8/13