# 📘 Chapter 10 - 泛型算法 (Generic Algorithms)

> 《C++ Primer》第 10 章学习笔记导航  
> 本章涵盖：标准库算法、迭代器、Lambda 表达式与函数对象

---

## 📑 章节导航

### 10.1 概述 🎯
算法与迭代器、算法永不执行容器操作  
[→ 查看笔记](./note_10_1_overview.md)

### 10.2 初识泛型算法 🔍
只读算法、写容器元素算法、重排容器元素算法  
[→ 查看笔记](./note_10_2_a_first_look_at_the_algorithms.md)

### 10.3 定制操作 ⚙️
向算法传递函数、Lambda 表达式、参数绑定  
[→ 查看笔记](./note_10_3_customizing_operations.md)

### 10.4 再探迭代器 🔄
插入迭代器、流迭代器、反向迭代器、移动迭代器  
[→ 查看笔记](./note_10_4_revisiting_iterators.md)

### 10.5 泛型算法结构 🏗️
5 类迭代器、算法形参模式、命名规范  
[→ 查看笔记](./note_10_5_structure_of_generic_algorithms.md)

### 10.6 特定容器算法 📦
list 和 forward_list 专属算法  
[→ 查看笔记](./note_10_6_container_specific_algorithms.md)

### 章节总结 📝
第 10 章核心概念速查表  
[→ 查看总结](./note_10_chapter_summary.md)

### 练习题 💪
本章所有练习代码  
[→ 查看代码](./exercises/)

---

## 🎯 知识点索引

<details>
<summary><b>🎯 概述</b></summary>

- [算法与头文件 algorithm/numeric](./note_10_1_overview.md)
- [算法不直接操作容器](./note_10_1_overview.md)
- [迭代器作为算法与容器的桥梁](./note_10_1_overview.md)

</details>

<details>
<summary><b>🔍 初识泛型算法</b></summary>

- [只读算法 find/count/accumulate/equal](./note_10_2_a_first_look_at_the_algorithms.md)
- [写元素算法 fill/fill_n/copy/transform](./note_10_2_a_first_look_at_the_algorithms.md)
- [重排算法 sort/unique/remove_if](./note_10_2_a_first_look_at_the_algorithms.md)

</details>

<details>
<summary><b>⚙️ 定制操作</b></summary>

- [向算法传递函数](./note_10_3_customizing_operations.md)
- [Lambda 表达式 [capture](params){body}](./note_10_3_customizing_operations.md)
- [Lambda 捕获列表 [=]/[&]/[this]](./note_10_3_customizing_operations.md)
- [参数绑定 bind](./note_10_3_customizing_operations.md)
- [ref/cref 引用包装器](./note_10_3_customizing_operations.md)

</details>

<details>
<summary><b>🔄 再探迭代器</b></summary>

- [插入迭代器 back_inserter/front_inserter/inserter](./note_10_4_revisiting_iterators.md)
- [流迭代器 istream_iterator/ostream_iterator](./note_10_4_revisiting_iterators.md)
- [反向迭代器 reverse_iterator](./note_10_4_revisiting_iterators.md)

</details>

<details>
<summary><b>🏗️ 泛型算法结构</b></summary>

- [5 类迭代器](./note_10_5_structure_of_generic_algorithms.md)
- [输入/输出迭代器](./note_10_5_structure_of_generic_algorithms.md)
- [前向/双向/随机访问迭代器](./note_10_5_structure_of_generic_algorithms.md)
- [算法命名规范 _if/_copy](./note_10_5_structure_of_generic_algorithms.md)

</details>

<details>
<summary><b>📦 特定容器算法</b></summary>

- [list::sort/merge/remove/reverse/unique](./note_10_6_algorithms_specific_to_containers.md)
- [forward_list 算法](./note_10_6_algorithms_specific_to_containers.md)
- [为什么 list 需要专属算法](./note_10_6_algorithms_specific_to_containers.md)

</details>

---

## 💡 核心速查

**常用算法**
```cpp
#include <algorithm>
#include <numeric>

// 查找
auto it = find(vec.begin(), vec.end(), value);
bool exists = binary_search(vec.begin(), vec.end(), value);  // 要求有序

// 排序
sort(vec.begin(), vec.end());                          // 升序
sort(vec.begin(), vec.end(), greater<int>());          // 降序
stable_sort(vec.begin(), vec.end(), cmp);              // 稳定排序

// 数值运算
int sum = accumulate(vec.begin(), vec.end(), 0);       // 求和
inner_product(a.begin(), a.end(), b.begin(), 0);       // 内积

// 复制与替换
copy(src.begin(), src.end(), dest.begin());
replace(vec.begin(), vec.end(), old_val, new_val);
transform(src.begin(), src.end(), dest.begin(), op);

// 删除（实际上是将不删除的元素前移）
auto end_unique = unique(vec.begin(), vec.end());
vec.erase(end_unique, vec.end());  // 真正删除
```

**Lambda 表达式**
```cpp
// 基本形式 [capture](parameters) -> return_type { body }

// 示例
auto f = [](int a, int b) { return a + b; };
int sum = f(1, 2);  // 3

// 捕获列表
[=]     // 隐式值捕获所有变量
[&]     // 隐式引用捕获所有变量
[=, &x]  // 值捕获所有，x 引用捕获
[&, x]  // 引用捕获所有，x 值捕获
[this]  // 捕获当前对象指针

// 使用示例
int threshold = 10;
count_if(vec.begin(), vec.end(), 
         [threshold](int x) { return x > threshold; });
```

**插入迭代器**
```cpp
#include <iterator>

vector<int> vec;
auto it = back_inserter(vec);  // 插入迭代器
*it = 42;  // 等价于 vec.push_back(42)

// 配合算法使用
fill_n(back_inserter(vec), 10, 0);  // 添加 10 个 0
```

**流迭代器**
```cpp
// 输入流迭代器
istream_iterator<int> in(cin), eof;
vector<int> vec(in, eof);  // 从 cin 读取 int 直到 EOF

// 输出流迭代器
ostream_iterator<int> out(cout, " ");
copy(vec.begin(), vec.end(), out);  // 输出用空格分隔
```

**bind 绑定**
```cpp
#include <functional>
using namespace std::placeholders;

// 将函数与参数绑定
auto check_size = bind(
    [](const string &s, int sz) { return s.size() < sz; },
    _1, 6  // 第一个参数占位，第二个固定为 6
);

// 配合算法
count_if(words.begin(), words.end(), check_size);
```

---

## 🔗 相关链接

← [上一章：第 9 章 - 顺序容器](../Chapter09_sequential_containers/README.md)  
→ [下一章：第 11 章 - 关联容器](../Chapter11_associative_containers/README.md)  
↑ [返回根目录](../README.md)

---

> 🕊️ *"泛型算法是 C++ 标准库的精华，配合 Lambda 和迭代器，可以编写简洁而高效的代码。"*
