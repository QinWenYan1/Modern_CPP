# 📘 Chapter 17 - C++ 开发工具链 (Dev Toolchain)

> C++ 工程实践必备工具  
> 本章涵盖：构建系统（Makefile/CMake）、调试器（GDB）、静态库与动态库的创建与链接

---

## 📑 章节导航

### 17.1 C++ 开发工具链 🛠️
Makefile 构建系统、CMake、GDB 调试器、静态库与动态库的创建与链接  
[→ 查看笔记](./note_17_1_dev_tools.md)

---

## 🎯 知识点索引

<details>
<summary><b>🔧 Makefile 构建系统</b></summary>

- [Makefile 基本语法](./note_17_1_dev_tools.md)
- [规则（target, prerequisite, recipe）](./note_17_1_dev_tools.md)
- [变量与自动变量](./note_17_1_dev_tools.md)
- [模式规则与隐式规则](./note_17_1_dev_tools.md)
- [多目录项目管理](./note_17_1_dev_tools.md)

</details>

<details>
<summary><b>🐛 GDB 调试器</b></summary>

- [编译时加 `-g` 选项](./note_17_1_dev_tools.md)
- [断点（breakpoint）与观察点（watchpoint）](./note_17_1_dev_tools.md)
- [单步执行（step, next, continue）](./note_17_1_dev_tools.md)
- [调用栈（backtrace, frame）](./note_17_1_dev_tools.md)
- [变量查看与内存检查](./note_17_1_dev_tools.md)
- [core dump 分析](./note_17_1_dev_tools.md)

</details>

<details>
<summary><b>📦 静态库与动态库</b></summary>

- [静态库 `.a`（Linux）/ `.lib`（Windows）](./note_17_1_dev_tools.md)
- [动态库 `.so`（Linux）/ `.dylib`（macOS）/ `.dll`（Windows）](./note_17_1_dev_tools.md)
- [`ar` 与 `ranlib` 创建静态库](./note_17_1_dev_tools.md)
- [编译时链接 vs 运行时加载（`dlopen`）](./note_17_1_dev_tools.md)
- [头文件管理与 `-I`, `-L`, `-l` 选项](./note_17_1_dev_tools.md)

</details>

<details>
<summary><b>🔨 CMake 构建系统生成器</b></summary>

- [CMake 是什么与为什么需要它](./note_17_1_dev_tools.md)
- [CMakeLists.txt 基础语法与核心指令](./note_17_1_dev_tools.md)
- [变量与缓存（`-D` 覆盖）](./note_17_1_dev_tools.md)
- [常用预定义变量清单（`PROJECT_*` / `CMAKE_CURRENT_*` / `*_OUTPUT_PATH` / `*_FLAGS`）](./note_17_1_dev_tools.md)
- [头文件搜索路径与可见性（PRIVATE/PUBLIC/INTERFACE）](./note_17_1_dev_tools.md)
- [find_package 引入第三方库](./note_17_1_dev_tools.md)
- [构建流程与构建类型（Debug/Release）](./note_17_1_dev_tools.md)
- [完整构建实例](./note_17_1_dev_tools.md)
- [进阶：构建配置与目标属性（`target_compile_options` 等）](./note_17_1_dev_tools.md)

</details>

---

## 💡 核心速查

**Makefile 基础模板**
```makefile
CXX = g++
CXXFLAGS = -std=c++17 -Wall -g
TARGET = prog
OBJS = main.o foo.o bar.o

$(TARGET): $(OBJS)
	$(CXX) $(CXXFLAGS) -o $@ $^

%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c -o $@ $<

.PHONY: clean
clean:
	rm -f $(OBJS) $(TARGET)
```

**GDB 常用命令**
```bash
g++ -g -o prog prog.cpp          # 编译时加调试信息
gdb ./prog                        # 启动 GDB
(gdb) break main                  # 在 main 处设断点
(gdb) run                         # 运行程序
(gdb) next                        # 单步（不进入函数）
(gdb) step                        # 单步（进入函数）
(gdb) print var                   # 查看变量值
(gdb) backtrace                   # 查看调用栈
(gdb) continue                    # 继续运行
```

**静态库与动态库**
```bash
# 静态库
g++ -c foo.cpp -o foo.o
ar rcs libfoo.a foo.o             # 创建静态库
g++ main.cpp -L. -lfoo -o prog    # 链接静态库

# 动态库（Linux）
g++ -fPIC -c foo.cpp -o foo.o
g++ -shared -o libfoo.so foo.o    # 创建动态库
g++ main.cpp -L. -lfoo -o prog    # 链接动态库

# 动态库（macOS）
g++ -dynamiclib -o libfoo.dylib foo.cpp
```

**CMake 构建**
```cmake
# CMakeLists.txt 最小配置
cmake_minimum_required(VERSION 3.10)
project(MyApp CXX)
add_executable(MyApp main.cpp)
target_link_libraries(MyApp PRIVATE MyLib)
```
```bash
# 标准构建流程（源外构建）
mkdir build && cd build
cmake ..                                # 配置：生成 Makefile
cmake --build .                         # 编译
cmake .. -DCMAKE_BUILD_TYPE=Release     # Release 构建
cmake .. -DMY_OPTION=ON                 # 覆盖缓存变量
```

---

## 🔗 相关链接

← [上一章：第 16 章 - 模板与泛型编程](../Chapter16_templates_and_generic_programming/README.md)  
↑ [返回根目录](../README.md)

---

> 🛠️ *"掌握工具链，才是从写代码到交付软件的关键一步。"*
