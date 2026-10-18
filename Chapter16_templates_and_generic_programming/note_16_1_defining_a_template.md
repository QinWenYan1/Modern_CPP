# 📘 16.1 定义模板 (Defining a Template)

> 来源说明：C++ Primer 第16章第1节 | 本节涵盖：函数模板与类模板的定义、实例化、模板参数、成员模板及显式实例化控制

---

## 🧠 核心概念总览（严格按原文顺序）

* [*知识点1: 函数重载的问题与模板的引入*](#id1)
* [*知识点2: 函数模板的定义*](#id2)
* [*知识点3: 模板参数列表*](#id3)
* [*知识点4: 函数模板的实例化*](#id4)
* [*知识点5: 模板类型参数(Template Type Parameters)*](#id5)
* [*知识点6: typename与class关键字*](#id6)
* [*知识点7: 非类型模板参数(Nontype Template Parameters)*](#id7)
* [*知识点8: inline和constexpr函数模板*](#id8)
* [*知识点9: 编写类型无关代码(Writing Type-Independent Code)*](#id9)
* [*知识点10: 模板编译(Template Compilation)*](#id10)
* [*知识点11: 类模板(Class Templates)定义*](#id11)
* [*知识点12: 实例化类模板(Instantiating a Class Template)*](#id12)
* [*知识点13: 模板作用域内的模板类型引用*](#id13)
* [*知识点14: 类模板的成员函数(Member Functions)*](#id14)
* [*知识点15: Blob的构造函数*](#id15)
* [*知识点16: 类模板成员函数的实例化*](#id16)
* [*知识点17: 在类模板代码中简化模板类名的使用*](#id17)
* [*知识点18: 在类模板体外使用类模板名*](#id18)
* [*知识点19: 类模板与友元(Class Templates and Friends)*](#id19)
* [*知识点20: 模板类型别名(Template Type Aliases)*](#id20)
* [*知识点21: 类模板的static成员*](#id21)
* [*知识点22: 模板参数命名*](#id22)
* [*知识点23: 模板参数与作用域*](#id23)
* [*知识点24: 使用类成员作为类型(Using Class Members That Are Types)*](#id24)
* [*知识点25: 默认模板参数(Default Template Arguments)*](#id25)
* [*知识点26: 类模板的默认模板参数*](#id26)
* [*知识点27: 成员模板(Member Templates)*](#id27)
* [*知识点28: 普通类(非模板类)的成员模板*](#id28)
* [*知识点29: 类模板的成员模板*](#id29)
* [*知识点30: 实例化与成员模板*](#id30)
* [*知识点31: 控制实例化(Controlling Instantiations)*](#id31)
* [*知识点32: 显式实例化(Explicit Instantiation)*](#id32)
* [*知识点33: 实例化定义实例化所有成员*](#id33)
* [*知识点34: 运行时绑定删除器(Binding the Deleter at Run Time)*](#id34)
* [*知识点35: 编译时绑定删除器(Binding the Deleter at Compile Time)*](#id35)

---

<a id="id1"></a>
## ✅ 知识点1: 函数重载的问题与模板的引入

**理论**
* **核心问题**：编写比较两个值的函数时，需要为不同类型定义多个重载函数
* **代码重复**：这些函数几乎完全相同，**唯一的区别是参数类型**，而函数体完全相同
* **维护困难**：为每种比较类型重复编写函数体既繁琐又容易出错
* **局限性**：编写程序时必须知道所有可能想要比较的类型，无法处理用户自定义类型

**教材示例代码**
```cpp
// returns 0 if the values are equal, -1 if v1 is smaller, 1 if v2 is smaller
int compare(const string &v1, const string &v2)
{
    if (v1 < v2) return -1;
    if (v2 < v1) return 1;
    return 0;
}

int compare(const double &v1, const double &v2)
{
    if (v1 < v2) return -1;
    if (v2 < v1) return 1;
    return 0;
}
```

**注意点**
* ⚠️ **代码冗余**：重载方案导致代码重复，违反DRY原则
* 💡 **模板优势**：可以定义一个**函数模板(Function Template)** 来替代多个重载函数

---

<a id="id2"></a>
## ✅ 知识点2: 函数模板的定义

**理论**
* **OOP vs 泛型编程**：OOP的类型在**运行时**通过虚机制动态绑定，而泛型编程（模板）的类型在**编译时**实例化确定，生成具体类型代码。
* **核心概念**：<b>函数模板(Function Template)</b>是一个公式，可以从中生成特定类型的函数版本
* **定义方式**：使用关键字 `template` 开始，后跟**模板参数列表(Template Parameter List)**

**教材示例代码**
```cpp
template <typename T>
int compare(const T &v1, const T &v2)
{
    if (v1 < v2) return -1;
    if (v2 < v1) return 1;
    return 0;
}
```
**注意点**
* 📋 **关键字**：`template` 和 `typename` 是模板定义的核心关键字
* 🔧 **类型参数**：`T` 是**类型参数(Type Parameter)**，在函数体内用作类型名


---

<a id="id3"></a>
## ✅ 知识点3: 模板参数列表

**理论**
* **语法结构**：模板定义以关键字 `template` 开始，后跟用尖括号 `<` 和 `>` 括起来的**模板参数列表**
* **参数类型**：模板参数列表是一个或多个**模板参数(Template Parameters)** 的逗号分隔列表
* **非空要求**：模板参数列表**不能为空**

**教材示例代码**
```cpp
// 多参数示例（后续内容）
template <typename T, class U> 
```

**注意点**
* ⚠️ **重要限制**：在模板定义中，**模板参数列表不能为空**
* 📋 **参数类比**：模板参数列表的行为很像函数参数列表，但模板参数表示的是类型或值

---

<a id="id4"></a>
## ✅ 知识点4: 函数模板的实例化

**理论**
* **实例化过程**：调用函数模板时，编译器(通常)使用**函数的调用实参**来推断**模板实参(Template Arguments)**
* **类型绑定**：编译器使用推断出的模板参数来**实例化(Instantiate)** 函数的特定版本
* **实例(Instance)**：编译器创建模板的一个新"实例"，用实际的模板实参替换对应的模板形参

**教材示例代码**
```cpp
// 调用示例
cout << compare(1, 0) << endl;  // T is int
```

**注意点**
* 🔄 **自动推导**：对于 `compare(1, 0)`，编译器推断 `T` 为 `int`，并绑定到模板参数 `T`
* 📋 **生成代码**：编译器会生成 `int compare(const int&, const int&)` 的具体函数

---

<a id="id5"></a>
## ✅ 知识点5: 模板类型参数(Template Type Parameters)

**理论**
* **用途**：类型参数(Type Parameter)可用作类型说明符，与内置类型或类类型说明符的使用方式相同
* **应用范围**：
  - 指定返回类型
  - 函数参数类型
  - 变量声明
  - 类型转换(cast)

**教材示例代码**
```cpp
// ok: same type used for the return type and parameter
template <typename T> 
T foo(T* p)
{
    T tmp = *p;  // tmp will have the type to which p points
    // ...
    return tmp;
}
```

**注意点**
* 💡 **一致性**：同一个类型参数 `T` 可以用于返回类型、参数类型和局部变量类型
* 🔄 **类型统一**：确保函数内各处的类型一致性

---

<a id="id6"></a>
## ✅ 知识点6: `typename`与`class`关键字

**理论**
* **关键字选择**：每个类型参数前必须加关键字 `class` 或 `typename`
* **等价性**：这两个关键字在模板参数列表中含义相同，可以互换使用
* **历史原因**：`typename` 是在模板已经广泛使用后才加入C++的，一些程序员仍习惯只使用 `class`

**教材示例代码**
```cpp
// error: must precede U with either typename or class
template <typename T, U> T calc(const T&, const U&);

// ok: no distinction between typename and class in a template parameter list
template <typename T, class U> calc (const T&, const U&);
```

**注意点**
* ⚠️ **错误示例**：`U` 前缺少 `typename` 或 `class` 会导致编译错误
* 📋 **最佳实践**：`typename` 更清晰地表明后面跟着的是类型名，**推荐使用**

---

<a id="id7"></a>
## ✅ 知识点7: 非类型模板参数(Nontype Template Parameters)

**理论**
* **概念**：除了定义类型参数外，还可以定义接受**非类型参数(Nontype Parameters)** 的模板
* **表示内容**：非类型参数代表一个值而非类型，通过使用特定类型名指定
* **实参要求**：实例化时，非类型参数被用户提供的值或编译器推断的值替换，这些值必须是**常量表达式**
* **重要区别**：非类型模板参数（NTTP） 给你的不是"运行时变量"，而是编译期常量，且这个值会成为类型的一部分。
* **可用类型**：可以是整型、指向对象或函数的指针或(lvalue)引用
* **用途**：如下面例子，非类型模板参数让你把数字（如数组大小、循环次数）直接"写死"在类型里，编译器会针对这个具体数字生成最优代码，省去运行时的内存分配和计算开销。


**教材示例代码**
```cpp
template<typename T, size_t N>
void print_array(const T (&arr)[N]) {  // N自动推导为数组长度
    for(size_t i = 0; i < N; ++i) 
        cout << arr[i];
}

int a[5] = {1,2,3,4,5};
print_array(a);  // 自动实例化为 print_array<int, 5>，不会越界
```

**注意点**
* ⚠️ **常量要求**：用于非类型模板参数的模板实参必须是**常量表达式**
* 📋 **数组引用**：此例中参数是对数组的引用，用于处理字符串字面量（`const char`数组）
* 💡 **数组大小**：编译器根据字符串长度（含空终止符）推断 `N` 和 `M`

---

<a id="id8"></a>
## ✅ 知识点8: `inline`和`constexpr`函数模板

**理论**
* **声明方式**：函数模板可以像非模板函数一样声明为 `inline` 或 `constexpr`
* **位置要求**：`inline` 或 `constexpr` 说明符放在模板参数列表之后，返回类型之前

**教材示例代码**
```cpp
// ok: inline specifier follows the template parameter list
template <typename T> inline T min(const T&, const T&);

// error: incorrect placement of the inline specifier
inline template <typename T> T min(const T&, const T&);
```

**注意点**
* ⚠️ **位置错误**：将 `inline` 放在 `template` 之前是错误的
* 📋 **内联特性**：类模板内定义的成员函数隐式内联

---

<a id="id9"></a>
## ✅ 知识点9: 编写类型无关代码(Writing Type-Independent Code)

**理论**
* **核心原则1**：模板函数参数应为 **`const` 引用**，确保函数可用于**不可复制的类型**，并提高大对象的效率
* **核心原则2**：函数体内仅使用 `<` 比较操作，减少对类型的要求（只需支持 `<`，不需要 `>`）

**教材示例代码**
```cpp
// 预期使用方式（需要同时支持 < 和 >）
// if (v1 < v2) return -1;
// if (v1 > v2) return 1;

// 实际实现（仅需支持 <）
if (v1 < v2) return -1;
if (v2 < v1) return 1;
```

**注意点**
* 💡 **最佳实践**：模板程序应尽量减少对实参类型的要求
* 🔄 **可移植性**：如需真正关心类型独立性和可移植性，应使用标准库函数对象如 `less<T>`

---

<a id="id10"></a>
## ✅ 知识点10: 模板编译(Template Compilation)

**理论**
* **编译时机**：编译器看到模板定义时**不生成代码**，只有在实例化特定版本时才生成代码
* **组织方式**：与非模板代码不同，模板代码通常将定义和声明都放在**头文件**中
* **错误检测三阶段**：
  1. **模板编译阶段**：检查语法错误（如分号缺失）
  2. **模板使用阶段**：检查参数数量、类型匹配
  3. **实例化阶段**：检查类型相关错误（如不支持某操作符）

**教材示例代码**
```cpp
// 错误示例：Sales_data没有<操作符
Sales_data data1, data2;
cout << compare(data1, data2) << endl;  // error: no < on Sales_data
```

**注意点**
* ⚠️ **头文件要求**：**函数模板**和**类模板成员函数**的定义通常放在头文件中
* ⚠️ **调用者责任**：必须保证传递给模板的实参支持模板使用的任何操作，且操作行为正确

---

<a id="id11"></a>
## ✅ 知识点11: 类模板(Class Templates)定义

**理论**
* **概念**：**类模板(Class Template)** 是生成类的蓝图，与函数模板不同，编译器**不能**推断类模板的模板参数类型
* **使用方式**：使用类模板时必须提供**显式模板实参(Explicit Template Arguments)** 列表，放在类名后的尖括号中
* **Blob类示例**：类似于 `StrBlob`，但可用于几乎任何元素类型，用户需指定元素类型

**教材示例代码**
```cpp
template <typename T> class Blob {
public:
    typedef T value_type;
    typedef typename std::vector<T>::size_type size_type;
    // constructors
    Blob();
    Blob(std::initializer_list<T> il);
    // number of elements in the Blob
    size_type size() const { return data->size(); }
    bool empty() const { return data->empty(); }
    // add and remove elements
    void push_back(const T &t) {data->push_back(t);}
    void push_back(T &&t) { data->push_back(std::move(t)); }
    void pop_back();
    // element access
    T& back();
    T& operator[](size_type i);  // defined in § 14.5 (p. 566)
private:
    std::shared_ptr<std::vector<T>> data;
    // throws msg if data[i] isn't valid
    void check(size_type i, const std::string &msg) const;
};
```

**函数模板和类模板的区别**
1. **实例化方式不同**：
    - **函数模板**通常可以通过函数实参自动推导模板参数
    - **类模板**通常需要显式指定模板参数，但 C++17 起支持类模板实参推导
2. **实例化结果不同**：
    - **函数模板**实例化后生成具体的函数
    - **类模板**实例化后生成具体的类类型
3. **默认模板参数**：
    - 函数模板和类模板都支持默认模板参数，但函数模板从 C++11 才开始支持
4. **模板特化不同**：
    - **函数模板**只支持全特化，不支持偏特化（但可以重载）
    - **类模板**既支持全特化，也支持偏特化
5. **使用方式不同**：
    - **函数模板**通过函数调用使用，可以隐式推导或显式指定模板参数
    - **类模板**主要用于定义类型，通常通过 `vector<int>` 这样的形式使用，也可以在 C++17 起通过 CTAD 推导

**注意点**
* 📋 **模板参数使用**：在类模板定义中，使用模板参数 `T` 作为元素类型的占位符
* 🔧 **类型定义**：使用 `typedef` 和 `typename` 定义相关类型

---

<a id="id12"></a>
## ✅ 知识点12: 实例化类模板(Instantiating a Class Template)

**理论**
* **显式实参**：使用类模板时，必须提供**显式模板实参(Explicit Template Arguments)**，绑定到模板参数
* **实例化过程**：编译器使用这些模板实参从模板实例化出特定类
* **类型特定版本**：每个不同的元素类型会生成不同的类

**教材示例代码**
```cpp
Blob<int> ia;                 // empty Blob<int>
Blob<int> ia2 = {0,1,2,3,4};  // Blob<int> with five elements

Blob<string> names;           // Blob that holds strings
Blob<double> prices;          // different element type
```

**注意点**
* 💡 **独立类**：每个类模板的实例化构成一个独立的类，`Blob<string>` 与 `Blob<int>` 之间没有特殊关系，无权访问彼此的私有成员
* 🔄 **等价性**：`ia` 和 `ia2` 使用相同的类型特定版本 `Blob<int>`

---

<a id="id13"></a>
## ✅ 知识点13: 模板作用域内的模板类型引用

**理论**
* **类模板名不是类型名**：类模板名本身不是类型名，实例化后的类型总是包含模板实参
* **类模板体内使用**：在类模板代码中，通常不使用实际类型名作为模板实参，而是使用**模板自身的参数**作为模板实参
    * 比如类模板内部用 T 作为类型占位符（而非写死 int/string），实例化时自动替换成具体类型。
* **data成员示例**：`data` 成员使用 `Blob` 的类型参数 `T` 来表明它指向持有 `T` 类型对象的 `vector` 的实例化

**教材示例代码**
```cpp
// 在Blob<T>定义内部
std::shared_ptr<std::vector<T>> data;

// 当实例化 Blob<string> 时，data 成为：
// shared_ptr<vector<string>>
```

**注意点**
* 💡 **参数传递**：在类模板体内，可以将模板参数 `T` 传递给其他模板（如 `vector<T>`）
* 🔄 **自动替换**：当用户实例化特定 `Blob`（如 `Blob<string>`），所有 `T` 的出现都被替换为具体类型

---

<a id="id14"></a>
## ✅ 知识点14: 类模板的成员函数(Member Functions)

**理论**
* **定义位置**：可以在类体内或类体外定义类模板的成员函数
* **类体内定义**：在类模板体内定义的成员函数**隐式内联(Implicitly Inline)**
* **类体外定义**：在类模板体外定义成员函数时，必须以关键字 `template` 开始，后跟类的模板参数列表

**教材示例代码**
```cpp
// 类体外定义成员函数的格式
template <typename T>
Blob<T>::member-name(parm-list)
```

**注意点**
* 📋 **模板参数列表**：类模板外定义成员时，需要先写类的模板参数列表
* 🔄 **作用域规则**：在类模板体内，我们不在类的作用域内，直到类名被看到

---

<a id="id15"></a>
## ✅ 知识点15: `Blob`的构造函数

**理论**
* **默认构造函数**：分配空 `vector`，将指针存储在 `data` 中
* **初始化列表构造函数**：接受 `std::initializer_list<T>` 参数，使用 `T` 作为元素类型
* **构造函数定义格式**：类模板外定义构造函数时，需先声明模板参数，再指定所属类

**教材示例代码**
```cpp
// 默认构造函数
template <typename T>
Blob<T>::Blob() : data(std::make_shared<std::vector<T>>()) { }

// 初始化列表构造函数
template <typename T>
Blob<T>::Blob(std::initializer_list<T> il):
    data(std::make_shared<std::vector<T>>(il)) { }

// 使用示例
Blob<string> articles = {"a", "an", "the"};
```

**注意点**
* 💡 **类型兼容性**：初始化列表构造函数的参数类型为 `initializer_list<string>`，列表中每个字符串字面量隐式转换为 `string`
* 🔧 **内存分配**：两个构造函数都分配新的 `vector`，后者从参数 `il` 初始化该 `vector`

---

<a id="id16"></a>
## ✅ 知识点16: 类模板成员函数的实例化

**理论**
* **延迟实例化**：默认情况下，类模板的成员函数只有在**程序使用该成员函数**时才被实例化
* **优势**：可以实例化一个类，其类型可能不满足某些成员操作的要求（只要不使用那些成员）

**教材示例代码**
```cpp
// 示例：使用Blob<int>只实例化所需成员
Blob<int> squares = {0,1,2,3,4,5,6,7,8,9};
for (size_t i = 0; i != squares.size(); ++i)
    squares[i] = i*i;  // 实例化 operator[] 和 size()
```

**注意点**
* 📋 **实例化范围**：上述代码实例化了 `Blob<int>` 类和三个成员函数：`operator[]`、`size` 和初始化列表构造函数
* ⚠️ **未使用不实例化**：如果成员函数未被使用，则不会被实例化

---

<a id="id17"></a>
## ✅ 知识点17: 在类模板代码中简化模板类名的使用

**理论**
* **简化规则**：在类模板自身的作用域内，可以使用**模板名而不带实参**
* **等价性**：编译器将类模板体内的**引用**视为已提供与模板自身参数匹配的模板实参

**教材示例代码**
```cpp
// 前缀递增和递减成员返回 BlobPtr&，而非 BlobPtr<T>&
// 在类模板体内，BlobPtr 被视为 BlobPtr<T>
template <typename T> class BlobPtr {
public:
    BlobPtr& operator++();  // 前缀操作符，返回类型简化
    BlobPtr& operator--();
    // ...
};
```

**注意点**
* 💡 **作用域内简化**：在 `BlobPtr<T>` 作用域内，`BlobPtr` 等价于 `BlobPtr<T>`
* 📋 **编译器处理**：编译器将模板体内的引用视为已提供与模板自身参数匹配的模板实参

---

<a id="id18"></a>
## ✅ 知识点18: 在类模板体外使用类模板名

**理论**
* **返回类型外**：在返回类型处（在类作用域外），必须指定返回类型是类模板的实例化版本
* **函数体内**：在函数体内，我们处于类作用域内，无需重复模板实参

**教材示例代码**
```cpp
// postfix: increment/decrement the object but return the unchanged value
template <typename T>
BlobPtr<T> BlobPtr<T>::operator++(int)  // 返回类型在类作用域外，需指定<T>
{
    // no check needed here; the call to prefix increment will do the check
    BlobPtr ret = *this;  // 函数体内，BlobPtr 等价于 BlobPtr<T>
    ++*this;              // advance one element; prefix ++ checks the increment
    return ret;           // return the saved state
}
```

**注意点**
* ⚠️ **作用域边界**：类名后的作用域运算符(::)标志着类作用域的开始，在此之前需完整指定模板实参
* 💡 **内部简化**：在类作用域内定义 `ret` 时，无需写成 `BlobPtr<T>`

---

<a id="id19"></a>
## ✅ 知识点19: 类模板与友元(Class Templates and Friends)

**理论**
* **一对一友元**：类模板与另一模板（类或函数）建立友元关系时，最常见的形式是建立**对应实例化**之间的友元关系
* **友元声明**：当友元本身是模板时，类授予友元的控制权决定是否包含所有实例化或特定实例化
* **通用/特定友元**：
  - 可以让另一个模板的每个实例成为友元
  - 或限制友元关系为特定实例化
* **通用友元指定方法**：为了让让每个实例成为友元，我们需要让友元使用一个不同于本类的模版参数
* **需要前置声明的情况**

| 场景         | 代码特征                           | 需不需要前置声明 | 原因                              |
| ---------- | ------------------------------ | -------- | ------------------------------- |
| **类内定义**   | `friend xxx { ... }` 带函数体      | ❌ 不需要    | 编译器当场看到定义，直接认识                  |
| **通用友元**:一对多   | `template <X> friend xxx` | ❌ 不需要    | 当场引入新 `template`，编译器当场认识        |
| **特定实例友元**：一对一 | `friend xxx<T>` 带尖括号       | ✅ **需要** | 编译器需提前知道 `xxx` 是模板，否则把 `<` 当小于号 |


**教材示例代码**
```cpp
// 前置声明
template <typename> class BlobPtr;
template <typename> class Blob;  // needed for parameters in operator==
template <typename T>
    bool operator==(const Blob<T>&, const Blob<T>&);

// 一对一友元
template <typename T> class Blob {
    // each instantiation of Blob grants access to the version of
    // BlobPtr and the equality operator instantiated with the same type
    friend class BlobPtr<T>;
    friend bool operator==<T>
        (const Blob<T>&, const Blob<T>&);
    // ...
};

//所有实例通用化， 一对多友元
template <typename T> class Blob {
    // 关键：使用与类模板不同的模板参数（如 X）
    template <typename X> friend class BlobPtr;  // 所有人都是朋友！
};
```

**注意点**
* 📋 **前置声明**：友元声明前需要前置声明模板本身，并且可以省略 `T`占位符
    * 这是为了向编译器预告该模板是一个独立存在实体（非局部作用域或临时定义），从而确保模板实例化时能准确地识别并绑定友元关系，避免编译期因无法找到匹配的模板声明而导致错误。
* **遮蔽问题(shadowing)**：
    * 类模板参数`T`作用域是整个类
    * 如果你在类内再写 template `<typename T>`，新的 `T` 会覆盖外面的, 代码如下：
    ```cpp
    template <typename T>
    class Screen {
        // 如果你写：
        template <typename T>  // 这里的 H, W 会遮蔽类模板的 H, W
        friend std::ostream& operator<<(std::ostream&, const Screen<T>&);
        // 现在这个 H, W 是友元模板自己的，不是类的了，会搞混！
    };
    ```

* 🔧 **参数匹配**：友元声明使用 `Blob` 的模板参数作为自身的模板实参，限制为相同类型的实例化
* 💡 **C++11特性**：可以将模板类型参数设为友元, `friend Type;`（将实例化类型设为友元）`Type`就是使用`Type`模版类的友元了：
    ```cpp
        template <typename Type> class Bar {
            friend Type; // grants access to the type used to instantiate Bar
            // ...
        };
    ```
* 内置类或者类类型都可以

---

<a id="id20"></a>
## ✅ 知识点20: 模板类型别名(Template Type Aliases)

**理论**
* **`typedef`限制**：由于模板不是类型，不能定义引用模板的 `typedef`（如不能定义引用 `Blob<T>` 的 `typedef`）
* **C++11类型别名**：新标准允许为类模板定义**类型别名(Type Alias)**，使用 `using`
* **类家族同义词**：模板类型别名是**一族类**的同义词

**教材示例代码**
```cpp
// C++11: 模板类型别名
template<typename T> using twin = pair<T, T>;
twin<string> authors;  // authors is a pair<string, string>

// 固定部分参数
template <typename T> using partNo = pair<T, unsigned>;
partNo<string> books;   // books is a pair<string, unsigned>
partNo<Vehicle> cars;   // cars is a pair<Vehicle, unsigned>
```

**注意点**
* 💡 **语法**：`template<参数列表> using 别名 = 模板<参数>;`
* 🔄 **部分绑定**：可以固定一个或多个模板参数（如 `partNo` 固定第二个参数为 `unsigned`）

---

<a id="id21"></a>
## ✅ 知识点21: 类模板的static成员

**理论**
* **每个实例独立的`static`成员**：与其他类一样，类模板可以声明 `static` 成员，**每个实例化**有独立的 `static` 成员实例。
* **共享性**：特定实例化类型的所有对象共享相同的 `static` 成员。
* **定义方式**：像定义模板成员函数一样定义模板类的 `static` 数据成员。

**教材示例代码**
```cpp
template <typename T> class Foo {
public:
    static std::size_t count() { return ctr; }
private:
    static std::size_t ctr;
};

// static成员的定义和初始化
template <typename T>
size_t Foo<T>::ctr = 0;  // define and initialize ctr

// 使用示例
Foo<int> fi, fi2, fi3;  // 共享 Foo<int>::ctr
Foo<string> fs;         // 独立的 Foo<string>::ctr
```

**注意点**
* 📋 **访问方式**：通过类类型的对象或通过作用域运算符直接访问，必须引用特定实例化
* ⚠️ **实例化时机**：`static` 成员函数只有**在使用时**才被实例化

---

<a id="id22"></a>
## ✅ 知识点22: 模板参数命名

**理论**
* **命名自由**：与函数参数名一样，模板参数名没有内在含义，可以使用任何名字
* **习惯用法**：通常将类型参数命名为 `T`，但可以使用任何名称

**教材示例代码**
```cpp
// 使用Foo作为类型参数名
template <typename Foo> Foo calc(const Foo& a, const Foo& b)
{
    Foo tmp = a;  // tmp has the same type as the parameters and return type
    // ...
    return tmp;   // return type and parameters have the same type
}
```

**注意点**
* 💡 **一致性**：虽然可以任意命名，但保持命名习惯有助于代码可读性

---

<a id="id23"></a>
## ✅ 知识点23: 模板参数与作用域

**理论**
* **作用域规则**：模板参数遵循常规作用域规则，在**声明后、模板声明或定义结束前可用**
* **名称隐藏**：模板参数会**隐藏**外层作用域中相同名字的声明
* **不可重用**：与大多数其他上下文不同，模板参数名**不能在模板参数列表中重用**

**教材示例代码**
```cpp
typedef double A;
template <typename A, typename B> void f(A a, B b)
{
    A tmp = a;  // tmp has same type as the template parameter A, not double
    double B;   // error: redeclares template parameter B
}

// 错误：非法重用模板参数名V
template <typename V, typename V>  // error: illegal reuse of template parameter name V
```

**注意点**
* ⚠️ **重定义错误**：在模板体内不能用模板参数名定义变量（会导致重定义错误）
* ⚠️ **列表内唯一性**：模板参数名在参数列表中只能出现一次

---

<a id="id24"></a>
## ✅ 知识点24: 使用类成员作为类型(Using Class Members That Are Types)

**理论**
* **依赖名问题**：当编译器遇到如 `T::mem` 的代码时，在实例化前无法知道 `mem` 是类型还是 `static` 数据成员
* **默认假设**：默认情况下，C++假定通过作用域运算符访问的名字**不是类型**
* **`typename`关键字**：要告诉编译器一个名字是类型，必须使用关键字 `typename`（注意：不能用 `class`）

**教材示例代码**
```cpp
// 使用typename告知编译器这是类型
template <typename T>
typename T::value_type top(const T& c)
{
    if (!c.empty())
        return c.back();
    else
        return typename T::value_type();  // 值初始化元素
}
```

**注意点**
* ⚠️ **必须显式声明**：当需要告知编译器某名代表类型时，**必须使用 `typename`**，不能用 `class`
* 💡 **应用场景**：常用于访问容器类的成员类型（如 `value_type`、`iterator` 等）

---

<a id="id25"></a>
## ✅ 知识点25: 默认模板参数(Default Template Arguments)

**理论**
* **类似函数默认参数**：可以像为函数参数提供默认实参一样，为模板参数提供**默认模板实参**
* **C++11扩展**：新标准允许为函数模板和类模板提供默认实参，旧版只允许类模板有默认实参
* **默认函数参数结合**：可以提供默认模板参数和对应的默认函数参数

**教材示例代码**
```cpp
// compare has a default template argument, less<T>
// and a default function argument, F()
template <typename T, typename F = less<T>>
int compare(const T &v1, const T &v2, F f = F())
{
    if (f(v1, v2)) return -1;
    if (f(v2, v1)) return 1;
    return 0;
}

// 使用示例
bool i = compare(0, 42);  // uses less; i is -1
bool j = compare(item1, item2, compareIsbn);  // 使用自定义比较器
```

**注意点**
* 📋 **推导机制**：当 `compare` 用三个参数调用时，第三个参数的类型用于推断 `F`
* 💡 **默认规则**：模板参数默认实参的规则与函数默认实参类似，右侧参数有默认则左侧也可有默认

---

<a id="id26"></a>
## ✅ 知识点26: 类模板的默认模板参数

**理论**
* **使用默认值**：使用带默认实参的类模板时，可以在模板名后使用空尖括号 `<>` 表示使用默认类型

**教材示例代码**
```cpp
template <class T = int> class Numbers {  // by default T is int
public:
    Numbers(T v = 0): val(v) { }
private:
    T val;
};

Numbers<long double> lots_of_precision;
Numbers<> average_precision;  // empty <> says we want the default type
// 实例化为 Numbers<int>
```

**注意点**
* ⚠️ **空括号**：`Numbers<>` 中的空 `<>` 表示使用默认模板实参
* 📋 **总是需要括号**：使用类模板时，即使使用默认实参也必须带 `<>`（不同于函数模板）

---

<a id="id27"></a>
## ✅ 知识点27: 成员模板(Member Templates)

**理论**
* **定义**：类（普通类或类模板）可以有本身是模板的成员函数，这样的成员称为**成员模板(Member Templates)**
* **限制**：成员模板**不能是虚函数(`virtual`)**
     * **核心原因**在于：虚函数表（vtable）的大小必须在编译时固定，而成员模板在编译时可能实例化出无限多个版本，编译器无法预知需要预留多少表项。
* **独立性**：类模板和成员模板有各自独立的模板参数

**注意点**
* 📋 **应用场景**：常用于构造函数（接受迭代器范围）或函数调用运算符（如删除器）
* 🔄 **灵活性**：允许类成员独立于类自身的模板参数进行模板化

---

<a id="id28"></a>
## ✅ 知识点28: 普通类(非模板类)的成员模板

**理论**
* **应用示例**：定义类似 `unique_ptr` 使用的默认删除器的类，但带调试功能
* **设计**：类有重载的函数调用运算符，接受指针并在给定指针上执行 `delete`

**教材示例代码**
```cpp
// function-object class that calls delete on a given pointer
class DebugDelete {
public:
    DebugDelete(std::ostream &s = std::cerr): os(s) { }
    // as with any function template, the type of T is deduced by the compiler
    template <typename T> void operator()(T *p) const
    {
        os << "deleting unique_ptr" << std::endl; 
        delete p;
    }
private:
    std::ostream &os;
};

// 使用
double* p = new double;
DebugDelete d;  // an object that can act like a delete expression
d(p);  // calls DebugDelete::operator()<double*>, which deletes p
int* ip = new int;
DebugDelete()(ip);  // calls operator()<int*> on a temporary DebugDelete object
```

**注意点**
* 💡 **类型推导**：与普通函数模板一样，`T` 的类型由编译器推导
* 🔄 **作为删除器**：可用作 `unique_ptr` 的删除器类型

---

<a id="id29"></a>
## ✅ 知识点29: 类模板的成员模板

**理论**
* **双重模板**：可以定义类模板的成员模板，此时类和成员各自有独立的模板参数列表
* **定义格式**：在类模板体外定义成员模板时，必须提供**类模板和成员模板**的参数列表（类参数在前）

**教材示例代码**
```cpp
template <typename T> class Blob {
    template <typename It> Blob(It b, It e);  // 成员模板声明
    // ...
};

// 类模板外的定义
template <typename T>       // 类的类型参数
template <typename It>      // 构造函数的类型参数
Blob<T>::Blob(It b, It e):
    data(std::make_shared<std::vector<T>>(b, e)) { }
```

**注意点**
* 📋 **参数列表顺序**：先写**类的模板**参数列表，后写成员**自己的模板**参数列表
* 💡 **应用场景**：`Blob`的迭代器范围构造函数，支持从不同序列类型构造

---

<a id="id30"></a>
## ✅ 知识点30: 实例化与成员模板

**理论**
* **双重实例化**：实例化类模板的成员模板时，必须同时提供类和函数模板的实参
* **推导规则**：类模板参数由对象类型确定，成员模板参数通常由编译器从调用实参推断

**教材示例代码**
```cpp
int ia[] = {0,1,2,3,4,5,6,7,8,9};
vector<long> vi = {0,1,2,3,4,5,6,7,8,9};
list<const char*> w = {"now", "is", "the", "time"};

Blob<int> a1(begin(ia), end(ia));           // It 推断为 int*
Blob<int> a2(vi.begin(), vi.end());         // It 推断为 vector<long>::iterator
Blob<string> a3(w.begin(), w.end());        // It 推断为 list<const char*>::iterator
```

**注意点**
* 🔄 **自动推导**：`a1` 中 `T` 是 `int`，构造函数自己的参数类型从 `begin(ia)` 和 `end(ia)` 推断为 `int*`
* 📋 **不同实例**：`a2` 使用已实例化的 `Blob<int>`，但构造函数实例化为 `vector<long>::iterator` 版本

---

<a id="id31"></a>
## ✅ 知识点31: 控制实例化(Controlling Instantiations)

**理论**
* **实例化开销**：模板在使用时生成实例化，相同模板可能在多个文件中实例化，在大系统中开销显著
* **显式实例化**：通过**显式实例化(Explicit Instantiation)** 避免开销，形式为：
    1. `extern template declaration`（声明）
    2.  `template declaration`（定义）
* 其中声明是指**所有模板形参均被模板实参替换后**的类或函数声明。
* 在特定编译单元中显式指定模板实例后，就可以避免多个文件重复实例化同一个模板。

**教材示例代码**
```cpp
// 显式实例化声明（不生成代码）
extern template class Blob<string>;
extern template int compare(const int&, const int&);

// 显式实例化定义（生成代码）
template int compare(const int&, const int&);
template class Blob<string>;
```

**注意点**
* ⚠️ **位置要求**：`extern` 声明必须出现在使用实例化的代码之前


---

<a id="id32"></a>
## ✅ 知识点32: 显式实例化(Explicit Instantiation)

**理论**
* **声明与定义**：
  - `extern template declaration`：实例化声明，承诺在其他地方有非 `extern` 定义
  - `template declaration`：实例化定义，生成代码
* **使用限制**：可以有多个 `extern` 声明，但必须有且只有一个定义

**教材示例代码**
```cpp
// Application.cc
extern template class Blob<string>;  // 实例化将出现在别处
extern template int compare(const int&, const int&);
Blob<string> sa1, sa2;  // 使用extern声明的实例化
int i = compare(a1[0], a2[0]);  // 实例化将出现在别处
Blob<int> a1 = {0,1,2,3,4,5,6,7,8,9};

// templateBuild.cc（必须链接到Application）
template int compare(const int&, const int&);
template class Blob<string>;  // 实例化所有成员
```

**代码解析**
* `compare<int>`, `Blob<string>` 不会在`Application.cc`给实例化。
* 因为编译器知道他们的模版一定在其他文件里存在。 
* 当在`templateBuild.cc`里面编译器看到了实例化的定义然后再生成代码
* 因此 **链接要求**：`Application.o` 必须链接包含实例化定义的 `templateBuild.o`

**注意点**
* ⚠️ **单一定义规则**：每个显式实例化声明必须在程序某处有显式实例化定义
* 📋 **代码生成**：编译器看到实例化定义时生成代码，看到 `extern` 声明时不生成


---
<a id="id33"></a>
## ✅ 知识点33: 实例化定义实例化所有成员

**理论**
* **完整实例化**：类模板的实例化定义会实例化该模板的**所有成员**，包括内联成员函数
* **使用限制**：因此，显式实例化**只能**用于能与模板所有成员一起使用的类型

**教材示例代码**
```cpp
// 显式实例化定义示例
template class Blob<string>;  // 实例化Blob<string>的所有成员
```

**注意点**
* ⚠️ **严格性**：与按需实例化不同，显式实例化定义会实例化所有成员，即使未使用的成员也会被实例化
* 💡 **适用场景**：适用于类型完全支持模板所有操作的场景

---

<a id="id34"></a>
## ✅ 知识点34: 运行时绑定删除器(Binding the Deleter at Run Time)

**理论**
* 标准库的智能指针给我们了一个很好的模版设计例子
* **`shared_ptr`的`delete`策略**：
    1. 由于`shared_ptr`支持自定义删除器（如`delete`、`free` 或自定义函数），且允许在**运行时**才确定甚至**中途更换**删除器类。
    2. 这就导致编译期无法确定删除器的具体类型。
    3. 因此不能将其作为固定类型的成员变量直接存储在对象内，必须通过指针间接存储，并在**运行时动态**跳转访问并调用。
* **存储方式**：删除器必须间接存储（如作为指针或封装指针的类，如 `function` 对象）
* **运行时开销**：由于删除器间接存储，调用 `del(p)` 需要运行时跳转到存储在 `del` 中的位置执行代码

**教材示例代码**
```cpp
// 假设shared_ptr实现
// value of del known only at run time; call through a pointer
del ? del(p) : delete p;  // del(p) requires run-time jump to del's location
```

**注意点**
* 📋 **灵活性**：运行时绑定允许在 `shared_ptr` 生命周期内改变删除器类型（使用 `reset`）
* ⚠️ **性能成本**：运行时跳转带来一定性能开销

---

<a id="id35"></a>
## ✅ 知识点35: 编译时绑定删除器(Binding the Deleter at Compile Time)

**理论**
* **`unique_ptr`策略**：`unique_ptr` 有两个模板参数，一个表示管理的指针类型，另一个表示删除器类型
* **类型包含**：删除器类型是 `unique_ptr` 类型的一部分，编译时已知
* **存储方式**：删除器可直接存储在每个 `unique_ptr` 对象中（作为成员）
* **性能优势**：无运行时开销，删除器调用可能在编译时内联

**教材示例代码**
```cpp
// del bound at compile time; direct call to the deleter is instantiated
del(p);  // no run-time overhead
```

**注意点**
* 💡 **性能对比**：编译时绑定避免运行时间接调用开销
* ⚠️ **灵活性权衡**：用户自定义删除器更复杂（需显式指定为模板参数）
* 📋 **设计选择**：运行时绑定便于用户覆盖删除器，编译时绑定提供更好性能

---

## 🔑 核心要点总结

1. **模板定义与实例化**：模板定义不生成代码，只有在实例化（使用特定类型）时才生成；类模板需显式指定类型实参，函数模板通常可自动推导

2. **模板参数类型**：包括**类型参数**（`typename`/`class`）和**非类型参数**（整型、指针、引用，必须是常量表达式）

3. **编译模型**：模板通常定义在头文件中；编译错误分三阶段检测（定义语法、使用检查、实例化类型检查）

4. **类模板特性**：每个实例化是独立类；成员函数延迟实例化（使用时才实例化）；类体内可简写模板名（不加 `<T>`）

5. **成员模板**：类（普通或模板）可有成员模板，用于构造函数（迭代器范围）等场景；类外定义需两个模板参数列表

6. **显式实例化控制**：使用 `extern template` 避免多文件重复实例化开销，但需确保单一定义；显式实例化定义会实例化所有成员

7. **删除器绑定策略**：`shared_ptr` 运行时绑定删除器（灵活但有开销），`unique_ptr` 编译时绑定删除器（零开销但类型复杂）

---

## 📌 考试速记版

**关键语法速查**：
```cpp
// 函数模板
template <typename T> T func(const T& a, const T& b) { }

// 类模板
template <typename T> class ClassName { /* T 作为类型使用 */ };

// 类模板成员函数类外定义
template <typename T> ret-type ClassName<T>::member(parms) { }

// 非类型模板参数
template <unsigned N> void func(const char (&arr)[N]);

// 显式实例化声明（不生成代码）
extern template class vector<string>;

// 显式实例化定义（生成代码）
template class vector<string>;
```

**重要区别对比**：

| 特性 | 函数模板 | 类模板 |
|------|---------|--------|
| **类型推导** | 编译器自动推导 | 必须显式提供实参 |
| **默认实参** | C++11起支持 | 支持，可用 `<>` 使用默认 |
| **实例化时机** | 调用时 | 对象定义时 |
| **成员实例化** | 整体实例化 | 按需实例化（使用时） |

**typename vs class**：
- 模板参数列表中：等价，可互换
- 告知编译器某名是类型时：**必须用 `typename`**（如 `typename T::value_type`）

**常见错误陷阱**：
- ❌ 类模板外定义成员忘记 `template <typename T>`
- ❌ 在类模板体内定义变量与模板参数同名（重定义错误）
- ❌ 非类型模板参数使用非常量表达式
- ❌ 忘记类模板实例间无特殊关系（不能互相访问私有成员）

**记忆口诀**：
> 模板参数两类分，类型值类要区分；  
> 函数推导类显式，实例代码用才生；  
> typename 告诉编译器，嵌套类型需标明；  
> 显式实例化省开销，extern声明需配对。