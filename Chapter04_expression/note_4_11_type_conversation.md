# 📘 Chapter 4.11 Type Conversions (类型转换)

> 本节内容源自 C++ Primer 4.11 节，主要讲解了 C++ 中的类型转换机制，包括隐式转换和显式转换。隐式转换由编译器自动执行，而显式转换需要通过强制类型转换操作符来请求。

---

## 🧠 本节核心概念（提纲目录）

*   类型转换概述与分类
*   隐式转换发生情况
*   算术转换与整型提升
*   无符号操作数的特殊情况
*   其他隐式转换（数组到指针、指针转换、布尔转换等）
*   显式转换
*   `static_const`
*   `void*` 指针
*   `const_const`
*   `reinterpret_cast`
*   `dynamic_cast`
*   旧式转型与使用建议

---

## ✅ 类型转换概述与分类 (Overview and Categories)

**定义 / 理论**
*   当两种类型相关时，可以使用一种类型的对象或值代替另一种类型，这称为**类型转换**。
*   C++ 中的类型转换分为两大类：
    1.  **隐式转换 (Implicit Conversions)**：由编译器自动执行，无需程序员干预，操作会去尽量保护变量精确度
    2.  **显式转换 (Explicit Conversions)**：程序员通过强制类型转换操作符明确请求

**教材示例代码**
```cpp
int ival = 3.541 + 3; 
// 隐式转换：3(int→double保护变量精度), 结果(double→int)
// 1. 整数3被转换为double类型
// 2. 执行浮点数加法，结果为double
// 3. double结果被截断为int，用于初始化ival
```

**注意点**
*   ⚠️ 隐式转换虽然方便，但有时会导致精度损失或意想不到的行为，特别是从浮点型到整型的转换会丢失小数部分。

---

## ✅ 隐式转换发生情况 (When implicit conversations occur)

编译器会在以下情况**自动转换**运算数

*   大多数表达式中，小于`int`的整形，会被提升为更大的整形类
*   在条件表达式中，非布尔表达式转换为布尔值
*   初始化中， 初始化器转换为变量类
*   赋值中，右边运算数类转换为左边运算数类
*   在混合运算中， 编译器会将所有运算数转换为同一个类
*   函数调用也会有类型转换

---
## ✅ 算术转换与整型提升 (Arithmetic Conversions and Integral Promotions)

**定义 / 理论**

*   **算术转换**：当不同类型的数据一起运算时，编译器会自动将它们转换为**其中最“大”**、最精确的类型，比如：
    *   如果有一个操作数是 `long double`，无论其他类为什么，都转换为 `long double`
    *   浮点数和整型混合运算时，整型都会转换为正确的浮点类

*   **整型提升**：将较小的整型类型提升为更大的整型：
    *   `bool`, `char`, `unsigned char`, `signed char`, `short`,`unsigned short` → `int` (**如果该类所有可能值能放到int里面**) 
    否则，→ `unsigned int`
    *   宽字符类型（如 `wchar_t`, `char16_t`, `char32_t`）→ 能完整容纳其所有可能值的最小标准整数类型
        (`int`,`unsigned int`,`long`,`unsigned long`,`long long`, `unsigned long long`)

---
## ✅ 无符号操作数的特殊情况 (Operands of Unsigned Type)


1.  **整型提升优先** (Integral Promotions First)
    *    所有小整型（`bool`, `char`, `short`, `unsigned short` 等）首先被提升为 `int` 或 `unsigned int`。

2.  **符号相同** (Same Signedness)
    *    如果提升后的类型符号相同（同为有符号或同为无符号），则**较小类型的操作数转换为较大的类型**

3.  **符号不同** (Different Signedness) - 这是最关键的部分：
    *    **无符号类型 >= 有符号类型**（指类型的大小/范围）：
        *   有符号操作数 → 无符号类型
        *   例如：`int` + `unsigned int` → `unsigned int`
        *   如果有符号操作数是一个负数，值的转换为unsigned表现在 **2_1章节primitive_type**描述
    *    **有符号类型 > 无符号类型**（指类型的大小/范围）
        *   **如果** 无符号类型的所有值都能存放在有符号类型中则 **无符号 → 有符号**
        *   否则 **有符号类 → 无符号类**
        *   例如：在 `long` = `int` 的系统上，`long` + `unsigned int` → `unsigned int`
        *   若在 `long` > `int` 大的系统上, `long` + `unsigned int` → `long`


**教材示例代码**
```cpp
bool flag;      char cval; 
short sval;     unsigned short usval;
int ival;       unsigned int uival; 
long lval;      unsigned long ulval;
float fval;     double dval;

3.14159L + 'a';     // 'a'→int, 然后int→long double
dval + ival;        // ival→double
dval + fval;        // fval→double
ival = dval;        // dval→int (截断)
flag = dval;        // dval→bool (0→false, 非0→true)
cval + fval;        // cval→int, 然后int→float
sval + cval;        // sval和cval都→int
cval + lval;        // cval→long
ival + uival;       // ival→unsigned int
usval + ival;       // 根据相对大小转换
uival + lval;       // 根据相对大小转换
```

