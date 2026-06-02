# 📘 Chapter 15 - 面向对象程序设计 (Object-Oriented Programming)

> 《C++ Primer》第 15 章学习笔记导航  
> 本章涵盖：继承、虚函数、多态、抽象基类与访问控制

---

## 📑 章节导航

### 15.1 OOP：概述 🎯
面向对象基本概念、继承与动态绑定  
[→ 查看笔记](./note_15_1_an_overview.md)

### 15.2 定义基类和派生类 🏗️
基类定义、派生类定义、虚函数、override/final  
[→ 查看笔记](./note_15_2_defining_base_and_derived_classes.md)

### 15.3 虚函数 🔀
动态绑定、虚函数调用、纯虚函数、抽象基类  
[→ 查看笔记](./note_15_3_virtual_functions.md)

### 15.4 抽象基类 📐
纯虚函数、抽象类、接口继承  
[→ 查看笔记](./note_15_4_abstract_base_classes.md)

### 15.5 访问控制与继承 🔒
public/protected/private 继承、using 声明改变访问权限  
[→ 查看笔记](./note_15_5_access_control_and_inheritance.md)

### 15.6 继承中的类作用域 🎯
名字查找、名字隐藏、虚函数与作用域  
[→ 查看笔记](./note_15_6_class_scope_under_inheritance.md)

### 15.7 构造函数与拷贝控制 🔄
派生类构造、析构、拷贝与移动  
[→ 查看笔记](./note_15_7_constructors_and_copy_control.md)

### 15.8 容器与继承 📦
容器中存放继承层次对象、智能指针的使用  
[→ 查看笔记](./note_15_8_container_and_inheritance.md)

### 15.9 文本查询程序再探 📖
使用面向对象重新实现文本查询程序  
[→ 查看笔记](./note_15_9_text_queries_revisited.md)

### 章节总结 📝
第 15 章核心概念速查表  
[→ 查看总结](./note_15_summary.md)

### 练习题 💪
本章所有练习代码  
[→ 查看代码](./exercises/)

---

## 🎯 知识点索引

<details>
<summary><b>🎯 OOP 概述</b></summary>

- [继承概念](./note_15_1_oop_an_overview.md)
- [基类与派生类](./note_15_1_oop_an_overview.md)
- [虚函数与动态绑定](./note_15_1_oop_an_overview.md)
- [多态性](./note_15_1_oop_an_overview.md)

</details>

<details>
<summary><b>🏗️ 定义基类和派生类</b></summary>

- [基类定义](./note_15_2_defining_base_and_derived_classes.md)
- [派生类定义 class D : public B](./note_15_2_defining_base_and_derived_classes.md)
- [虚函数 virtual](./note_15_2_defining_base_and_derived_classes.md)
- [override 关键字](./note_15_2_defining_base_and_derived_classes.md)
- [final 关键字](./note_15_2_defining_base_and_derived_classes.md)
- [派生类对象组成](./note_15_2_defining_base_and_derived_classes.md)

</details>

<details>
<summary><b>🔀 虚函数</b></summary>

- [动态绑定机制](./note_15_3_virtual_functions.md)
- [虚函数调用](./note_15_3_virtual_functions.md)
- [纯虚函数 = 0](./note_15_3_virtual_functions.md)
- [虚析构函数](./note_15_3_virtual_functions.md)
- [回避虚函数机制](./note_15_3_virtual_functions.md)

</details>

<details>
<summary><b>📐 抽象基类</b></summary>

- [纯虚函数定义](./note_15_4_abstract_base_classes.md)
- [抽象类不能实例化](./note_15_4_abstract_base_classes.md)
- [接口继承](./note_15_4_abstract_base_classes.md)
- [重构继承层次](./note_15_4_abstract_base_classes.md)

</details>

<details>
<summary><b>🔒 访问控制与继承</b></summary>

- [public 继承](./note_15_5_access_control_and_inheritance.md)
- [protected 成员](./note_15_5_access_control_and_inheritance.md)
- [protected 继承](./note_15_5_access_control_and_inheritance.md)
- [private 继承](./note_15_5_access_control_and_inheritance.md)
- [using 改变访问权限](./note_15_5_access_control_and_inheritance.md)
- [友元与继承](./note_15_5_access_control_and_inheritance.md)

</details>

<details>
<summary><b>🎯 继承中的类作用域</b></summary>

- [名字查找顺序](./note_15_6_scope_under_inheritance.md)
- [名字隐藏/遮蔽](./note_15_6_scope_under_inheritance.md)
- [使用作用域运算符访问被隐藏成员](./note_15_6_scope_under_inheritance.md)
- [虚函数与作用域](./note_15_6_scope_under_inheritance.md)

</details>

<details>
<summary><b>🔄 构造函数与拷贝控制</b></summary>

