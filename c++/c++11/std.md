# std::function

https://www.cnblogs.com/heartchord/p/5017071.html

是函数包装器模板，能包装任何类型的可调用元素（**函数、函数指针、类成员函数指针或任意类型的函数对象**）

```c++
struct Minus
{
    int operator() (int i, int j)
    {
        return i - j;
    }
};
function<int(int, int)> f = Minus();//函数对象和普通函数处理方式一样
```

结合bind使用来包装类普通成员函数

```c++
Math m;
function<int(int, int)> f = bind(&Math::Minus<int>, &m, placeholders::_1, placeholders::_2);
//类普通成员函数的第一个参数是this指针，所以需要用bind
```

# std::result_of

https://blog.csdn.net/hanxiaoyong_/article/details/120618869

https://blog.csdn.net/qq_31175231/article/details/77165279

result_of主要用于目标函数定义的类型推导中，在C++中auto也会自动推导类型，但是初始值不赋值时，auto是不能推导出目标类型，但result_of是可以推导出类型。

result_of的用法如下：

```c++
template <class Fn, class... ArgTypes>
struct result_of<Fn(ArgTypes...)>
```

使用result_of推导的类型是函数Fn(ArgTypes...)的返回类型

```c++
// result_of example
#include <iostream>
#include <type_traits>
int fn(int) {return int();}                            // function
typedef int(&fn_ref)(int);                             // function reference
typedef int(*fn_ptr)(int);                             // function pointer
struct fn_class { int operator()(int i){return i;} };  // function-like class
int main() {
  typedef std::result_of<decltype(fn)&(int)>::type A;  // int，其中decltype(funName)返回函数funName的函数类型
  typedef std::result_of<fn_ref(int)>::type B;         // int
  typedef std::result_of<fn_ptr(int)>::type C;         // int
  typedef std::result_of<fn_class(int)>::type D;       // int
  std::cout << std::boolalpha;
  std::cout << "typedefs of int:" << std::endl;
  std::cout << "A: " << std::is_same<int,A>::value << std::endl;
  std::cout << "B: " << std::is_same<int,B>::value << std::endl;
  std::cout << "C: " << std::is_same<int,C>::value << std::endl;
  std::cout << "D: " << std::is_same<int,D>::value << std::endl;
  return 0;
}
```

# std::lock_guard

声明一个局部的lock_guard对象，在其构造函数中进行加锁，在其析构函数中进行解锁。最终的结果就是：在定义该局部对象的时候加锁（调用构造函数），出了该对象作用域的时候解锁（调用析构函数）。

```c++
#include <mutex>
void A(){
    {
        std::lock_guard<std::mutex> lock(mutex);
        std::cout << "获得锁" << i++ <<std::endl;
    }
    std::cout << "释放锁" <<std::endl;
}
```

也可以手动锁定，见c++多线程.md



# initializer_list

`std::initializer_list`（C++11标准）是一个标准库类型，用于提供一种方便的方式来初始化 STL 容器、构造函数等。它允许我们使用花括号 `{}` 进行列表初始化，同时可以保证类型安全和编译时错误检查。

`std::initializer_list` 是一个轻量级容器，只能读取其元素，不能修改，类似于 `const` 数组。它是一个模板类，定义在 `<initializer_list>` 头文件中。

`std::initializer_list` 的定义如下：

```cpp
cpp
namespace std {
    template <class T>
    class initializer_list {
    public:
        using value_type      = T;
        using reference       = const T&;
        using const_reference = const T&;
        using size_type       = size_t;

        const T* begin() const noexcept;
        const T* end() const noexcept;
        size_t size() const noexcept;
    };
}
```

通过 `std::initializer_list` 我们可以使用花括号 `{}` 初始化容器或对象，例如：

```cpp
c++
std::vector<int> v = {1, 2, 3, 4, 5};
std::pair<int, std::string> p = {1, "hello"};
```

在构造函数中使用 `std::initializer_list` 也很方便：

```cpp
c++
class MyClass {
public:
    MyClass(std::initializer_list<int> list) {
        for (auto it = list.begin(); it != list.end(); ++it) {
            // ...
        }
    }
};
```

总之，`std::initializer_list` 提供了一种方便的方式来进行列表初始化，可以使代码更加简洁、易于理解。
