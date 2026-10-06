# 📘 2.4 Section — const 限定符 (const Qualifier)

> 本章笔记基于《C++ Primer》第五版整理，详细介绍了 C++ 中的 const 限定符系统。

---

## 🧠 本章核心概念

- ✯ const 限定符基本概念
- ✯ const 对象的定义和初始化
- ✯ 初始化与 const
- ✯ 默认情况下，const 对象仅在文件内有效
- ✯ const 的引用 (References to const)
  - const 引用的特性
  - 初始化和 const 引用
  - 对 const 的引用可能引用一个并非 const 的对象
- ✯ 指针和 const (Pointers and const)
  - 指向 const 的指针
  - const 指针
  - 顶层 const 和底层 const
- ✯ constexpr 和常量表达式 (constexpr and Constant Expressions)
  - 常量表达式
  - constexpr 变量
  - 字面值类型
  - 指针和 constexpr

---

## ✅ const 限定符基本概念

有时我们希望定义一个其值不能被改变的变量。例如，我们可能想要使用一个变量来表示缓冲区大小。使用变量使我们可以很容易地在需要时改变缓冲区的大小。另一方面，我们也希望防止代码无意中给用来表示缓冲区大小的变量赋予新值。

- 我们可以用关键字 `const` 对变量的类型加以限定来做到这一点：

  ```cpp
  const int bufSize = 512;    // input buffer size
  ```

- 这样定义了 `bufSize` 是一个常量。任何试图对 `bufSize` 赋值的行为都将引发错误：

  ```cpp
  bufSize = 512; // error: attempt to write to const object
  ```

### 📝 const 作用及用法
**作用：**
1. const 修饰成员变量，定义成 const 常量，相较于宏常量`#define`，可进行类型检查，节省内存空间，提高了效率
2. const 修饰函数参数，使得传递过来的函数参数的值不能改变
3. **在成员变量中**:
    - const 成员变量只能在类内声明、定义，在构造函数初始化列表中初始化
    - const 成员变量只在某个对象的生存周期内是常量，对于整个类而言却是可变的，因为类可以创建多个对象，不同类的 const 成员变量的值是不同的。因此不能在类的声明中初始化 const 成员变量，类的对象还没有创建，编译器不知道他的值。
4. **在成员函数中**:
    - **不能修改成员变量的值**，除非有 mutable 修饰;只能访问成员变量
    - **不能调用非常量成员函数**，以防修改成员变量的值

### 📝 初始化和 const

**`const` 对象必须显式初始化**：因为 `const` 对象在创建后其值就不能再改变，所以 `const` 对象必须显式初始化。和其他对象一样，初始化值可以是任意复杂的表达式：

```cpp
const int i = get_size();  // ok: initialized at run time
const int j = 42;          // ok: initialized at compile time
const int k;               // error: k is uninitialized const
```

### 📝 初始化和 const

- 一个对象能做什么，取决于它的类型。`const` 类型的对象基本能做普通对象能做的事，只不过**不能改它的值**，像算术运算、转成 bool 这些操作还是照样能用。


- 给另一个对象赋值或者用它来初始化，并不会改原来的对象的值，所以就算是 `const` 也没问题。也就是说，不管谁是 `const`，只要是拷贝或者初始化，都是安全的：

```cpp
int i = 42;
const int ci = i;          // ok: the value in i is copied into ci
int j = ci;                // ok: the value in ci is copied into j
```

虽然 `ci` 是 `const int`，但它里面的值还是普通的 `int`。`const` 只是在你**想改它**的时候才起作用。像把 `ci` 拷贝给 `j` 这种操作，其实只是**读一下它的值**，所以不会触发 `const` 限制。拷贝完成后，`j` 是自己的变量，跟 `ci` 没什么关系了。


### 📝 默认情况下，const 对象仅在文件内有效

- **编译时常量替换**：当一个 `const` 对象用编译时常量初始化时，比如我们对 bufSize 的定义：

```cpp
const int bufSize = 512;   // input buffer size
```

编译器将在编译过程中把用到该变量的地方都替换成对应的值。也就是说，编译器会用 512 替换代码中所有用到 bufSize 的地方。

- **多文件访问的问题**：如果一个程序有多个文件，而你在某些文件里用到了 `const` 变量，编译器必须能看到它的初始值，因为它要在编译时进行替换。所以，每个用到这个 `const` 的文件都得自己提供一份变量的定义，不然就会出错。

