# 📘 17.1 并发基础与线程管理 (Concurrency Basics and Thread Management)

> 本节涵盖：并发与并行、线程创建、传参与生命周期、join/detach、移动与异常安全、线程标识及线程局部存储。正文以 C++11 为基础，使用 C++17 工作草案 N4659 核对规则。
> 来源说明：参考旧 17.1 笔记、Manning 的第 1、2 章公开导览，以及已读取的标准草案正文。导览不是《C++ Concurrency in Action》第 2 版原著正文；具体对应关系见文末。
> 当前为重组候选稿，旧笔记保持原样，尚未替换。

---

## 🧠 核心概念总览

1. [*知识点1: 并发、并行与进程/线程*](#id1)
2. [*知识点2: std::thread 的创建与执行*](#id2)
3. [*知识点3: 线程传参、引用捕获与对象生命周期*](#id3)
4. [*知识点4: join、detach、joinable 与线程对象状态*](#id4)
5. [*知识点5: 线程所有权与移动*](#id5)
6. [*知识点6: 线程析构、异常与 RAII 管理*](#id6)
7. [*知识点7: 线程标识、休眠与让出执行*](#id7)
8. [*知识点8: thread_local 与线程局部存储*](#id8)

---

<a id="id1"></a>
## ✅ 知识点1: 并发、并行与进程/线程

**并发描述多个任务在同一段时间内推进；并行强调多个任务在同一时刻实际执行。**

**理论**

- **并发(concurrency)**：例如编辑器一边响应输入，一边检查文件变化；任务可以交替执行，也可以同时执行。
- **并行(parallelism)**：例如多个处理器核心分别处理一批数据的不同部分。
- 单核上通过任务切换可以实现并发；创建多个线程本身不保证这些线程一定同时执行。

| 比较 | 进程(process) | 线程(thread) |
|---|---|---|
| 基本角色 | 运行程序的资源与隔离边界 | 进程中的执行流 |
| 地址空间 | 不同进程通常有独立地址空间 | 同一进程中的线程共享地址空间 |
| 数据交流 | 常借助管道、套接字或共享内存等机制 | 可以直接访问共享对象，但需要正确协调访问 |

- 多线程既可用于提高吞吐量，也可用于保持界面响应、分离不同工作。
- C++11 引入标准化线程库及支持多线程的内存模型；此前通常依赖平台线程接口。

**注意点**

> ⚠️ **关键区分**：多线程不保证提速；线程管理、同步和调度也会产生开销。
> 💡 **理解技巧**：并发关注“多件事如何一起推进”，并行关注“能否同时做”。

---

<a id="id2"></a>
## ✅ 知识点2: std::thread 的创建与执行

**向 `std::thread` 构造函数传入可调用对象并成功构造后，新线程就已经启动，不需要再调用 start**

**理论**

- 使用 `std::thread` 需要头文件 `<thread>`；执行入口可以是普通函数、lambda 或函数对象等**可调用对象(callable)**。
- `std::thread t;` 是默认构造，仅创建一个不关联执行线程的对象。
- `std::thread t(task);` 则尝试启动执行 `task` 的线程。创建失败会抛出 `std::system_error`。
- 新线程与调用者随后并发推进；不能仅凭代码书写顺序推断两者后续工作的先后。
- **线程对象(thread object)** 与 **执行线程(thread of execution)** 要分开理解：前者用于管理和查询后者，两个生命周期不一定相同。

**示例/实践**


```cpp
#include <iostream>
#include <thread>

int main() {
    std::thread t([](){
        std::cout << "hello world." << std::endl;
    });
    t.join();
    return 0;
}
```

- 构造 `t` 时启动新线程，执行 lambda；`main` 是原有执行流的入口。
- `t.join()` 等待关联线程结束；它不是启动线程的操作，具体规则见知识点4。

**注意点**

> ⚠️ **关键区分**：传给 `std::thread` 的函数即使有返回值，该值也不会自动交回调用者。异步结果的获取属于后续 future 专题。
> 🔄 **知识关联**：旧笔记[知识点2](./note_17_1_thread_basics.md#id2)介绍线程创建；[知识点4](./note_17_1_thread_basics.md#id4)提供本例，原文件均未改动。

---

<a id="id3"></a>
## ✅ 知识点3: 线程传参、引用捕获与对象生命周期

**创建线程时，普通变量默认按值传递；要让线程访问原变量，可以使用 std::ref 传递引用。传引用时，必须保证原变量在线程使用期间没有被销毁**

**理论**

- `std::thread(f, args...)` 会在构造线程对象的线程中保存可调用对象及参数的副本；根据传入值的类别，可以发生拷贝或移动。
- **普通左值参数不会自动按引用传递**。要让线程函数访问原对象，可显式使用 `<functional>` 中的 `std::ref`；只读引用可使用 `std::cref`。
- 指针的副本仍指向原对象；复制指针不会复制目标对象，也不会延长其生命周期。
- lambda 的值捕获保存相应值，引用捕获访问原对象。捕获指针、`this` 或引用时，同样要考虑目标对象的生命周期。
- **生命周期安全不等于访问同步**：对象仍存活，也不代表多个线程可以无保护地同时修改它。

**示例/实践（自拟）**

```cpp
#include <functional>
#include <thread>

void increment(int& value) {
    ++value;
}

void example() {
    int value = 0;
    std::thread t(increment, std::ref(value));
    t.join();
    // value == 1：调用者在 join 返回后才访问结果。
}
```

- `value` 在线程使用期间仍存活，调用者也没有在等待期间访问它。
- 如果改为分离线程，然后让函数返回，就不能再保证线程访问 `value` 时该对象仍存活。
- 类似地，`[&value]` 不会延长 `value` 的生命周期；`[value]` 保存一份值，但若这个值是指针，仍需检查指向对象的生命周期。

**注意点**

> ⚠️ **警告**：线程在局部对象生命周期结束后仍通过指针或引用访问它，会产生未定义行为(undefined behavior)。不能靠“线程大概已经执行完”保证安全。
> 💡 **理解技巧**：检查两个独立问题——“对象还在吗？”和“访问是否正确同步？”

**来源**：[线程构造][std-ctor]、[ref/cref][std-ref]、[lambda 捕获][std-capture]（标准正文已读）；Manning [第 2 章导览][ccia-ch2] §2.1.1、§2.2（非原著正文）。

---

<a id="id4"></a>
## ✅ 知识点4: join、detach、joinable 与线程对象状态

**joinable 判断线程对象是否仍关联一个执行线程，不用于判断该线程当前是否还在运行。**

**理论**

| 操作 | 作用 | 成功执行后的结果 |
|---|---|---|
| `t.joinable()` | 查询是否仍有关联线程 | 不改变对象状态，也不等待线程 |
| `t.join()` | 当前线程等待 `t` 关联的线程结束 | `t` 不再关联该线程 |
| `t.detach()` | 解除对象与执行线程的关联，不等待其结束 | `t` 不再关联该线程，执行线程可能仍继续运行 |

**按生命周期判断状态：**

- **默认构造后**：没有关联线程，`joinable()` 为 `false`。
- **成功启动线程后**：存在关联线程，`joinable()` 为 `true`。
- **线程函数已返回，但尚未 join 或 detach**：关联仍在，`joinable()` **仍为 `true`**。
- **成功 join 或 detach 后**：关联解除，`joinable()` 为 `false`。
- **被移动后的源对象**：关联交给了另一对象，源对象的 `joinable()` 为 `false`。
- **对象析构时**：若 `joinable()` 为 `true`，调用 `std::terminate()`；否则不会因此终止程序。

**示例/实践（自拟片段）**

```cpp
std::thread t([] {});
bool associated = t.joinable(); // true，即使 lambda 已经执行完。
t.join();
bool associated_after = t.joinable(); // false。
```

**注意点**

> ⚠️ **警告**：不能对非 joinable 对象调用 `join()` 或 `detach()`，也不能让线程 join 自己；这些错误会涉及 `std::system_error`。同一个线程对象也不能被多个线程无同步地操作。
> 💡 **理解技巧**：线程“干完活”与管理对象“办完收尾手续”是两件事。
> 🔄 **知识关联**：成功 `join()` 还建立线程完成到调用者返回之间的同步关系；`detach()` 不提供这种完成等待保证。

**来源**：[线程成员函数][std-members]、[线程析构][std-dtor]、[thread 类][std-class]（标准正文已读）；旧笔记[知识点3](./note_17_1_thread_basics.md#id3)仅供对照，旧文未修改。

---

<a id="id5"></a>
## ✅ 知识点5: 线程所有权与移动

**std::thread 不可复制、可以移动；移动转移的是关联线程的管理责任，不是创建另一个执行线程。**

**理论**

- 同一个执行线程不会同时由两个 `std::thread` 对象表示，因此复制构造和复制赋值被删除。
- **移动构造(move construction)** 将关联转交给新对象，源对象变为空状态。
- **移动赋值(move assignment)** 还需要检查目标对象：如果目标仍然 `joinable()`，会调用 `std::terminate()`。
- 转移后，由新对象的所有者安排后续 `join()` 或 `detach()`。

**示例/实践（自拟片段，另需 `<utility>`）**

```cpp
std::thread first([] {});
std::thread second(std::move(first));
// first.joinable() == false；second.joinable() == true。
second.join();
```

- 无论工作函数有没有提前结束，移动后的关联状态仍按上述规则判断。
- 给一个仍然 joinable 的对象赋入另一线程，不会替你等待或分离原线程；应先完成原关联的处理。

**注意点**

> ⚠️ **关键区分**：`std::move` 只是转换表达式；真正转移关联的是随后调用的移动构造或移动赋值。
> 🔄 **知识关联**：移动语义的基础见[13.6 移动对象](../Chapter13_copy_control/note_13_6_moving_objects.md)。

**来源**：[thread 类][std-class]、[线程构造][std-ctor]、[移动赋值][std-assign]（标准正文已读）；Manning [第 2 章导览][ccia-ch2] §2.3（非原著正文）。

---

<a id="id6"></a>
## ✅ 知识点6: 线程析构、异常与 RAII 管理

**线程管理必须覆盖正常返回和异常退出；只在函数末尾写一次 join，可能被提前退出绕过。**

**理论**

需要区分三个位置：

1. **构造阶段失败**：创建线程或保存参数时发生的异常在构造线程对象的线程中处理。
2. **启动后，调用者抛出异常**：可能跳过原本计划执行的 `join()`；如果因此析构一个 joinable 对象，就会触发终止。
3. **工作线程的入口函数有异常逃出**：调用 `std::terminate()`；这个异常不会自动在调用者的 `join()` 处重新抛出。

**资源获取即初始化(RAII)** 可以将收尾责任绑定到对象生命周期：正常离开作用域，或异常向外层处理器传播并进行**栈展开(stack unwinding)** 时，管理对象的析构函数负责等待线程。

**示例/实践（自拟作用域管理器）**

```cpp
#include <thread>

class join_on_exit {
    std::thread& worker_;
public:
    explicit join_on_exit(std::thread& worker) noexcept : worker_(worker) {}
    join_on_exit(const join_on_exit&) = delete;
    join_on_exit& operator=(const join_on_exit&) = delete;

    ~join_on_exit() {
        if (worker_.joinable()) {
            worker_.join();
        }
    }
};

void run() {
    int value = 0;
    std::thread worker([&value] { value = 1; });
    join_on_exit guard(worker);
    // 后续正常离开，或发生向外层处理器传播的异常时，guard 先等待线程。
}
```

- 声明顺序是 `value → worker → guard`，析构顺序相反：先等待线程，再销毁线程对象和被引用的数据。
- 这个示例要求管理器始终在创建者线程中使用，引用的线程对象仍有效，且没有其他地方并发操作或转移它。
- 管理器只负责收尾，不负责捕获工作线程的异常，也不保证工作线程一定能够结束。

**注意点**

> ⚠️ **警告**：析构中等待可能阻塞；如果工作线程等着管理者释放锁，仍可能死锁。`join()` 本身也可能抛异常，若从此析构函数逃出会触发终止；示例依赖合法的 join 使用条件，不是任意场景下的完整线程封装。
> 💡 **理解技巧**：RAII 将收尾绑定到作用域，但不会自动解决线程之间的依赖关系。

**来源**：[线程构造与未捕获异常][std-ctor]、[线程析构][std-dtor]、[栈展开][std-unwind]（标准正文已读）；Manning [第 2 章导览][ccia-ch2] §2.1.3、§2.3（非原著正文）。《Effective Modern C++》第 37 条仅定位到目录，未使用其正文。

---

<a id="id7"></a>
## ✅ 知识点7: 线程标识、休眠与让出执行

**get_id 用于识别线程；sleep 和 yield 影响调度，不用于证明另一个线程已经完成工作。**

**理论**

| 接口 | 含义与边界 |
|---|---|
| `t.get_id()` | 获取 `t` 关联线程的标识；没有关联线程时返回默认构造的 `std::thread::id` |
| `std::this_thread::get_id()` | 获取当前执行线程的标识 |
| `std::this_thread::sleep_for(duration)` | 让当前线程按相对时长休眠；调度等因素可能使返回更晚 |
| `std::this_thread::sleep_until(time_point)` | 等待给定时刻；需要注意所用时钟及可能的时钟调整 |
| `std::this_thread::yield()` | 给实现一次重新调度的机会，不保证另一个特定线程一定运行 |

- 线程 ID 应作为标识使用，不应解释成有固定意义的数字。已终止且不再可 join 的线程，其 ID 值可以被后续线程复用。
- **时间经过不构成线程同步**：让当前线程睡一会儿，既不能保证对方已经结束，也不能为共享数据访问建立同步关系。

**示例/实践（自拟片段）**

```cpp
#include <chrono>
#include <thread>

void pause_briefly() {
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
}
```

若要等待一个线程完成，应使用 `join()`；若要等待某个条件或结果，应使用相应同步设施，而不是猜测休眠时间。

**注意点**

> ⚠️ **关键区分**：`joinable()` 不是“运行中”检查，线程 ID 也不是完成通知，`sleep/yield` 更不是同步工具。
> 🔄 **知识关联**：旧笔记[知识点3](./note_17_1_thread_basics.md#id3)中的 `get_id()` 在此扩展，旧正文不变。

**来源**：[thread 类与线程标识][std-class]、[this_thread][std-this]、[时间约定][std-timing]（标准正文已读）；Manning [第 2 章导览][ccia-ch2] §2.5（非原著正文）。

---

<a id="id8"></a>
## ✅ 知识点8: thread_local 与线程局部存储

**thread_local 让每个线程拥有各自的变量实例，适合不需要跨线程共享的状态。**

**理论**

- 从 C++11 起，可用 `thread_local` 指定**线程存储期(thread storage duration)**。
- 不同线程通过该名字访问各自的实例；一个线程修改自己的实例，不会直接修改另一个线程的实例。
- 实例按规则在首次需要使用之前完成初始化；已构造对象在线程退出时销毁，而不是每次离开使用它的函数时销毁。
- 常见用途包括线程私有计数、临时工作状态、缓存和错误信息；共享汇总仍需另行安排。

**示例/实践（自拟）**

```cpp
#include <thread>

thread_local int calls = 0;

void count_once() {
    ++calls;
}

int main() {
    std::thread t(count_once);
    t.join();
    // 工作线程的 calls 曾变为 1；主线程自己的 calls 仍是 0。
}
```

**注意点**

> ⚠️ **警告**：`thread_local` 只分开变量实例。如果变量是指针，多份指针仍可能指向同一个共享对象；共享目标的生命周期和同步问题不会因此消失。
> 💡 **理解技巧**：把本来不必共享的状态分给各个线程，可以减少共享访问的需求。
> 📋 **术语提醒**：TLS 是 thread-local storage（线程局部存储）的缩写。

**来源**：[C++17 草案：线程存储期][std-tls]（正文已读）；本点为标准资料补充，不归称为已读取的 CCIA 或 EMC 正文内容。

---

## 🔑 核心要点总结

1. 并发让任务在同一时间段推进，并行强调同时执行；多线程不保证提速。
2. `std::thread` 管理一个执行线程；`joinable()` 表示仍有关联，不表示工作仍在进行。
3. 引用传参和捕获必须同时满足对象生命周期与访问同步要求，复制指针不能延长目标对象的生命。
4. 线程所有权可以移动；销毁或覆盖一个 joinable 的线程对象会触发终止，异常路径也必须安排收尾。
5. `sleep/yield` 不提供同步；`thread_local` 分离线程私有状态，但不能自动保护它指向的共享数据。

## 📌 考试速记版

| 常见提问 | 回答要点 |
|---|---|
| 线程函数结束后，joinable 一定变成 false 吗？ | 不一定；未 join、detach 或转移关联时，仍然是 true |
| thread 析构一定会终止程序吗？ | 只有析构时仍 joinable，才会因此调用 `std::terminate()` |
| detach 会停止线程吗？ | 不会；它解除管理对象与执行线程的关联 |
| thread 能复制吗？ | 不能，可以移动；移动赋值时目标不能仍然 joinable |
| join 能重新抛出工作线程的异常吗？ | 不能；异常不能逃出工作线程入口，应在该线程处理或显式传递 |
| ref 或引用捕获能延长局部对象生命周期吗？ | 不能；对象必须活到线程最后一次使用它之后 |
| 睡眠足够久可以替代同步吗？ | 不可以，经过的时间不是同步关系 |
| thread_local 可以保存跨线程总计数吗？ | 各线程有独立实例，若要总计数，需要显式汇总与协调 |

**记忆口诀**：启动即运行，关联看 joinable；引用看寿命，退出管收尾；休眠不作同步，私有不等于共享安全。

---

## 📚 材料使用与旧稿对照

**已读取的资料**：旧 17.1 全文；Manning 第 1、2 章公开导览；正文中链接的 C++17 工作草案条文。标准草案用于核对语言及库行为，导览用于定位主题和阅读线索。

**未读取的原著正文**：《C++ Concurrency in Action》第 2 版正文入口返回 403；《Effective Modern C++》第 37 条只核对了[出版社目录][emc-toc]，本稿未使用该条正文，也未复制其中示例。不得将上述两项记为“原著正文已读”。

| 本稿知识点 | CCIA 第 2 版导览定位 | 旧稿使用情况 | 正文依据 |
|---|---|---|---|
| 1 | §1.1—1.3 | 旧知识点1：背景参考，原文不变 | 出版社第 1 章导览；旧笔记 |
| 2 | §1.4、§2.1.1 | 旧知识点2：概念参考；旧知识点4：代码原样复制到本稿 | 标准 thread 类与构造 |
| 3 | §2.1.1、§2.2 | 新增，未改旧知识点 | 标准线程构造、ref/cref、lambda 捕获 |
| 4 | §2.1.2、§2.1.4 | 旧知识点3：主题对照；旧警告尚未修改 | 标准线程成员与析构 |
| 5 | §2.3 | 新增，未改旧知识点 | 标准线程类、移动构造与赋值 |
| 6 | §2.1.3、§2.3 | 新增，未改旧知识点 | 标准异常、析构、栈展开；出版社导览的 RAII 思路 |
| 7 | §2.5 | 旧知识点3：get_id 主题参考，原文不变 | 标准线程标识、this_thread、时间约定 |
| 8 | 不作原著归属 | 新增，未改旧知识点 | 标准线程存储期 |

本稿未修改旧笔记，旧稿的四个知识点及总结仍保存在[原文件](./note_17_1_thread_basics.md)。是否替换、迁移或修改旧文，须另行确认。

[ccia-ch1]: https://livebook.manning.com/learn/c-plus-plus-concurrency-in-action-second-edition/chapter-1
[ccia-ch2]: https://livebook.manning.com/learn/c-plus-plus-concurrency-in-action-second-edition/chapter-2
[emc-toc]: https://www.oreilly.com/library/view/effective-modern-c/9781491908419/ch07.html
[std-class]: https://timsong-cpp.github.io/cppwp/n4659/thread.thread.class
[std-ctor]: https://timsong-cpp.github.io/cppwp/n4659/thread.thread.constr
[std-members]: https://timsong-cpp.github.io/cppwp/n4659/thread.thread.member
[std-dtor]: https://timsong-cpp.github.io/cppwp/n4659/thread.thread.destr
[std-assign]: https://timsong-cpp.github.io/cppwp/n4659/thread.thread.assign
[std-this]: https://timsong-cpp.github.io/cppwp/n4659/thread.thread.this
[std-timing]: https://timsong-cpp.github.io/cppwp/n4659/thread.req.timing
[std-tls]: https://timsong-cpp.github.io/cppwp/n4659/basic.stc.thread
[std-ref]: https://timsong-cpp.github.io/cppwp/n4659/refwrap.helpers
[std-capture]: https://timsong-cpp.github.io/cppwp/n4659/expr.prim.lambda.capture
[std-unwind]: https://timsong-cpp.github.io/cppwp/n4659/except.ctor
