# 📘 Chapter 13 - 拷贝控制 (Copy Control)

> 《C++ Primer》第 13 章学习笔记导航  
> 本章涵盖：拷贝构造函数、拷贝赋值运算符、移动语义与三/五法则

---

## 📑 章节导航

### 13.1 拷贝、赋值与销毁 📋
拷贝构造函数、拷贝赋值运算符、析构函数  
[→ 查看笔记](./note_13_1_copy_assign_destroy.md)

### 13.2 拷贝控制和资源管理 🔄
行为像值的类、行为像指针的类  
[→ 查看笔记](./note_13_2_copy_control_and_resource_management.md)

### 13.3 交换操作 🔄
swap 函数、拷贝并交换技术  
[→ 查看笔记](./note_13_3_swap.md)

### 13.4 拷贝控制示例 📝
Message 和 Folder 类的实现  
[→ 查看笔记](./note_13_4_a_copy_control_example.md)

### 13.5 动态内存管理类 🧠
StrVec 类的实现（类似 vector）  
[→ 查看笔记](./note_13_5_classes_that_manage_dynamic_memory.md)

### 13.6 对象移动 ➡️
右值引用、移动构造函数、移动赋值运算符  
[→ 查看笔记](./note_13_6_moving_objects.md)

### 章节总结 📝
第 13 章核心概念速查表  
[→ 查看总结](./note_13_summary.md)

### 练习题 💪
本章所有练习代码  
[→ 查看代码](./exercises/)

---

## 🎯 知识点索引

<details>
<summary><b>📋 拷贝、赋值与销毁</b></summary>

- [拷贝构造函数](./note_13_1_copy_assign_and_destroy.md)
- [拷贝赋值运算符 =](./note_13_1_copy_assign_and_destroy.md)
- [析构函数 ~Class()](./note_13_1_copy_assign_and_destroy.md)
- [三五法则](./note_13_1_copy_assign_and_destroy.md)
- [使用 =default](./note_13_1_copy_assign_and_destroy.md)
- [使用 =delete 阻止拷贝](./note_13_1_copy_assign_and_destroy.md)

</details>

<details>
<summary><b>🔄 拷贝控制和资源管理</b></summary>

- [行为像值的类](./note_13_2_copy_control_and_resource_management.md)
- [行为像指针的类](./note_13_2_copy_control_and_resource_management.md)
- [引用计数](./note_13_2_copy_control_and_resource_management.md)
- [写时复制](./note_13_2_copy_control_and_resource_management.md)

</details>

<details>
<summary><b>🔄 交换操作</b></summary>

- [swap 函数](./note_13_3_swap.md)
- [拷贝并交换技术 copy-and-swap](./note_13_3_swap.md)
- [异常安全](./note_13_3_swap.md)

</details>

<details>
<summary><b>📝 拷贝控制示例</b></summary>

- [Message 类设计](./note_13_4_a_copy_control_example.md)
- [Folder 类设计](./note_13_4_a_copy_control_example.md)
- [双向关联管理](./note_13_4_a_copy_control_example.md)

</details>

<details>
<summary><b>🧠 动态内存管理类</b></summary>

- [StrVec 类实现](./note_13_5_classes_that_manage_dynamic_memory.md)
- [分配器 allocator](./note_13_5_classes_that_manage_dynamic_memory.md)
- [移动迭代器](./note_13_5_classes_that_manage_dynamic_memory.md)

</details>

<details>
<summary><b>➡️ 对象移动</b></summary>

- [右值引用 &&](./note_13_6_moving_objects.md)
- [move 函数](./note_13_6_moving_objects.md)
- [移动构造函数](./note_13_6_moving_objects.md)
- [移动赋值运算符](./note_13_6_moving_objects.md)
- [移动后源对象状态](./note_13_6_moving_objects.md)
- [移动迭代器](./note_13_6_moving_objects.md)
- [引用限定成员函数](./note_13_6_moving_objects.md)

</details>

---

## 💡 核心速查

**三五法则**
```cpp
// 需要自定义以下之一，通常需要全部定义：
// 1. 析构函数
// 2. 拷贝构造函数
// 3. 拷贝赋值运算符
// 4. 移动构造函数 (C++11)
// 5. 移动赋值运算符 (C++11)

class HasPtr {
public:
    // 构造函数
    HasPtr(const string &s = string()) : 
        ps(new string(s)), i(0) {}
    
    // 拷贝构造函数
    HasPtr(const HasPtr &p) : 
        ps(new string(*p.ps)), i(p.i) {}
    
    // 拷贝赋值运算符（拷贝并交换）
    HasPtr& operator=(HasPtr rhs) {
        swap(*this, rhs);
        return *this;
    }
    
    // 析构函数
    ~HasPtr() { delete ps; }
    
    // 移动构造函数
    HasPtr(HasPtr &&p) noexcept : 
        ps(p.ps), i(p.i) { p.ps = nullptr; }
    
    // 移动赋值运算符
    HasPtr& operator=(HasPtr &&p) noexcept {
        if (this != &p) {
            delete ps;
            ps = p.ps;
            i = p.i;
            p.ps = nullptr;
        }
        return *this;
    }
};
```

**右值引用与移动**
```cpp
// 右值引用绑定到临时对象
int &&rr1 = 42;  // OK
// int &r1 = 42;   // 错误！

// std::move 将左值转为右值
int lval = 42;
int &&rr2 = std::move(lval);  // 可以绑定，但之后不要再用 lval

// 移动构造函数示例
class StrVec {
public:
    StrVec(StrVec &&s) noexcept  // 不抛异常
        : elements(s.elements), first_free(s.first_free), cap(s.cap) {
        s.elements = s.first_free = s.cap = nullptr;  // 置空源对象
    }
};
```

**拷贝并交换技术**
```cpp
// 形参按值传递（触发拷贝构造）
HasPtr& HasPtr::operator=(HasPtr rhs) {
    swap(*this, rhs);  // 与 *this 交换
    return *this;      // rhs 销毁，释放原 *this 内存
}

// 同时处理拷贝和赋值
HasPtr hp1("hi"), hp2("hello");
hp1 = hp2;              // 拷贝赋值
hp1 = std::move(hp2);   // 移动赋值
```

**阻止拷贝**
```cpp
class NoCopy {
public:
    NoCopy() = default;
    NoCopy(const NoCopy&) = delete;        // 阻止拷贝
    NoCopy& operator=(const NoCopy&) = delete;  // 阻止赋值
};
```

**引用限定成员函数**
```cpp
class Widget {
public:
    // 只能被左值调用
    void process() & { /* ... */ }
    
    // 只能被右值调用
    void process() && { /* ... */ }
};

Widget w;
w.process();           // 调用 & 版本
Widget().process();      // 调用 && 版本
```

---

## 🔗 相关链接

← [上一章：第 12 章 - 动态内存](../Chapter12_dynamic_memory/README.md)  
→ [下一章：第 14 章 - 重载运算与类型转换](../Chapter14_overloaded_operations_and_conversions/README.md)  
↑ [返回根目录](../README.md)

---

> 🕊️ *"拷贝控制是 C++ 资源管理的核心，理解三五法则与移动语义是写出高效现代 C++ 代码的关键。"*