- **默认行为**：为了支持这种用法，同时避免对同一变量的重复定义，默认情况下，`const` 对象被设定为仅在文件内有效。当多个文件中出现了同名的 `const` 变量时，其实等同于在不同文件中分别定义了独立的变量。

- **跨文件共享 const 对象**：有时候我们希望一个 `const` 变量能在多个文件中共用，而不是每个文件都单独生成一份。尤其当它的初始值比较复杂时，我们就想像普通变量一样：在一个文件里定义，其他文件只引用。这样就不会重复生成多个变量了。

- **解决方案**：为了在多个文件之间共享 `const` 对象，必须在变量的定义之前添加 **`extern`** 关键字：

```cpp
// file_1.cc defines and initializes a const that is accessible to other files
extern const int bufSize = fcn();
// file_1.h
extern const int bufSize; // same bufSize as defined in file_1.cc
```

在这个例子里，`file_1.cc` 里定义并初始化了 bufSize。因为它是 `const`，如果想让别的文件也能用它，就必须加上 `extern`，否则默认只在当前文件可见。

`file_1.h` 头文件中的声明也由 `extern` 做了限定，其作用是指明 `bufSize` 并非本文件所独有，它的定义将在别处出现。

#### 📝 注意
如果想在多个文件间共享 `const` 对象，必须在变量的定义之前添加 **`extern`** 关键字。

---

## ✅ const 的引用 (References to const)

可以把引用绑定到 `const` 对象上，就像绑定到其他对象上一样，我们称之为**对常量的引用（reference to const）**。与普通引用不同的是，对常量的引用不能被用作修改它所绑定的对象：

```cpp
const int ci = 1024;
const int &r1 = ci;        // ok: both reference and underlying object are const
r1 = 42;                   // error: r1 is a reference to const
int &r2 = ci;              // error: non const reference to a const object
```


### 📝 术语：const 引用是对 const 的引用
- `const` 引用”只是说引用的是一个 `const` 对象，而不是说引用本身是 `const`，因为引用从来就不能重新绑定，不需要也不能加 `const` 修饰引用本身。

### 📝 初始化和对 const 的引用

- 引用的类型必须与其所引用对象的类型一致，但是有两个例外。
- 第一个例外情况是在初始化常量引用时允许用任意表达式作为初始值，只要该表达式的结果能够转换成引用的类型即可。尤其，允许为一个常量引用绑定**非常量的对象**、**字面值**，**甚至是个一般表达式**：

```cpp
int i = 42;
const int &r1 = i;        // we can bind a const int& to a plain int object
const int &r2 = 42;       // ok: r1 is a reference to const
const int &r3 = r1 * 2;   // ok: r3 is a reference to const
int &r4 = r * 2;          // error: r4 is a plain, non const reference
```

理解这种例外情况的最简单办法是弄清楚当我们把一个引用绑定到另外一种类型上时到底发生了什么：

```cpp
double dval = 3.14;
const int &ri = dval;
```

此处 `ri` 引用了一个 `int` 类型。对 `ri` 的操作应该是整数运算，但是 `dval` 却是一个双精度浮点数而非整数。因此为了确保让 `ri` 绑定一个整数，编译器把上述代码变成了如下形式：

```cpp
const int temp = dval;    // create a temporary const int from the double
const int &ri = temp;     // bind ri to that temporary
```

- 在这种情况下，ri 绑定了一个**临时量（temporary）对象**。所谓临时量对象就是当编译器需要一个空间来暂存表达式的求值结果时临时创建的一个未命名的对象。C++ 程序员们常常把临时量对象简称为临时量。

- 现在考虑一下如果 `ri` 不是常量会发生什么情况。如果 `ri` 不是常量，就可以给 `ri` 赋值，这样做会改变 `ri` 所引用的对象。但是 `ri` 绑定的是一个临时量而非 `dval`。程序员既然让 `ri` 引用 `dval`，就肯定想通过 `ri` 改变 `dval` 的值，可一旦给 `ri` 赋了值，真正被改变的是临时量对象，而非 `dval`，这样程序员的真实意图就无法实现了。正是由于这个原因，C++ 语言规定这种行为是非法的。

### 📝 对 const 的引用可能引用一个并非 const 的对象

- 常量引用只是说“不能通过这个引用来改对象”，但对象本身可能还是可以被改的。
- 也就是说，如果这个对象是个普通变量，你虽然不能通过这个 const 引用去改它，但可以通过别的方式去改，比如直接用变量名、用另一个非常量引用等:

