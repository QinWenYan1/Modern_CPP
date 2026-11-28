# 第 17 章逐知识点来源对照

核对日期为 2026-11-29。每篇交付时，在聊天中给出同一份逐点对照，在此保留具体位置。下列 35 项对应当前工作区的 17.1–17.4。定位指向用来核对知识点的正文，范围是所列结论和例子。用户阅读、整篇验收及本轮任务另见[项目状态](../STATE.md)和[当前计划](../PLAN.md)。

## 材料与读法

- **Williams** 指用户提供的 `Anthony Williams - C++并发编程实战(中文版) (0) - libgen.lc.pdf`。PDF 第 3 页列有英文书名 *C++ Concurrency in Action* 和副标题 *Practical Multithreading*，正文按这份中文 C++11 版本的节号定位。第二版网页目录不作为该 PDF 的正文证据。
- **Meyers** 指用户提供的英文 `Scott_Meyers_Effective_Modern_C++.pdf`。按 Item（条款）号定位。本文所有页码均为 **PDF 阅读器从 1 开始的物理页码**，例如该 PDF 第 260 页对应印刷页 242。两本书在原工作区的仓库同级 `ebook/`，换机器时按文件名定位。
- **旧笔记** 限于下列三份。第 17.1 旧稿 ①–④ 为并发背景、启动线程、线程标识与等待、首个多线程程序。第 17.2 旧稿 ①–④ 为互斥量与锁，⑤–⑦ 为 future 与 packaged_task，⑧–⑩ 为条件变量、生产消费与通知选择。旧 17.3 供对照原子及内存模型。具体用到的点在表内链接到原锚点。[旧 17.1][old1] · [旧 17.2][old2] · [旧 17.3][old3]
- **正文核对**表示本次从两份 PDF 的实际页面提取并读取了所列内容。直接沿用的旧例、原书清单片段、综合推导和自拟示例分开标注。没有历史讲义或聊天明示时，表中的可复核出处只说明这次核对到什么，初稿实际采用哪份资料仍待历史记录佐证。缺少正文支撑时在对应行标明。

<a id="s17-1"></a>
## 17.1 并发基础与线程管理

17.1 页首保留旧起草说明，记录了使用 N4659 正文及 Manning 第二版目录导览的历史。本表重新用获准的本地材料核对主体结论。历史上仅据导览定位的部分，不记作读过第二版正文。

