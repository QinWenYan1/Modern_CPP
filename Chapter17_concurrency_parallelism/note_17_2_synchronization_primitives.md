# 📘 [第 17 章] 并发与并行 (Concurrency and Parallelism)

> 来源说明：《现代 C++ 教程：高速上手 C++》第 7 章「并行与并发」7.2–7.4 节 | 本节涵盖：`std::mutex` 与 RAII 锁（`lock_guard` / `unique_lock`）、期物 `std::future` 与 `std::packaged_task`、条件变量 `std::condition_variable` 与生产者-消费者模型

---

## 🧠 核心概念总览

- [*知识点1: 互斥量与临界区*](#id1)
- [*知识点2: std::mutex 与手动 lock/unlock 的陷阱*](#id2)
- [*知识点3: std::lock_guard：RAII 互斥锁*](#id3)
- [*知识点4: std::unique_lock：更灵活的独占锁*](#id4)
- [*知识点5: 期物 std::future：访问异步操作的结果*](#id5)
- [*知识点6: future 作为线程同步屏障*](#id6)
- [*知识点7: std::packaged_task：封装可调用目标*](#id7)
- [*知识点8: 条件变量 std::condition_variable：为避免死锁而生*](#id8)
- [*知识点9: 示例：生产者-消费者模型*](#id9)
- [*知识点10: notify_one vs notify_all 与互斥锁的粒度局限*](#id10)

---

<a id="id1"></a>
## ✅ 知识点1: 互斥量与临界区

**mutex（互斥量）是并发技术的核心之一，C++11 引入了 mutex 相关的类。**

我们在操作系统、亦或是数据库的相关知识中已经了解过了有关并发技术的基本知识，`mutex` 就是其中的核心之一。

- **互斥量(mutex)**：保证同一时刻只有一个线程进入**临界区(critical section)** 的同步原语
- C++11 引入了 `mutex` 相关的类，其所有相关的函数都放在 **`<mutex>` 头文件**中

> 📋 **术语提醒**：临界区(critical section) 指访问共享资源的那段代码——同一时刻只能被一个线程执行
> 🔄 **知识关联**：操作系统课中的互斥概念在此落地为 C++ 标准库类型——理论不变，接口标准化

---

<a id="id2"></a>
## ✅ 知识点2: std::mutex 与手动 lock/unlock 的陷阱

**`std::mutex` 是最基本的互斥量类，但实际编码中最好不要直接调用 `lock()`/`unlock()`——异常路径上极易漏解锁。**

- `std::mutex` 是 C++11 中**最基本的互斥量类**，通过构造 `std::mutex` 对象创建互斥量
- 成员函数 **`lock()`** 进行上锁，**`unlock()`** 进行解锁
- **实际编写代码的过程中，最好不去直接调用成员函数**：
  - 直接调用就需要在**每个临界区的出口处**调用 `unlock()`
  - **当然，还包括异常**——临界区中抛出异常时，`unlock()` 根本执行不到

> ⚠️ **关键警告**：手动 lock/unlock 的最大敌人是**异常安全**——任何一个异常出口都会让互斥量永远保持锁住状态，进而死锁
> 💡 **理解技巧**：「拿到锁就必须在每个出口还锁」听着简单，但 return、break、throw 都是出口——这正是 RAII 要解决的问题（见知识点3）

---

<a id="id3"></a>
## ✅ 知识点3: std::lock_guard：RAII 互斥锁

**`std::lock_guard` 用 RAII 机制管理互斥量：构造上锁、析构解锁，天生异常安全。**

- C++11 为互斥量提供了一个 **RAII 机制的模板类 `std::lock_guard`**
- **RAII 在不失代码简洁性的同时，很好地保证了代码的异常安全性**
- RAII 用法下，对于临界区的互斥量的创建只需要在**作用域的开始部分**完成
- **异常安全原理**：C++ 保证了所有**栈对象在生命周期结束时会被销毁**——无论函数正常返回、还是在中途抛出异常，都会引发**栈回溯(stack unwinding)**，也就自动调用了 `unlock()`

**示例/实践**
```cpp
#include <iostream>
#include <mutex>
#include <thread>

int v = 1;

void critical_section(int change_v) {
    static std::mutex mtx;
    std::lock_guard<std::mutex> lock(mtx);

    // 执行竞争操作
    v = change_v;

    // 离开此作用域后 mtx 会被释放
}

int main() {
    std::thread t1(critical_section, 2), t2(critical_section, 3);
    t1.join();
    t2.join();

    std::cout << v << std::endl;
    return 0;
}
```

> ⚠️ **细节警告**：原书注解——若抛出的异常**没有被捕获**，此时**由实现定义是否进行栈回溯**（锁未必释放）
> 💡 **理解技巧**：`lock_guard` = 「作用域锁」——它的生命周期就是锁的持有期，出了大括号必然解锁
> 🔄 **知识关联**：RAII 是 C++ 核心惯用法，与智能指针管理内存是同一思想（见第 12 章动态内存与智能指针）

---

<a id="id4"></a>
## ✅ 知识点4: std::unique_lock：更灵活的独占锁

**`std::unique_lock` 以独占所有权方式管理互斥量的上锁/解锁，可在任意位置显式 lock/unlock，是并发编程的推荐选择。**

- `std::unique_lock` 是相对于 `std::lock_guard` 出现的，**更加灵活**
- `unique_lock` 的对象会以**独占所有权**（没有其他的 `unique_lock` 对象同时拥有某个 `mutex` 对象的所有权）的方式管理 `mutex` 对象上的上锁和解锁操作
- **在并发编程中，推荐使用 `std::unique_lock`**
- **与 `lock_guard` 的关键区别**：
  - `std::lock_guard` **不能**显式地调用 `lock` 和 `unlock`
  - `std::unique_lock` 可以在声明后的**任意位置**调用 → 可以**缩小锁的作用范围，提供更高的并发度**
- **条件变量配合**：如果用到了 `std::condition_variable::wait`，则**必须使用 `std::unique_lock`** 作为参数

**示例/实践**
```cpp
#include <iostream>
#include <mutex>
#include <thread>

int v = 1;

void critical_section(int change_v) {
    static std::mutex mtx;
    std::unique_lock<std::mutex> lock(mtx);
    // 执行竞争操作
    v = change_v;
    std::cout << v << std::endl;
    // 将锁进行释放
    lock.unlock();

    // 在此期间，任何人都可以抢夺 v 的持有权

    // 开始另一组竞争操作，再次加锁
    lock.lock();
    v += 1;
    std::cout << v << std::endl;
}

int main() {
    std::thread t1(critical_section, 2), t2(critical_section, 3);
    t1.join();
    t2.join();
    return 0;
}
```

> ⚠️ **关键区分**：`lock_guard` 简单省心但不能中途解锁；`unique_lock` 稍有开销但可灵活收放——需要中途释放或配合条件变量时必须用 `unique_lock`
> 💡 **理解技巧**：示例中 `unlock()` 与再次 `lock()` 之间的窗口期，其他线程可以进入临界区——「锁只罩住真正需要互斥的代码」
> 🔄 **知识关联**：`unique_lock` 的独占所有权语义与 `unique_ptr` 一致（独占、可移动、不可拷贝）

---

<a id="id5"></a>
## ✅ 知识点5: 期物 std::future：访问异步操作的结果

**`std::future` 提供了一条访问异步操作结果的途径，把「等结果」这件事标准化了。**

期物（Future）表现为 `std::future`，它提供了一个**访问异步操作结果**的途径。

**引入动机（C++11 之前的做法）：**
试想主线程 A 希望新开辟一个线程 B 去执行某个我们预期的任务，并返回一个结果。而线程 A 可能正在忙其他的事情，无暇顾及 B 的结果——所以我们会很自然地希望能够在某个特定的时间获得线程 B 的结果。在 `std::future` 被引入之前，通常的做法是：

1. 创建一个线程 A，在线程 A 里启动任务 B
2. 当准备完毕后**发送一个事件**，并将结果保存在**全局变量**中
3. 主线程 A 里正在做其他的事情，当需要结果的时候，调用一个**线程等待函数**来获得执行的结果

- C++11 提供的 `std::future` **简化了这个流程**，可以用来**获取异步任务的结果**


> 💡 **理解技巧**：`future` 就像快递单号——任务（包裹）发出后你继续忙，凭单号随时可以查询、领取结果
> 📋 **术语提醒**：期物(future) 是「未来才就绪的值」的句柄；与之配对、负责写入结果的一方叫期约 `std::promise`（本教程未展开）

---

<a id="id6"></a>
## ✅ 知识点6: future 作为线程同步屏障

**`future` 天然可用作简单的线程同步手段——屏障（barrier）。**

- 既然 `future` 能在某个时刻取得异步任务的结果，我们很容易能够想象到把它作为一种**简单的线程同步手段**，即**屏障(barrier)**
- 在 `future` 上调用 `wait()`：当前线程**阻塞到期物的完成**——效果等价于在代码中设置一道屏障

> 💡 **理解技巧**：`join()` 等的是「线程结束」，`wait()` 等的是「结果就绪」——粒度不同：一个线程可以产出结果后继续干别的
> 🔄 **知识关联**：屏障思想在并行计算中很常见（多阶段计算的阶段分界），配合 `get()` 即「先同步、再取数」

---

<a id="id7"></a>
## ✅ 知识点7: std::packaged_task：封装可调用目标

**`std::packaged_task` 封装任何可调用目标用于异步调用，`get_future()` 取出配对的 future 实施线程同步。**

- `std::packaged_task` 可以用来**封装任何可以调用的目标**，从而用于实现**异步的调用**
- 模板参数为**要封装函数的类型**（如 `std::packaged_task<int()>`）
- 在封装好要调用的目标后，可以使用 **`get_future()`** 来获得一个 `std::future` 对象，以便之后实施线程同步

**示例/实践**
```cpp
#include <iostream>
#include <future>
#include <thread>

int main() {
    // 将一个返回值为7的 lambda 表达式封装到 task 中
    // std::packaged_task 的模板参数为要封装函数的类型
    std::packaged_task<int()> task([](){return 7;});
    // 获得 task 的期物
    std::future<int> result = task.get_future(); // 在一个线程中执行 task
    std::thread(std::move(task)).detach();
    std::cout << "waiting...";
    result.wait(); // 在此设置屏障，阻塞到期物的完成
    // 输出执行结果
    std::cout << "done!" << std:: endl << "future result is "
              << result.get() << std::endl;
    return 0;
}
```

- `task.get_future()`：取得与任务关联的 `std::future<int>`
- `std::thread(std::move(task)).detach()`：在新线程中执行任务并**分离**（主线程不 join 它）
- `result.wait()`：**屏障**——阻塞到期物完成
- `result.get()`：**取回执行结果**（输出 7）

> ⚠️ **关键区分**：`packaged_task` 不可拷贝，只能 **`std::move`** 转移给线程——与 `unique_lock` / `unique_ptr` 的所有权语义一致
> 📋 **术语提醒**：`detach()` 分离线程后线程对象与执行脱钩（无需 join）；本例靠 `future` 的 `wait()`/`get()` 保证结果可见
> 💡 **理解技巧**：三件套分工——`packaged_task` 打包任务、`future` 取结果、`thread` 提供执行场所

---

<a id="id8"></a>
## ✅ 知识点8: 条件变量 std::condition_variable：为避免死锁而生

**互斥操作不够用时引入条件变量：让线程「睡着等条件」，由其他线程唤醒，避免忙等待死锁。**

- 条件变量 `std::condition_variable` 是**为了解决死锁而生**，当互斥操作不够用而引入的
- **动机场景**：线程可能需要**等待某个条件为真**才能继续执行——
  - 若用一个**忙等待**循环反复抢锁检查条件，可能会导致**所有其他线程都无法进入临界区**使得条件为真
  - 结果：**死锁**
- `condition_variable` 对象被创建出现主要就是用于**唤醒等待线程**从而避免死锁
- **`notify_one()`**：唤醒**一个**等待线程
- **`notify_all()`**：通知**所有**等待线程

**注意点**
> ⚠️ **关键区分**：互斥锁解决「不能同时进」，条件变量解决「条件不满足时怎么办」——前者管互斥，后者管同步（等待/通知）
> 💡 **理解技巧**：`wait()` 时线程**释放锁并睡眠**，被唤醒后**重新获得锁**再继续——所以等待期间生产者才有机会进临界区改变条件
> 🔄 **知识关联**：`wait()` 必须配合 `std::unique_lock`（见知识点4）——等待过程需要中途解锁、醒来再加锁

---

<a id="id9"></a>
## ✅ 知识点9: 示例：生产者-消费者模型

**队列 + 互斥锁 + 条件变量 + 通知标志，构建经典的生产者-消费者模型。**

**示例/实践**
```cpp
#include <queue>
#include <chrono>
#include <mutex>
#include <thread>
#include <iostream>
#include <condition_variable>


int main() {
    std::queue<int> produced_nums;
    std::mutex mtx;
    std::condition_variable cv;
    bool notified = false;  // 通知信号

    // 生产者
    auto producer = [&]() {
        for (int i = 0; ; i++) {
            std::this_thread::sleep_for(std::chrono::milliseconds(900));
            std::unique_lock<std::mutex> lock(mtx);
            std::cout << "producing " << i << std::endl;
            produced_nums.push(i);
            notified = true;
            cv.notify_all(); // 此处也可以使用 notify_one
        }
    };
    // 消费者
    auto consumer = [&]() {
        while (true) {
            std::unique_lock<std::mutex> lock(mtx);
            while (!notified) {  // 避免虚假唤醒
                cv.wait(lock);
            }
            // 短暂取消锁，使得生产者有机会在消费者消费空前继续生产
            lock.unlock();
            // 消费者慢于生产者
            std::this_thread::sleep_for(std::chrono::milliseconds(1000));
            lock.lock();
            while (!produced_nums.empty()) {
                std::cout << "consuming " << produced_nums.front() << std::endl;
                produced_nums.pop();
            }
            notified = false;
        }
    };

    // 分别在不同的线程中运行
    std::thread p(producer);
    std::thread cs[2];
    for (int i = 0; i < 2; ++i) {
        cs[i] = std::thread(consumer);
    }
    p.join();
    for (int i = 0; i < 2; ++i) {
        cs[i].join();
    }
    return 0;
}
```

**关键点拆解：**
- **生产者**：每 900ms 生产一个数 → 入队 → 置 `notified = true` → `cv.notify_all()`
- **消费者**：`while (!notified) cv.wait(lock);`——用 **while 循环**包裹 `wait()`，**避免虚假唤醒(spurious wakeup)**
- 消费前**短暂 `unlock()`**：使得生产者有机会在消费者消费完之前继续生产
- 1 个生产者线程 + 2 个消费者线程，最后全部 `join()`

**注意点**
> ⚠️ **关键警告**：`wait()` 必须用 **while（而非 if）** 检查条件——线程可能被**虚假唤醒**（条件并未真就绪就被唤醒），醒来必须重新检查
> 💡 **理解技巧**：`cv.wait(lock)` 的内部动作 = 原子地「释放锁 + 睡眠」，被唤醒后「重新拿锁再返回」——所以参数必须是可中途操作的 `unique_lock`
> 🔄 **知识关联**：`notified` 标志本身被多线程读写，为什么它能安全工作？这正是 [note_17_3 内存模型](./note_17_3_atomic_and_memory_model.md) 要回答的问题

---

<a id="id10"></a>
## ✅ 知识点10: notify_one vs notify_all 与互斥锁的粒度局限

**多消费者场景不建议用 notify_one；但互斥锁的排他性决定了无法真正并行消费——需要粒度更细的手段。**

- 在生产者中我们虽然可以使用 `notify_one()`，但**实际上并不建议在此处使用**：
  - 在**多消费者**的情况下，我们的消费者实现中**简单放弃了锁的持有**（消费前 unlock）
  - 这使得可能让**其他消费者争夺此锁**，从而更好地利用多个消费者之间的并发
  - `notify_one` 只唤醒一个，其余消费者错过争夺机会，白白浪费了这份并发空间
- **话虽如此**：因为 `std::mutex` 的**排他性**，我们根本无法期待多个消费者能**真正意义上并行消费**队列中生产的内容
- **结论**：我们仍需要**粒度更细的手段**

**注意点**
> ⚠️ **关键区分**：`notify_one` 只醒一个（可能错过该醒的、并发度低）；`notify_all` 全醒自行竞争（安全但有**惊群**开销）——按等待者数量与角色选择
> 💡 **理解技巧**：互斥锁是「一把大锁锁整个队列」——想真正并行消费就得缩小锁粒度（如无锁队列、分段锁），属于进阶话题
> 🔄 **知识关联**：「粒度更细的手段」之一正是原子操作——见 [note_17_3 原子操作与内存模型](./note_17_3_atomic_and_memory_model.md)

---

## 🔑 核心要点总结

1. **`std::mutex`（`<mutex>`）是基本互斥量**，但不要手动 `lock()`/`unlock()`——异常出口会漏解锁导致死锁；应使用 RAII 锁管理
2. **`lock_guard` 简单异常安全但不能中途解锁；`unique_lock` 独占所有权、可任意位置 lock/unlock、缩小锁范围提高并发度**——配合条件变量 `wait()` 时必须用 `unique_lock`
3. **`std::future` 提供访问异步结果的途径**，可作线程同步屏障（barrier）；`std::packaged_task` 封装可调用目标，`get_future()` 取 future，`wait()` 等就绪、`get()` 取结果
4. **`std::condition_variable` 为避免死锁而生**：`wait()`（必须 while 检查防虚假唤醒）、`notify_one()`/`notify_all()`；生产者-消费者是其经典场景
5. **互斥锁的排他性决定了无法真正并行消费**——更高并发需要粒度更细的手段（如原子操作，见 note_17_3）

## 📌 考试速记版

| 工具 | 头文件 | 一句话作用 |
|---|---|---|
| `std::mutex` | `<mutex>` | 最基本互斥量，`lock()`/`unlock()`（勿手动用） |
| `std::lock_guard` | `<mutex>` | RAII 锁，作用域内持有，**不能**中途解锁 |
| `std::unique_lock` | `<mutex>` | 灵活独占锁，可任意 lock/unlock，**条件变量必须用它** |
| `std::future` | `<future>` | 异步结果句柄，`wait()` 当屏障、`get()` 取结果 |
| `std::packaged_task` | `<future>` | 封装可调用目标，`get_future()` 配对 future（只能 move） |
| `std::condition_variable` | `<condition_variable>` | 条件等待/唤醒：`wait` / `notify_one` / `notify_all` |

- **易混淆对比**：`lock_guard` vs `unique_lock`（能否显式解锁）；`join()` vs `wait()`（等线程结束 vs 等结果就绪）；`notify_one` vs `notify_all`（醒一个 vs 全醒竞争）
- **常见陷阱**：① 手动 unlock 遇异常 → 死锁；② `wait` 用 if 不用 while → 虚假唤醒漏检；③ 忘记 `packaged_task` 要 `std::move`

**记忆口诀**：互斥 RAII 两把锁，guard 简单 unique 活；future 取数当屏障，条件变量防死锁；wait 要配 while 查，虚假唤醒不放过。