```cpp
int i = 42;
int &r1 = i;              // r1 bound to i
const int &r2 = i;        // r2 also bound to i; but cannot be used to change i
r1 = 0;                   // r1 is not const; i is now 0
r2 = 0;                   // error: r2 is a reference to const
```

r2 绑定（非常量）整数 i 是合法的。然而，不允许通过 r2 修改 i。尽管如此，i 的值仍然允许通过其他途径修改，既可以直接给 i 赋值，也可以通过像 r1 一样绑定到 i 的其他引用来修改。

---

## ✅ 指针和 const (Pointers and const)

### 📝 指向 const 的指针

- 与引用一样，也可以令指针指向常量或非常量。
- 类似于常量引用，<b>指向常量的指针（pointer to const）</b>不能用于改变其所指对象的值
- 要想存储常量对象的地址，只能使用指向常量的指针：

  ```cpp
  const double pi = 3.14;   // pi is const; its value may not be changed
  double *ptr = &pi;        // error: ptr is a plain pointer
  const double *cptr = &pi; // ok: cptr may point to a double that is const
  *cptr = 42;               // error: cannot assign to *cptr
  ```

- 指针的类型必须与其所指对象的类型一致，但是有两个例外（规则同样适用于引用）：
  - 基类指针，可以指向派生类对象（以后讲）
  - 另外一个例外是允许令一个**指向常量的指针**指向一个**非常量对象**（这节重点讲）：

  ```cpp
  double dval = 3.14;       // dval is a double; its value can be changed
  cptr = &dval;             // ok: but can't change dval through cptr
  ```

- 指向常量的指针（`const` 指针）只是说**不能通过这个指针去改值**，但并不代表那个值本身不能被改。如果这个对象本身不是 `const`，你还是可以通过别的方法去修改它

- 换句话说，所谓指向常量的指针或引用，不过是指针或引用"**自以为**"所指的是常量，因此指针或引用也就不会允许做改变对象值的操作。

#### 💡 提示
- 把指针或引用想象成指向 const 的指针或引用，可以理解为"自以为指向常量"的指针或引用。

### 📝 const 指针

- 指针是对象，而引用不是，因此指针可以像其他对象一样被定义为常量。<b> 常量指针（const pointer）</b>在定义时**必须初始化**，而且一旦初始化，其值（即指向的地址）就不能再更改。
- 指向常量的指针不需要立即初始化。
- 将 `*` 放在 `const` 关键字之前，表示指针本身是常量，而不是它指向的值：

```cpp
int errNumb = 0;
int *const curErr = &errNumb;  // curErr will always point to errNumb
const double pi = 3.14159;
const double *const pip = &pi; // pip is a const pointer to a const object
```

理解复杂声明最好的方法是**从右往左读**。在这个例子里，`const` 紧挨着 `curErr`，说明 curErr 本身是个常量；再往左是 `*`，表示 curErr 是一个常量指针；最左边的 `int` 则说明它指向一个 `int` 类型的对象。类似地，pip 是一个常量指针，指向的是一个**常量对象**。

- 指针本身是一个常量并不意味着不能通过指针修改其所指对象的值，能否这样做完全依赖于**所指对象的类型**。
- 例如，`curErr` 指向一个普通的（非常量）整数，当然可以用 `curErr` 去修改 `errNumb` 的值：

```cpp
*curErr = 0; // ok: reset the value of the object to which curErr is bound
```

类似的，也不能给 pip 赋值让其指向其他对象，但是如果该对象的值不是常量，我们就能通过 pip 改变其所指对象的值。

### 📝 顶层 const 和底层 const

![alt text](images/2.png)

前面说过，**指针本身是对象**，它又可以指向另一个对象。
所以我们得分别考虑两个“常量性”：

* 指针本身能不能变（指向别的地方）
* 指针指向的对象能不能变

为此，C++ 把这两个概念分开命名：

* **顶层 const（top-level const）**：指的是对象本身是常量，比如“这个指针不能改动指向”
* **底层 const (low-level const)**：指的是对象指向的内容是常量，比如“通过这个指针不能改内容”

**顶层 `const` VS 底层 `const`**：

- 顶层 const 适用于所有类型；而底层 const 主要和指针、引用这类复合类型打交道。
**指针类型可以同时拥有顶层 const 和底层 const**，这是它相比其他类型特别的地方。

- 更通用地说，**顶层 const** 表示对象本身是常量，适用于任何类型，比如基本类型、类、指针等；而**底层 const** 则用于基础类的复合类型，说明该类型所指或所引用的对象是常量。