| 当前知识点 | 可复核的资料正文（PDF 页） | 旧稿、例子与核对边界 |
|---|---|---|
| [1. 并发、并行与进程/线程](../../Chapter17_concurrency_parallelism/note_17_1_thread_management.md#id1) | Williams §1.1.1 计算机系统中的并发、§1.1.2 并发的途径，10–14 页。§1.2.1–1.2.3，14–16 页。§1.3.1–1.3.2，16–17 页。 | [旧 17.1 ①][o1_1] 对应 C++11 前的背景。进程与线程的对比、并发与并行的解释为整理。 |
| [2. std::thread 的创建与执行](../../Chapter17_concurrency_parallelism/note_17_1_thread_management.md#id2) | Williams §2.1.1 启动线程，22–24 页。附录 D.7.1 线程构造，519–520 页。Meyers Item 35，259–260 页核对普通线程入口的返回值。 | [旧 17.1 ②][o1_2] 的创建规则。保留[旧 17.1 ④][o1_4] 的 `hello world.` 代码。 |
| [3. 线程传参、引用捕获与对象生命周期](../../Chapter17_concurrency_parallelism/note_17_1_thread_management.md#id3) | Williams §2.2 向线程函数传递参数，29–32 页。§2.1.1 清单 2.1，24–25 页。附录 A.5.1 捕获局部变量，334–336 页。 | 整数自增与 `std::ref(value)` 是笔记自拟例。书中传引用与局部对象失效的例子是其核对依据。 |
| [4. join、detach、joinable 与线程对象状态](../../Chapter17_concurrency_parallelism/note_17_1_thread_management.md#id4) | Williams §2.1.2–2.1.4，25–28 页。附录 D.7.1，520–524 页。Meyers Item 37，268–270 页列出可汇合与不可汇合的状态。 | 承接[旧 17.1 ③][o1_3] 的 `join`。状态表和 `associated_after` 片段为整理。 |
| [5. 线程所有权与移动](../../Chapter17_concurrency_parallelism/note_17_1_thread_management.md#id5) | Williams §2.3 转移线程所有权，32–35 页。附录 D.7.1，520–521 页。 | 移动后由谁负责收尾，覆盖仍关联线程的对象时的后果均有正文支撑。变量 `first`、`second` 为自拟演示。 |
| [6. 线程析构、异常与 RAII 管理](../../Chapter17_concurrency_parallelism/note_17_1_thread_management.md#id6) | Williams §2.1.1 清单 2.1，24–25 页。§2.1.3 特殊情况下的等待，清单 2.2、2.3，25–27 页。构造/析构见附录 D.7.1，519–520 页。Meyers Item 37，269–275 页可交叉核对 RAII。 | 沿用书中 `f()` 和 `thread_guard` 两种写法。例中辅助函数的示意实现与书中两份清单分别看待。 |
| [7. 线程标识、休眠与让出执行](../../Chapter17_concurrency_parallelism/note_17_1_thread_management.md#id7) | Williams §2.5 识别线程，38–39 页。D.7.1 的 `get_id`，523 页。D.7.2 的 `this_thread` 接口，524–526 页。 | [旧 17.1 ③][o1_3] 对应标识。休眠 10 毫秒片段是自拟示例。 |
| [8. thread_local 与线程局部存储](../../Chapter17_concurrency_parallelism/note_17_1_thread_management.md#id8) | Williams 附录 A.8 线程本地变量，341–342 页，含每线程独立实例、线程退出时析构、跨线程指针的寿命。 | 线程私有整数计数器为自拟示例。 |

<a id="s17-2"></a>
## 17.2 共享数据与互斥同步

| 当前知识点 | 可复核的资料正文（PDF 页） | 旧稿、例子与核对边界 |
|---|---|---|
| [1. 竞态条件使结果依赖执行顺序，数据竞争导致未定义行为](../../Chapter17_concurrency_parallelism/note_17_2_sharing_data_between_threads.md#id1) | Williams §3.1、§3.1.1–3.1.2，40–43 页，图 3.1。§5.1.1–5.1.2，109–111 页区分内存位置和数据竞争。 | [旧 17.2 ①][o2_1] 的临界区概念是前置。双链表来自原书，电影票和普通整数是解释用例。 |
| [2. 读写链表共用一把 mutex，由 lock_guard 自动解锁](../../Chapter17_concurrency_parallelism/note_17_2_sharing_data_between_threads.md#id2) | Williams §3.2.1 C++ 中使用互斥量，43–44 页，清单 3.1。 | [旧 17.2 ①][o2_1]、[②][o2_2]、[③][o2_3] 提供互斥量、手动解锁与 RAII 的已有概念。链表代码来自原书。 |
| [3. 传出内部指针或引用可能让调用者绕过锁](../../Chapter17_concurrency_parallelism/note_17_2_sharing_data_between_threads.md#id3) | Williams §3.2.2，44–46 页，清单 3.2。§5.2.7，122–123 页阐述针对 `shared_ptr` 的原子访问。Meyers Item 19，143–144 页阐述共享计数。 | 泄露内部地址的回调来自清单。指针变量、不同副本和目标对象三层边界是结合引用计数与 §5.1.2 的整理。 |
| [4. 线程安全栈应把判空、取值和删除合成一次出栈操作](../../Chapter17_concurrency_parallelism/note_17_2_sharing_data_between_threads.md#id4) | Williams §3.2.3，46–52 页，清单 3.3–3.5。§5.1.1–5.1.2，109–111 页。Meyers Item 6，61–62 页展示 `vector<bool>` 的位存储与代理。 | 合并出栈步骤的接口来自原书。编号 42、17 和时序图为教学演绎。不同下标的说明综合了位存储与内存位置规则。 |
| [5. 用 std::lock 一起获取多把锁，避免加锁顺序造成死锁](../../Chapter17_concurrency_parallelism/note_17_2_sharing_data_between_threads.md#id5) | Williams §3.2.4–3.2.5，52–59 页，清单 3.6。§7.1.2–7.1.4，187–188 页解释饥饿、活锁及进展。 | 交换代码取自清单 3.6。持锁等待线程、回调的环形等待和饥饿表里的排队情境是综合说明。 |
| [6. 用 unique_lock 把独立计算移到锁外，缩短持锁时间](../../Chapter17_concurrency_parallelism/note_17_2_sharing_data_between_threads.md#id6) | Williams §3.2.6–3.2.8，59–63 页，`defer_lock`、移动、`get_and_process_data` 与比较两个对象的局限。§4.1.1，72–74 页核对等待时解锁。 | 承接[旧 17.2 ④][o2_4] 的 `unique_lock`。取数据、锁外计算、加锁写回选自 §3.2.8。 |
| [7. call_once 和局部 static 保护初始化，后续并发修改仍需同步](../../Chapter17_concurrency_parallelism/note_17_2_sharing_data_between_threads.md#id7) | Williams §3.3.1 保护共享数据的初始化过程，64–67 页，清单 3.11、3.12 及局部静态对象示例。 | 收、发共享同一 `once_flag` 的代码按书例简化。后续收发需要单独判断同步，依据原书的初始化边界。 |
| [8. 读写锁允许并发读取，const 查询写缓存时也要独占](../../Chapter17_concurrency_parallelism/note_17_2_sharing_data_between_threads.md#id8) | Williams §3.3.2，67–69 页，清单 3.13 的 Boost DNS 缓存。Meyers Item 16，121–124 页核对 `const` 查询内部写缓存。 | DNS 表根据书中两个成员函数整理。缓存可能写数据的补充来自 Meyers。 |
| [9. 递归锁允许同一线程重复加锁，定时锁支持限时等待](../../Chapter17_concurrency_parallelism/note_17_2_sharing_data_between_threads.md#id9) | Williams §3.3.3 嵌套锁，69 页。D.5.1–5.4，476–491 页，定时获取见 D.5.3，485–486 页。 | 递归加锁、尝试取得锁和限时获取的对照表，以及批量写入场景，按接口条件整理。 |

<a id="s17-3"></a>
## 17.3 条件变量与线程协作

| 当前知识点 | 可复核的资料正文（PDF 页） | 旧稿、例子与核对边界 |
|---|---|---|
| [1. 队列为空时，消费者可以睡眠等待任务到来](../../Chapter17_concurrency_parallelism/note_17_3_condition_variables.md#id1) | Williams §4.1，71–72 页的轮询与 100 毫秒休眠。§4.1.1，72–74 页引入条件变量。 | [旧 17.2 ⑧][o2_8] 的等待需求。消费者刚睡下时到货的时间顺序为讲解。 |
| [2. 队列保存任务，通知提醒消费者检查队列](../../Chapter17_concurrency_parallelism/note_17_3_condition_variables.md#id2) | Williams §4.1.1，72–74 页，清单 4.1。Meyers Item 39，281–285 页核对条件与通知的区别。 | [旧 17.2 ⑨][o2_9] 提供 `produced_nums`、`notified` 和生产者入队片段。 |
| [3. wait 在睡眠前释放锁，返回前重新拿锁](../../Chapter17_concurrency_parallelism/note_17_3_condition_variables.md#id3) | Williams §4.1.1，清单 4.1，72–74 页。D.2.1 的 `wait` 与谓词重载，388–389 页。 | 条件变量的已有概念见[旧 17.2 ⑧][o2_8]。带谓词的等待片段来自清单，时序图是整理。 |
| [4. 通知可能先到，睡前检查队列就不会错过已有任务](../../Chapter17_concurrency_parallelism/note_17_3_condition_variables.md#id4) | Williams D.2.1 的 `notify_one/all`、`wait`，387–389 页。Meyers Item 39，281–285 页描述先通知后等待的情形。 | 结合[旧 17.2 ⑨][o2_9] 的队列状态，例中的任务 42、不同到达顺序为演绎。 |
| [5. 醒来后重新检查条件，再决定是否取任务](../../Chapter17_concurrency_parallelism/note_17_3_condition_variables.md#id5) | Williams §4.1.1，73–74 页，D.2.1，388–389 页。Meyers Item 39，282–284 页。 | [旧 17.2 ⑨][o2_9] 的 `while (!notified)`。真正收到通知、但其他消费者先取走任务的分支是结合旧例的解释。 |
| [6. wait_and_pop 把等待和出队合起来，任务在锁外处理](../../Chapter17_concurrency_parallelism/note_17_3_condition_variables.md#id6) | Williams §4.1.2 使用条件变量构建线程安全队列，75–79 页，清单 4.3–4.5。清单 4.4 是等待取值，清单 4.5 的 `try_pop(T&)` 在 79 页。 | [旧 17.2 ⑨][o2_9] 在同一轮中取完一批。与书中一次取一项的接口做对照。任务 1、2、3 为解释。 |
| [7. 一项任务可唤醒一个消费者，所有人都要响应时再广播](../../Chapter17_concurrency_parallelism/note_17_3_condition_variables.md#id7) | Williams §4.1.2，79–80 页，比较一个数据项与多方响应事件。D.2.1，387–388 页。 | [旧 17.2 ⑩][o2_10] 比较两种通知。消费者都醒来但只有一人领到任务的表格为整理。 |
| [8. 限时等待要守住同一个截止时刻](../../Chapter17_concurrency_parallelism/note_17_3_condition_variables.md#id8) | Williams §4.3.2–4.3.4，93–97 页，尤其 §4.3.3 清单 4.11（95–96 页）。D.2.1 的限时等待，389–393 页。 | 500 毫秒的 `wait_loop` 来自清单。第 490 毫秒被唤醒的时间线为讲解。 |
| [9. 关闭队列时叫醒所有消费者，取完剩余任务再退出](../../Chapter17_concurrency_parallelism/note_17_3_condition_variables.md#id9) | Williams §4.1.1 清单 4.1，72–74 页，最后一块数据令循环退出。§4.1.2，79–80 页，所有等待者都要响应时广播。 | 从[旧 17.2 ⑨][o2_9] 的持续循环扩展。共享 `closed`、排空后返回及两个消费者退出是笔记选定的设计，书中没有这段关闭代码。 |

<a id="s17-4"></a>
## 17.4 异步任务与结果传递

| 当前知识点 | 可复核的资料正文（PDF 页） | 旧稿、例子与核对边界 |
|---|---|---|
| [1. 用 future 接收一次任务的结果](../../Chapter17_concurrency_parallelism/note_17_4_asynchronous_tasks_and_futures.md#id1) | Williams §4.2、§4.2.1，80–81 页，清单 4.6。Meyers Item 35，259–260 页，Item 38，276–277 页的共享状态。 | [旧 17.2 ⑤][o2_5] 提供 future 的已有概念。整数例来自清单。当前段末新增的 `std::optional` 对照另见下方待核对项。 |
| [2. async 提交函数，执行策略决定函数何时运行](../../Chapter17_concurrency_parallelism/note_17_4_asynchronous_tasks_and_futures.md#id2) | Williams §4.2.1，81–83 页，D.4.5，474–475 页。Meyers Item 36，263–268 页解释默认、异步、延迟策略。 | 在清单 4.6 的计算场景上显式设置 `launch::deferred`，是笔记改写。 |
| [3. wait 等就绪，get 取结果，限时等待还要识别延迟任务](../../Chapter17_concurrency_parallelism/note_17_4_asynchronous_tasks_and_futures.md#id3) | Williams §4.3.2，94 页的 35 毫秒例。D.4.1 的 `valid/wait/get` 及限时等待，451–454 页。Meyers Item 36，265–266 页。 | [旧 17.2 ⑥][o2_6] 使用等待“屏障”的说法。笔记区分等结果与等线程退出。 |
| [4. 用 packaged_task 把任务交给指定线程，提交者用 future 取回结果](../../Chapter17_concurrency_parallelism/note_17_4_asynchronous_tasks_and_futures.md#id4) | Williams §4.2.2 任务与期望，83–86 页，清单 4.8、4.9。 | 第一段 `return 7` 是[旧 17.2 ⑦][o2_7] 的例子。GUI 线程的任务队列来自清单 4.9，整数任务的队列时序图为改写。 |
| [5. 用 promise 写入结果，用 future 取回结果](../../Chapter17_concurrency_parallelism/note_17_4_asynchronous_tasks_and_futures.md#id5) | Williams §4.2.2，83–84 页。§4.2.3，86–87 页，清单 4.10 发包分支。D.4.4，468–471 页。D.7.1，522–523 页的 `join`。Meyers Item 35，259–260 页对照普通线程与任务取值。 | [旧 17.2 ⑦][o2_7] 对应打包任务。两种 `3 + 4` 是自拟对照。发完一个包时写入布尔值是书中场景。先前聊天已报告两种片段在 C++11 下验证得到 7。 |
| [6. 计算失败也能交付，调用者在 get 处收到异常](../../Chapter17_concurrency_parallelism/note_17_4_asynchronous_tasks_and_futures.md#id6) | Williams §4.2.4 为“期望”存储“异常”，87–89 页。D.4.4，470–472 页。 | 负数开平方与 `set_exception(current_exception())` 有正文对应。“负责交付者离开”和 `broken_promise` 见 89 页。 |
| [7. 多个线程读取同一结果时，各持一份 shared_future 副本](../../Chapter17_concurrency_parallelism/note_17_4_asynchronous_tasks_and_futures.md#id7) | Williams §4.2.5 多个线程的等待，89–91 页，图 4.1。D.4.1 的 `share`，451 页。D.4.2，454–460 页。 | 以两个读者取同一个整数和 Mermaid 图解释副本共享状态，属于笔记重述。 |
| [8. 最后一个 future 析构时，可能要等 async 启动的线程结束](../../Chapter17_concurrency_parallelism/note_17_4_asynchronous_tasks_and_futures.md#id8) | Meyers Item 38，276–280 页，尤其 278 页的三个条件。Williams D.4.5，474–475 页。 | 是否由 async 创建、是否实际异步、是否是最后一个关联者。任务已结束时无须再等。用 `write_report` 示意，非原书完整程序。 |
| [9. 只需通知一次完成时，用 `promise<void>` 和 `future<void>`](../../Chapter17_concurrency_parallelism/note_17_4_asynchronous_tasks_and_futures.md#id9) | Meyers Item 39，284–288 页，比较一次性通知与条件变量，并用共享 future 通知多方。Williams §4.1.2 末、§4.2，79–80 页。 | 设置 `port = 8080` 的启动通知为自拟示例，依据书中的一次性事件。 |

<a id="optional-gap"></a>
## 待补证的位置

17.4 知识点 1 的 C++17 `std::optional` 与 `std::nullopt` 对比在当前笔记中可见。清单 4.6 只支撑 future 一侧。本次对两份 PDF 正文和三份指定旧稿未定位到相应 optional 说明，因此将这部分标为**来源待确认**。保留原文，等用户给出来源或决定处理。[问题 A5](../AUDIT.md#a5)

后续笔记写入或用户改动某一知识点时，重新核对受影响的行。聊天报告应交代书中正文与笔记补例分别支撑什么。

[old1]: ../../Chapter17_concurrency_parallelism/note_17_1_thread_basics.md
[o1_1]: ../../Chapter17_concurrency_parallelism/note_17_1_thread_basics.md#id1
[o1_2]: ../../Chapter17_concurrency_parallelism/note_17_1_thread_basics.md#id2
[o1_3]: ../../Chapter17_concurrency_parallelism/note_17_1_thread_basics.md#id3
[o1_4]: ../../Chapter17_concurrency_parallelism/note_17_1_thread_basics.md#id4
[old2]: ../../Chapter17_concurrency_parallelism/note_17_2_synchronization_primitives.md
[o2_1]: ../../Chapter17_concurrency_parallelism/note_17_2_synchronization_primitives.md#id1
[o2_2]: ../../Chapter17_concurrency_parallelism/note_17_2_synchronization_primitives.md#id2
[o2_3]: ../../Chapter17_concurrency_parallelism/note_17_2_synchronization_primitives.md#id3
[o2_4]: ../../Chapter17_concurrency_parallelism/note_17_2_synchronization_primitives.md#id4
[o2_5]: ../../Chapter17_concurrency_parallelism/note_17_2_synchronization_primitives.md#id5
[o2_6]: ../../Chapter17_concurrency_parallelism/note_17_2_synchronization_primitives.md#id6
[o2_7]: ../../Chapter17_concurrency_parallelism/note_17_2_synchronization_primitives.md#id7
[o2_8]: ../../Chapter17_concurrency_parallelism/note_17_2_synchronization_primitives.md#id8
[o2_9]: ../../Chapter17_concurrency_parallelism/note_17_2_synchronization_primitives.md#id9
[o2_10]: ../../Chapter17_concurrency_parallelism/note_17_2_synchronization_primitives.md#id10
[old3]: ../../Chapter17_concurrency_parallelism/note_17_3_atomic_and_memory_model.md
