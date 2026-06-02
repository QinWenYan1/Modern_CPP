# 📘 Chapter 07 - 类 (Classes)

> 《C++ Primer》第 7 章学习笔记导航  
> 本章涵盖：类的定义、访问控制、构造函数、友元与成员函数

---

## 📑 章节导航

### 7.1 定义抽象数据类型 🏗️
类定义、成员函数、this 指针、const 成员函数  
[→ 查看笔记](./note_7_1_defining_abstract_data_type.md)

### 7.2 访问控制与封装 🔒
访问说明符、友元、封装原则  
[→ 查看笔记](./note_7_2_access_control_and_encapsulation.md)

### 7.3 类的其他特性 ⭐
类成员、返回 this 的成员函数、类类型与友元  
[→ 查看笔记](./note_7_3_additional_class_features.md)

### 7.4 类的作用域 🎯
类作用域、名字查找、类型名与普通成员  
[→ 查看笔记](./note_7_4_class_scope.md)

### 7.5 构造函数再探 🔄
构造函数初始化列表、委托构造函数、默认构造函数  
[→ 查看笔记](./note_7_5_constructor_revisited.md)

### 7.6 类的静态成员 🌐
静态成员定义、使用、静态成员函数  
[→ 查看笔记](./note_7_6_static_class_member.md)

### 章节总结 📝
第 7 章核心概念速查表  
[→ 查看总结](./note_7_chapter_summary.md)

### 练习题 💪
本章所有练习代码  
[→ 查看代码](./exercises/)

---

## 🎯 知识点索引

<details>
<summary><b>🏗️ 定义抽象数据类型</b></summary>

- [类定义](./note_7_1_defining_abstract_data_types.md)
- [成员函数](./note_7_1_defining_abstract_data_types.md)
- [this 指针](./note_7_1_defining_abstract_data_types.md)
- [const 成员函数](./note_7_1_defining_abstract_data_types.md)
- [类作用域与成员函数](./note_7_1_defining_abstract_data_types.md)

</details>

<details>
<summary><b>🔒 访问控制与封装</b></summary>

- [访问说明符 public/private](./note_7_2_access_control_and_encapsulation.md)
- [封装原则](./note_7_2_access_control_and_encapsulation.md)
- [友元函数](./note_7_2_access_control_and_encapsulation.md)
- [友元类](./note_7_2_access_control_and_encapsulation.md)
- [类接口设计](./note_7_2_access_control_and_encapsulation.md)

</details>

<details>
<summary><b>⭐ 类的其他特性</b></summary>

- [类成员类型](./note_7_3_additional_class_features.md)
- [可变数据成员 mutable](./note_7_3_additional_class_features.md)
- [返回 *this 的成员函数](./note_7_3_additional_class_features.md)
- [基于 const 的重载](./note_7_3_additional_class_features.md)
- [类类型与友元关系](./note_7_3_additional_class_features.md)

</details>

<details>
<summary><b>🎯 类的作用域</b></summary>

- [类作用域](./note_7_4_class_scope.md)
- [名字查找](./note_7_4_class_scope.md)
- [类型名](./note_7_4_class_scope.md)
- [作用域运算符 ::](./note_7_4_class_scope.md)

</details>

<details>
<summary><b>🔄 构造函数再探</b></summary>

- [构造函数初始化列表](./note_7_5_constructor_revisited.md)
- [委托构造函数](./note_7_5_constructor_revisited.md)
- [默认构造函数](./note_7_5_constructor_revisited.md)
- [隐式类类型转换](./note_7_5_constructor_revisited.md)
- [explicit 关键字](./note_7_5_constructor_revisited.md)

</details>

<details>
<summary><b>🌐 类的静态成员</b></summary>

- [静态数据成员](./note_7_6_static_class_members.md)
- [静态成员函数](./note_7_6_static_class_members.md)
- [静态成员定义与初始化](./note_7_6_static_class_members.md)
- [静态成员使用场景](./note_7_6_static_class_members.md)

</details>

---

## 💡 核心速查

**类定义基础**
```cpp
class Sales_data {
public:
    // 构造函数
    Sales_data() = default;
    Sales_data(const string &s) : bookNo(s) {}
    
    // 成员函数
    string isbn() const { return bookNo; }
    Sales_data& combine(const Sales_data &);
    
private:
    // 数据成员
    string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};
```

**this 指针**
```cpp
// 隐式指向调用对象
Sales_data& Sales_data::combine(const Sales_data &rhs) {
    units_sold += rhs.units_sold;
    revenue += rhs.revenue;
    return *this;  // 返回调用对象本身
}
```

**构造函数初始化列表**
```cpp
class ConstRef {
public:
    // 必须使用初始化列表的情况
    ConstRef(int ii) : ci(ii), ri(ii), i(ii) {}
private:
    int i;
    const int ci;    // const 成员
    int &ri;         // 引用成员
};
```

**友元**
```cpp
class Sales_data {
    // 友元声明
    friend Sales_data add(const Sales_data &, const Sales_data &);
    friend class Window_mgr;  // 友元类
    friend std::istream &read(std::istream &, Sales_data &);
};
```

**静态成员**
```cpp
class Account {
public:
    static double getRate() { return interestRate; }
    static void setRate(double);
private:
    static double interestRate;  // 声明
    static constexpr int period = 30;  // 编译期初始化
};

// 类外定义
double Account::interestRate = 0.02;
```

**explicit 构造函数**
```cpp
class Sales_data {
public:
    explicit Sales_data(const string &s) : bookNo(s) {}
    // 禁止隐式转换
};

Sales_data item1("978-0-201-70353-5");  // OK
// Sales_data item2 = "978-0-201-70353-5";  // 错误：隐式转换被禁止
```

---

## 🔗 相关链接

← [上一章：第 6 章 - 函数](../Chapter06_functions/README.md)  
→ [下一章：第 8 章 - IO 库](../Chapter08_IO_library/README.md)  
↑ [返回根目录](../README.md)

---

> 🕊️ *"类是 C++ 面向对象编程的基础，封装与抽象是设计良好接口的核心。"*
