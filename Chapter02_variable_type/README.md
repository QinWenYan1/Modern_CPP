# 📘 Chapter 02 - 变量和基本类型 (Variable and Basic Types)

> 《C++ Primer》第 2 章学习笔记导航  
> 本章涵盖：C++ 原始类型系统、变量定义、复合类型与类型处理

---

## 📑 章节导航

### 2.1 原始内置类型 🧱
算术类型、字面值、类型转换规则  
[→ 查看笔记](./note_2_1_primitive_type.md)

### 2.2 变量 📦
变量定义、初始化、作用域与生命周期  
[→ 查看笔记](./note_2_2_variable.md)

### 2.3 复合类型 🔗
引用、指针、复合声明与指针/引用对比

[→ 查看笔记](./note_2_3_compound_types.md)

### 2.4 类型限定符 🛡️
`const`、`constexpr`、`volatile` 等修饰符详解  
[→ 查看笔记](./note_2_4_qualifier.md)

### 2.5 类型处理 🔄
类型别名、auto 类型推导、decltype 与类型转换  
[→ 查看笔记](./note_2_5_dealing_with_type.md)

### 2.6 自定义数据结构 🏗️
结构体、类基础与数据抽象入门  
[→ 查看笔记](./note_2_6_defining_data_structure.md)

### 章节总结 📝
第 2 章核心概念速查表  
[→ 查看总结](./note_2_chapter_summary.md)

### 练习题 💪
本章所有练习代码  
[→ 查看代码](./exercises/)

---

## 🎯 知识点索引

<details>
<summary><b>🧱 原始内置类型</b></summary>

- [算术类型分类](./note_2_1_primitive_type.md#-原始类型类别)
- [整型类型全览](./note_2_1_primitive_type.md#-类型全览)
- [浮点型特性](./note_2_1_primitive_type.md#浮点)
- [机器级表示](./note_2_1_primitive_type.md#-机器级表示-machine-level-representation)
- [符号型 vs 无符号型](./note_2_1_primitive_type.md#-符号型-vs-无符号型)
- [类型转换规则](./note_2_1_primitive_type.md#-类型转换-type-conversions)
- [无符号类型陷阱](./note_2_1_primitive_type.md#-减法运算的陷阱)
- [字面值类型](./note_2_1_primitive_type.md#-字面值-literals)
- [转义序列](./note_2_1_primitive_type.md#转义序列-escape-sequences)

</details>

<details>
<summary><b>📦 变量</b></summary>

- [变量定义与声明](./note_2_2_variable.md)
- [列表初始化](./note_2_2_variable.md)
- [默认初始化](./note_2_2_variable.md)
- [变量作用域](./note_2_2_variable.md)
- [生命周期管理](./note_2_2_variable.md)

</details>

<details>
<summary><b>🔗 复合类型</b></summary>

- [引用类型](./note_2_3_compound_types.md)
- [指针基础](./note_2_3_compound_types.md)
- [const 限定符](./note_2_3_compound_types.md)
- [顶层 const 与底层 const](./note_2_3_compound_types.md)
- [指针与引用的区别](./note_2_3_compound_types.md#id1)

</details>

<details>
<summary><b>🛡️ 类型限定符</b></summary>

- [const 修饰符](./note_2_4_qualifier.md)
- [constexpr 常量表达式](./note_2_4_qualifier.md)
- [volatile 易变修饰符](./note_2_4_qualifier.md)
- [限定符组合规则](./note_2_4_qualifier.md)

</details>

<details>
<summary><b>🔄 类型处理</b></summary>

- [类型别名 typedef/using](./note_2_5_dealing_with_type.md)
- [auto 类型推导](./note_2_5_dealing_with_type.md)
- [decltype 类型获取](./note_2_5_dealing_with_type.md)
- [显式类型转换](./note_2_5_dealing_with_type.md)

</details>

<details>
<summary><b>🏗️ 自定义数据结构</b></summary>

- [struct 结构体定义](./note_2_6_defining_data_structure.md)
- [类数据成员](./note_2_6_defining_data_structure.md)
- [成员函数基础](./note_2_6_defining_data_structure.md)
- [头文件保护](./note_2_6_defining_data_structure.md)

</details>

---

## 💡 核心速查

**基本类型**
```cpp
bool b = true;           // 布尔
char c = 'a';            // 字符
int i = 42;              // 整型
long l = 42L;            // 长整型
float f = 3.14f;         // 单精度浮点
double d = 3.14;         // 双精度浮点
```

**类型选择原则**
```cpp
// 整数运算用 int
// 浮点计算用 double
// 确定非负用 unsigned
// 避免用 char/bool 参与算术运算
```

**复合类型**
```cpp
int i = 42;
int &r = i;              // 引用，r 是 i 的别名
int *p = &i;             // 指针，p 存储 i 的地址
const int ci = 42;       // const 限定，不可修改
```

**类型处理**
```cpp
using IntPtr = int*;     // 类型别名 C++11
auto x = 42;             // 自动类型推导
decltype(x) y = x;       // 获取表达式类型
```

---

## 🔗 相关链接

← [上一章：第 1 章 - 开始](../Chapter01_get_started/README.md)  
→ [下一章：第 3 章 - 字符串、向量和数组](../Chapter03_String_Vector_Array/README.md)  
↑ [返回根目录](../README.md)

---

> 🕊️ *"理解类型是掌握 C++ 的第一步，一切高级特性都建立在类型系统之上。"*
