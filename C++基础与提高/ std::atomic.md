#  std::atomic



#### std::atomic_flag

std::atomic_flag是一个原子的布尔类型，可支持两种原子操作：

-   test_and_set, 如果atomic_flag对象被设置，则返回true; 如果atomic_flag对象未被设置，则设置之，返回false
-   clear. 清楚atomic_flag对象

　　std::atomic_flag可用于多线程之间的同步操作，类似于linux中的信号量。使用atomic_flag可实现mutex。下例子中的13、15配套使用对14上锁。

```C++
#include <iostream>
#include <atomic>
#include <vector>
#include <thread>
#include <sstream>

const int ths_num = 100;

std::atomic_flag lock = ATOMIC_FLAG_INIT;
std::stringstream stream;

void append_numer(int x) {
    while (lock.test_and_set());
    stream << "thread#" << x << "\n";
    lock.clear();
}

int main (int argc, char **argv) {
    std::vector<std::thread> ths;
    ths.reserve(ths_num);
    for (int i = 0; i < ths_num; ++i)
        ths.push_back(std::thread(append_numer, i));
    for (int i = 0; i < ths_num; ++i)
        ths[i].join();
    std::cout << stream.str();
    return 0;
}
```



####std::atomic

* 一般atomic原子操作，支持++,--,-=,+=其他的不支持

#### std::async

* async一般叫创建异步任务
* async有时候并不创建线程
* deferred延迟调用，并且不创建新线程，延迟到future对象调用get或者wait的时候才执行线程函数，如果没有调用get或者wait，那么mythread不会执行

##### std::launch::async

```
std::future<int> res = std::async(std::launch::async, mythread);
```
* 强制异步任务在新线程上执行

##### std::launch::async|std::launch::deferred
```
std::future<int> res = std::async(std::launch::async | std::launch::deferred, mythread);
```
* 可能是创建新线程立即执行
* 也可能是没有创建新线程，并且延迟到res.get()才开始执行任务入口函数

##### 不带额外参数，只给async函数一入口函数名
```
std::future<int> res = std::async(std::launch::async | std::launch::deferred, mythread);
```
* 默认值是std::launch::async|std::launch::deferred

##### 系统如何自行决定
* std::async和std::thread
* std::thread创建线程，如果系统资源紧张，创建线程异常，整个程序就会报异常错误
* std::async创建异步任务，可能创建，也可能不创建，容易拿到入口函数的返回值
* async创建失败不会引起系统崩溃，可能执行在主线程上，不创建新的线程
* 如果一定要创建std::launched::async, 可能是程序崩溃
* 默认写法判定系统是否创建线程需要wait_for

```
std::future_status status = result.wait_for(std::chrono::seconds(0));
if (status == std::future_status::deferred) {
  //delay exec 
}
```
