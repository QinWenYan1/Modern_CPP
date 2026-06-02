# 📘 Chapter 14 - 重载运算与类型转换 (Overloaded Operations and Conversions)

> 《C++ Primer》第 14 章学习笔记导航  
> 本章涵盖：运算符重载、类型转换、函数对象与 Lambda

---

## 📑 章节导航

### 14.1 基本概念 🎯
运算符重载规则、选择成员或非成员实现  
[→ 查看笔记](./note_14_1_basic_concept.md)

### 14.2 输入和输出运算符 📥📤
重载 << 和 >> 运算符  
[→ 查看笔记](./note_14_2_input_and_output_operators.md)

### 14.3 算术和关系运算符 ➕
+、-、*、/、==、!=、<、> 等  
[→ 查看笔记](./note_14_3_arithmetic_and_relational_operators.md)

### 14.4 赋值运算符 📝
复合赋值、拷贝赋值、移动赋值  
[→ 查看笔记](./note_14_4_assignment_operators.md)

### 14.5 下标运算符 🔢
重载 operator[]  
[→ 查看笔记](./note_14_5_subscript_operators.md)

### 14.6 递增和递减运算符 ⬆️
前置与后置 ++、--  
[→ 查看笔记](./note_14_6_increment_and_decrement_operators.md)

### 14.7 成员访问运算符 🔍
解引用 *、箭头 -> 运算符  
[→ 查看笔记](./note_14_7_member_access_operators.md)

### 14.8 函数调用运算符 📞
重载 ()、函数对象、Lambda  
[→ 查看笔记](./note_14_8_function_call_operators.md)

### 14.9 重载、类型转换与运算符 🔄
类型转换运算符、避免二义性  
[→ 查看笔记](./note_14_9_overloading_conversions_operators.md)

### 章节总结 📝
第 14 章核心概念速查表  
[→ 查看总结](./note_14_summary.md)

### 练习题 💪
本章所有练习代码  
[→ 查看代码](./exercises/)

---

## 🎯 知识点索引

<details>
<summary><b>🎯 基本概念</b></summary>

- [运算符重载规则](./note_14_1_basic_concepts.md)
- [成员函数 vs 非成员函数](./note_14_1_basic_concepts.md)
- [不能重载的运算符](./note_14_1_basic_concepts.md)
- [运算符优先级与结合性](./note_14_1_basic_concepts.md)

</details>

<details>
<summary><b>📥📤 输入输出运算符</b></summary>

- [重载 << 输出运算符](./note_14_2_input_and_output_operators.md)
- [重载 >> 输入运算符](./note_14_2_input_and_output_operators.md)
- [非成员实现](./note_14_2_input_and_output_operators.md)
- [错误处理](./note_14_2_input_and_output_operators.md)

</details>

<details>
<summary><b>➕ 算术和关系运算符</b></summary>

- [算术运算符 + - * /](./note_14_3_arithmetic_and_relational_operators.md)
- [相等运算符 == !=](./note_14_3_arithmetic_and_relational_operators.md)
- [关系运算符 < > <= >=](./note_14_3_arithmetic_and_relational_operators.md)
- [非成员实现建议](./note_14_3_arithmetic_and_relational_operators.md)

</details>

<details>
<summary><b>📝 赋值运算符</b></summary>

- [复合赋值 += -= *= /=](./note_14_4_assignment_operators.md)
- [拷贝赋值 =](./note_14_4_assignment_operators.md)
- [移动赋值 =](./note_14_4_assignment_operators.md)
- [返回 *this](./note_14_4_assignment_operators.md)

</details>

<details>
<summary><b>🔢 下标运算符</b></summary>

- [operator[] 重载](./note_14_5_subscript_operator.md)
- [const 与非 const 版本](./note_14_5_subscript_operator.md)
- [下标类型](./note_14_5_subscript_operator.md)

</details>

<details>
<summary><b>⬆️ 递增递减运算符</b></summary>

- [前置 ++ --](./note_14_6_increment_and_decrement_operators.md)
- [后置 ++(int) --(int)](./note_14_6_increment_and_decrement_operators.md)
- [区分前置后置](./note_14_6_increment_and_decrement_operators.md)

