---
{"uid":null,"source":null,"aliases":null,"tags":["mutex","unique_lock","note"],"created":"2022-07-04 11:00:51","updated":"2023-03-07 17:10:42","title":"mutex 、unique_lock、lock_guard、conditional_variable 源码浅析","dg-publish":true,"permalink":"/Pages/mutex、unique_lock、lock_guard、conditional_variable 源码浅析/","dgPassFrontmatter":true,"noteIcon":""}
---


# Mutex 、unique_lock、lock_guard、conditional_variable 源码浅析

## Mutex 类

mutex 类继承自 `__mutex_base`，主要包括以下部分

1. 构造函数负责初始化 `pthread_mutex_t mutex `（结构体）
2. 析构函数负责清除结构体成员
3. 禁用拷贝构造，拷贝复制

```cpp
 class __mutex_base
  {
  protected:
    typedef __gthread_mutex_t			__native_type;

    __native_type  _M_mutex;

    __mutex_base() noexcept
    {
      // XXX EAGAIN, ENOMEM, EPERM, EBUSY(may), EINVAL(may)
      __GTHREAD_MUTEX_INIT_FUNCTION(&_M_mutex);
    }

    ~__mutex_base() noexcept { __gthread_mutex_destroy(&_M_mutex); }
#endif

    __mutex_base(const __mutex_base&) = delete;
    __mutex_base& operator=(const __mutex_base&) = delete;
  };

```

此外，`mutex` 在 `__mutex_base` 的基础上封装了 `lock` 与 `unlock` 方法，即对互斥量进行上锁与解锁操作，过程比较简单。

### Mutex 为什么需要禁止拷贝构造和赋值运算符？

具体见：[[Pages/Mutex类为什么需要禁止拷贝构造和赋值运算符？\|Mutex类为什么需要禁止拷贝构造和赋值运算符？]]

以下代码对互斥量测试，假如我们取消了 mutex 类禁止拷贝构造相关的代码，即允许 mutex 对象进行拷贝构造或者拷贝赋值，会发生什么？

```cpp
#include "header.h"

#include <mutex>
#include <pthread.h>

mutex m;
mutex b;
int cnt=0;

void *f2(void *){
    b.lock();
    for(int i=0; i<10; i++){
        cnt++;
    }
    b.unlock();
}
int main(){
    pthread_t t1, t2;
    m.lock();
    b = m;
    m.unlock();
    pthread_create(&t1, NULL, f2, NULL);
    pthread_create(&t2, NULL, f2, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    cout << cnt;
}
```

使用 gdb 来调试以上程序

```
(gdb) p m
$1 = {<std::__mutex_base> = {_M_mutex = {__data = {
        __lock = 0, __count = 0, __owner = 0, 
        __nusers = 0, __kind = 0, __spins = 0, 
        __elision = 0, __list = {__prev = 0x0, 
          __next = 0x0}}, 
      __size = '\000' <repeats 39 times>, 
      __align = 0}}, <No data fields>}

// 执行lock之后
(gdb) p m
$3 = {<std::__mutex_base> = {_M_mutex = {__data = {
        __lock = 1, __count = 0, __owner = 7522, 
        __nusers = 1, __kind = 0, __spins = 0, 
        __elision = 0, __list = {__prev = 0x0, 
          __next = 0x0}}, 
      __size = "\001\000\000\000\000\000\000\000b\035\000\000\001", '\000' <repeats 26 times>, 
      __align = 1}}, <No data fields>}

// 由于b复制的m，lock标志位为1，所以再次调用b.lock()时，进入阻塞
(gdb) thread 2
[Switching to thread 2 (Thread 0x7ffff7a4e700 (LWP 8569))]
#0  __lll_lock_wait (
    futex=futex@entry=0x5555555631c0 <b>, private=0)
    at lowlevellock.c:52
52      lowlevellock.c: No such file or directory.

```

## unique_lock

- 控制 mutex 的所有权  
- 在构造的时候可以选择对给定的锁上锁（默认），或者延迟上锁，或者尝试上锁  
- 构造完之后，可以执行 lock,unlock 操作  
- 析构的时候解锁  
- 支持拷贝构造和拷贝赋值，转移所的所有权

>![note]  
>只有在多线程环境内 unique_lock 构造的时候才会上锁

```cpp
// 只有在多线程环境下，该变量才会被激活
static inline int
__gthread_active_p (void)
{
  static void *const __gthread_active_ptr
    = __extension__ (void *) &GTHR_ACTIVE_PROXY;
  return __gthread_active_ptr != 0;
}
```

## lock_guard

- 同 c++17 的 scoped_lock 一样，实现了 RAII 机制（ [[Pages/同步原语\|同步原语]] ）  
- 只有构造函数和析构函数，构造时上锁，析构时解锁

## condition_variable

```cpp
// condition_variable example
#include <iostream>           // std::cout
#include <thread>             // std::thread
#include <mutex>              // std::mutex, std::unique_lock
#include <condition_variable> // std::condition_variable
 
std::mutex mtx;
std::condition_variable cv;
bool ready = false;
 
void print_id(int id) {
	std::unique_lock<std::mutex> lck(mtx);
	while (!ready) cv.wait(lck);
	// ...
	std::cout << "thread " << id << '\n';
}
 
void go() {
	std::unique_lock<std::mutex> lck(mtx);
	ready = true;
	// cv.notify_all(); // 这是重点
    cv.notify_one();
}
 
int main()
{
	std::thread threads[10];
	// spawn 10 threads:
	for (int i = 0; i < 10; ++i)
		threads[i] = std::thread(print_id, i);
 
	std::cout << "10 threads ready to race...\n";
	go();                       // go!
 
	for (auto& th : threads) th.join();
 
	return 0;
}
```

- `cond.notify_one() ` 只有一个线程获得锁，执行，其他线程仍阻塞  
- `cond.notify_all()` 所有线程都获得锁，打印完，结束

### 源码分析

- `cv` 通常配合 `unique_lock` 一起使用  
- `unique_lock` 先对 `mutex` 上锁

```cpp
void print_id(int id) {
	std::unique_lock<std::mutex> lck(mtx);
	while (!ready) {
		cv.wait(lck);
	}
	// ...
	std::cout << "thread " << id << '\n';
}
```

在 `cv.wait(lck)` 内部，`cv` 先对自己的 `mutex` 上锁，因为 `cv` 同时有多个线程在执行，同时如果有的线程先退出，那么 `cv` 的 `mutex` 将会析构，从而使得其他线程崩溃，所以 `mutex` 要转成要给 `shared_ptr`  
`__unlock` 对传来的锁进行解锁， 然后将独占锁转移到一个临时变量上，这样跳出 scope 之后，这一段能够反复执行

```cpp
    template<typename _Lock>
      void
      wait(_Lock& __lock)
      {
	shared_ptr<mutex> __mutex = _M_mutex;
	unique_lock<mutex> __my_lock(*__mutex);
	_Unlock<_Lock> __unlock(__lock);
	// *__mutex must be unlocked before re-locking __lock so move
	// ownership of *__mutex lock to an object with shorter lifetime.
	unique_lock<mutex> __my_lock2(std::move(__my_lock));
	_M_cond.wait(__my_lock2);
      }

```

- 使用谓词的 wait 能够避免唤醒丢失和惊群现象。
- 发送方在接收方进入等待前发送通知。
- 惊群是说即使没有发送通知，仍然可能唤醒
