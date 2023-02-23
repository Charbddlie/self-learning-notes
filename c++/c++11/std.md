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