- [派生类构造函数](./note_15_7_constructors_and_copy_control.md)
- [基类构造函数调用](./note_15_7_constructors_and_copy_control.md)
- [派生类析构函数](./note_15_7_constructors_and_copy_control.md)
- [虚析构函数重要性](./note_15_7_constructors_and_copy_control.md)
- [合成拷贝控制与继承](./note_15_7_constructors_and_copy_control.md)
- [删除的拷贝控制与继承](./note_15_7_constructors_and_copy_control.md)

</details>

<details>
<summary><b>📦 容器与继承</b></summary>

- [容器中不能存放不同类型对象](./note_15_8_containers_and_inheritance.md)
- [使用智能指针管理](./note_15_8_containers_and_inheritance.md)
- [vector<shared_ptr<Base>>](./note_15_8_containers_and_inheritance.md)

</details>

<details>
<summary><b>📖 文本查询程序再探</b></summary>

- [面向对象设计](./note_15_9_text_query_program_revisited.md)
- [Query_base 抽象基类](./note_15_9_text_query_program_revisited.md)
- [WordQuery/NotQuery/AndQuery/OrQuery](./note_15_9_text_query_program_revisited.md)
- [eval 虚函数](./note_15_9_text_query_program_revisited.md)

</details>

---

## 💡 核心速查

**基本继承语法**
```cpp
// 基类
class Quote {
public:
    Quote() = default;
    Quote(const string &book, double sales_price) :
        bookNo(book), price(sales_price) {}
    
    virtual double net_price(size_t n) const {
        return n * price;
    }
    
    virtual ~Quote() = default;  // 虚析构函数！
    
private:
    string bookNo;
protected:
    double price = 0.0;
};

// 派生类
class Bulk_quote : public Quote {
public:
    Bulk_quote() = default;
    Bulk_quote(const string&, double, size_t, double);
    
    // 覆盖虚函数
    double net_price(size_t) const override;
    
    // 阻止进一步覆盖
    void some_func() final;
};
```

**动态绑定**
```cpp
// 通过基类指针/引用调用虚函数时发生动态绑定
Quote item;           // 基类对象
Bulk_quote bulk;      // 派生类对象

Quote *p = &item;     // p 指向 Quote
p = &bulk;            // p 指向 Bulk_quote

double ret = p->net_price(20);  // 调用 Bulk_quote::net_price
```

**访问控制**
```cpp
class Base {
public:
    void pub_mem();     // 公有成员
protected:
    int prot_mem;       // 受保护成员（派生类可访问）
private:
    char priv_mem;      // 私有成员（仅类内可访问）
};

struct Pub_Derv : public Base {
    void memfcn() {
        prot_mem = 1;   // OK：派生类可访问 protected 成员
        // priv_mem = 1; // 错误：不能访问 private 成员
    }
};
```

**纯虚函数与抽象类**
```cpp
class Disc_quote : public Quote {
public:
    Disc_quote() = default;
    Disc_quote(const string& book, double price, size_t qty, double disc) :
        Quote(book, price), quantity(qty), discount(disc) {}
    
    double net_price(size_t) const = 0;  // 纯虚函数
    
protected:
    size_t quantity = 0;
    double discount = 0.0;
};

// Disc_quote 是抽象类，不能实例化
// Disc_quote disc;  // 错误！
```

**虚析构函数（重要！）**
```cpp
class Base {
public:
    virtual ~Base() = default;  // 基类必须有虚析构函数
};

class Derived : public Base {
public:
    ~Derived() { /* 释放 Derived 资源 */ }
};

// 使用
Base *p = new Derived();
delete p;  // 正确：先调用 ~Derived()，再调用 ~Base()
```

**override 与 final**
```cpp
struct B {
    virtual void f1(int) const;
    virtual void f2();
    void f3();
};

struct D1 : B {
    void f1(int) const override;  // OK：正确覆盖
    // void f2(int) override;      // 错误：参数不匹配
    // void f3() override;         // 错误：f3 不是虚函数
    // void f4() override;         // 错误：B 没有 f4
};

struct D2 final : B {  // D2 不能被继承
    void f2() override final;  // 后续派生类不能覆盖 f2
};
// struct D3 : D2 { };  // 错误：D2 是 final
```

**容器中存储继承层次对象**
```cpp
vector<shared_ptr<Quote>> basket;
basket.push_back(make_shared<Quote>("0-201-82470-1", 50));
basket.push_back(make_shared<Bulk_quote>("0-201-82470-1", 50, 10, 0.25));

// 动态绑定正确工作
cout << basket.back()->net_price(15) << endl;  // 调用 Bulk_quote::net_price
```

---

## 🔗 相关链接

← [上一章：第 14 章 - 重载运算与类型转换](../Chapter14_overloaded_operations_and_conversions/README.md)  
↑ [返回根目录](../README.md)

---

> 🕊️ *"面向对象是 C++ 的核心范式，继承实现代码复用，多态实现接口复用，虚函数是连接二者的桥梁。"*
