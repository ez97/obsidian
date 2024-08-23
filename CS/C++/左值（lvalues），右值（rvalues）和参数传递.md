左值和右值的详细情况太复杂了，所有的值类型都为三种中的一种：

1. 纯右值（prvalue，pure rvalue），计算内置运算符（built-in operator）的值，或者初始化一个对象（拥有一个结果对象）
2. 消亡值（xvalue，expiring value），表示资源可以重用的对象
3. 左值（lvalue，left value），（其计算值）决定对象的身份

其中，广义左值（glvalue，generalized lvalue）包括xvalue和lvalue
接下来具体举例

## 左值（lvalue）

表达式（不一定是赋值表达式）后依然存在的持久对象。

比较罕见的对象有：

```C++
// 函数
void foo() {}
void baz()
{
    // `foo` is lvalue
    // address may be taken by built-in address-of operator
    void (*p)() = &foo;
}
// 模板参数对象
struct foo {};
template <foo a>
void baz()
{
    const foo* obj = &a;  // `a` is an lvalue, template parameter object
}
// 非static类成员函数（不能取地址），static是可以的，不依赖于对象实例化
struct foo
{
    void m() {} // non-static member function
};
void baz()
{
    foo a;
    // `a.m` is a prvalue, hence the address cannot be taken by built-in
    // address-of operator
    void (foo::*p1)() = &a.m; // ill-formed
    void (foo::*p2)() = &foo::m; // OK: pointer to member function
}

```

另外一个常见的是左值引用返回，常见于运算符重载的实现中（为了链式求值）

```C++
int& a_ref()
{
    static int a{3};
    return a;
}
 
void foo()
{
    a_ref() = 5;  // `a_ref()` is lvalue, function call whose return type is lvalue reference
}
```

此外就是一些根据意义就能方便判断的类型，比如`*p`, `a[n]`, `a.m`, `p->m`, `p->*mp`，使用它们可以“操控它们代表的对象”

`static_cast`也是可以将对象转换为左值引用的，比如`static_cast<int&>(x)`，一个常见的场景是派生类转换为基类的引用（注意这里体现了左值的多态性，即它表示的对象的动态类型（即派生类）不一定是其静态类型（即基类）），一般情况下是隐式进行的：

```C++
class Base {
public:
    int baseValue = 1;
};

class Derived : public Base{
public:
    int derivedValue = 2;
};

int main(){
    Derived d;
    Base& b = static_cast<Base&>(d);
    Base& b2 = d; // the same
    return 0;
}
```

不过在多重继承的情况下需要显示指定（否则编译器不知道转换成Base1还是Base2）

具体的特性有：
1.  左值到右值，数组到指针，函数到指针，均可以隐式转换为右值（C语言的包袱）
2.  可以用内置寻址运算符&获取地址，比如`&++i`，`&std::endl`
3.  可被修改，即可以在=，+=等运算符的左边
4.  可用作初始化左值引用，如`int& x = y`

## 纯右值（prvalue）

表达式结束后就不再存在的临时对象。
查了下cppref，基本没有什么反直觉的，注意字符串字面量不是就行了
性质和左值基本是反的，比较典型有不能取地址，不能赋值或修改
需要注意的是，右值可以用于初始化const左值引用，此时右值生命周期将会被延长：

```C++
const int& create_temp() {
    return 42;  // 返回右值
}
int main() {
    const int& ref1 = create_temp();
    {
        const int& ref2 = ref1;
        std::cout << ref2 << std::endl; 
    }  // ref2 离开作用域，但不影响临时对象的生命周期
    std::cout << ref1 << std::endl; 
    return 0;  // ref1 离开作用域，临时对象被销毁
}
```

## 消亡值（xvalue）

即将被销毁、却能够被移动的值，同时拥有glvalue和rvalue的性质

一个常见的场景是：

```C++
std::vector<int> foo() {  
    std::vector<int> temp = {1, 2, 3, 4};  
    return temp;  
}  
  
std::vector<int> v = foo();
```

C++11之后，这里的左值`temp`会被隐式`static_cast<std::vector<int> &&>(temp)`，进而避免了大块数据的复制

## 右值引用，以及转换关系