</details>

<details>
<summary><b>🔍 成员访问运算符</b></summary>

- [解引用运算符 *](./note_14_7_member_access_operators.md)
- [箭头运算符 ->](./note_14_7_member_access_operators.md)
- [智能指针实现](./note_14_7_member_access_operators.md)

</details>

<details>
<summary><b>📞 函数调用运算符</b></summary>

- [operator() 重载](./note_14_8_function_call_operator.md)
- [函数对象](./note_14_8_function_call_operator.md)
- [含状态的函数对象](./note_14_8_function_call_operator.md)
- [Lambda 是函数对象](./note_14_8_function_call_operator.md)
- [标准库函数对象](./note_14_8_function_call_operator.md)

</details>

<details>
<summary><b>🔄 类型转换运算符</b></summary>

- [类型转换运算符 operator type()](./note_14_9_overloading_conversions_and_operators.md)
- [explicit 类型转换](./note_14_9_overloading_conversions_and_operators.md)
- [避免二义性](./note_14_9_overloading_conversions_and_operators.md)
- [转换与运算符重载](./note_14_9_overloading_conversions_and_operators.md)

</details>

---

## 💡 核心速查

**输出运算符 <<**
```cpp
class Sales_data {
public:
    // 友元声明（需要访问私有成员）
    friend ostream &operator<<(ostream &os, const Sales_data &item);
};

ostream &operator<<(ostream &os, const Sales_data &item) {
    os << item.isbn() << " " << item.units_sold << " " << item.revenue;
    return os;  // 返回左值以支持链式调用
}
```

**算术运算符（对称性要求非成员）**
```cpp
// 非成员实现，保持对称性
Sales_data operator+(const Sales_data &lhs, const Sales_data &rhs) {
    Sales_data sum = lhs;
    sum += rhs;  // 使用复合赋值
    return sum;
}
```

**下标运算符**
```cpp
class StrVec {
public:
    string& operator[](size_t n) { return elements[n]; }
    const string& operator[](size_t n) const { return elements[n]; }
private:
    string *elements;
};
```

**递增递减运算符**
```cpp
class StrBlobPtr {
public:
    // 前置
    StrBlobPtr& operator++() {
        check(curr, "increment past end");
        ++curr;
        return *this;
    }
    
    // 后置（int 参数区分）
    StrBlobPtr operator++(int) {
        StrBlobPtr ret = *this;
        ++*this;
        return ret;
    }
};
```

**箭头运算符**
```cpp
class StrBlobPtr {
public:
    string& operator*() const { return (*check(curr, "dereference"))[curr]; }
    string* operator->() const { return &this->operator*(); }
};

// 使用
StrBlobPtr p;
*p = "hello";       // 解引用
string len = p->size();  // 箭头
```

**函数对象**
```cpp
// 定义函数对象类
struct Add {
    int operator()(int a, int b) const { return a + b; }
};

Add add;
int sum = add(1, 2);  // 3

// 含状态的函数对象
struct PrintString {
    ostream &os;
    char sep;
    PrintString(ostream &o = cout, char c = ' ') : os(o), sep(c) {}
    void operator()(const string &s) const { os << s << sep; }
};
```

**类型转换运算符**
```cpp
class SmallInt {
public:
    SmallInt(int i = 0) : val(i) {
        if (i < 0 || i > 255) throw out_of_range("Bad value");
    }
    
    // 转换为 int
    explicit operator int() const { return val; }
    
private:
    size_t val;
};

SmallInt si = 10;
int i = static_cast<int>(si);  // 显式转换
// int j = si;  // 错误：explicit 禁止隐式转换
```

---

## 🔗 相关链接

← [上一章：第 13 章 - 拷贝控制](../Chapter13_copy_control/README.md)  
→ [下一章：第 15 章 - 面向对象程序设计](../Chapter15_object_oriented_programming/README.md)  
↑ [返回根目录](../README.md)

---

> 🕊️ *"运算符重载让自定义类型像内置类型一样自然工作，类型转换则让不同类型间的协作更加灵活。"*
