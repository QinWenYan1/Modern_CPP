# 📘 Chapter 17 - 并发与并行 (Concurrency and Parallelism)

> **章节定位**：C++ 拓展部分第一篇——素材取自《现代 C++ 教程：高速上手 C++》第 7 章，覆盖 C++11 并发编程语言层支持，上接《C++ Primer》语法体系，下启第 18 章工程工具链。  
> 本章涵盖：`std::thread` 线程基础、互斥量(mutex)与 RAII 锁、期物(future)与条件变量(condition variable)、原子操作(atomic)与内存模型(memory model)

---

## 📋 章节导航与重难点

### 17.1 [并行基础 (Thread Basics)](./note_17_1_thread_basics.md)
- **核心**：
  - C++11 首次将线程纳入语言标准（此前仅 `pthread` / Windows API，难以跨平台）
  - `std::thread` 创建执行的线程实例（`<thread>` 头文件、可调用目标）
  - 基本操作：`get_id()` 取线程 ID、`join()` 汇合等待
- **难点**：
  - 创建线程 ≠ 等待线程——两路执行流并发推进
  - 线程对象销毁前必须 `join()` 或 `detach()`，否则程序直接终止

### 17.2 [同步原语：互斥量、期物与条件变量 (Synchronization Primitives)](./note_17_2_synchronization_primitives.md)
- **核心**：
  - `std::mutex` 与临界区(critical section)；RAII 锁 `std::lock_guard`（异常安全）
  - `std::unique_lock`：独占所有权、任意位置 lock/unlock、缩小锁范围
  - 期物 `std::future` + `std::packaged_task`：`get_future()` / `wait()` / `get()`，可作线程同步屏障(barrier)
  - 条件变量 `std::condition_variable`：`notify_one()` / `notify_all()`，生产者-消费者模型
- **难点**：
  - 手动 `lock()`/`unlock()` 的异常安全陷阱——每个出口（含异常）都要解锁
  - `wait()` 必须用 **while** 包裹检查条件，防虚假唤醒(spurious wakeup)
  - 配合 `condition_variable::wait` 时必须使用 `std::unique_lock`
  - `notify_one` vs `notify_all` 的抉择；互斥锁排他性 → 无法真正并行消费

### 17.3 [原子操作与内存模型 (Atomic Operations and Memory Model)](./note_17_3_atomic_and_memory_model.md)
- **核心**：
  - `std::atomic`：把原子写最小化为单条 CPU 指令；`fetch_add` 等成员函数与运算符重载
  - `std::atomic<T>::is_lock_free`：原子可行性检查（CPU 架构与内存对齐）
  - 四种一致性模型：线性一致性 > 顺序一致性 > 因果一致性 > 最终一致性
  - 六种 `std::memory_order` → 四种同步模型：relaxed / release-consume / release-acquire / seq_cst（默认）
  - CAS 原语：`compare_exchange_weak` vs `compare_exchange_strong`
- **难点**：
  - `volatile` 不能做同步——竞争 + 指令重排 + CPU 乱序 → 未定义行为
  - happens-before 与屏障语义：release 是向后屏障、acquire 是向前屏障
  - `compare_exchange_weak` 可能虚假失败，必须配循环重试
  - 默认 `seq_cst` 最安全但有性能开销，放松内存序是进阶优化

---

## 💡 核心速查（原书总结）

C++11 语言层提供了并发编程的相关支持，本章介绍了 `std::thread`、`std::mutex`、`std::future` 这些并发编程中不可回避的重要工具；此外还介绍了 C++11 最重要的几个特性之一的『内存模型』，它们为 C++ 在标准化高性能计算中提供了重要的基础。

| 需求 | 工具 |
|---|---|
| 创建线程 | `std::thread` + `join()` / `detach()` |
| 互斥临界区 | `std::lock_guard`（简单）/ `std::unique_lock`（灵活） |
| 等异步结果 | `std::future`（配 `std::packaged_task`） |
| 等条件成立 | `std::condition_variable`（while + `wait()`） |
| 无锁计数/标志 | `std::atomic`（默认 `seq_cst`，按需放松） |

---

## ✏️ 原书习题

1. **线程池(Thread Pool)**：请编写一个简单的线程池，提供如下功能：
   ```cpp
   ThreadPool p(4); // 指定四个工作线程

   // 将任务在池中入队，并返回一个 std::future
   auto f = pool.enqueue([](int life) {
       return meaning;
   }, 42);

   // 从 future 中获得执行结果
   std::cout << f.get() << std::endl;
   ```
2. **原子互斥锁**：请使用 `std::atomic<bool>` 实现一个互斥锁。

> 💡 习题解答后续将补充到 `exercises/` 目录

---

## 📚 进一步阅读

- [C++ 并发编程（中文版）](https://book.douban.com/subject/26386925/)
- [线程支持库文档（cppreference）](https://en.cppreference.com/w/cpp/thread)
- Herlihy, M. P., & Wing, J. M. (1990). Linearizability: a correctness condition for concurrent objects. *ACM Transactions on Programming Languages and Systems*, 12(3), 463–492.

---

## 🔗 相关链接

← [上一章：第 16 章 - 模板与泛型编程](../Chapter16_templates_and_generic_programming/README.md)  
→ [下一章：第 18 章 - C++ 开发工具](../Chapter18_extension/README.md)  
↑ [返回根目录](../README.md)

---

> 🧵 *"并发不是让程序跑得更快，而是让程序在正确的前提下跑得更快。"*