```C++
#include <iostream>  
#include <string>  
  
void reference(std::string& str) {  
    std::cout << "左值" << std::endl;  
}  
void reference(std::string&& str) {  
    std::cout << "右值" << std::endl;  
}  
  
int main()  
{  
    std::string lv1 = "string,"; // lv1 是一个左值  
    // std::string&& r1 = lv1; // 非法, 右值引用不能引用左值  
    std::string&& rv1 = std::move(lv1); // 合法, std::move可以将左值转移为右值  
    std::cout << rv1 << std::endl; // string,  
  
    const std::string& lv2 = lv1 + lv1; // 合法, 常量左值引用能够延长临时变量的生命周期  
    // lv2 += "Test"; // 非法, 常量引用无法被修改  
    std::cout << lv2 << std::endl; // string,string,  
  
    std::string&& rv2 = lv1 + lv2; // 合法, 右值引用延长临时对象生命周期  
    rv2 += "Test"; // 合法, 非常量引用能够修改临时变量  
    std::cout << rv2 << std::endl; // string,string,string,Test  
  
    reference(rv2); // 输出左值  
  
    return 0;  
}
```

比较需要注意的是，右值可以被const左值引用延长周期，但**非const左值引用不行**，而右值引用怎么都可以

另外需要注意，**右值引用本身是左值**，它满足可修改，有持久性的特性

一个简单的例子：

```C++
void foo(int&& v){}
int &&x = 2;
foo(x); // 非法，x是左值，而foo的实参必须是右值
```
## 引用折叠，auto&&和完美转发

比较弱智的语法细节

```C++
typedef int&& rref;
int n;
rref&& r4 = 1;
int&& && r5 = 1; // ill formed
```

与多级指针不同，C++明面上是禁止“对引用的引用的”，但实际上由于模板参数推导的需要，放开了它的限制。在使用typedef和template的时候，能够实际突破这个限制

```C++
void foo(int& v){}
void bar(int&& v){}
int x = 2;
int &lref = x; // left ref
int && rref = 3; // right ref
foo(x);
foo(3);
foo(lref);
bar(rref);
```

这里`v`是对传入参数的引用，但这里绑定的是引用的原始对象，跳过了“引用的引用”的陷阱
当右值引用引入后，由此产生了一套规则，即引用折叠（Reference collapsing）

| 函数形参类型 | 实参参数类型 | 推导后函数形参类型 |
| :----: | :----: | :-------: |
|   T&   |  左引用   |    T&     |
|   T&   |  右引用   |    T&     |
|  T&&   |  左引用   |    T&     |
|  T&&   |  右引用   |    T&&    |

对于（模板）函数，通用引用（Universal References）的方法是使用`T&&`，而lambda表达式中才能使用`auto&&`（这里需要注意，`T&&`才是特例，对于其它类型，比如`int&&`，是限定死参数的左/右值性的）

不过这样仍然有问题，因为传入进去的仍然是左值，实参作为左值引用和作为右值引用的信息丢失了，无论实参是`int&&`还是`int&`，在传参过程二次调用无法识别，`T&&`知道不代表二次调用的函数`bar`知道：

```C++
void bar(int&& v){
    cout << "right" << endl;
}
void bar(int& v){
    cout << "left" << endl;
}
template <typename T>
void foo(T&& v){
    bar(v);
}
int x = 2;
foo(x); // output : left
foo(2); // output : left
```

解决方法就是使用`std::forward`进行完美转发（Perfect Forwarding）

```C++
void bar(int&& v){
    cout << "right" << endl;
}
void bar(int& v){
    cout << "left" << endl;
}
template <typename T>
void foo(T&& v){
    bar(std::forward<T>(v));
}
foo(x); // output : left
foo(2); // output : right
```

从结果来看，和`static_cast<T&&>(v)`差不多

	TODO : forward的具体实现原理

```C++
template<typename _Tp>  
constexpr _Tp&& forward(typename std::remove_reference<_Tp>::type& __t) noexcept  
{ return static_cast<_Tp&&>(__t); }  
  
template<typename _Tp>  
constexpr _Tp&& forward(typename std::remove_reference<_Tp>::type&& __t) noexcept  
{  
    static_assert(!std::is_lvalue_reference<_Tp>::value, "template argument"  
        " substituting _Tp is an lvalue reference type");  
    return static_cast<_Tp&&>(__t);  
}
```
