# 📘 Chapter 06 - 函数 (Functions)

> 《C++ Primer》第 6 章学习笔记导航  
> 本章涵盖：函数定义、参数传递、返回类型、重载与函数指针

---

## 📑 章节导航

### 6.1 函数基础 🎯
函数定义、调用、形参与实参、局部对象、`extern "C"`

[→ 查看笔记](./note_6_1_function_basics.md)

### 6.2 参数传递 📦
值传递、引用传递、const 形参、数组形参  
[→ 查看笔记](./note_6_2_argument_passing.md)

### 6.3 返回类型与 return 语句 ↩️
返回值、返回引用、列表初始化返回值、递归  
[→ 查看笔记](./note_6_3_return_type_statement.md)

### 6.4 重载函数 🔀
函数重载、重载与作用域、重载解析  
[→ 查看笔记](./note_6_4_overloaded_function.md)

### 6.5 特殊用途语言特性 ⭐
默认实参、内联函数、constexpr 函数  
[→ 查看笔记](./note_6_5_features_for_specialized_uses.md)

### 6.6 函数匹配 🎯
候选函数、可行函数、最佳匹配  
[→ 查看笔记](./note_6_6_function_matching.md)

### 6.7 函数指针 🔗
函数指针定义、使用、回调函数  
[→ 查看笔记](./note_6_7_pointers_to_functions.md)

### 章节总结 📝
第 6 章核心概念速查表  
[→ 查看总结](./note_6_chapter_summary.md)

### 练习题 💪
本章所有练习代码  
[→ 查看代码](./exercises/)

---

## 🎯 知识点索引

<details>
<summary><b>🎯 函数基础</b></summary>

- [函数定义](./note_6_1_function_basics.md)
- [函数调用](./note_6_1_function_basics.md)
- [形参与实参](./note_6_1_function_basics.md)
- [局部对象与静态局部变量](./note_6_1_function_basics.md)
- [函数声明](./note_6_1_function_basics.md)
- [extern "C" 的作用](./note_6_1_function_basics.md#extern-c)

</details>

<details>
<summary><b>📦 参数传递</b></summary>

- [值传递](./note_6_2_argument_passing.md)
- [引用传递](./note_6_2_argument_passing.md)
- [const 形参](./note_6_2_argument_passing.md)
- [数组形参](./note_6_2_argument_passing.md)
- [可变参数](./note_6_2_argument_passing.md)

</details>

<details>
<summary><b>↩️ 返回类型与 return</b></summary>

- [返回值](./note_6_3_return_type_statement.md)
- [返回引用](./note_6_3_return_type_statement.md)
- [列表初始化返回](./note_6_3_return_type_statement.md)
- [递归函数](./note_6_3_return_type_statement.md)
- [返回数组指针](./note_6_3_return_type_statement.md)

</details>

<details>
<summary><b>🔀 重载函数</b></summary>

- [函数重载定义](./note_6_4_overloaded_function.md)
- [重载与作用域](./note_6_4_overloaded_function.md)
- [const 形参重载](./note_6_4_overloaded_function.md)
- [重载解析步骤](./note_6_4_overloaded_function.md)

</details>

<details>
<summary><b>⭐ 特殊用途语言特性</b></summary>

- [默认实参](./note_6_5_features_for_specialized_uses.md)
- [内联函数](./note_6_5_features_for_specialized_uses.md)
- [constexpr 函数](./note_6_5_features_for_specialized_uses.md)
- [调试帮助 assert](./note_6_5_features_for_specialized_uses.md)

</details>

<details>
<summary><b>🎯 函数匹配</b></summary>

- [候选函数](./note_6_6_function_matching.md)
- [可行函数](./note_6_6_function_matching.md)
- [最佳匹配](./note_6_6_function_matching.md)
- [类型转换等级](./note_6_6_function_matching.md)

</details>

<details>
<summary><b>🔗 函数指针</b></summary>

- [函数指针定义](./note_6_7_pointers_to_functions.md)
- [使用函数指针](./note_6_7_pointers_to_functions.md)
- [函数指针作为形参](./note_6_7_pointers_to_functions.md)
- [返回函数指针](./note_6_7_pointers_to_functions.md)

</details>

---

## 💡 核心速查

**函数定义**
```cpp
// 基本函数
return_type function_name(parameter_list) {
    // function body
    return value;
}

// 示例
int add(int a, int b) {
    return a + b;
}
```

**参数传递**
```cpp
// 值传递（拷贝）
void f(int x);

// 引用传递（避免拷贝）
void f(int &x);

// const 引用（只读访问）
void f(const int &x);

// 指针传递
void f(int *p);
```

**函数重载**
```cpp
// 同名不同参数列表
void print(int x);
void print(double x);
void print(const string &s);
```

**默认实参**
```cpp
// 从右向左提供默认值
void f(int a, int b = 0, int c = 10);
f(1);        // f(1, 0, 10)
f(1, 2);     // f(1, 2, 10)
f(1, 2, 3);  // f(1, 2, 3)
```

**constexpr 函数**
```cpp
constexpr int square(int x) {
    return x * x;
}
constexpr int val = square(5);  // 编译期计算
```

**函数指针**
```cpp
// 定义函数指针
bool (*pf)(const string &, const string &);

// 赋值
pf = lengthCompare;

// 调用
bool b = pf("hello", "world");
```

---

## 🔗 相关链接

← [上一章：第 5 章 - 语句](../Chapter05_statement/README.md)  
→ [下一章：第 7 章 - 类](../Chapter07_classes/README.md)  
↑ [返回根目录](../README.md)

---

> 🕊️ *"函数是 C++ 程序的基本构建块，掌握函数机制是模块化编程的核心。"*
