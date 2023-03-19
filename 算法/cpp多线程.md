## 互斥锁

```cpp
#include <mutex>

std::mutex my_mutex;

void my_func() {
    // Lock the mutex before accessing the shared resource
    std::lock_guard<std::mutex> guard(my_mutex);

    // Access the shared resource here
    // ...

    // The mutex will be automatically unlocked when the guard goes out of scope
}

int main() {
    // Create some threads to execute my_func()
    std::thread thread1(my_func);
    std::thread thread2(my_func);

    // Wait for the threads to finish
    thread1.join();
    thread2.join();

    return 0;
}

```

在上面的示例中，`std::mutex` 类对象 `my_mutex` 用于控制对共享资源的访问。`std::lock_guard` 类模板用于在使用互斥量时自动锁定和解锁互斥量。`std::lock_guard` 对象 `guard` 在构造函数中锁定互斥量，并在析构函数中自动解锁互斥量。

在使用互斥量时，需要确保在访问共享资源之前先锁定互斥量，并在访问完毕后及时释放互斥量。这可以通过使用 `std::lock_guard` 或 `std::unique_lock` 等 RAII 类模板来实现。当使用 RAII 类模板时，无需手动锁定或解锁互斥量，因为 RAII 对象的构造函数和析构函数会自动完成这些操作。

## 信号量

力扣不给用，只能用cv和mutex实现

```cpp
#include <semaphore>
// 信号量
sem_t semaphore;

void threadFunc(int threadId) {
    std::cout << "Thread " << threadId << " waiting..." << std::endl;

    // 等待信号量
    sem_wait(&semaphore);

    std::cout << "Thread " << threadId << " started." << std::endl;

    // 停留几秒钟
    std::this_thread::sleep_for(std::chrono::seconds(3));

    std::cout << "Thread " << threadId << " finished." << std::endl;

    // 发送信号量
    sem_post(&semaphore);
}

int main() {
    // 初始化信号量
    //sem 是指向要初始化的信号量的指针。
    //pshared 是一个标志，指示信号量是在进程间共享还是在同一进程内的线程之间共享。如果 pshared 的值为 0，则信号量将在同一进程内的线程之间共享；否则，信号量将在进程之间共享。
    //value 是信号量的初始值。如果信号量是二元信号量（即只能取 0 或 1），则 value 的值应为 0 或 1；否则，value 的值应该是一个正整数。
    sem_init(&semaphore, 0, 1);

    // 创建两个线程
    // 1.可调用对象	2.参数
    std::thread thread1(threadFunc, 1);
    std::thread thread2(threadFunc, 2);

    // 等待线程结束
    thread1.join();
    thread2.join();

    // 销毁信号量
    sem_destroy(&semaphore);

    return 0;
}
```

可以自己实现信号量（算法题中，不支持sem_init()时）

```cpp
class Semaphore {
private:
    int n_;
    mutex mu_;
    condition_variable cv_;

public:
    Semaphore(int n): n_{n} {}

public:
    void wait() {
        unique_lock<mutex> lock(mu_);
        if (!n_) {
            cv_.wait(lock, [this]{return n_;});//!!!需要手动让lambda捕获this
        }
        --n_;
    }

    void signal() {
        unique_lock<mutex> lock(mu_);
        ++n_;
        cv_.notify_one();
    }
};

//然后在题目给出的类中使用构造函数初始化它们（那不就可以使用原版信号量了？）
class A{
public:
    A(){ //题目给的
        
    }
}
```



## 条件变量

在 C++ 中，条件变量（condition variable）是一种用于多线程编程的同步原语，用于等待某个条件的变化。条件变量通常与互斥量（mutex）一起使用，以便线程能够安全地等待条件变量的变化。

使用条件变量需要遵循以下一般步骤：

1. 定义一个互斥量，用于控制对共享数据的访问。
2. 定义一个条件变量，用于等待条件的变化。
3. 在等待线程中，使用 `std::unique_lock` 对象锁定互斥量，并在条件变量上等待条件的变化。
4. 在更改条件的线程中，使用 `std::lock_guard` 或 `std::unique_lock` 对象锁定互斥量，更改条件，然后通知等待线程条件的变化。

下面是一个简单的示例，演示如何使用条件变量等待一个条件的变化：

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void worker() {
    // do some work ...
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one();
}

int main() {
    std::thread t(worker);
    {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [](){ return ready; });
    }
    std::cout << "Worker thread completed." << std::endl;
    t.join();
    return 0;
}
```

在上面的示例中，worker 线程执行某些工作，并在工作完成后更改 ready 标志，并通知等待条件变量的线程。在主线程中，使用 `std::unique_lock` 对象锁定互斥量，然后在条件变量上等待 ready 标志的变化。`cv.wait` 方法会将主线程阻塞，直到 ready 变为 true，然后继续执行下面的代码。

需要注意的是，在等待条件变量时，必须使用 `std::unique_lock` 对象锁定互斥量。这是因为 `std::condition_variable` 类的 wait 方法会自动释放锁，并在等待时阻塞线程。在通知条件变量后，等待线程会重新获取锁并继续执行。

```cpp
cv.notify_one();//唤醒等待在该条件变量上的一个线程（如果满足它wait的条件）
cv.notify_all();//唤醒所有等待的线程(wait条件满足)
```

cv.notify_one()不是唤醒一个必然满足条件的线程，如果线程不满足条件就会睡回笼觉，也不叫醒别的线程，就可能会死锁?

notify_all()对所有线程的条件都测试一遍，效率反而高。

## RAII对象

在 C++ 中，`unique_lock` 和 `lock_guard` 都是用于管理互斥量（mutex）的 RAII 类，它们可以确保在离开作用域时自动释放锁，从而避免忘记手动释放锁而导致死锁等问题。虽然它们的主要功能相同，但它们之间还是有一些区别的。

1. 所有权的转移

`unique_lock` 对象的所有权可以被转移，因此它可以用于支持更多的锁定策略。例如，可以将 `unique_lock` 对象作为函数参数传递，并在函数内部使用 `std::move` 转移所有权，以便在函数返回时自动释放锁。

相反，`lock_guard` 对象的所有权不能被转移，因此它通常用于简单的锁定场景，不需要更复杂的所有权管理。

2. 可以延迟锁定的时间

`unique_lock` 对象可以在构造时选择是否立即锁定互斥量，或者在后续时间手动锁定互斥量。这使得 `unique_lock` 对象更加灵活，可以用于实现更多的锁定策略。例如，可以使用 `std::defer_lock` 参数构造一个未锁定的 `unique_lock` 对象，然后在后续时间手动锁定互斥量。

相反，`lock_guard` 对象只能在构造时自动锁定互斥量，因此它无法延迟锁定的时间。

3. 可以在锁定期间释放和重新获取锁

`unique_lock` 对象可以在锁定期间释放锁，并在后续时间重新获取锁。这使得 `unique_lock` 对象更加灵活，可以用于实现更多的锁定策略。例如，**可以使用 `std::condition_variable` 在等待某个条件变量的过程中，释放锁以允许其他线程访问共享数据。**

相反，`lock_guard` 对象无法在锁定期间释放和重新获取锁，因此它不能用于这种情况。

总的来说，`unique_lock` 比 `lock_guard` 更灵活，可以支持更多的锁定策略，但也会带来一些额外的开销。在简单的锁定场景中，`lock_guard` 更加简洁和高效，而在需要更复杂的锁定策略时，`unique_lock` 则更加适合。