特别地，**指针类型**既可以有顶层 const（指针本身不变），也可以有底层 const（指向的内容不变），这使它与其他类型相比更具特殊性。


```cpp
int i = 0;
int *const p1 = &i;        // we can't change the value of p1; const is top-level
const int ci = 42;         // we cannot change ci; const is top-level
const int *p2 = &ci;       // we can change p2; const is low-level
const int *const p3 = p2;  // right-most const is top-level, left-most is low-level
const int &r = ci;         // const in reference types is always low-level
```

- 拷贝对象时，顶层 `const` 不影响拷贝，会被忽略；只有底层 `const` 会真正影响拷贝的行为。


```cpp
i = ci;  // ok: copying the value of ci; top-level const in ci is ignored
p2 = p3; // ok: pointed-to type matches; top-level const in p3 is ignored
```

- 拷贝操作并**不会**改变被拷贝对象的值，因此，拷入和拷出的对象是否是常量都没什么影响。

- 另一方面，**底层 `const` 的限制是必须遵守的**。拷贝对象时，源对象和目标对象要么拥有相同的底层 `const` 限定，要么类型之间能转换。一般来说，**可以把非常量拷贝给常量，但不能把常量拷贝给非常量**：

```cpp
int *p = p3;      // error: p3 has a low-level const but p doesn't
p2 = p3;          // ok: p2 has the same low-level const qualification as p3
p2 = &i;          // ok: we can convert int* to const int*
int &r = ci;      // error: can't bind a plain int& to a const int object
const int &r2 = i; // ok: can bind const int& to plain int
```

- `p3` 既有顶层 `const` 也有底层 `const`。
- 当我们拷贝 `p3` 时，可以忽略它的顶层 `const`，但是不能忽略它指向一个 `const` 类型这一事实。
- 因此，我们不能用 `p3` 来初始化 `p`，因为 `p` 指向一个普通的（非 `const`）`int`。
- 另一方面，我们可以把 `p3` 赋给 `p2`。两个指针具有相同的（底层 `const`）类型。`p3` 是 `const` 指针（即，它有顶层 `const`）这一事实无关紧要。

---

## ✅ constexpr 和常量表达式 (constexpr and Constant Expressions)

<b>常量表达式（constant expression）</b>是指值不会改变并且在**编译过程中就能得到计算结果**的表达式
- **字面值**属于常量表达式，
- const只是运行时态才能算出的常量
- 用**常量表达式**初始化的 const 对象也是常量表达式。正如我们将要看到的，在语言中有几种上下文要求用到常量表达式

一个对象（或表达式）是不是常量表达式由它的**数据类型和初始值共同决定**，例如：

```cpp
const int max_files = 20;      // max_files is a constant expression
const int limit = max_files + 1; // limit is a constant expression
int staff_size = 27;           // staff_size is not a constant expression
const int sz = get_size();     // sz is not a constant expression
```

尽管 `staff_size` 的初始值是个字面值常量，但由于它的数据类型只是一个普通 `int` 而非 `const int`，所以它不属于常量表达式。另一方面，尽管 `sz` 本身是一个常量，但它的具体值直到运行时才能获得，所以也不是常量表达式。

### 📝 constexpr 变量

在复杂系统中，我们往往难以判断一个初始值是否真的是常量表达式。虽然可以用看似“常量”的表达式初始化 `const` 变量，但在实际使用中常会发现它并不符合常量表达式的要求。换句话说，**定义时看起来没问题，使用时才发现不合规**。


C++11 新标准规定，允许将变量声明为 **constexpr 类型**以便由编译器来验证变量的值是否是一个常量表达式。声明为 constexpr 的变量一定是：
- 一个常量
- 而且必须用常量表达式初始化

```cpp
constexpr int mf = 20;         // 20 is a constant expression
constexpr int limit = mf + 1;  // mf + 1 is a constant expression
constexpr int sz = size();     // ok only if size is a constexpr function
```

尽管不能使用普通函数作为 `constexpr` 变量的初始值，但是我们将在后面看到，新标准允许定义一种特殊的 `constexpr` 函数。这种函数应该足够简单以使得编译时就可以计算其结果，这样就能用 `constexpr` 函数去初始化 `constexpr` 变量了。

#### ⭐ 最佳实践
一般来说，如果你认定变量是一个常量表达式，那就把它声明成 constexpr 类型。

