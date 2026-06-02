# 📘 Chapter 11 - 关联容器 (Associative Containers)

> 《C++ Primer》第 11 章学习笔记导航  
> 本章涵盖：map、set、unordered_map、unordered_set 与关联容器操作

---

## 📑 章节导航

### 11.1 使用关联容器 🗺️
map、set 基本使用与场景  
[→ 查看笔记](./note_11_1_using_an_associative_container.md)

### 11.2 关联容器概述 📊
关联容器类型、键类型要求、pair 类型  
[→ 查看笔记](./note_11_2_overview_of_the_associative_container.md)

### 11.3 关联容器操作 🔧
关联容器迭代器、添加元素、删除元素、下标操作  
[→ 查看笔记](./note_11_3_operations_on_associative_containers.md)

### 11.4 无序容器 🔀
无序容器管理、哈希函数、自定义哈希  
[→ 查看笔记](./note_11_4_unordered_containers.md)

### 章节总结 📝
第 11 章核心概念速查表  
[→ 查看总结](./note_11_chapter_summary.md)

### 练习题 💪
本章所有练习代码  
[→ 查看代码](./exercises/)

---

## 🎯 知识点索引

<details>
<summary><b>🗺️ 使用关联容器</b></summary>

- [map 映射](./note_11_1_using_an_associative_container.md)
- [set 集合](./note_11_1_using_an_associative_container.md)
- [单词计数示例](./note_11_1_using_an_associative_container.md)
- [忽略常见词示例](./note_11_1_using_an_associative_container.md)

</details>

<details>
<summary><b>📊 关联容器概述</b></summary>

- [有序容器 map/set/multimap/multiset](./note_11_2_overview_of_associative_containers.md)
- [无序容器 unordered_map/unordered_set](./note_11_2_overview_of_associative_containers.md)
- [键类型要求](./note_11_2_overview_of_associative_containers.md)
- [自定义比较函数](./note_11_2_overview_of_associative_containers.md)
- [pair 类型与初始化](./note_11_2_overview_of_associative_containers.md)

</details>

<details>
<summary><b>🔧 关联容器操作</b></summary>

- [关联容器迭代器](./note_11_3_operations_on_associative_containers.md)
- [insert/emplace 添加元素](./note_11_3_operations_on_associative_containers.md)
- [erase 删除元素](./note_11_3_operations_on_associative_containers.md)
- [下标操作 map[key]](./note_11_3_operations_on_associative_containers.md)
- [访问元素 find/count](./note_11_3_operations_on_associative_containers.md)
- [lower_bound/upper_bound/equal_range](./note_11_3_operations_on_associative_containers.md)

</details>

<details>
<summary><b>🔀 无序容器</b></summary>

- [哈希表实现](./note_11_4_the_unordered_containers.md)
- [无序容器性能](./note_11_4_the_unordered_containers.md)
- [桶接口 bucket](./note_11_4_the_unordered_containers.md)
- [自定义哈希函数](./note_11_4_the_unordered_containers.md)
- [自定义相等判断](./note_11_4_the_unordered_containers.md)

</details>

---

## 💡 核心速查

**关联容器类型**
```cpp
// 有序容器（基于红黑树）
map<Key, T>           // 键值对，键唯一
set<Key>             // 键集合，键唯一
multimap<Key, T>      // 键值对，键可重复
multiset<Key>        // 键集合，键可重复

// 无序容器（基于哈希表）
unordered_map<Key, T>     
unordered_set<Key>       
unordered_multimap<Key, T>
unordered_multiset<Key>
```

**map 基本操作**
```cpp
map<string, int> word_count;

// 插入
word_count.insert({"hello", 1});
word_count.insert(make_pair("world", 2));
word_count.emplace("foo", 3);

// 下标访问（不存在则插入默认值）
word_count["hello"]++;  // 访问并递增
int count = word_count["bar"];  // 不存在则插入 0

// 查找
auto it = word_count.find("hello");  // 返回迭代器或 end
size_t n = word_count.count("hello");  // 返回键出现次数（map 为 0 或 1）

// 删除
word_count.erase("hello");
word_count.erase(it);
```

**set 基本操作**
```cpp
set<int> s;

// 插入
s.insert(42);
s.emplace(10);

// 检查存在性
if (s.find(42) != s.end()) { /* 存在 */ }
if (s.count(42)) { /* 存在 */ }

// 删除
s.erase(42);
```

**pair 类型**
```cpp
// 定义与初始化
pair<string, int> p1("hello", 1);
auto p2 = make_pair("world", 2);
pair<string, int> p3 = {"foo", 3};  // C++11

// 访问
string key = p1.first;
int val = p1.second;

// 比较（先比 first，再比 second）
if (p1 < p2) { /* ... */ }
```

**无序容器自定义哈希**
```cpp
// 为自定义类型定义哈希
struct Sales_data {
    string isbn() const;
    // ...
};

namespace std {
    template<>
    struct hash<Sales_data> {
        size_t operator()(const Sales_data &s) const {
            return hash<string>()(s.isbn());
        }
    };
}
```

**范围查找（multimap/multiset）**
```cpp
multimap<string, int> mm;

// 查找所有键为 "key" 的元素
auto range = mm.equal_range("key");
for (auto it = range.first; it != range.second; ++it) {
    // 处理每个匹配元素
}

// 或使用 lower_bound/upper_bound
auto beg = mm.lower_bound("key");  // 第一个 >= key
auto end = mm.upper_bound("key");  // 第一个 > key
```

---

## 🔗 相关链接

← [上一章：第 10 章 - 泛型算法](../Chapter10_generic_algorithms/README.md)  
→ [下一章：第 12 章 - 动态内存](../Chapter12_dynamic_memory/README.md)  
↑ [返回根目录](../README.md)

---

> 🕊️ *"关联容器是数据组织和快速查找的利器，map 用于键值映射，set 用于集合运算，选择有序或无序取决于是否需要顺序遍历。"*