**注意点**
*   ⚠️ 涉及无符号类型的转换可能导致意外的结果，特别是当有符号值为负数时。

---

## ✅ 其他隐式转换 (Other Implicit Conversions)

**定义 / 理论**
除了算术转换，C++ 还定义了其他几种隐式转换：

1.  **数组到指针的转换**：在大多数表达式中，数组自动转换为指向其首元素的指针。
    ```cpp
    int ia[10];     // 10个整数的数组
    int* ip = ia;   // ia转换为指向首元素的指针
    ```
2.  **数组不会发生转换的情况**:
    *   对其使用`decltype`,`sizeof`或`typeid`(19_2章节)
    *   对其使用地址运算符`&`
    *   初始化对数组引用

2.  **指针转换**：
    *   整数值 0 和 `nullptr` 可转换为任何指针类型
    *   指向非 const 的指针可转换为 `void*`
    *   指向任何类型的指针可转换为 `const void*`
    *   继承也会影响指针转换 (15_2章节)

3.  **转换为 bool**：算术或指针类型可自动转换为 bool。
    *   零值 → `false`
    *   非零值 → `true`

4.  **转换为 const**：指向非常量的指针或引用，安全地转换为指向常量的指针或引用。这被称为"添加底层const"。
    *   但是反向转换 - 去除 low-level const **不被允许**
    ```cpp
    int i;
    const int &j = i;    // OK: 非const→const引用
    const int *p = &i;   // OK: 非const地址→const指针
    int &r = j, *q = p;  // error: 不允许const指针转换为nonconst 
    ```

5.  **类类型定义的转换**：类可以定义编译器自动应用的转换。比如c风格字符串 → 标准库string类
    ```cpp
    string s, t = "a value";  // 字符串字面值→string
    while (cin >> s)          // istream→bool
    ```

**注意点**
*   ⚠️ 数组到指针的转换有几个例外：不适用于 `decltype`、取地址(`&`)、`sizeof` 或 `typeid` 的操作数。

---

## ✅ 显式转换（四种命名转换）(Explicit Conversions)

**定义 / 理论**
有时需要显式要求类型转换，这称为**强制类型转换**或**转型**。
命名转换的语法格式为：

```cpp
cast_name<type>(expression);
```
* **`type`**：目标转换类型
* **`expression`**：要转换的值或表达式
* 如果 `type` 是引用类型，则结果是一个左值，其他情况为右值
* `cast-name` 确定执行哪种转换类型。

---

## ✅ `static_cast`

-   任何明确定义的类型转换都可以（除了底层 const）。
-   这种转换常用于**将大的算术类型转换为小的算术类型**
-   也常用于**编译器不会自动进行的类型转换**，比如去获取一个存放在`void*`的值
-   使用他常常也是告诉程序和编译器我们意识到了但也不关心潜在精度缺失
-   使用显式转换会关掉编译器的警告信息   

```cpp
double slope = static_cast<double>(j) / i;  // 强制浮点除法
void* p = &d;
double* dp = static_cast<double*>(p);       // 将void*转换回原始类型
```
**解析**
-   我们使用`void*`去储存一个指针然后将其再转换为原类型 (这并**不会**造成对地址的重新解释，void*只用于存储地址)
-   这样可以让指针值得到保护
-   转换后的结果是原地址值
-   然而我们必须确认**转换后的类就是实际原本的指针类**
-   如果两者不匹配， 结果会未定义
---
## ✅ `void*` 指针

---
## ✅ `const_cast`
-   用于只改变运算对象的**底层 const**，  用于改变对象类型是非法的
-   使用其他转换形式去改变const是编译时报错的 
-   这样一来，编译器**不会**再禁止我们对目标对象的写入
-   如果目标对象本身是非const，那么写入是合法的
-   但是如果是为了用于对const对象写入的话， 那么结果未定义
```cpp
const char *cp;
// error: static_cast 无法去掉const 
char *q = static_cast<char*>(cp);
static_cast<string>(cp); // ok: 将字符串字面值转换为字符串
const_cast<string>(cp); // error: const_cast 无法改变类型
```
⚠️ 通过 `cp` 写入原是 const 的对象是未定义行为

---

## ✅ `reinterpret_cast`
“重新解释内存布局”的操作（极其危险！）。
```cpp
int* ip;
char* pc = reinterpret_cast<char*>(ip);  // 危险：将int*视为char*
string str(pc);                          // 可能导致运行时错误
```

**解释**
-   我们必须记住`ip`是一个`int*`指针，而不是`char*`指针
-   任何将其当作`char*`使用都极可能会**报错在运行时**
-   因为使用这种转换不会报错或者警告
-   编译器不会知道指针原本类型， 而是假定其为`char*`
-   所以初始化`str`是正确的，尽管是无意义的
-   因此回溯回去找问题是极其困难的

