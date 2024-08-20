## 编译器自动生成的函数

在满足某些情况下，编译器会为类自动生成6种函数：
1. 默认构造函数（Default Constructor）
2. 拷贝构造函数（Copy Constructor）
3. 移动构造函数（Move Constructor） （C++11）
4. 析构函数（Destructor）
5. 拷贝赋值运算符（Copy Assignment Operator）
6. 移动赋值运算符（Move Assignment Operator）（C++11）

如果想要保留可以使用`default`

```C++
struct Pig{
    Pig(int x) {}
    Pig() = default;
};
```

## 列表初始化（Brace Initialization or List Initialization）

列表初始化不属于编译器自动生成的函数，参数的顺序与类成员声明顺序一致

```C++
struct Pig {
	std::string name;
	int weight;
}

int main(){
	Pig pig1 = {"name1", 100}; // right
	Pig pig2{"name2", 200};    // right
	// Pig pig3("name3", 300); // wrong（和C++98兼容）
}
```

也可以部分初始化，剩下的交给类定义内手动指定的初始化值。但是由于初始化列表里的值的顺序必须严格遵守类中成员的声明顺序，因此有默认值的必须后置

```C++
struct Pig{
    // std::string str = "123";
    // int x;
    int x;
    std::string str = "123";
};

int main(){
    Pig pig1{2};
    return 0;
}
```

C++20可以手动指定成员名称，可以规避掉这个问题

```C++
	Pig pig4{.weight = 400} // C++20
```

这种构造方式不能被`default`恢复

## 1. 默认构造函数（Default Constructor）

失效条件：
1. 显式定义了任何一种构造函数（包括含参的）
2. 类成员无默认构造函数
3. 基类没有默认构造函数

效果：对每一个成员都调用默认（无参数）构造函数

但对POD（Plain-old-data）不会，包括：
（1）基础类型，如`int`，`float`
（2）指针类型
（3）完全由上述类型组成的类
可以看出就是C原来有的特性。如本例中，`name`在这里被调用构造函数`std::string name()`，而`weight`和C里只声明不定义的`int`一样，是随机值。

```c++
struct Pig {
	std::string name;
	int weight;
}
```

手动指定初始化值可以避免POD问题。手动指定初始值的方法可以单独存在，也可以和初始化列表合并使用，只要不冲突就可以。

```C++
struct Pig {
	std::string name{"pig1"};
	// std::string name = "pig2"; 也是可以的
	// std::string name(pig2); 错误
	int weight{0};
	// int weight{}; 等价零初始化
	// 只有什么都不写会是随机值
}
// 混合使用等价：
struct Pig {
	std::string name;
	int weight{0};
	Pig() : name("pig1") {} 
}
```

注意这里的默认构造函数的语法是`Pig pig1`而不是`Pig pig()`，后者会被识别为函数调用
## 2. 拷贝构造函数（Copy Constructor）

失效条件：
1. 显式定义了拷贝构造函数
2. 定义了移动构造函数或移动赋值运算符
3. 任何非静态数据成员或基类没有拷贝构造函数


## explicit关键字

阻止隐式转换，这个直接照搬cppreference的最清晰

```C++
struct A
{
    A(int) { }      // converting constructor
    A(int, int) { } // converting constructor (C++11)
    operator bool() const { return true; }
};
 
struct B
{
    explicit B(int) { }
    explicit B(int, int) { }
    explicit operator bool() const { return true; }
};
 
int main()
{
    A a1 = 1;      // OK: copy-initialization selects A::A(int)
    /* 
    其中最坑的就是这种单变量初始化的隐式转换
    一个void foo(A a)函数，能够接受foo(1)的调用
    因为这里的1被隐式转换为了A(1)，非常有迷惑性
    */
    A a2(2);       // OK: direct-initialization selects A::A(int)
    A a3 {4, 5};   // OK: direct-list-initialization selects A::A(int, int)
    A a4 = {4, 5}; // OK: copy-list-initialization selects A::A(int, int)
    A a5 = (A)1;   // OK: explicit cast performs static_cast
    if (a1) { }    // OK: A::operator bool()
    bool na1 = a1; // OK: copy-initialization selects A::operator bool()
    bool na2 = static_cast<bool>(a1); // OK: static_cast performs direct-initialization
 
//  B b1 = 1;      // error: copy-initialization does not consider B::B(int)
    B b2(2);       // OK: direct-initialization selects B::B(int)
    B b3 {4, 5};   // OK: direct-list-initialization selects B::B(int, int)
//  B b4 = {4, 5}; // error: copy-list-initialization does not consider B::B(int, int)
    B b5 = (B)1;   // OK: explicit cast performs static_cast
    if (b2) { }    // OK: B::operator bool()
//  bool nb1 = b2; // error: copy-initialization does not consider B::operator bool()
    bool nb2 = static_cast<bool>(b2); // OK: static_cast performs direct-initialization
 
    [](...){}(a4, a5, na1, na2, b5, nb2); // may suppress "unused variable" warnings
}
```

	Note : 感觉还不如多看cppref，懒得写了