### 📝 字面值类型

- 由于常量表达式必须在编译时就能求值，因此用于 `constexpr` 声明的类型受到限制。通常，这些可以用于`constexpr`的类型比较简单、值显而易见，被称为**字面类型**（literal type）


- 目前为止，我们接触过的字面值类型包括：**算术类型**、**引用和指针**。而像 `Sales_item` 类、IO 库、`string` 类型都不是字面值类型，因此不能用作 `constexpr`。

- 尽管指针和引用可以是 `constexpr`，它们的初始值仍有限制：必须是 `nullptr`、`0`，或指向固定地址的对象。

- 函数体内的普通局部变量**地址不固定**，因此不能作为 `constexpr` 指针的初始值。相反，定义在函数体外或具有静态存储期的对象地址固定，可以用来初始化 `constexpr` 指针或绑定 `constexpr` 引用。我们将在之后看到更多细节。

### 📝 指针和 constexpr

- 必须明确一点，在 `constexpr` 声明中如果定义了一个指针，限定符 `constexpr` 仅对指针有效，与指针所指的对象无关：

```cpp
const int *p = nullptr;       // p is a pointer to a const int
constexpr int *q = nullptr;   // q is a const pointer to int
```

- `p` 和 `q` 的类型相差很大，`p` 是一个指向常量的指针，而 `q` 是一个常量指针，其中关键在于 `constexpr` 把它所定义的对象置为了顶层 `const`

- 与其他常量指针类似，`constexpr` 指针既可以指向常量也可以指向一个非常量：

```cpp
constexpr int *np = nullptr;  // np is a constant pointer to int that is null
int j = 0;
constexpr int i = 42;         // type of i is const int
// i and j must be defined outside any function
constexpr const int *p = &i; // p is a constant pointer to the const int i
constexpr int *p1 = &j; // p1 is a constant pointer to the int j
```

---

## 🍃 小结

### 📚 核心知识体系

**const 限定符概念**：
- const 使对象成为常量，防止值被意外修改
- const 对象必须初始化，因为创建后值不能改变
- const 对象默认仅在文件内有效，支持编译时常量替换

**const 引用特性**：
- 对 const 的引用不能用于修改所绑定的对象
- 可以用字面值、表达式或不同类型初始化 const 引用
- 可能绑定到非 const 对象，但仍不能通过该引用修改对象

**指针与 const**：
- 指向 const 的指针：不能通过指针修改所指对象
- const 指针：指针本身是常量，不能改变指向
- 可以组合：const 指针指向 const 对象

**constexpr 和常量表达式**：
- constexpr 变量必须用常量表达式初始化
- 字面值类型包括算术类型、引用和指针
- constexpr 指针是顶层 const，可指向 const 或非 const 对象

### ⚡ 关键概念精要

**顶层 const vs 底层 const**：
- 顶层 const：对象本身是常量（适用于任何类型）
- 底层 const：指针或引用所指/引用的对象是常量
- 拷贝时：顶层 const 被忽略，底层 const 必须匹配

**类型转换规则**：
- 非常量可以转换为常量
- 常量不能转换为非常量
- const 引用可以绑定到临时对象

**文件间共享**：
- 默认 const 对象仅文件内可见
- 使用 extern 可以跨文件共享 const 对象

### 🛡️ 实践防护策略

**const 最佳实践**：
- 尽可能使用 const：提高代码安全性和可读性
- const 对象必须初始化：避免未定义行为
- 合理使用 const 引用：避免不必要的拷贝

**指针 const 使用**：
- 明确区分指向 const 的指针和 const 指针
- 理解顶层 const 和底层 const 的差异
- 正确处理 const 指针的类型转换

**constexpr 使用**：
- 优先使用 constexpr 声明常量表达式
- 理解字面值类型的限制
- 正确区分 constexpr 指针的含义

**设计原则**：
- 接口设计中合理使用 const：明确函数是否修改参数
- 成员函数的 const 正确性：区分 const 和非 const 成员函数
- 返回值的 const 设计：防止意外修改

### 🔗 学习路径指引

const 限定符为后续概念奠定基础：
- **函数**：const 参数和 const 成员函数
- **类**：const 成员变量和 const 对象
- **迭代器**：const_iterator 的使用
- **模板**：const 模板参数和 const 表达式
- **现代 C++**：constexpr 和编译时计算

掌握 const 限定符是编写安全、高效 C++ 代码的重要基础，有助于构建健壮的类型系统和接口设计。