**注意点**
*   ⚠️ `reinterpret_cast` 平台相关、风险高，只有完全理解类型和编译器实现时才能安全使用。

---

## ✅ `dynamic_cast`
**用途**：支持运行时类型识别（将在第19章介绍）。

**注意点**
*   ⚠️ 强制类型转换干扰了正常的类型检查，应尽可能避免使用。

---

## ✅ 旧式转型与使用建议 (Old-Style Casts and Advice)

*   旧式类型转换是C++早期版本中使用的两种显式转换语法形式
*   这两种形式在功能上等同于新式命名转换，但缺乏明确性和安全性，**增加代码阅读难度**
*   并且可能意外执行危险的 `reinterpret_cast`，导致未定义行为
*   现代C++强烈建议避免使用旧式转换，优先使用新式命名转换

**两种旧式转换语法**
```cpp
// 1. 函数风格转换
type (expr);

// 2. C语言风格转换  
(type) expr;
```

**旧式转换的实际行为**
*   旧式转换会根据上下文自动选择最"匹配"的新式转换行为：
    *   如果 `static_cast` 或 `const_cast` 合法 → 执行相应的命名转换
    *   如果上述都不合法 → 执行 `reinterpret_cast`（最危险的情况！）

**示例代码**
```cpp
int* ip = /* ... */;
char* pc = (char*) ip; // 旧式转换，实际上执行 reinterpret_cast<char*>(ip)
```

**注意点**
*   💡 **最佳实践**：始终优先使用新式命名转换（`static_cast`, `const_cast`, `dynamic_cast`, `reinterpret_cast`）

---

## ✅ 核心建议总结

*   **避免使用类型转换**：类型转换会干扰正常的类型检查系统，应尽可能避免
*   **特别警惕 `reinterpret_cast`**：这种转换总是危险的，应极度谨慎使用
*   **谨慎使用 `const_cast`**：除在重载函数上下文中有用外，其他使用通常表示设计缺陷
*   **减少使用 `static_cast` 和 `dynamic_cast`**：这些转换应该很少需要
*   **彻底避免旧式转换**：旧式转换缺乏明确性，应使用新式命名转换替代

###  使用原则

每次考虑使用类型转换时，应该：
1.  仔细思考是否能通过其他方式达到相同效果
2.  如果转换确实不可避免，应限制被转换值的使用范围减少错误，通过详细记录所有关于涉及类型的假设
3.  优先选择最能明确表达意图的新式命名转换
4.  **避免使用类型转换**：类型转换会干扰正常的类型检查系统，应尽可能避免

**终极建议**：在C++编程中，类型转换应该是最后的选择，而不是首选的解决方案。良好的设计通常可以避免大多数类型转换的需求。

---

## 🔑 本节总结

*   C++类型转换分为隐式转换（编译器自动完成）和显式转换（程序员强制请求）两大类，核心目标是保证类型安全的同时提供必要的灵活性。
*   隐式转换遵循一套复杂但明确的规则：算术转换以保持精度为优先，整型提升先于其他转换，无符号类型参与运算时需要特别小心符号差异带来的意外结果。
*   除算术转换外，还存在数组到指针、指针转换、布尔转换和常量性添加等多种重要隐式转换，每种都有其特定规则和例外情况。
*   显式转换应作为最后手段，C++提供的四种命名转换各司其职：`static_cast`用于相对安全的明确定义转换，`const_cast`专门处理常量性，`reinterpret_cast`用于危险的低级重新解释，`dynamic_cast`支持运行时类型识别。
*   旧式C风格转换应彻底避免，因其行为不明确且隐蔽性强，现代C++强烈推荐使用命名转换以明确表达转换意图。
*   最根本的原则是尽可能避免使用类型转换，良好设计可以消除大多数转换需求。必须使用时，应限制转换值的作用域并详细记录类型假设。

---

## 📌 考试速记版

**转换基本原则**
- 能隐不显，尽量避免
- 无号小心，负会变正
- 小整先提，向宽看齐

**隐式转换关键点**
- 条件非bool→bool（0假非0真）
- 数组名→首元素指针（除&、sizeof等）
- 非const→const可，const→非const不可
- 类类型定义转换只走一步

**四种命名转换速记**
- `static_cast`：安全明确转换（算术、void*回调、类层次）
- `const_cast`：仅改常量性（去const，慎用！）
- `reinterpret_cast`：位重解释（极端危险，平台相关）
- `dynamic_cast`：运行时类型检查（后续讲）

**终极建议**
- 弃旧式(`(type)`)，用新式(`xxx_cast<type>`)
- 写cast前三思：能否避免？范围能否限制？
- 记牢：reinterpret最危，const_cast慎用