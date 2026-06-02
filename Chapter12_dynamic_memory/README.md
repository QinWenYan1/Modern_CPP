# 📘 Chapter 12 - 动态内存 (Dynamic Memory)

> 《C++ Primer》第 12 章学习笔记导航  
> 本章涵盖：智能指针、new/delete、动态数组与内存管理

---

## 📑 章节导航

### 12.1 动态内存与智能指针 🧠
shared_ptr、unique_ptr、weak_ptr、make_shared  
[→ 查看笔记](./note_12_1_dynamic_memory_and_smart_pointers.md)

### 12.2 直接管理内存 ⚠️
new/delete、new[]/delete[]、空悬指针  
[→ 查看笔记](./note_12_2_dynamic_arrays.md)

### 12.3 使用标准库：文本查询程序 📖
综合示例：使用智能指针实现文本查询  
[→ 查看笔记](./note_12_3_using_the_library_a_text-query_program.md)

### 章节总结 📝
第 12 章核心概念速查表  
[→ 查看总结](./note_12_chapter_summary.md)

### 练习题 💪
本章所有练习代码  
[→ 查看代码](./exercises/)

---

## 🎯 知识点索引

<details>
<summary><b>🧠 动态内存与智能指针</b></summary>

- [shared_ptr 共享所有权](./note_12_1_dynamic_memory_and_smart_pointers.md)
- [make_shared 安全创建](./note_12_1_dynamic_memory_and_smart_pointers.md)
- [unique_ptr 独占所有权](./note_12_1_dynamic_memory_and_smart_pointers.md)
- [weak_ptr 弱引用](./note_12_1_dynamic_memory_and_smart_pointers.md)
- [引用计数机制](./note_12_1_dynamic_memory_and_smart_pointers.md)
- [循环引用问题](./note_12_1_dynamic_memory_and_smart_pointers.md)

</details>

<details>
<summary><b>⚠️ 直接管理内存</b></summary>

- [new 表达式](./note_12_2_dynamic_arrays.md)
- [delete 表达式](./note_12_2_dynamic_arrays.md)
- [动态数组 new[]/delete[]](./note_12_2_dynamic_arrays.md)
- [空悬指针](./note_12_2_dynamic_arrays.md)
- [内存泄漏](./note_12_2_dynamic_arrays.md)
- [allocator 类](./note_12_2_dynamic_arrays.md)

</details>

<details>
<summary><b>📖 文本查询程序示例</b></summary>

- [程序设计](./note_12_3_using_the_library_a_text_query_program.md)
- [数据结构](./note_12_3_using_the_library_a_text_query_program.md)
- [使用 shared_ptr 共享数据](./note_12_3_using_the_library_a_text_query_program.md)

</details>

---

## 💡 核心速查

**shared_ptr（共享所有权）**
```cpp
#include <memory>

// 创建
shared_ptr<int> p1 = make_shared<int>(42);
auto p2 = make_shared<vector<int>>(10, 0);  // 10 个 0

// 拷贝（引用计数 +1）
shared_ptr<int> p3 = p1;

// 引用计数
size_t n = p1.use_count();  // 共享对象的指针数量
bool unique = p1.unique();  // use_count() == 1 ?

// 重置
p1.reset();          // 减 1，若变为 0 则释放
p1.reset(new int(10));  // 指向新对象

// 自定义删除器
shared_ptr<FILE> file(fopen("data.txt", "r"), fclose);
```

**unique_ptr（独占所有权）**
```cpp
// 创建
unique_ptr<int> p1(new int(42));
// unique_ptr 不支持拷贝
// unique_ptr<int> p2 = p1;  // 错误！

// 移动所有权
unique_ptr<int> p2 = std::move(p1);  // p1 变为空

// 释放所有权
int *raw = p2.release();  // p2 放弃所有权，返回裸指针
delete raw;  // 必须手动释放

// reset
p2.reset();  // 释放当前对象
```

**weak_ptr（弱引用）**
```cpp
// 创建
shared_ptr<int> sp = make_shared<int>(42);
weak_ptr<int> wp = sp;

// 检查是否有效
if (auto sp2 = wp.lock()) {  // 若对象存在返回 shared_ptr
    // 使用 sp2
}

// expired 检查
if (!wp.expired()) { /* 对象还存在 */ }
```

**动态数组（不推荐，优先用容器）**
```cpp
// 分配
int *arr = new int[100];  // 100 个未初始化 int
int *arr2 = new int[100]();  // 值初始化为 0

// 释放（必须用 delete[]）
delete[] arr;
delete[] arr2;

// 智能指针管理数组
unique_ptr<int[]> up(new int[10]);
up[0] = 1;  // 支持下标访问
// up.release();  // 自动调用 delete[]
```

**allocator（原始内存分配）**
```cpp
allocator<string> alloc;
auto const p = alloc.allocate(100);  // 分配 100 个 string 空间

// 构造
auto q = p;
alloc.construct(q++, "hello");
alloc.construct(q++, 10, 'c');  // "cccccccccc"

// 销毁
while (q != p) {
    alloc.destroy(--q);
}

// 释放
alloc.deallocate(p, 100);
```

---

## ⚠️ 最佳实践

1. **优先使用智能指针**，避免直接使用 new/delete
2. **使用 make_shared** 而不是 shared_ptr 构造函数
3. **unique_ptr** 用于明确的所有权转移场景
4. **weak_ptr** 用于打破循环引用
5. **优先使用标准容器**（vector、string）而不是动态数组

---

## 🔗 相关链接

← [上一章：第 11 章 - 关联容器](../Chapter11_associative_containers/README.md)  
→ [下一章：第 13 章 - 拷贝控制](../Chapter13_copy_control/README.md)  
↑ [返回根目录](../README.md)

---

> 🕊️ *"现代 C++ 中，裸指针几乎总是意味着资源管理问题。智能指针让内存管理自动化，是写出安全代码的关键。"*
