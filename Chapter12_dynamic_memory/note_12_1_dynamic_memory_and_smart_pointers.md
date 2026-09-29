# 📘 12.1 Dynamic Memory and Smart Pointers (动态内存和智能指针)

> 来源说明：《C++ Primer》12.1；知识点1–5结合[小林 coding：什么是指针？指针的大小及用法？](https://www.xiaolincoding.com/interview/cpp.html#什么是指针-指针的大小及用法)补充，示例为整理编写，并参考下文链接中的 C++ 标准草案核对关键规则。
> 知识点19结合[小林 coding：什么是野指针和悬空指针？](https://www.xiaolincoding.com/interview/cpp.html#什么是野指针和悬空指针)补充，示例为整理编写。
> 本节涵盖：原始指针的概念、大小与用途，野指针与悬空指针，动态内存的手动管理（new/delete），以及智能指针（shared_ptr、unique_ptr、weak_ptr）的原理和使用方法。

---

## 🧠 核心概念总览

* [*知识点1: 原始指针的概念与大小*](#id1)
* [*知识点2: 指向普通对象与常量的指针*](#id2)
* [*知识点3: 函数指针*](#id3)
* [*知识点4: 数据成员指针与成员函数指针*](#id4)
* [*知识点5: this 指针*](#id5)
* [*知识点6: 动态内存与对象生命周期*](#id6)
* [*知识点7: 动态内存管理的问题与智能指针简介*](#id7)
* [*知识点8: shared_ptr 类的基本使用*](#id8)
* [*知识点9: make_shared 函数*](#id9)
* [*知识点10: shared_ptr 的拷贝、赋值与引用计数*](#id10)
* [*知识点11: shared_ptr 自动销毁对象与释放内存*](#id11)
* [*知识点12: 使用动态内存的类示例 - StrBlob*](#id12)
* [*知识点13: 直接使用 new 和 delete 管理内存*](#id13)
* [*知识点14: 使用 new 动态分配和初始化对象*](#id14)
* [*知识点15: 动态分配 const 对象与内存耗尽*](#id15)
* [*知识点16: 使用 delete 释放动态内存*](#id16)
* [*知识点17: 动态分配对象的生存期与常见错误*](#id17)
* [*知识点18: 重置指针与悬挂指针*](#id18)
* [*知识点19: 野指针与悬空指针*](#id19)
* [*知识点20: 使用 shared_ptr 配合 new*](#id20)
* [*知识点21: 不要混合使用普通指针和智能指针*](#id21)
* [*知识点22: 不要用 get() 初始化或赋值另一个智能指针*](#id22)
* [*知识点23: shared_ptr 的其他操作（reset与unique）*](#id23)
* [*知识点24: 智能指针与异常*](#id24)
* [*知识点25: 智能指针与“哑”类（自定义删除器）*](#id25)
* [*知识点26: unique_ptr 独占所有权*](#id26)
* [*知识点27: 传递和返回 unique_ptr*](#id27)
* [*知识点28: 向 unique_ptr 传递删除器*](#id28)
* [*知识点29: weak_ptr 弱引用*](#id29)
* [*知识点30: 使用 weak_ptr 的示例 - StrBlobPtr*](#id30)

---

<a id="id1"></a>
## ✅ 知识点1: 原始指针的概念与大小

**理论**
* **原始指针(raw pointer)**: 
    - 通常指 `T*` 这样的内置指针类型，用于与智能指针类区分。
    - 指向对象的指针可以保存对象地址，通过 **解引用(dereference)** 间接访问对象。
* **指针本身与所指对象是两个不同的实体**：
    - `p` 是指针对象，`*p` 表示所指对象，`&p` 则是指针对象自己的地址
* **指针大小与目标类型大小要分开理解**：
    - `sizeof(p)` 查询指针类型的大小，`sizeof(*p)` 查询所指类型的大小
    * 常见 64 位程序中的 `int*`、`char*` 等等通常都是 8 字节
    * **大小由具体实现决定**
        - 不能仅凭计算机是“64 位”推断；成员指针也不能直接套用普通指针的大小结论
        - 常见平台上，32 位程序的普通对象指针通常占 4 字节，64 位程序通常占 8 字节
        - 这里看的是程序的编译目标，而不只是电脑是否为 64 位。
        - 成员指针属于不同的类型，其大小不一定与普通指针相同；实际大小可用 sizeof 查询


**示例代码**
```cpp
int value = 10;
int* p = &value;

*p = 20;                       // 修改 value，不是修改 p 的指向
const auto pointer_bytes = sizeof(p);  // int* 的大小
const auto object_bytes = sizeof(*p);  // int 的大小
```

**注意点**
> 🔄 指针定义、赋值与解引用详见[2.3 复合类型](../Chapter02_variable_type/note_2_3_compound_types.md)；`sizeof` 的不求值特性详见[4.9 sizeof 运算符](../Chapter04_expression/note_4_9_size_of_operator.md)。

---

<a id="id2"></a>
## ✅ 知识点2: 指向普通对象与常量的指针

**理论**
* **指针可以指向内置类型对象，也可以指向类对象**:
    - **不必使用 `new` 才能获得对象指针**，可以直接对已有对象取地址。
* 对于类对象指针: 
    - `p->member` 等价于 `(*p).member`，用于访问数据成员或调用成员函数
* **指向常量的指针(pointer to const)** 写作 `const T*`：
    - 不能通过它修改目标对象；它也可以指向非常量对象，限制的是 **通过该指针进行修改**
* 注意与 **const 指针(const pointer)** `T* const` 区分：
    - 前者限制通过指针修改对象
    - 后者限制改变指针自身的指向

**示例代码**
```cpp
struct Item {
    int count = 0;
};

Item item;
Item* p = &item;
p->count = 3;                 // 修改 item.count

const Item* read_only = &item;
int n = read_only->count;     // 可以读取，n 为 3
// read_only->count = 4;      // 错误：不能通过它修改 count
item.count = 4;               // 正确：item 本身不是 const 对象
```

**注意点**
> ⚠️ **持有地址不等于拥有资源**：这里的 `p` 指向局部对象，不能对它执行 `delete`；对象生命周期与释放规则在[知识点6](#id6)及后续 `new/delete` 内容中展开。
> 💡 `const T*` 中的 `const` 描述所指类型的访问限制，不意味着指针本身不能重新赋值。
> 🔄 完整的 const 组合与类型转换规则参见[2.4 const 限定符](../Chapter02_variable_type/note_2_4_qualifier.md)，此处只保留用途示例。

---

<a id="id3"></a>
## ✅ 知识点3: 函数指针

**理论**
* **函数指针(function pointer)**: 用于指定函数，通过它可以间接调用该函数。
* `int (*f)(int, int)`:
    - 声明一个指针，目标函数接收两个 `int`，返回 `int`；括号将 `*f` 结合在一起
* **普通函数名可以转换为函数指针**，调用时可写 `f(a, b)` 或 `(*f)(a, b)`

**示例代码**
```cpp
int add(int a, int b) {
    return a + b;
}

int main() {
    int (*f)(int, int) = &add;
    int result = f(2, 3);     // result 为 5
    return result == 5 ? 0 : 1;
}
```

**注意点**
> ⚠️ 去掉括号写成 `int* f(int, int)`，声明的就是返回 `int*` 的函数。
> 💡 函数指针可以用来传递“要执行哪个函数”，例如回调(callback)；它不承担动态对象的释放责任。
> 🔄 函数指针的参数传递、重载匹配和复杂声明已经在[6.7 函数指针](../Chapter06_functions/note_6_7_pointers_to_functions.md)讲解，这里不重复展开。

---

<a id="id4"></a>
## ✅ 知识点4: 数据成员指针与成员函数指针

**理论**
* **指向具体对象成员的普通指针**：
    - `&obj.value`：已经定位到 `obj` 中的那个成员对象
* **数据成员指针(pointer to data member) 定义**：
    - `int Counter::*`：指定类的某个非静态数据成员，使用时还需要提供具体对象
* **成员函数指针(pointer to member function) 定义**：
    - `void (Counter::*)(int)`：指定普通非静态成员函数，调用时同样需要结合具体对象
* 形成成员指针使用 **`&类名::成员名`**；访问方式取决于左侧是对象还是对象指针：

    | 操作 | 通过对象 | 通过对象指针 |
    | --- | --- | --- |
    | 访问数据成员 | `obj.*member` | `ptr->*member` |
    | 调用成员函数 | `(obj.*action)(参数)` | `(ptr->*action)(参数)` |

**示例代码**
```cpp
struct Counter {
    int value = 0;

    void add(int n) {
        value += n;
    }
};

int main() {
    Counter a;
    Counter b;
    Counter* ptr = &b;

    int* direct = &a.value;                   // 普通 int*，指向 a.value
    int Counter::* member = &Counter::value; // 数据成员指针
    void (Counter::* action)(int) = &Counter::add; // 成员函数指针

    *direct = 10;             // a.value 为 10
    a.*member = 20;           // a.value 为 20
    ptr->*member = 30;        // b.value 为 30
    (a.*action)(5);           // a.value 为 25
    (ptr->*action)(7);        // b.value 为 37

    return a.value == 25 && b.value == 37 ? 0 : 1;
}
```

**注意点**
> ⚠️ **成员指针与普通指针属于不同类型类别**：不能把 `int Counter::*` 当成 `int*`，也不能把这里的成员函数指针当成普通 `void (*)(int)`。
> ⚠️ 形成普通非静态成员函数指针时，必须写 `&Counter::add`，不能像普通函数那样省略 `&`。参见[标准草案：取地址与成员指针](https://eel.is/c++draft/expr.unary.op)。
> 💡 同一个成员指针可以用于不同对象：`member` 指定“哪个成员”，`a` 或 `b` 指定“哪个对象”。
> 🔄 静态成员没有绑定到某个对象，对静态数据成员或静态成员函数取地址得到的是普通指针；静态成员基础参见[7.6 静态类成员](../Chapter07_classes/note_7_6_static_class_member.md)。

---

<a id="id5"></a>
## ✅ 知识点5: `this` 指针

**理论**
* **`this` 指针(this pointer)**:
    -  在普通非静态成员函数中指向调用该函数的当前对象，可以用 `this->member` 显式访问成员
* 当参数与数据成员同名时，可用 `this->` 明确表示数据成员
* 对类 `Counter` 而言：
    - 普通非 `const`、非 `volatile` 成员函数中的 `this` 类型为 `Counter*`
    - 对应的 `const` 成员函数中为 `const Counter*`。
* **`this` 是指针类型的纯右值(prvalue)表达式，不是一个可以重新赋值的指针变量**
    - 不能因为 `this = ...` 不合法，就把其实际类型写成 `Counter* const`。参见[标准草案：this](https://eel.is/c++draft/expr.prim.this)。

**示例代码**
```cpp
class Counter {
public:
    void set(int value) {
        this->value = value;  // 左侧为成员，右侧为参数
    }

    int get() const {
        return this->value;  // const 成员函数通过 const Counter* 读取
    }

private:
    int value = 0;
};
```

**注意点**
> ⚠️ **静态成员函数没有 `this`**，因为调用它不需要某个具体对象。
> 💡 将 `this` 类比为隐式传入的对象地址，有助于理解调用机制；这个类比不等于一段真实的函数参数声明。
> 🔄 成员函数调用、const 成员函数与返回 `*this` 的应用参见[7.1 定义抽象数据类型](../Chapter07_classes/note_7_1_defining_abstract_data_type.md#id7)。关于 `this` 的实际类型，以本知识点的精确定义为准。

---

<a id="id6"></a>
## ✅ 知识点6: 动态内存与对象生命周期

**理论**
* **核心主旨总结**：
    * 程序中的对象有其**生命周期(lifetime)**。
    * **全局对象**(global objects)和**局部静态对象**(local static objects)在程序启动/首次使用时分配，在程序结束时销毁,使用的是**静态内存** (static memory)
    * **局部自动对象**(local, automatic objects)在进入/退出其定义块时创建和销毁,使用的是**栈内存**（stack memory）
* 除了自动和静态对象，C++允许我们<b>动态分配(dynamically allocate)</b>对象。
    * 动态分配对象的生存期与其创建位置无关，它们**一直存在直到被显式释放**，使用的是**堆内存** (heap memory/ free store)


**注意点**
* ⚠️ **对象生命周期的重要性**：理解不同存储类别（静态、自动、动态）对象的生命周期是管理内存和避免错误的基础。
* ⚠️ **重点**：正确地释放动态对象是一个复杂且容易出错的源头。


---

<a id="id7"></a>
## ✅ 知识点7: 动态内存管理的问题与智能指针简介

**理论**
* **核心主旨总结**：
    * C++通过一对运算符管理动态内存：**`new`**（分配并可选地初始化对象，返回指针）和 **`delete`**（销毁对象并释放内存）。
* **动态内存的问题在于很难确保在正确的时间释放内存**：
    * 忘记释放会导致**内存泄漏(memory leak)**；在仍有指针指向内存时释放会导致**无效指针(dangling pointer)**。
* 为了更安全、更容易地使用动态内存，新标准库提供了两种<b>智能指针(smart pointer)</b>类型：
    * **`shared_ptr`**（允许多个指针指向同一对象）
    * **`unique_ptr`**（“独占”所指对象）
    * 还有一个伴随类<b>`weak_ptr`</b>（弱引用）。
    * 它们都定义在 `<memory>` 头文件中。

**教材示例代码**
```cpp
// 代码片段展示了智能指针是模板，需要指定类型
shared_ptr<string> p1;    // 可以指向string的shared_ptr
shared_ptr<list<int>> p2; // 可以指向int列表的shared_ptr
```

**注意点**
* ⚠️ **内存管理难题**：手动管理动态内存是C++程序错误的常见来源。
* 💡 **解决方案**：智能指针像常规指针一样使用，但关键区别在于它们能<b>自动删除(auto-delete)</b>所指向的对象。


---

<a id="id8"></a>
## ✅ 知识点8: `shared_ptr` 类的基本使用

**理论**
* **核心主旨总结**：
    * `shared_ptr` 是模板，创建时必须提供指向的类型。
    * **默认初始化的智能指针保存着一个空指针(nullptr)**。
* 使用智能指针的方式与使用普通指针类似：
    * 解引用操作返回所指对象
    * 在条件中使用智能指针是检查其是否为空。

**教材示例代码**
```cpp
shared_ptr<string> p1; // 默认初始化，持有空指针
if (p1 != nullptr) {
    *p1 = "hi"; // 如果p1非空，则解引用并赋值
}
```

**Table 12.1. `shared_ptr` 和 `unique_ptr` 通用的操作**


| 操作 | 描述 |
| --- | --- |
| `shared_ptr<T>` `unique_ptr<T> up` | 空智能指针，可以指向类型为 `T` 的对象。 |
| `p` | 将 `p` 用作条件判断；如果 `p` 指向一个对象，则为 `true`。 |
| `*p` | 解引用 `p` 以获得 `p` 所指向的对象。 |
| `p->mem` | `(*p).mem` 的简写。 |
| `p.get()` | 返回 `p` 中保存的指针。**小心使用**：当智能指针被释放时，该操作返回的指针所指向的对象也会消失。 |
| `swap(p, q)` `p.swap(q)` | 交换 `p` 和 `q` 中的指针。 |


**Table 12.2. `shared_ptr`特有操作**

| 操作 | 描述 |
| --- | --- |
| `make_shared<T>(args)` | 返回一个 `shared_ptr`，指向一个动态分配的类型为 `T` 的对象。使用 `args` 初始化该对象。 |
| `shared_ptr<T> p(q)` | `p` 是 `shared_ptr q` 的拷贝；此操作会递增 `q` 中的计数。`q` 中的指针必须能转换为 `T*`。 |
| `p = q` | `p` 和 `q` 都是 `shared_ptr`，且所持指针可相互转换。此操作递减 `p` 的引用计数，递增 `q` 的引用计数；若 `p` 的计数变为 0，则释放其原有内存。 |
| `p.unique()` | 如果 `p.use_count()` 为 1，返回 `true`；否则返回 `false`。 |
| `p.use_count()` | 返回与 `p` 共享对象的智能指针数量；可能是一个耗时操作，主要用于调试目的。 |

---

<a id="id9"></a>
## ✅ 知识点9: `make_shared` 函数

**理论**
* **核心主旨总结**：
    * **最安全**的动态内存分配和使用方式是调用库函数 **`make_shared`**。
    * 它在动态内存中分配并初始化一个对象，并返回指向该对象的`shared_ptr`。
* 调用时必须指定要创建的对象类型（模板参数）：
    * `make_shared`使用其参数来**直接构造**给定类型的对象，如同`emplace`一样。
    * 如果不传参数，则对象被**值初始化(value-initialized)**。
* 通常使用 `auto` 来简化定义。

**教材示例代码**
```cpp
// shared_ptr指向一个值为42的int
shared_ptr<int> p3 = make_shared<int>(42);
// p4指向一个值为“9999999999”的string
shared_ptr<string> p4 = make_shared<string>(10, '9');
// p5指向一个值初始化为0的int
shared_ptr<int> p5 = make_shared<int>();
// 使用auto
auto p6 = make_shared<vector<string>>(); // 指向动态分配的空vector<string>
```

**注意点**
* 💡 **最佳实践**：
    * **优先使用 `make_shared`** 而不是直接 `new`，因为它更安全，将分配和绑定智能指针合为一步。


---

<a id="id10"></a>
## ✅ 知识点10: `shared_ptr` 的拷贝、赋值与引用计数

**理论**
* **核心主旨总结**：
    * 每个`shared_ptr`都会记录有多少其他`shared_ptr`指向同一个对象
    * 每个`shared_ptr`所指向的内存都有一个关联的计数器，称为**引用计数(reference count)**。
* **拷贝或赋值`shared_ptr`时，引用计数递增**（例如，初始化另一个`shared_ptr`、作为赋值右值、传参或返回值，因为有拷贝发生）。
* **当给`shared_ptr`赋新值或`shared_ptr`本身被销毁时（如离开作用域），其引用计数递减**。
* **一旦某个`shared_ptr`的引用计数变为0，它会自动释放所管理的对象**。

**教材示例代码**
```cpp
auto p = make_shared<int>(42); // p指向的对象有一个使用者
auto q(p); // p和q指向同一对象，该对象有两个使用者
auto r = make_shared<int>(42); // r指向的对象有一个使用者
r = q; // 给r赋值，使其指向q的对象
        // 增加q所指对象的引用计数
        // 减少r原先所指对象的引用计数（变为0，自动释放）
```

**注意点**
* 🔧 **实现细节**：
    * 具体使用计数器还是其他数据结构来跟踪共享状态由实现决定，关键是类会自动跟踪并适时释放对象。


---

<a id="id11"></a>
## ✅ 知识点11: `shared_ptr` 自动销毁对象与释放内存

**理论**
* **核心主旨总结**：
    * 当最后一个指向某对象的`shared_ptr`被销毁时，`shared_ptr`类通过其<b>析构函数(destructor)</b>自动销毁该对象并释放内存。
* 析构函数控制类类型**对象销毁时**发生什么，通常用于释放对象分配的资源（如`string`释放字符内存，`vector`释放元素内存）。
* `shared_ptr`的析构函数递减其所指对象的引用计数，如果计数变为0，则销毁对象并释放内存。

**教材示例代码**
```cpp
shared_ptr<Foo> factory(T arg) {
    return make_shared<Foo>(arg); // shared_ptr会负责删除这段内存
}

void use_factory(T arg) {
    shared_ptr<Foo> p = factory(arg);
    // 使用p
} // p离开作用域，它指向的内存被自动释放

shared_ptr<Foo> use_factory(T arg) {
    shared_ptr<Foo> p = factory(arg);
    return p; // 返回时复制p，引用计数递增
} // p离开作用域，但所指内存并未释放，因为还有其他shared_ptr指向它
```
**代码解析**
* 在第一个`use_factory`函数中：
    * 因为`p`对于`use_factory`函数是本地，`use_factory`结束后`p`同样也会被销毁。
    * 当`p`被销毁，引用计数就会减少并被检查。
    * 在这个例子中, `p`是唯一指向这个`shared_ptr`内存的对象。
    * 因此这块内存也会被释放。
* 在第二个`use_factory`函数中：
    * 函数返回了一个`p`的副本到调用点。
    * 复制`shared_ptr`会增加引用计数。
    * 因此即使`p`被销毁，但这一块内存还暂时不会被销毁
    * 因此程序会正确运行但是可能浪费内存，如果我们忽视了这个返回值。

**注意点**
* 💡 **关键优势**：
    * `shared_ptr`通过在不再被使用时自动释放动态对象，显著降低了动态内存管理的难度，尤其是在函数返回时。
* ⚠️ **内存释放时机**：
    * **内存直到最后一个`shared_ptr`消失才会被释放**。需要管理好`shared_ptr`的生命周期。
* ⚠️ **注意容器中的`shared_ptr`**：
    * 如果将`shared_ptr`放入容器，之后不再需要所有元素，**务必记得手动擦除(erase)不再需要的`shared_ptr`元素**
    * 因为容器里每个`shared_ptr`都被赋值给了容器元素，所以counter至少都是1
    * 否则可能导致内存浪费。

---

<a id="id12"></a>
## ✅ 知识点12: 使用动态内存的类示例 - `StrBlob`

**理论**
* **核心主旨总结**：
程序使用动态内存主要有**三个目**的：
    1. **不知道需要多少对象**
    2. **不知道所需对象的精确类型**
    3. **在多个对象间共享数据，分享共同的状态**
* 本节定义一个使用动态内存让多个对象共享底层数据的类`StrBlob`
    * 它使用`shared_ptr`来管理一个动态分配的`vector<string>`，从而实现拷贝时共享元素。
    * 如果使用一般`vector`的成员函数来共享的话。
    * 当保存这个`vector`的成员对象离开作用域，那这`vector`就会不复存在。
    * 所以欠妥， 因为其他对象就无法共享使用了，故我们使用动态内存来保存`vector`。
* `StrBlob`类定义包括构造函数:
    * `size`、`empty`、`push_back`、`pop_back`、`front`、`back`等成员，以及一个私有的`check`辅助函数用于边界检查。

**教材示例代码**
```cpp
class StrBlob {
public:
    typedef std::vector<std::string>::size_type size_type;
    StrBlob();
    StrBlob(std::initializer_list<std::string> il);
    size_type size() const { return data->size(); }
    bool empty() const { return data->empty(); }
    void push_back(const std::string &t) { data->push_back(t); }
    void pop_back();
    std::string& front();
    std::string& back();
private:
    std::shared_ptr<std::vector<std::string>> data;
    void check(size_type i, const std::string& msg) const;
};

// 构造函数
StrBlob::StrBlob(): data(make_shared<vector<string>>()) { }
StrBlob::StrBlob(initializer_list<string> il):
              data(make_shared<vector<string>>(il)) { }

// check 函数
void StrBlob::check(size_type i, const string &msg) const {
    if (i >= data->size())
        throw out_of_range(msg);
}

// 元素访问成员
string& StrBlob::front() {
    check(0, "front on empty StrBlob");
    return data->front();
}
string& StrBlob::back() {
    check(0, "back on empty StrBlob");
    return data->back();
}
void StrBlob::pop_back() {
    check(0, "pop_back on empty StrBlob");
    data->pop_back();
}
```

**注意点**
* 🔄 **拷贝、赋值和销毁**：
    * `StrBlob`使用默认合成版本的操作：赋值，复制，以及析构。
    * 其唯一的数据成员是`shared_ptr`，因此拷贝`StrBlob`会拷贝这个`shared_ptr`，从而增加引用计数。
    * 当最后一个指向底层`vector`的`StrBlob`被销毁时，`vector`会被自动销毁。


---

<a id="id13"></a>
## ✅ 知识点13: 直接使用 new 和 delete 管理内存

**理论**
* **核心主旨总结**：
    * 语言本身定义了`new`和`delete`运算符来直接分配和释放动态内存。
    * **与使用智能指针相比，直接使用这些运算符管理内存更容易出错**。
* 直接管理内存的类无法依赖默认合成版本的拷贝、赋值和销毁操作成员，需要自己定义。
* **强烈建议在阅读第13章（拷贝控制）之前，类只应使用智能指针来分配动态内存**。

**注意点**
* ⚠️ **警告**：**动态内存的管理非常棘手**。
* 💡 **最佳实践**：使用智能指针的程序通常更易于编写和调试。

---

<a id="id14"></a>
## ✅ 知识点14: 使用 new 动态分配和初始化对象

**理论**
* **核心主旨总结**：
    * `new`在自由存储区（堆）上构造一个<b>未命名(unnamed)</b>的对象，并返回指向该对象的指针。
* <b>默认情况下，动态分配的对象是默认初始化(default-initialized)</b>的。
    * 内置类型或复合类型的值未定义；类类型由默认构造函数初始化。
* 还可以使用以下方式初始化：
    * **直接初始化(direct initialization)**（圆括号）
    * 或**列表初始化(list initialization)**（花括号）
    * **值初始(value-initialization)**（类型名后跟空括号）

* 可以使用`auto`从单个初始化器推断要分配的对象的类型。


**教材示例代码**
```cpp
int *pi = new int; // pi指向一个动态分配的、未命名的、未初始化的int
string *ps = new string; // 初始化为空字符串
int *pi2 = new int(1024); // 直接初始化，值为1024
string *ps2 = new string{10, '9’}; // 列表初始化，*ps2为"9999999999"
vector<int> *pv = new vector<int>{0,1,2,3,4,5,6,7,8,9}; // 列表初始化
string *ps3 = new string(); // 值初始化为空字符串
int *pi3 = new int(); // 值初始化为0
auto p1 = new auto(obj); // p指向obj类型的对象，用obj初始化
// auto p2 = new auto{a,b,c}; // 错误：初始化器必须为单个
```

**注意点**
* 🔄 **默认初始化 vs 值初始化**：
    * 对于**类类型**，两者效果相同（都调用默认构造函数）。
    * 对于**内置类型**，**值初始化的对象有明确定义的值（如0），而默认初始化的对象值未定义**。
* ⚠️ **`auto`的限制**：
    * 使用`auto`进行类型推断时，**初始化器必须为单个且放在圆括号内**。
* ⚠️ **最佳实践**：
    * 与初始化变量一样，初始化动态分配的对象也是个好主意。

---

<a id="id15"></a>
## ✅ 知识点15: 动态分配 const 对象与内存耗尽

**理论**
* **核心主旨总结**：
    * 可以使用`new`分配<b>`const`对象</b>, 动态分配的`const`对象**必须被初始化**。
    * 具有默认构造函数的类类型可以隐式初始化，其他类型必须显式初始化。
    * 由于分配的对象是`const`的，`new`返回的是**指向const的指针**。
* **当自由存储空间耗尽时，`new`表达式会失败**。
    * 默认情况下，`new`会抛出`bad_alloc`异常。
    * 可以通过<b>定位new(placement new)</b>形式传递`nothrow`对象来阻止`new`抛出异常，此时若分配失败则返回空指针。
    * `bad_alloc`和`nothrow`定义在`<new>`头文件中。

**教材示例代码**
```cpp
// 分配并初始化一个const int
const int *pci = new const int(1024);
// 分配一个默认初始化的const空字符串
const string *pcs = new const string;

int *p1 = new int; // 如果分配失败，new抛出std::bad_alloc
int *p2 = new (nothrow) int; // 如果分配失败，new返回空指针
```

**注意点**

* 🔧 **错误处理**：使用`nothrow`的`new`允许程序在内存分配失败时进行恢复（检查返回的指针是否为空），而不是直接崩溃。

---

<a id="id16"></a>
## ✅ 知识点16: 使用 delete 释放动态内存

**理论**
* **核心主旨总结**：
    * 通过 **`delete`表达式**将动态内存归还给系统。
    * `delete`表达式接受一个指向我们想要释放的对象的指针。
* `delete`执行两个动作：
    * **销毁给定的指针指向的对象**，
    * 并**释放对应的内存**。
* **传递给`delete`的指针必须指向动态分配的内存或者是空指针**。
* **释放非`new`分配的内存，或多次释放同一个指针值，是未定义行为**。
* 虽然`const`对象的值不能修改，但对象本身可以被销毁。
    * 通过对其指针执行`delete`来释放`const`动态对象。

**教材示例代码**
```cpp
int i, *pi1 = &i, *pi2 = nullptr;
double *pd = new double(33), *pd2 = pd;
delete i;   // 错误：i不是指针
delete pi1; // 未定义：pi1指向局部变量（非动态分配）
delete pd;  // 正确
delete pd2; // 未定义：pd2指向的内存已被释放
delete pi2; // 正确：删除空指针总是安全的

const int *pci = new const int(1024);
delete pci; // 正确：删除一个const对象
```

**注意点**
* ⚠️ **严重错误**：
    * 编译器可能无法检测出对非动态分配指针或已释放指针的`delete`操作，这些错误很隐蔽且危险。
* 💡 **安全操作**：**删除空指针总是安全的**。

---

<a id="id17"></a>
## ✅ 知识点17: 动态分配对象的生存期与常见错误

**理论**
* **核心主旨总结**：
    * **通过内置指针（而非智能指针）管理的动态对象，直到被显式删除之前都会一直存在**。

* 返回（智能指针之外的）动态内存指针的函数给调用者增加了负担:
    * 调用者必须记得删除内存。
    * 调用者忘记删除是常见错误。

* 当内置指针离开作用域时，其指向的动态内存**不会**自动释放。
    * 如果这是指向该内存的唯一指针，内存将无法被访问和释放，导致内存泄漏。
* 直接内存管理有三个常见问题：
    1. **忘记删除内存导致的内存泄漏**。
    2. **使用已删除的对象**。
    3. **同一内存被删除两次**，比如两个指向同一内存的指针各被删除两次。
* 为了避免这些问题，我们可以：
    * **通过独占使用智能指针可以避免所有这些问题**。
    * 智能指针只在没有其他智能指针指向该内存时才删除它。

**教材示例代码**
```cpp
Foo* factory(T arg) {
    return new Foo(arg); // 调用者负责删除此内存
}

void use_factory(T arg) {
    Foo *p = factory(arg);
    // 使用p但不删除它
} // p离开作用域，但它指向的内存未被释放！（内存泄漏）

// 修正版本1：在函数内删除
void use_factory(T arg) {
    Foo *p = factory(arg);
    delete p; // 记得释放
}
// 修正版本2：返回指针，让调用者删除
Foo* use_factory(T arg) {
    Foo *p = factory(arg);
    return p; // 调用者必须删除内存
}
```

**注意点**
* ⚠️ **错误倾向**：
    * 这类错误远比发现和修复它们容易。

---

<a id="id18"></a>
## ✅ 知识点18: 重置指针与悬挂指针

**理论**
* **核心主旨总结**：
    * 删除指针后，该指针变为**无效(invalid)**
    * 在许多机器上它仍持有或指向（已释放）动态内存的地址，成为**悬挂指针(dangling pointer)**
* 避免悬挂指针问题的一个方法：
    - 是在指针本身即将离开作用域之前才删除其关联的内存
* 如果需要保留指针：
    - 可以在使用`delete`后**将其置为`nullptr`**，以明确表示该指针不指向任何对象


**教材示例代码**
```cpp
int *p(new int(42)); // p指向动态内存
auto q = p;          // p和q指向同一内存
delete p;            // p和q均失效
p = nullptr;         // 表明p不再绑定到对象（但q仍是悬挂指针！）
```

**注意点**
* ⚠️ **悬挂指针的危险**：
    * 悬挂指针具有未初始化指针的所有问题。
* ⚠️ **重置指针的局限性**：
    * 重置一个指针（如置为`nullptr`）只对该指针本身有效，**对其他仍指向（已释放）内存的指针没有影响**。
    * 在真实系统中，找出所有指向同一内存的指针异常困难。

---

<a id="id19"></a>
## ✅ 知识点19: 野指针与悬空指针

**理论**
* **悬空指针**：曾经指向有效对象，但该对象的生命周期已经结束，指针没有随之更新
    * 前一知识点展示了动态内存被释放后的情况；
    * **局部对象离开作用域**也会造成悬空，不一定要发生 `delete` 或 `free`
    * 核心是**所指对象已经不再有效**，不能因为原地址看起来没变，就继续通过它访问原对象
* **野指针(wild pointer)**是常见的非正式说法，本处按原文区分，指没有确定的有效指向的指针，典型情况是**未初始化的普通局部指针**。
    * 在函数内写 `int* p;`，并不意味着 `p` 自动等于 `nullptr`；使用前应为它赋予有效地址或空指针值。
    * 不要把这个结论套用到所有存储期：
        1. 例如没有显式初始化的全局指针
        2. `static` 指针会经过零初始化而得到空指针值。
* **空指针(null pointer)**具有确定的空值，表示不指向对象或函数。它与未初始化指针不同，但**同样不能用于访问所指对象**。

| 对比项 | 野指针（本处用法） | 悬空指针 | 空指针 |
| --- | --- | --- | --- |
| 典型来源 | 普通局部指针未初始化 | 所指对象销毁或存储被释放 | 初始化或赋值为 `nullptr` |
| 核心问题 | 没有建立确定的有效指向 | 原来的对象已不再有效 | 明确表示没有目标 |
| 能否通过它读写对象 | 不能 | 不能 | 不能 |
| 应对方式 | 使用前初始化为有效地址或 `nullptr` | 跟踪对象生命周期，失效后停止访问 | 使用前确认已指向仍有效的对象 |

**示例代码**
```cpp
void uninitialized_example() {
    int* p;               // 未初始化的普通局部指针，俗称野指针
    // if (p != nullptr) {} // 错误：判断本身就会读取未初始化的指针值
    // *p = 10;           // 错误：没有有效目标

    p = nullptr;          // 先给 p 一个确定的空值；赋值不会读取旧值
    int value = 10;
    p = &value;           // 现在才建立了有效指向
    *p = 20;              // 正确：value 仍在生命周期内
}
```

```cpp
void dangling_example() {
    int* p = nullptr;
    {
        int value = 42;
        p = &value;
        *p = 43;          // 正确：value 此时仍然存在
    }                     // value 生命周期结束，p 成为悬空指针

    // int result = *p;   // 错误：不能读取已销毁的对象
    // delete p;          // 错误：这个地址并非由 new 分配
    p = nullptr;          // 清除当前指针保存的失效指向
}
```

**注意点**
> 💡 **记忆区别**：野指针尚未建立可靠指向，悬空指针的原目标已经失效，空指针则明确表示没有目标。
> 🔄 `delete` 后置空及多个指针指向同一对象的例子已在[知识点18：重置指针与悬挂指针](#id18)讲解，这里不重复；普通局部对象的生命周期见[知识点6](#id6)。
> 📋 “野指针”在不同资料中可能有更宽泛的用法；本节采用原文区分，避免与“悬空指针”混用。

---

<a id="id20"></a>
## ✅ 知识点20: 使用 `shared_ptr` 配合 `new`

**理论**
* **核心主旨总结**：
    * 可以用`new`返回的指针来初始化智能指针。
    * **接受指针的智能指针构造函数是`explicit`的**，因此**不能将内置指针隐式转换为智能指针，必须使用直接初始化形式**。
* 默认情况下，用于初始化智能指针的指针必须指向动态内存，因为默认智能指针使用`delete`释放关联对象。
* 但可以绑定智能指针到指向其他类型资源的指针，前提是提供自定义的操作（删除器）来代替`delete`。
* `shared_ptr` 的构造与 `reset` 操作（表格）：

| 操作                       | 说明                                                                         |
| ------------------------ | -------------------------------------------------------------------------- |
| `shared_ptr<T> p(q)`     | `p` 管理内置指针 `q` 所指向的对象；`q` 必须指向由 `new` 分配的内存，并且必须可转换为 `T*`。                 |
| `shared_ptr<T> p(u)`     | `p` 从 `unique_ptr u` 中接管对象的所有权；`u` 被置为空。                                   |
| `shared_ptr<T> p(q, d)`  | `p` 接管内置指针 `q` 所指向的对象；`q` 必须可转换为 `T*`。`p` 在释放对象时使用可调用对象 `d`（而不是 `delete`）。 |
| `shared_ptr<T> p(p2, d)` | `p` 是 `shared_ptr p2` 的一个拷贝（如表 12.2 所述），但在释放对象时使用可调用对象 `d`（而不是 `delete`）。  |
| `p.reset()`              | 如果 `p` 是唯一指向该对象的 `shared_ptr`，则释放该对象；随后将 `p` 置为空。                          |
| `p.reset(q)`             | 若 `p` 是唯一指向其当前对象的 `shared_ptr`，则释放该对象；随后令 `p` 指向内置指针 `q`（若未传 `q`，则置为空）。    |
| `p.reset(q, d)`          | 与 `reset(q)` 类似，但若提供了 `d`，则在释放 `q` 时调用 `d`，否则使用 `delete`。                  |


**教材示例代码**
```cpp
shared_ptr<int> p1; // 可以指向double的shared_ptr
shared_ptr<int> p2(new int(42)); // p2指向值为42的int

shared_ptr<int> p1 = new int(1024); // 错误：必须使用直接初始化
shared_ptr<int> p2(new int(1024));  // 正确：使用直接初始化

shared_ptr<int> clone(int p) {
    return new int(p); // 错误：隐式转换为shared_ptr<int>
}
shared_ptr<int> clone(int p) {
    // 正确：显式创建shared_ptr<int>
    return shared_ptr<int>(new int(p));
}
```

**注意点**
* ⚠️ **语法细节**：
    * 因为构造函数是`explicit`的，所以初始化或返回时**不能使用拷贝初始化（`=`），必须使用直接初始化（圆括号）**。


---

<a id="id21"></a>
## ✅ 知识点21: 不要混合使用普通指针和智能指针

**理论**
* **核心主旨总结**：
    * `shared_ptr`只能与其他作为自身副本的`shared_ptr`协调销毁。
    * 内置指针无法和`shared_ptr`一起参与counter，即使两者都共同指向同一内存
    * **使用`make_shared`可以避免无意中将同一块内存绑定到多个独立创建的`shared_ptr`上**。
* 当我们将一个`shared_ptr`绑定到一个普通指针时，我们就将管理该内存的责任交给了那个`shared_ptr`。
    * **一旦将责任交给`shared_ptr`，就不应该再使用内置指针来访问`shared_ptr`现在所指向的内存**。
    * 因为当我们使用内置指针来访问的时候，`shared_ptr`已经释放了访问的内存
    * 导致dangling pointer


**教材示例代码**
```cpp
void process(shared_ptr<int> ptr) {
    // 使用ptr
} // ptr离开作用域被销毁，但引用计数不可能为0，内存不会被删。

// 正确用法
shared_ptr<int> p(new int(42)); // 引用计数为1
process(p); // 拷贝p，其引用计数在process中至少为2
int i = *p; // 正确：process结束，引用计数变回1

// 危险用法
int *x(new int(1024)); // x是普通指针
// process(x); // 错误：不能将int*转换为shared_ptr<int>
process(shared_ptr<int>(x)); // 合法，但内存会被删除！
int j = *x; // 未定义：x是悬挂指针！
```
**代码解析**
* 如危险用法中的`process`所示：
    * 虽然不能将内置指针传递给期望`shared_ptr`的函数，但可以传递一个（临时的）由内置指针显式构造的`shared_ptr`。
    * 然而，这样做很可能导致错误，因为临时`shared_ptr`在`process`函数执行结束被销毁后。
    * 而当时的初始化的`shared_ptr`也不会因为还有一个内置指针而将counter提高为2。 
    * 因此`shared_ptr`指向的内存也会被销毁。
    * 原始内置指针会变成悬挂指针。

**注意点**
* ⚠️ **严重警告**：**使用内置指针访问智能指针拥有的对象是危险的**，因为我们可能不知道该对象何时被销毁。
* 💡 **核心原则**：确保一块动态内存从分配开始就只由一组`shared_ptr`管理，避免普通指针介入。

---

<a id="id22"></a>
## ✅ 知识点22: 不要用`get()`初始化或赋值另一个智能指针

**理论**
* **核心主旨总结**：
    * `get()`函数返回一个指向智能指针所管理对象的内置指针。
    * 此函数用于需要向**无法使用智能指针**的代码传递内置指针的情况。
* **使用`get()`返回的指针的代码不能删除该指针**。
* **严重错误**：
    * **将另一个智能指针绑定到`get()`返回的指针上**。
    * 这会导致两个独立的`shared_ptr`指向同一内存，各自拥有一个引用计数。
    * 当其中一个被销毁时，内存会被释放，使另一个成为悬挂指针，后续销毁时会导致双重释放。

**教材示例代码**
```cpp
shared_ptr<int> p(new int(42)); // 引用计数为1
int *q = p.get(); // 正确：但不要用q做任何可能导致其指针被删除的事
{ // 新作用域
    // 未定义：两个独立的shared_ptr指向同一内存
    shared_ptr<int>(q);
} // 块结束，q被销毁，q指向的内存被释放
int foo = *p; // 未定义：p指向的内存已被释放
```

**注意点**
* ⚠️ **警告**：
    * **`get()`仅用于向明确不会删除指针的代码传递访问权限。尤其不要用`get()`返回的指针去初始化或赋值给另一个智能指针**。

---

<a id="id23"></a>
## ✅ 知识点23: `shared_ptr`的其他操作（`reset`与`unique`）

**理论**
* **核心主旨总结**：
    * 可以使用`reset`将一个新指针分配给`shared_ptr`。
    * 和赋值操作一样，`reset`会更新引用计数，并在适当时删除`p`原来指向的对象，然后指向新分配的指针。
    * 实现**共享读，单独写**的目的，避免多个`shared_ptr`对同一个内存的无意修改。
* `reset`成员常与`unique`一起使用，以控制由多个`shared_ptr`共享的对象的更改。
    * 在更改底层对象前，检查我们是否是唯一用户。
    * 如果不是，则在更改前制作一个新副本。

**教材示例代码**
```cpp
p = new int(1024);       // 错误：不能将指针赋值给shared_ptr
p.reset(new int(1024));  // 正确：p指向一个新对象

if (!p.unique())
    p.reset(new string(*p)); // 我们不是唯一用户；分配一个新副本
*p += newVal; // 现在知道我们是唯一指针了，可以安全地更改对象
```

**注意点**
* 🔧 **`unique()`和`use_count()`**：
    * `unique()`在`use_count()`为1时返回`true`。
    * `use_count()`返回共享对象（引用计数）的数量，可能是一个慢速操作，主要用于调试。
* 💡 **写时复制(Copy-on-Write)**：
    * 使用`unique()`和`reset(new T(*p))`是一种实现写时复制逻辑的模式，在需要修改共享数据时创建副本。

---

<a id="id24"></a>
## ✅ 知识点24: 智能指针与异常

**理论**
* **核心主旨总结**：
    * 使用异常处理的程序需要确保在发生异常时资源被正确释放。
    * **使用智能指针是确保资源被释放的一种简单方法**。
* 当函数退出时（无论是正常处理还是由于异常），所有局部对象都会被销毁。
    * 如果局部对象是智能指针，其析构函数会检查引用计数并在计数为0时释放内存。
    * 这即使在函数提前退出时也能保证内存释放。
* 相反，如果我们直接管理内存并在`new`之后、对应`delete`之前发生异常，且异常未在函数内被捕获，那么该内存将永远无法被释放。

**教材示例代码**
```cpp
void f() {
    shared_ptr<int> sp(new int(42)); // 分配一个新对象
    // 此处抛出异常且未在f内捕获
} // sp是局部对象，会被销毁。作为shared_ptr，它会自动释放内存。

void f() {
    int *ip = new int(42); // 动态分配一个新对象
    // 此处抛出异常且未在f内捕获
    delete ip; // 此语句不会被执行，内存泄漏！
}
```

**注意点**
* 💡 **关键优势**：智能指针（包括后面提到的带自定义删除器的用法）是<b>异常安全(exception-safe)</b>编程的重要工具，可以防止资源泄漏。

---

<a id="id25"></a>
## ✅ 知识点25: 智能指针与“哑”类（自定义删除器）

**理论**
* **核心主旨总结**：
    * 许多C++类（包括所有库类）定义了负责清理资源的**析构函数**(destructor)。
    * 但并非所有类都如此良好，特别是那些设计为C和C++共用的类，通常要求用户显式释放所用资源。
* 对于没有定义良好析构函数来释放资源的类，可能会遇到与使用动态内存类似的错误。
    * 我们可以使用管理动态内存的类似技术（智能指针）来管理它们。
* 默认情况下，`shared_ptr`假定它们指向动态内存，因此销毁时会对其持有的指针执行`delete`。
    * **为了用`shared_ptr`管理其他资源（如网络连接），我们必须定义一个删除函数(deleter)**。
    * 创建`shared_ptr`时，可以将删除器作为额外参数传递。

**教材示例代码**
```cpp
// 假设的网络库接口
struct destination; // 表示连接目标
struct connection; // 使用连接所需信息
connection connect(destination*); // 打开连接
void disconnect(connection); // 关闭给定连接

// 自定义删除器
void end_connection(connection *p) { disconnect(*p); }

void f(destination &d /* 其他参数 */) {
    connection c = connect(&d);
    // 创建shared_ptr，并传入删除器
    shared_ptr<connection> p(&c, end_connection);
    // 使用连接
    // 当f退出时，即使是异常退出，连接也会被正确关闭
}
```
**代码解析**
* 如果`connection`有析构器，其会在`f`函数结束后自动关闭
* 然而`connection`并没有
* 这种情况我们就可以使用`shared_ptr`去保障`connection`的正确关闭
* 当`p`被销毁，其不会执行`delete`， 而是执行`end_connection`
* 这样就保证了`p`能被正确释放即使出现exception或是正常退出

**注意点**
* ⚠️ **智能指针使用注意事项（坑）**：
    1. 不要用同一个内置指针值来初始化（或`reset`）多个智能指针。
    2. 不要`delete`从`get()`返回的指针。
    3. 不要用`get()`返回的指针来初始化或`reset`另一个智能指针。
    4. 如果使用`get()`返回的指针，记住当最后一个对应的智能指针消失时，该指针将失效。
    5. 如果用智能指针管理非`new`分配的资源，记得传递一个删除器。

---

<a id="id26"></a>
## ✅ 知识点26: `unique_ptr`独占所有权

**理论**
* **核心主旨总结**：
    * 一个`unique_ptr`<b>“拥有”(owns)</b>它指向的对象。
    * 与`shared_ptr`不同，**一次只能有一个`unique_ptr`指向一个给定对象**。
    * 当`unique_ptr`被销毁时，它指向的对象也被销毁。
* 与`shared_ptr`不同，**没有类似`make_shared`的标准库函数返回`unique_ptr`**。
    * 定义`unique_ptr`时，需将其绑定到`new`返回的指针，且必须**直接初始化**。
* `unique_ptr`拥有它指向的对象，所以`unique_ptr`**不支持普通的拷贝或赋值**。
    * 但可以通过调用`release`或`reset`**从另一个（非const）`unique_ptr`转移所有权**。

**`uniqeu_ptr`操作表**


| 操作                      | 说明                                                                         |
| ----------------------- | -------------------------------------------------------------------------- |
| `unique_ptr<T> u1`      | 空的 `unique_ptr`，可以指向类型为 `T` 的对象。`u1` 使用 `delete` 释放其指针。                    |
| `unique_ptr<T, D> u2`   | 空的 `unique_ptr`，可以指向类型为 `T` 的对象。`u2` 使用类型为 `D` 的可调用对象来释放其指针（而不是 `delete`）。 |
| `unique_ptr<T, D> u(d)` | 空的 `unique_ptr`，指向类型为 `T` 的对象，使用 `d`（类型为 `D` 的对象）来代替 `delete` 释放资源。        |
| `u = nullptr`           | 删除 `u` 当前指向的对象，并将 `u` 置为空。                                                 |
| `u.release()`           | 放弃对所指对象的控制；返回 `u` 原来持有的内置指针，并将 `u` 置为空。                                    |
| `u.reset()`             | 删除 `u` 当前指向的对象。                                                            |
| `u.reset(q)`            | 若提供内置指针 `q`，则使 `u` 指向该对象；在此之前会删除原来指向的对象。                                   |
| `u.reset(nullptr)`      | 删除 `u` 当前指向的对象，并将 `u` 置为空。                                                 |


**教材示例代码**
```cpp
unique_ptr<double> p1; // 可以指向double的unique_ptr
unique_ptr<int> p2(new int(42)); // p2指向值为42的int

unique_ptr<string> p1(new string("Stegosaurus"));
unique_ptr<string> p2(p1); // 错误：unique_ptr不能拷贝
unique_ptr<string> p3;
p3 = p1; // 错误：unique_ptr不能赋值

// 通过release和reset转移所有权
unique_ptr<string> p2(p1.release()); // release使p1为空，所有权转给p2
unique_ptr<string> p3(new string("Trex"));
p2.reset(p3.release()); // reset释放p2原内存，接管p3的所有权，p3置空

p2.release(); // 错误：p2不会释放内存，而且我们丢失了指针
auto p = p2.release(); // 正确，但我们必须记得delete(p)
```

**注意点**

* ⚠️ **`release()`的陷阱**：      
    * `release()`只是放弃控制权并返回指针，**不会释放内存**。
    * 如果不用另一个智能指针接住返回的指针，程序就必须负责释放该资源。
* 🔄 **与`shared_ptr`对比**：`shared_ptr`通过引用计数共享所有权；`unique_ptr`独占所有权，必须显式转移。

---

<a id="id27"></a>
## ✅ 知识点27: 传递和返回 `unique_ptr`

**理论**
* **核心主旨总结**：
    * 有一个例外规则：**我们可以拷贝或赋值一个即将被销毁的`unique_ptr`**。
    * 最常见的例子是从函数返回一个`unique_ptr`。
    * 编译器知道返回的对象即将被销毁，因此会执行一种特殊的“拷贝”（移动操作）§ 13.6.2。

**教材示例代码**
```cpp
unique_ptr<int> clone(int p) {
    return unique_ptr<int>(new int(p)); // 从内置指针创建并返回
}
// 或者返回局部对象的拷贝
unique_ptr<int> clone(int p) {
    unique_ptr<int> ret(new int(p));
    return ret;
}
```

**注意点**
* 🔄 **向后兼容**：
    * 早期库包含一个名为`auto_ptr`的类，具有`unique_ptr`的部分特性（不能放入容器或从函数返回）。
    * **虽然`auto_ptr`仍是标准库的一部分，但程序应使用`unique_ptr`**。

---

<a id="id28"></a>
## ✅ 知识点28: 向 `unique_ptr` 传递删除器

**理论**
* **核心主旨总结**：
    * 默认`unique_ptr`用`delete`释放对象。可以重写默认的删除器。
* **重写`unique_ptr`的删除器会影响`unique_ptr`的类型**: 
    * 与`shared_ptr`不同，必须在尖括号中提供删除器类型和指针所能指向的类型。
    * 在创建或`reset`这种类型的对象时，需提供指定类型的可调用对象（删除器）。
* 使用`decltype`来指定函数指针类型。因为`decltype(end_connection)`返回函数类型，必须添加`*`表示使用该类型的指针。

**教材示例代码**
```cpp
void f(destination &d /* other needed parameters */) {
    connection c = connect(&d);
    // 当p被销毁时，连接将被关闭
    unique_ptr<connection, decltype(end_connection)*>
        p(&c, end_connection);
    // 使用连接
    // 当f退出，即使因异常退出，连接也会被正确关闭
}
```

**注意点**
* 🔧 **与`shared_ptr`的区别**：
    * `unique_ptr`管理删除器的方式与`shared_ptr`不同（§16.1.6）。
    * 对于`unique_ptr`，删除器类型是模板参数的一部分，这可能在编译期带来效率优势。

---

<a id="id29"></a>
## ✅ 知识点29: `weak_ptr` 弱引用

**理论**
* **核心主旨总结**：
    * **`weak_ptr`是一种不控制所指向对象生存期的智能指针**。
    * 它指向一个由`shared_ptr`管理的对象。将一个`weak_ptr`绑定到一个`shared_ptr`**不会改变`shared_ptr`的引用计数**。
    * 一旦最后一个指向对象的`shared_ptr`被销毁，对象就会被释放，即使仍有`weak_ptr`指向它——因此得名“弱”共享。
* **不能直接使用`weak_ptr`访问其对象**，必须调用`lock()`。
    * `lock()`检查`weak_ptr`指向的对象是否仍存在。
    * 若存在，则返回一个指向该共享对象的`shared_ptr`，从而保证底层对象在该`shared_ptr`存在期间至少存在。
根据你提供的图片内容，我为你整理了一份清晰的 `weak_ptr` 操作汇总表格。

* **表格 12.5. `weak_ptr`操作汇总**

| 操作 / 函数 | 功能描述 |
| --- | --- |
| **`weak_ptr<T> w`** | 创建一个空的 `weak_ptr`，它可以指向类型为 `T` 的对象。 |
| **`weak_ptr<T> w(sp)`** | 创建一个与 `shared_ptr sp` 指向相同对象的 `weak_ptr`。类型 `T` 必须能转换为 `sp` 指向的类型。 |
| **`w = p`** | 赋值操作。`p` 可以是一个 `shared_ptr` 或 `weak_ptr`。赋值后 `w` 与 `p` 共享对象的所有权。 |
| **`w.reset()`** | 将 `w` 置为空（null）。 |
| **`w.use_count()`** | 返回与 `w` 共享对象的 `shared_ptr` 的数量。 |
| **`w.expired()`** | 若 `w.use_count()` 为 0，则返回 `true`；否则返回 `false`。 |
| **`w.lock()`** | 如果 `expired()` 为 `true`，返回一个空的 `shared_ptr`；否则返回一个指向 `w` 所指对象的 `shared_ptr`。 |

**教材示例代码**
```cpp
auto p = make_shared<int>(42);
weak_ptr<int> wp(p); // wp弱共享p；p的引用计数不变

if (shared_ptr<int> np = wp.lock()) { // 如果np不为null则为true
    // 在if内部，np与p共享其对象
}
```

**注意点**

* 💡 **主要用途**：`weak_ptr`用于打破`shared_ptr`的循环引用，或用于缓存、观察者模式等场景，它允许访问对象但不会延长其生命周期。

---

<a id="id30"></a>
## ✅ 知识点30: 使用`weak_ptr`的示例 - `StrBlobPtr`

**理论**
* **核心主旨总结**：
    * 作为`weak_ptr`用途的示例，教材定义了一个`StrBlob`的伴随指针类`StrBlobPtr`。
    * 它存储一个指向`StrBlob`数据成员的`weak_ptr`。
    * 通过使用`weak_ptr`，它不影响所指向`vector`的生存期，但可以防止用户尝试访问一个不存在的`vector`。
* `StrBlobPtr`有两个数据成员：
    * `wptr`（指向`StrBlob`中`vector`的`weak_ptr`）
    * 和`curr`（当前指示元素的索引）。它有一个`check`成员来验证解引用是否安全，该`check`使用`lock()`来获取`shared_ptr`，如果底层`vector`已被释放则抛出异常。
* `StrBlobPtr`类提供了`deref()`和`incr()`成员来模拟指针的解引用和递增操作。
    * `StrBlob`类需要将`StrBlobPtr`声明为友元，并添加`begin()`和`end()`成员返回指向自身的`StrBlobPtr`。

**教材示例代码**
```cpp
class StrBlobPtr {
public:
    StrBlobPtr(): curr(0) { }
    StrBlobPtr(StrBlob &a, size_t sz = 0): wptr(a.data), curr(sz) { }
    std::string& deref() const;
    StrBlobPtr& incr();
private:
    std::shared_ptr<std::vector<std::string>>
        check(std::size_t, const std::string&) const;
    std::weak_ptr<std::vector<std::string>> wptr;
    std::size_t curr;
};

std::shared_ptr<std::vector<std::string>>
StrBlobPtr::check(std::size_t i, const std::string &msg) const {
    auto ret = wptr.lock(); // vector还在吗？
    if (!ret)
        throw std::runtime_error("unbound StrBlobPtr");
    if (i >= ret->size())
        throw std::out_of_range(msg);
    return ret; // 否则，返回指向vector的shared_ptr
}

std::string& StrBlobPtr::deref() const {
    auto p = check(curr, "dereference past end");
    return (*p)[curr];
}

StrBlobPtr& StrBlobPtr::incr() {
    check(curr, "increment past end of StrBlobPtr");
    ++curr;
    return *this;
}

// StrBlob类中的新增成员
class StrBlob {
    friend class StrBlobPtr;
    // ...
    StrBlobPtr begin() { return StrBlobPtr(*this); }
    StrBlobPtr end() { auto ret = StrBlobPtr(*this, data->size()); return ret; }
};
```

**注意点**
* 🔧 **设计模式**：
    * `StrBlobPtr`是一个“智能”迭代器或指针类，它使用`weak_ptr`来安全地引用可能已被销毁的共享数据。
* ⚠️ **构造函数限制**：`StrBlobPtr`的构造函数接受非const的`StrBlob&`引用，因此**不能将`StrBlobPtr`绑定到const `StrBlob`对象**。
* 💡 **`weak_ptr`的典型应用**：`StrBlobPtr`展示了`weak_ptr`在创建不拥有资源、但需要安全地检查资源是否存在的“观察者”类时的典型用法。

---

## 🔑 核心要点总结
1. **原始指针负责间接访问**：指针自身与所指对象是不同实体；保存地址不自动承担生命周期管理，不能把每个原始指针都交给 `delete`。
2. **按用途区分指针**：对象指针用于访问对象，函数指针用于调用函数，非静态成员指针需要结合对象使用；`this` 表示当前对象的地址。
3. **明确所有权**：`shared_ptr` 共享所有权，`unique_ptr` 独占所有权，`weak_ptr` 只观察而不延长对象生命周期；选择取决于实际管理需求。
4. **安全创建与释放**：优先用 `make_shared` 创建共享对象，利用 RAII 自动释放资源；不要用 `get()` 得到的地址再创建独立的所有权管理者，也不要手动释放智能指针正在管理的对象。
5. **理解引用计数与删除器**：最后一个共享所有者释放对象；自定义删除器可将自动清理扩展到文件、连接等资源。

## 📌 考试速记版
* **野指针／悬空指针／空指针**：未建立可靠指向／原目标已失效／明确没有目标；非空不等于有效，置空只影响当前指针。
* **原始指针大小**：`sizeof(p)` 看指针类型，`sizeof(*p)` 看目标类型；具体字节数依赖实现。
* **const 区别**：`const T*` 不能通过指针修改对象；`T* const` 不能改变指针指向。
* **成员指针**：`T C::*` 指定数据成员，`R (C::*)(Args...)` 指定普通非静态成员函数；通过 `.*` / `->*` 结合对象使用。
* **`this`**：普通非 const、非 volatile 成员函数中为 `C*`，对应 const 成员函数中为 `const C*`；静态成员函数没有 `this`。
*   **`new`/`delete`**：手动管理，易错（泄漏、双重释放、悬挂指针）。
*   **`shared_ptr<T>`**：`make_shared<T>(args)`创建。拷贝/赋值增加引用计数。计数为0时自动`delete`。
*   **`unique_ptr<T>`**：`unique_ptr<T> p(new T(args))`创建。**不可拷贝，可移动**（`release()`, `reset()`）。独占所有权。
*   **`weak_ptr<T>`**：从`shared_ptr`创建。用`lock()`获取`shared_ptr`来安全访问。不增加引用计数。
*   **关键禁令**：
    *   禁止混用普通指针和智能指针管理同一内存。
    *   禁止用`get()`返回的指针创建另一个智能指针。
    *   `unique_ptr`不能拷贝赋值。
*   **自定义删除器**：管理非`new`资源或需要特殊清理时，在构造函数中传递可调用对象。
*   **异常安全**：智能指针保证在异常发生时资源能被正确释放。

**记忆口诀**：
原始指针存位置，对象生死分开记。
成员指针配对象，访问管理辨仔细。
野指针先初始化，悬空莫访旧对象。
`new/delete` 手动难，泄漏悬挂坑不断。
智能指针来帮忙，`shared`计数`unique`独享。
`make_shared` 是首选，`get()`乱用很危险。
`weak_ptr` 弱观察，`lock()`检查保平安。
自定义删除扩用途，RAII思想记心间。