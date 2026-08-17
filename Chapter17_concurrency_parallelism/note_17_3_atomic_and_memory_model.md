# 📘 [第 17 章] 并发与并行 (Concurrency and Parallelism)

> 来源说明：《现代 C++ 教程：高速上手 C++》第 7 章「并行与并发」7.5 节 | 本节涵盖：`std::atomic` 原子操作、`is_lock_free` 可行性检查、四种一致性模型、六种 `std::memory_order` 内存顺序与 CAS 原语

---

## 🧠 核心概念总览

- [*知识点1: 引入：优化与重排下的并发陷阱*](#id1)
- [*知识点2: 互斥锁的代价：为什么需要原子操作*](#id2)
- [*知识点3: std::atomic 与基本原子操作*](#id3)
- [*知识点4: is_lock_free：原子可行性检查*](#id4)
- [*知识点5: 一致性模型：四种强弱等级*](#id5)
- [*知识点6: 内存顺序总览：六种 memory_order*](#id6)
- [*知识点7: 宽松模型 memory_order_relaxed*](#id7)
- [*知识点8: 释放/消费模型 release/consume*](#id8)
- [*知识点9: 释放/获取模型与 CAS 原语*](#id9)
- [*知识点10: 顺序一致模型 memory_order_seq_cst*](#id10)

---

<a id="id1"></a>
## ✅ 知识点1: 引入：优化与重排下的并发陷阱

**编译器优化与 CPU 乱序执行会让「直觉上正确」的并发代码实际成为未定义行为。**

细心的读者可能会对上一节生产者-消费者模型的例子存在疑虑：编译器可能对变量 `notified` 存在优化，例如**将其作为一个寄存器的值**，从而导致消费者线程**永远无法观察到此值的变化**。为了解释清楚这个问题，我们需要进一步讨论从 C++11 起引入的**内存模型(memory model)** 这一概念。我们首先来看一个问题——下面这段代码输出结果是多少？

**示例/实践**
```cpp
#include <thread>
#include <iostream>

int main() {
    int a = 0;
    volatile int flag = 0;

    std::thread t1([&]() {
        while (flag != 1);

        int b = a;
        std::cout << "b = " << b << std::endl;
    });

    std::thread t2([&]() {
        a = 5;
        flag = 1;
    });

    t1.join();
    t2.join();
    return 0;
}
```

- **直觉分析**：`t2` 中 `a = 5;` 似乎总在 `flag = 1;` 之前得到执行；`t1` 中 `while (flag != 1)` 似乎保证了输出不会在标记被改变前执行——从逻辑上看，`b` 的值似乎应该等于 5
- **实际情况远比此复杂得多**：这段代码本身属于**未定义行为(undefined behavior, UB)**——
  1. 对于 `a` 和 `flag` 而言，它们在两个并行的线程中被读写，出现了**竞争(race)**
  2. 即便忽略竞争读写，仍然可能受 **CPU 的乱序执行**、**编译器对指令的重排**的影响，导致 `a = 5` 发生在 `flag = 1` 之后
- 从而 `b` **可能输出 0**

**注意点**
> ⚠️ **关键警告**：`volatile` **不能保证**多线程的可见性与有序性——它只禁止编译器把变量优化进寄存器，管不了 CPU 乱序与缓存；多线程同步不能用 `volatile`
> 💡 **理解技巧**：并发代码的正确性有三层敌人——线程交叉执行、编译器重排、CPU 乱序；直觉只覆盖第一层
> 🔄 **知识关联**：这里的 `flag`/`a` 问题正是 [note_17_2](./note_17_2_synchronization_primitives.md) 生产者-消费者例子中 `notified` 标志的理论解释

---

<a id="id2"></a>
## ✅ 知识点2: 互斥锁的代价：为什么需要原子操作

**互斥锁是操作系统级的强同步，编译出来是一组指令；对仅需原子操作的变量来说太苛刻。**

`std::mutex` 可以解决上面出现的并发读写问题，但互斥锁是**操作系统级**的功能——一个互斥锁的实现通常包含两条基本原理：

1. 提供线程间自动的状态转换，即**『锁住』这个状态**
2. 保障在互斥锁操作期间，所操作变量的**内存与临界区外进行隔离**

- 这是一组**非常强的同步条件**——换句话说，当最终编译为 CPU 指令时会表现为**非常多的指令**
- 这对于一个**仅需原子级操作（没有中间态）**的变量，似乎太苛刻了
- 现代 CPU 体系结构提供了 **CPU 指令级的原子操作**——这正是 `std::atomic` 的硬件基础

**注意点**
> 💡 **理解技巧**：互斥锁像「包下整个房间谈事」，原子操作像「递一张纸条」——递纸条不需要包房间
> 📋 **术语提醒**：原子操作(atomic operation) 指不可再分、**没有中间态**的操作——其他线程只能看到操作前或操作后的状态

---

<a id="id3"></a>
## ✅ 知识点3: std::atomic 与基本原子操作

**`std::atomic` 模板把一次原子写操作从一组指令最小化到单个 CPU 指令。**

- C++11 中引入了 **`std::atomic` 模板**：使得我们能实例化**原子类型**，并将一个原子写操作从一组指令，**最小化到单个 CPU 指令**。例如：`std::atomic<int> counter;`
- 为整数或浮点数的原子类型提供了基本的数值成员函数，举例来说，包括 **`fetch_add`、`fetch_sub`** 等
- 同时通过**重载**方便地提供了对应的 `+`、`-` 版本（`count++`、`count += 1` 等价于 `fetch_add`）

**示例/实践**
```cpp
#include <atomic>
#include <thread>
#include <iostream>

std::atomic<int> count = {0};

int main() {
    std::thread t1([](){
        count.fetch_add(1);
    });
    std::thread t2([](){
        count++;        // 等价于 fetch_add
        count += 1;     // 等价于 fetch_add
    });
    t1.join();
    t2.join();
    std::cout << count << std::endl;
    return 0;
}
```

- 三个线程共加 3 次，最终 `count` **必然输出 3**——原子操作没有中间态，不会丢更新

**注意点**
> ⚠️ **关键区分**：`count++` 对普通 `int` 是「读-改-写」一组指令（可被其他线程穿插），对 `std::atomic<int>` 是**单条原子指令**——这就是不丢更新的原因
> 💡 **理解技巧**：声明 `std::atomic<int> counter;` 之后，变量的每次读写默认都是原子的、线程安全的

---

<a id="id4"></a>
## ✅ 知识点4: is_lock_free：原子可行性检查

**并非所有类型都能提供原子操作——取决于 CPU 架构与内存对齐，用 `is_lock_free` 检查。**

- **并非所有的类型都能提供原子操作**，这是因为原子操作的可行性取决于：
  - 具体的 **CPU 架构**
  - 所实例化的类型结构是否能够满足该 CPU 架构对**内存对齐**条件的要求
- 因而我们总是可以通过 **`std::atomic<T>::is_lock_free`** 来检查该原子类型是否支持原子操作

**示例/实践**
```cpp
#include <atomic>
#include <iostream>

struct A {
    float x;
    int y;
    long long z;
};

int main() {
    std::atomic<A> a;
    std::cout << std::boolalpha << a.is_lock_free() << std::endl;
    return 0;
}
```

**注意点**
> 📋 **术语提醒**：`is_lock_free` 为 `false` 时该原子类型仍可使用，只是内部**退化为锁实现**——语义正确、性能不同
> 💡 **理解技巧**：内置小整型几乎总是 lock-free；大的自定义结构体通常不是——所以高性能并发代码倾向用「原子指针/索引」而非「原子大对象」

---

<a id="id5"></a>
## ✅ 知识点5: 一致性模型：四种强弱等级

**多线程在宏观上类似分布式系统；适度削弱同步条件换取效率，形成四种一致性模型。**

并行执行的多个线程，从某种宏观层面上讨论，可以粗略地视为一种**分布式系统**——在分布式系统中，任何通信乃至本地操作都需要消耗一定时间，甚至出现不可靠的通信。

**削弱同步条件的动机：** 如果强行将一个变量 `v` 在多个线程之间的操作设为原子操作——任何一个线程在操作完 `v` 后，其他线程均能**同步**感知到 `v` 的变化——则对于变量 `v` 而言，表现为**顺序执行的程序**，它并没有由于引入多线程而得到任何效率上的收益。**适当加速的办法 = 削弱原子操作在进程间的同步条件。**

从原理上看，每个线程可以对应为一个集群节点，而线程间的通信也几乎等价于集群节点间的通信。削弱进程间的同步条件，通常会考虑**四种不同的一致性模型**：

**1. 线性一致性(Linearizability)**——又称**强一致性**或**原子一致性**
- 要求：**任何一次读操作都能读到某个数据的最近一次写的数据**，并且**所有线程的操作顺序与全局时钟下的顺序是一致的**

```
        x.store(1)      x.load()
T1 ---------+----------------+------>


T2 -------------------+------------->
                x.store(2)
```
- 这种情况下线程 `T1`、`T2` 对 `x` 的两次写操作是原子的，且 `x.store(1)` 严格发生在 `x.store(2)` 之前，`x.store(2)` 严格发生在 `x.load()` 之前
- **线性一致性对全局时钟的要求是难以实现的**——这也是人们不断研究比这个一致性更弱条件下其他一致性算法的原因

**2. 顺序一致性(Sequential Consistency)**
- 同样要求**任何一次读操作都能读到数据最近一次写入的数据**，但**未要求与全局时钟的顺序一致**

```
        x.store(1)  x.store(3)   x.load()
T1 ---------+-----------+----------+----->


T2 ---------------+---------------------->
              x.store(2)
```
- 在顺序一致性的要求下，`x.load()` 必须读到最近一次写入的数据，因此 `x.store(2)` 与 `x.store(1)` **并无任何先后保障**——只要 `T2` 的 `x.store(2)` 发生在 `x.store(3)` 之前即可

**3. 因果一致性(Causal Consistency)**
- 要求进一步降低：只需要**有因果关系的操作顺序得到保障**，而非因果关系的操作顺序则不做要求

```
      a = 1      b = 2
T1 ----+-----------+---------------------------->


T2 ------+--------------------+--------+-------->
      x.store(3)         c = a + b    y.load()
```
- 上例中整个过程中**只有 `c` 对 `a` 和 `b` 产生依赖**，而 `x` 和 `y` 在此例子中表现为没有关系（但实际情况中我们需要更详细的信息才能确定 `x` 与 `y` 确实无关）——因此 `x.store(3)`、`y.load()`、`c = a + b` 的相对顺序可以任意

**4. 最终一致性(Eventual Consistency)**
- **最弱**的一致性要求：只保障某个操作在**未来的某个时间节点上会被观察到**，但并未要求被观察到的时间

```
    x.store(3)  x.store(4)
T1 ----+-----------+-------------------------------------------->


T2 ---------+------------+--------------------+--------+-------->
         x.read      x.read()           x.read()   x.read()
```
- 假设 `x` 的初始值为 0，则 `T2` 中四次 `x.read()` 结果可能但不限于以下情况：

```
3 4 4 4 // x 的写操作被很快观察到
0 3 3 4 // x 的写操作被观察到的时间存在一定延迟
0 0 0 4 // 最后一次读操作读到了 x 的最终值，但此前的变化并未观察到
0 0 0 0 // 在当前时间段内 x 的写操作均未被观察到，
        // 但未来某个时间点上一定能观察到 x 为 4 的情况
```

**注意点**
> ⚠️ **关键区分**：四种模型强弱排序——**线性 > 顺序 > 因果 > 最终**；越弱性能空间越大、编程心智负担越重
> 💡 **理解技巧**：全局时钟是分水岭——线性一致性要求「所有操作在真实时间上不错乱」，顺序一致性只要求「存在一个所有线程都认可的统一顺序」
> 🔄 **知识关联**：分布式系统中的共识与复制协议（如 Paxos/Raft）正是在这些一致性等级之间做取舍

---

<a id="id6"></a>
## ✅ 知识点6: 内存顺序总览：六种 memory_order

**C++11 为原子操作定义了六种 `std::memory_order` 选项，表达四种多线程间的同步模型。**

为了追求极致的性能，实现各种强度要求的一致性，C++11 为原子操作定义了**六种不同的内存顺序 `std::memory_order` 的选项**，表达了**四种多线程间的同步模型**：

| 同步模型 | 对应 memory_order 选项 | 强度 |
|---|---|---|
| 宽松模型 | `memory_order_relaxed` | 最弱：只保原子性 |
| 释放/消费模型 | `memory_order_release` + `memory_order_consume` | 弱：数据依赖可见 |
| 释放/获取模型 | `memory_order_release` + `memory_order_acquire`（+ `memory_order_acq_rel`） | 中：happens-before |
| 顺序一致模型 | `memory_order_seq_cst`（**默认值**） | 最强：全局统一顺序 |

**注意点**
> 📋 **术语提醒**：第六个选项 `memory_order_acq_rel` 用于读改写(read-modify-write) 操作，同时具有 acquire + release 语义——六种选项映射为四种模型
> ⚠️ **关键提醒**：不显式指定时，原子操作默认使用 **`memory_order_seq_cst`**（最强、最安全、也最慢）——指定更弱序是优化手段，需理解后才可使用
> 💡 **理解技巧**：内存顺序管的不是「这次原子操作本身」（它永远原子），而是「这次操作**周围**的其他读写能否被重排越过它」

---

<a id="id7"></a>
## ✅ 知识点7: 宽松模型 memory_order_relaxed

**单线程内的原子操作顺序执行、不允许重排；不同线程间原子操作的顺序是任意的。**

- **宽松模型(relaxed)**：
  - **单个线程内**的原子操作都是顺序执行的，**不允许指令重排**
  - **不同线程间**原子操作的顺序是**任意的**
- 通过 **`std::memory_order_relaxed`** 指定

**示例/实践**
```cpp
std::atomic<int> counter = {0};
std::vector<std::thread> vt;
for (int i = 0; i < 100; ++i) {
    vt.emplace_back([&](){
        counter.fetch_add(1, std::memory_order_relaxed);
    });
}

for (auto& t : vt) {
    t.join();
}
std::cout << "current counter:" << counter << std::endl;
```

- 100 个线程各自 `fetch_add(1, relaxed)`，最终结果**必然是 100**——relaxed 只放松「顺序」，不放松「原子性」

**注意点**
> ⚠️ **关键区分**：relaxed 保证**操作本身的原子性**，但不提供**任何跨线程的顺序与可见性保证**——适合计数器等只关心最终值、不关心顺序的场景
> 💡 **理解技巧**：把 relaxed 想成「各跑各的、只看总数」——没人关心谁先谁后，只关心每次加法都不丢

---

<a id="id8"></a>
## ✅ 知识点8: 释放/消费模型 release/consume

**消费方依赖释放方的某次写：consume 确保 load 时能观察到 release 所携带的那次写。**

- **释放/消费模型**开始**限制进程间的操作顺序**：如果某个线程需要修改某个值，但另一个线程会对该值的某次操作**产生依赖**，即后者依赖前者
- 具体而言：线程 A 完成了三次对 `x` 的写操作，线程 B **仅依赖其中第三次** `x` 的写操作、与前两次写行为无关——则当 A 主动 `x.release()`（即使用 `std::memory_order_release`）时，选项 **`std::memory_order_consume` 能够确保 B 在调用 `x.load()` 时候观察到 A 中第三次对 `x` 的写操作**

**示例/实践**
```cpp
// 初始化为 nullptr 防止 consumer 线程从野指针进行读取
std::atomic<int*> ptr(nullptr);
int v;
std::thread producer([&]() {
    int* p = new int(42);
    v = 1024;
    ptr.store(p, std::memory_order_release);
});
std::thread consumer([&]() {
    int* p;
    while(!(p = ptr.load(std::memory_order_consume)));

    std::cout << "p: " << *p << std::endl;
    std::cout << "v: " << v << std::endl;
});
producer.join();
consumer.join();
```

- 生产者 `release` 发布指针；消费者 `consume` 载入指针后，**通过该指针解引用的数据（`*p`）保证可见**
- 指针先初始化为 `nullptr`，防止 consumer 从野指针读取；consumer 自旋等待 `ptr` 非空

**注意点**
> ⚠️ **关键警告**：`consume` 只保护**有数据依赖**的读取（如通过 `p` 解引用 `*p`）；例中普通变量 `v` 与指针无依赖关系，其可见性要靠更强的 `acquire` 才稳妥（见知识点9）
> 📋 **术语提醒**：release/consume 中的「依赖」指**数据依赖(data dependency)**——读到的值被用于计算后续访问的地址或值

---

<a id="id9"></a>
## ✅ 知识点9: 释放/获取模型与 CAS 原语

**release 之前的所有写对 acquire 可见（happens-before）；release 是向后屏障，acquire 是向前屏障。**

- **释放/获取模型**进一步加紧对不同线程间原子操作的顺序的限制：
  - 在释放 `std::memory_order_release` 和获取 `std::memory_order_acquire` 之间**规定时序**
  - 发生在释放（release）操作之前的**所有写操作**，对其他线程的任何获取（acquire）操作都是**可见的**，亦即**发生顺序(happens-before)**
- **屏障语义**：
  - `std::memory_order_release` 确保了它**之前**的写操作不会发生在释放操作之后——是一个**向后的屏障(backward)**
  - `std::memory_order_acquire` 确保了它**之后**的读行为不会发生在该获取操作之前——是一个**向前的屏障(forward)**
  - **`std::memory_order_acq_rel`** 结合了这两者的特点，唯一确定了一个内存屏障，使得当前线程对内存的读写**不会被重排并越过此操作的前后**

**示例/实践**
```cpp
std::vector<int> v;
std::atomic<int> flag = {0};
std::thread release([&]() {
    v.push_back(42);
    flag.store(1, std::memory_order_release);
});
std::thread acqrel([&]() {
    int expected = 1; // must before compare_exchange_strong
    while(!flag.compare_exchange_strong(expected, 2, std::memory_order_acq_rel))
        expected = 1; // must after compare_exchange_strong
    // flag has changed to 2
});
std::thread acquire([&]() {
    while(flag.load(std::memory_order_acquire) < 2);

    std::cout << v.at(0) << std::endl; // must be 42
});
release.join();
acqrel.join();
acquire.join();
```

- `release` 线程先写 `v` 再 `flag.store(1, release)`；`acquire` 线程看到 `flag >= 2` 后读 `v.at(0)` **必然是 42**——release 之前的写对 acquire 之后的读可见
- 中间 `acqrel` 线程用 **CAS** 把 `flag` 从 1 改为 2

**CAS（比较交换原语，Compare-and-Swap primitive）：**
- `compare_exchange_strong(expected, desired, order)`：仅当当前值等于 `expected` 时替换为 `desired` 并返回 true；否则把当前值写回 `expected` 并返回 false
- 它有一个更弱的版本 **`compare_exchange_weak`**：**允许即便交换成功，也仍然返回 `false` 失败**
  - **原因**：某些平台上**虚假故障**导致的——具体而言，当 CPU 进行上下文切换时，另一线程加载同一地址产生的不一致
  - `compare_exchange_strong` 的**性能可能稍差于** `compare_exchange_weak`
  - 但大部分情况下，鉴于其使用的复杂度而言，**`compare_exchange_weak` 应该被优先考虑**

**注意点**
> ⚠️ **关键区分**：`weak` 可能「假失败」→ 必须配合**循环**重试（如示例的 while）；`strong` 无假失败但略慢——**循环里用 weak，单次判断用 strong**
> 💡 **理解技巧**：release/acquire 像「发令枪与终点线」——release 保证「我之前的活都干完了」，acquire 保证「我之后看到的都是成品」
> 🔄 **知识关联**：happens-before 是 C++ 内存模型的核心关系——本章习题 2（用 `std::atomic<bool>` 实现互斥锁）正是 release/acquire 语义的直接应用

---

<a id="id10"></a>
## ✅ 知识点10: 顺序一致模型 memory_order_seq_cst

**原子操作满足顺序一致性——最安全也最符合直觉，代价是可能的性能损耗。**

- **顺序一致模型**下，原子操作满足**顺序一致性**（见知识点5），进而**可能对性能产生损耗**
- 可显式通过 **`std::memory_order_seq_cst`** 进行指定；它也是原子操作的**默认**内存顺序

**示例/实践**
```cpp
std::atomic<int> counter = {0};
std::vector<std::thread> vt;
for (int i = 0; i < 100; ++i) {
    vt.emplace_back([&](){
        counter.fetch_add(1, std::memory_order_seq_cst);
    });
}

for (auto& t : vt) {
    t.join();
}
std::cout << "current counter:" << counter << std::endl;
```

- 这个例子与宽松模型的例子**本质上没有区别**，仅仅只是将原子操作的内存顺序修改为了 `memory_order_seq_cst`
- 有兴趣的读者可以自行编写程序，**测量这两种不同内存顺序导致的性能差异**

**注意点**
> 💡 **理解技巧**：seq_cst = 「所有线程看到同一个全局操作顺序」——最符合直觉所以是默认值；先用默认写对，再按需放松（relaxed/acquire）做优化
> ⚠️ **常见误区**：计数器这类「只改不依赖顺序」的场景，seq_cst 与 relaxed 结果相同但前者更慢——**默认序不等于零开销**
> 🔄 **知识关联**：四种同步模型 ≈ 四种一致性模型的工程落地：seq_cst ↔ 顺序一致性；release/acquire ↔ 因果级保证；relaxed ↔ 最终一致性思想

---

## 🔑 核心要点总结

1. **并发正确性的三层敌人**：线程交叉执行、编译器指令重排、CPU 乱序执行——`volatile` 解决不了，多线程无同步读写共享变量就是**未定义行为**
2. **`std::atomic` 把原子写最小化到单条 CPU 指令**（对比互斥锁的一整组指令）：`fetch_add`/`fetch_sub` 等成员函数 + 运算符重载；`is_lock_free` 检查可行性（取决于 CPU 架构与内存对齐）
3. **四种一致性模型**由强到弱：线性一致性（要求全局时钟）> 顺序一致性 > 因果一致性 > 最终一致性——本质是**削弱同步条件换取性能**
4. **六种 `memory_order` 表达四种同步模型**：relaxed（仅原子性）→ release/consume（数据依赖可见）→ release/acquire/acq_rel（happens-before，向后/向前屏障）→ seq_cst（顺序一致，**默认**）
5. **CAS 原语**：`compare_exchange_weak`（可能虚假失败、须配循环、性能优先）vs `compare_exchange_strong`（无假失败、略慢）——无锁编程的基石

---