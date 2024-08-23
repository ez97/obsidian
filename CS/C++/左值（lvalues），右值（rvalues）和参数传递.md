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

## 引用折叠



