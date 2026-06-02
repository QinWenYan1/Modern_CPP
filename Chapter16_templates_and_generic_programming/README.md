# 《C++ Primer》第 16 章学习笔记导航

本章涵盖：函数模板、类模板、模板实参推断、重载与模板、可变参数模板、模板特化

## 📑 小节笔记目录

### 16.1 定义模板
函数模板定义、类模板定义、模板参数、成员模板

[→ 查看笔记](./note_16_1_defining_a_template.md)

### 16.2 模板实参推断
类型推断规则、显式模板实参、尾置返回类型

[→ 查看笔记](./note_16_2_template_argument_deduction.md)

### 16.3 重载与模板
函数模板重载、模板与非模板重载、最佳匹配规则

[→ 查看笔记](./note_16_3_overloading_and_templates.md)

### 16.4 可变参数模板
参数包、包展开、转发参数包、递归处理

[→ 查看笔记](./note_16_4_variadic_templates.md)

### 16.5 模板特化
全特化、偏特化、类模板特化、函数模板特化

[→ 查看笔记](./note_16_5_template_specializations.md)

---

## 📊 章节总结

第 16 章核心概念速查表

[→ 查看总结](./note_16_summary.md)

---



## 🗂️ 核心概念索引

### 📋 定义模板

- [函数模板 template <typename T>](./note_16_1_defining_a_template.md)
- [类模板 template <class T>](./note_16_1_defining_a_template.md)
- [模板参数：类型参数、非类型参数](./note_16_1_defining_a_template.md)
- [模板作用域与模板类型别名](./note_16_1_defining_a_template.md)
- [成员模板](./note_16_1_defining_a_template.md)
- [实例化与代码膨胀](./note_16_1_defining_a_template.md)

### 🔍 模板实参推断

- [类型推断规则](./note_16_2_template_argument_deduction.md)
- [顶层const的推断](./note_16_2_template_argument_deduction.md)
- [引用类型的推断](./note_16_2_template_argument_deduction.md)
- [显式模板实参 <T>](./note_16_2_template_argument_deduction.md)
- [尾置返回类型 auto ->](./note_16_2_template_argument_deduction.md)
- [类型转换与模板](./note_16_2_template_argument_deduction.md)

### ⚖️ 重载与模板

- [函数模板重载](./note_16_3_overloading_and_templates.md)
- [模板与非模板同时匹配](./note_16_3_overloading_and_templates.md)
- [最佳匹配规则](./note_16_3_overloading_and_templates.md)
- [模板特化与重载区别](./note_16_3_overloading_and_templates.md)
- [可变参数模板匹配](./note_16_3_overloading_and_templates.md)

### 📦 可变参数模板

- [模板参数包 typename... Args](./note_16_4_variadic_templates.md)
- [函数参数包 Args... rest](./note_16_4_variadic_templates.md)
- [sizeof... 运算符](./note_16_4_variadic_templates.md)
- [包展开模式](./note_16_4_variadic_templates.md)
- [转发参数包 std::forward](./note_16_4_variadic_templates.md)
- [递归处理参数包](./note_16_4_variadic_templates.md)

### 🎯 模板特化

- [全特化 template <>](./note_16_5_template_specializations.md)
- [偏特化（类模板）](./note_16_5_template_specializations.md)
- [函数模板特化](./note_16_5_template_specializations.md)
- [类模板成员特化](./note_16_5_template_specializations.md)
- [特化与重载的选择](./note_16_5_template_specializations.md)

---

## 📝 代码示例

### 函数模板

```cpp
// 函数模板定义
template <typename T>
int compare(const T &v1, const T &v2) {
    if (v1 < v2) return -1;
    if (v2 < v1) return 1;
    return 0;
}

// 使用
compare<int>(1, 2);        // 显式指定
template <typename T> T foo(T* p) { /* ... */ }
```

### 类模板

```cpp
// 类模板定义
template <class T> class Blob {
public:
    typedef T value_type;
    typedef typename std::vector<T>::size_type size_type;
    
    // 构造函数
    Blob();
    Blob(std::initializer_list<T> il);
    
    // 成员函数
    size_type size() const { return data->size(); }
    bool empty() const { return data->empty(); }
    
    void push_back(const T &t) { data->push_back(t); }
    void push_back(T &&t) { data->push_back(std::move(t)); }
    
    T& back();
    T& operator[](size_type i);
    
private:
    std::shared_ptr<std::vector<T>> data;
    void check(size_type i, const std::string &msg) const;
};

// 实例化
Blob<int> ia;                    // 空Blob
Blob<int> ia2 = {0,1,2,3,4};     // 列表初始化
```

### 模板实参推断

```cpp
template <typename T, typename U>
auto sum(T t, U u) -> decltype(t + u) {
    return t + u;
}

// 使用
auto val = sum(1, 2.5);  // T=int, U=double, 返回类型推断为double
```

### 可变参数模板

```cpp
// 递归终止函数
template <typename T>
std::ostream &print(std::ostream &os, const T &t) {
    return os << t;
}

// 可变参数模板
template <typename T, typename... Args>
std::ostream &print(std::ostream &os, const T &t, const Args&... rest) {
    os << t << ", ";
    return print(os, rest...);  // 递归调用，展开参数包
}

// 使用
print(cout, 1, 2.5, "hello", 'c');  // 输出: 1, 2.5, hello, c
```

### 模板特化

```cpp
// 通用模板
template <typename T>
int compare(const T&, const T&);

// 全特化：const char* 版本
template <>
int compare(const char* const &p1, const char* const &p2) {
    return strcmp(p1, p2);
}

// 类模板偏特化
template <class T> struct remove_reference {
    typedef T type;
};

template <class T> struct remove_reference<T&> {
    typedef T type;
};

template <class T> struct remove_reference<T&&> {
    typedef T type;
};
```

### 转发参数包（完美转发）

```cpp
template <typename... Args>
void emplace_back(Args&&... args) {
    // 检查是否需要分配新内存
    check_n_alloc();
    
    // 在first_free指向的位置构造对象
    // std::forward 保持原始类型（左值/右值）
    alloc.construct(first_free++, std::forward<Args>(args)...);
}

// 使用：Args&& 是万能引用，std::forward 保持值类别
StrVec vec;
vec.emplace_back(10, 'c');     // 调用 string(size_t, char)
vec.emplace_back("hello");     // 调用 string(const char*)
```

---

## 🧭 章节导航

← [上一章：第 15 章 - 面向对象程序设计](../Chapter15_object_oriented_programming/README.md)

→ [下一章：第 17 章 - 标准库特殊设施](../Chapter17_specialized_library_facilities/README.md)

↑ [返回根目录](../README.md)

---

🕊️ *"模板是 C++ 泛型编程的基石，它让代码超越类型的限制，在编译期实现零成本抽象。"*
