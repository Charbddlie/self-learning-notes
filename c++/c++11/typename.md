# typename

https://feihu.me/blog/2014/the-origin-and-usage-of-typename/

traits 技术非常重要 std很常用 //todo

c++ primer里面说

```c++
template<typename T>
//等价于
template<class T>
```

直接告诉编译器`T::iterator`是类型而不是变量，只需用`typename`修饰，否则可能会被识别成变量，然后这行代码就变成了乘法

```c++
template <class T>
void foo() {
    typename T::iterator * iter;//即使没有歧义，也需要加上typename声明
    // ...

}
```

## 规则

- typename在下面情况下禁止使用：    
  - 模板定义之外，即typename只能用于模板的定义中
  - 非限定类型，比如前面介绍过的`int`，`vector<int>`之类
  - 基类列表中，比如`template <class T> class C1 : T::InnerType`不能在`T::InnerType`前面加typename
  - 构造函数的初始化列表中
- 如果类型是依赖于模板参数的限定名，那么在它之前**必须**加typename(除非是基类列表，或者在类的初始化成员列表中)
- 其它情况下typename是**可选**的，也就是说对于一个不是依赖名的限定名，该名称是可选的，例如`vector<int> vi;`

> 使用关键字typename代替关键字class指定模板类型形参也许更为直观，毕竟，可以使用内置类型（非类类型）作为实际的类型形参，而且，typename更清楚地指明后面的名字是一个类型名。但是，关键字typename是作为标准C++的组成部分加入到C++中的，因此旧的程序更有可能只用关键字class。

所以最好

```c++
template<typename A>
void foo(){
    //...
}
```

