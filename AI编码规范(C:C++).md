# C++ AI 编码规范

**版本**: 4.5.0  **日期**: 2026-06-24

> **适用范围**：所有 C++ 新代码；修改老代码时遵循最小变更原则（见 26.3），不强制重构未触及的代码
> **使用方式**：AI 应在编码前全文加载本规范；遇到规则冲突时按下方"优先级"裁决
> **优先级**（高→低）：安全规范（19）> 正确性（14）> 可读性（3）> 性能（13）> 风格（1-2）
> **与其他规范关系**：本规范为 C++ 专用；通用规则见《通用 AI 编码规范》；两者冲突时以本规范为准
> **版本规则**：遵循语义化版本（SemVer）

---
## 1. 代码格式

### 1.1 缩进
使用 4 空格缩进，**严禁**使用 Tab。

### 1.2 大括号与代码块格式
- 类、函数、控制语句、命名空间：大括号与关键字**必须**在同一行
- **禁止**大括号单独占一行

### 1.3 头文件引用顺序
1. 本文件对应实现文件（`.cc` / `.cpp`）
2. C 标准库
3. C++ 标准库
4. 第三方库
5. 项目内部头文件

组间**必须**用空行分隔。

### 1.4 文件编码与换行
- **必须**使用 UTF-8 无 BOM 编码
- **必须**使用 LF（`\n`）换行符，**严禁**使用 CRLF

## 2. 命名规范

### 2.1 类型命名

| 类型 | 后缀 | 示例 |
|------|------|------|
| class | `_t` | `manager_t` |
| struct | `_t` | `data_t` |
| enum | `_t` | `state_t` |
| using | `_t` | `id_t` |
| 接口 | `i_` 前缀 + `_t` | `i_handler_t` |

**必须** struct、enum、using 使用 `_t` 后缀。
**禁止**使用 `typedef`，统一使用 `using`（与 7.1 保持一致）。

### 2.2 枚举类型
- **必须**使用 `enum class`（scoped enum），**禁止**裸 `enum`
- 枚举值命名格式为 `类型_t::值`，**禁止**大写命名
- 示例：`state_t::running`、`color_t::dark_red`

### 2.3 变量命名
- **必须**使用下划线分隔的小写字母：`user_name`, `is_valid`
- **禁止**驼峰命名：`userName`, `bValid`

### 2.4 私有成员变量
- **必须**在名称末尾添加下划线 `_`
- **禁止**匈牙利命名、无后缀、下划线开头

### 2.5 私有成员函数
- **必须**使用下划线分隔的小写字母
- **建议**使用单下划线后缀 `_` 与私有成员变量风格保持一致（如 `init_()`）
- **禁止**使用双下划线前缀 `__`（C++ 保留标识符，使用属未定义行为）
- **禁止**使用单下划线前缀 `_`（避免与保留标识符边界混淆，且与成员变量后缀风格冲突）

### 2.6 常量命名
- **必须**使用全大写 + 下划线：`MAX_BUFFER_SIZE`
- **禁止**小写、k前缀

### 2.7 全局变量
- **必须**添加 `global_` 前缀
- **禁止** g_ 前缀或其他形式

### 2.8 命名空间
- **必须**使用小写字母

### 2.9 函数命名
- **必须**使用下划线分隔的小写字母
- 可保留 get/set 前缀

## 3. 注释规范

### 3.1 文件头注释
所有头文件**必须**包含文件头注释。

### 3.2 函数注释
- **所有函数必须包含注释**
- 重要函数**必须**详细说明：功能、参数、返回值、实现思路、使用示例
- **禁止**无注释或注释不完整

### 3.3 类注释
- 重要类**必须**包含注释，说明功能、设计思路、使用场景

## 4. 类结构规范

**必须**按以下顺序组织：
1. public 类型定义
2. protected 类型定义
3. private 成员变量
4. protected 成员变量
5. public 成员变量（谨慎使用）
6. private 成员函数（单下划线后缀 `_`，详见 2.5）
7. public 成员函数
8. 静态函数

## 5. 头文件规范

| 文件类型 | 后缀 | 用途 |
|----------|------|------|
| 头文件 | `.h` | 声明 |
| 源文件 | `.cc` / `.cpp` | 实现 |
| 头源合一 | `.hpp` | 声明+实现 |
| 模板实现 | `.inl` | 模板实现 |

- **必须**使用 `#pragma once`
- **必须**使用前向声明减少依赖

## 6. 接口设计规范

- **必须**使用 `i_` 前缀
- **必须**包含虚析构函数

## 7. 函数规范

### 7.1 关键规则
- 单参数构造函数**必须**使用 `explicit`
- 不抛异常的函数**必须**标记 `noexcept`
- 赋值运算符**必须**返回自身引用
- **禁止**过度使用 `inline`
- **必须**使用 `nullptr`，**严禁**使用 `NULL` 或 `0`
- **优先**使用列表初始化 `{}`；当 `{}` 会改变语义时（如 `std::vector<int> v(10, 0)` 计数构造）**必须**使用 `()`

  **正例**：
  ```cpp
  std::vector<int> v1{1, 2, 3};    // 列表初始化，3 个元素
  std::vector<int> v2(10, 0);      // 计数构造，10 个 0
  int x{42};                        // 禁窄化转换
  ```

  **反例**：
  ```cpp
  std::vector<int> v{10, 0};       // 错！变成 2 个元素 {10, 0}
  int x = 3.14;                    // 错！隐式窄化
  ```
- **必须**使用 `using`，**禁止** `typedef`
- **必须**显式声明特殊成员函数：优先 Rule of 0（`= default` 或不声明由编译器隐式生成）；一旦自定义任一特殊成员函数，则**必须**按 Rule of 5 显式声明全部（`= default` / `= delete` / 自定义）
- **必须**在成员变量声明处直接初始化（默认成员初始化），**禁止**在构造函数体中对成员做简单赋值初始化

### 7.2 lambda 表达式
- **必须**显式捕获需要的变量
- **禁止**捕获过多变量

### 7.3 函数设计规则
- 函数长度**建议**不超过 50 行；超过**必须**考虑拆分（单一职责，见第24条）
- 函数参数**建议**不超过 4 个；超过**必须**考虑封装为结构体或使用 builder 模式
- **必须**使用提前返回（guard clause）简化嵌套：

  **正例**：
  ```cpp
  bool process(const input_t& in) {
      if (!in.is_valid()) { return false; }   // 提前返回
      if (in.is_empty()) { return false; }
      // 主逻辑
      return true;
  }
  ```

  **反例**：
  ```cpp
  bool process(const input_t& in) {
      if (in.is_valid()) {
          if (!in.is_empty()) {
              // 深层嵌套
              return true;
          }
      }
      return false;
  }
  ```
- **必须**优先使用纯函数（无副作用）；成员函数 `const` 修饰见第12条
- **禁止**输出参数（非 const 引用作为返回值），**必须**使用返回值或 `std::optional` / `std::expected`

## 8. 移动语义

- **必须**遵循 Rule of 0/3/5：
  - **优先 Rule of 0**：类不声明任何特殊成员函数，依赖编译器隐式生成正确实现
  - **Rule of 3**：若自定义拷贝构造、拷贝赋值、析构之一，则**必须**三者都声明
  - **Rule of 5**：若涉及资源所有权，则**必须**显式声明全部五个特殊成员函数（拷贝构造/拷贝赋值/移动构造/移动赋值/析构）
- 移动构造函数和赋值运算符**必须**标记 `noexcept`
- **必须**正确使用参数传递规则：
  - 内置类型/小型结构：按值传递
  - 大型对象：const 引用
  - 需要修改：非const 引用
  - 需要移动：右值引用

## 9. 命名空间

- **禁止**嵌套过深（建议最多2-3层）
- **严禁**在头文件中使用 `using namespace`
- **禁止**在源文件中使用 `using namespace std`
- 文件内部符号**必须**放在匿名命名空间

## 10. 类型推导与现代特性

### 10.1 类型推导
- **必须**明确使用 `auto`，避免隐式类型推导
- `decltype` **必须**用于需要类型信息时
- 模板函数**必须**使用尾返回类型或 `decltype`
- `auto` 变量**必须**在声明时初始化
- **禁止** `auto` 的以下反模式：

  **反例**：
  ```cpp
  auto x = {1, 2, 3};           // 错！推导为 std::initializer_list<int>
  auto p = new widget_t();      // 错！推导为 widget_t*，应避免裸 new（见第17条）
  ```

### 10.2 结构化绑定与 if constexpr
- **必须**使用结构化绑定解构多返回值（替代 `std::tie`）：

  ```cpp
  auto [key, value] = *map_iter;            // 解构 map 节点
  auto [x, y, z] = get_position();          // 解构多返回值
  ```
- **必须**使用 `if constexpr` 替代 SFINAE 实现编译期分支：

  ```cpp
  template<typename T>
  void process(T&& v) {
      if constexpr (std::is_integral_v<T>) {
          // 整型路径
      } else if constexpr (std::is_floating_point_v<T>) {
          // 浮点路径
      }
  }
  ```

### 10.3 concept 与约束（C++20）
- 模板约束**必须**优先使用 `concept` 替代 SFINAE / `static_assert`：

  **正例**：
  ```cpp
  template<typename T>
      requires std::integral<T>
  T add(T a, T b) { return a + b; }

  // 或简写形式
  template<std::integral T>
  T add(T a, T b) { return a + b; }
  ```

  **反例**：
  ```cpp
  template<typename T, typename = std::enable_if_t<std::is_integral_v<T>>>
  T add(T a, T b) { return a + b; }   // 错！过时的 SFINAE 写法
  ```
- 自定义 concept **必须**使用 `_t` 后缀以外的命名（concept 是编译期谓词，不加 `_t`）：`template<typename T> concept hashable = ...;`

### 10.4 属性（Attributes）
- 返回值不应被忽略的函数**必须**标记 `[[nodiscard]]`：
  ```cpp
  [[nodiscard]] bool is_valid() const;
  [[nodiscard]] std::unique_ptr<widget_t> create();
  ```
- 暂未使用的变量/参数**必须**标记 `[[maybe_unused]]`，**禁止**用 `(void)` 强转：
  ```cpp
  void on_event([[maybe_unused]] const event_t& e) { ... }
  ```
- **禁止**使用 `[[deprecated]]` 之外的编译器扩展属性（如 `__attribute__((...))`），**必须**使用标准属性

## 11. 模板规范

- 模板参数**必须**使用有意义的名称：`template<typename T>`
- **必须**将模板实现放在 `.inl` 文件，由对应 `.h` 在末尾 `#include`（模板实现必须对使用方可见，否则会链接失败）
- 模板特化**必须**明确标注
- **禁止**将模板实现放入 `.cpp`（除非使用显式实例化）
- **禁止**在头文件中直接定义模板实现（应放入 `.inl`）

## 12. 函数修饰符

- 重写虚函数**必须**使用 `override`
- `const` 不需要修改成员函数**必须**标记 `const`
- `constexpr` 函数**必须**正确使用

## 13. 性能与最佳实践

### 13.1 通用规则
- **禁止**不必要拷贝，优先移动
- **禁止**在构造函数中做复杂操作
- **必须**使用 `std::optional` 表示可选值

### 13.2 返回值优化（RVO/NRVO）
- **必须**按值返回局部对象，依赖 RVO/NRVO，**禁止**返回 `std::move` 局部对象（会禁用 NRVO）：

  **正例**：
  ```cpp
  widget_t make_widget() {
      widget_t w;          // 局部对象
      w.configure();
      return w;            // NRVO 自动生效
  }
  ```

  **反例**：
  ```cpp
  widget_t make_widget() {
      widget_t w;
      return std::move(w); // 错！禁用 NRVO，强制移动
  }
  ```

### 13.3 容器使用
- **必须**在已知元素数量时调用 `reserve()` 预分配，避免多次重分配：
  ```cpp
  std::vector<int> v;
  v.reserve(n);            // 预分配
  for (int i = 0; i < n; ++i) { v.push_back(i); }
  ```
- **必须**优先使用 `emplace_back` / `emplace` 替代 `push_back` / `insert`，避免临时对象：
  ```cpp
  v.emplace_back(42, "name");   // 直接构造
  v.push_back(widget_t(42, "name")); // 错！多一次移动/拷贝
  ```
- **禁止**对顺序容器使用 `std::list`（缓存不友好），**必须**优先 `std::vector` / `std::deque`
- 容器选择详见**附录 B：容器选择决策树**

## 14. 错误处理

- **必须**默认使用错误码 / `std::optional` / `std::expected` 方案（与 noexcept 安全风格一致）
- **仅在库边界且必要时**可使用异常，且**必须**捕获具体异常类型，**禁止**裸 `try-catch`
- **禁止**在构造函数中抛出异常（使用工厂模式 + `std::optional` / `std::expected` 返回结果）
- **必须**在析构函数中处理清理工作，**禁止**抛出异常
- **必须**使用 `std::optional` 或 `std::expected` 表示可能失败的操作
- 日志记录**必须**包含错误上下文信息

**正例**：
```cpp
// 用 std::expected 表示可能失败的操作
std::expected<result_t, error_t> parse(std::string_view input);

// 库边界捕获具体异常类型
try {
    lib_call();
} catch (const std::filesystem::filesystem_error& e) {
    log_error(e.what());
    return std::unexpected(error_t::io_failure);
}
```

**反例**：
```cpp
try {
    lib_call();
} catch (...) {                    // 错！裸 catch-all
    // 错！默默吞掉异常
}
catch (const std::exception& e) {  // 错！过宽基类
    // ...
}
```

## 15. 日志规范

- **必须**使用统一日志系统，**禁止**直接使用 `cout` / `printf`
- 日志级别：`DEBUG` / `INFO` / `WARN` / `ERROR` / `FATAL`
- **禁止**在日志中输出敏感信息（密码、密钥等）
- 日志格式**必须**包含：时间戳、文件名、行号、日志级别、线程ID
- **禁止**在高频循环中输出日志

## 16. 并发规范

### 16.1 锁与同步原语
- **必须**使用 RAII 管理锁（`std::lock_guard` / `std::unique_lock` / `std::scoped_lock`）
- **禁止**在持锁期间执行耗时操作
- **必须**避免死锁：**禁止**嵌套锁，**必须**按固定顺序获取锁
- 需要同时获取多把锁时，**必须**使用 `std::scoped_lock`（自动死锁避免）：
  ```cpp
  std::scoped_lock lk(mutex_a_, mutex_b_);   // 原子获取多把锁
  ```
- 读多写少场景**必须**使用 `std::shared_mutex` + `std::shared_lock`：
  ```cpp
  mutable std::shared_mutex mutex_;
  // 读端
  std::shared_lock lk(mutex_);
  // 写端
  std::unique_lock lk(mutex_);
  ```
- **禁止**使用 `std::recursive_mutex`（设计气味，通常说明锁粒度有问题）

### 16.2 线程管理
- **必须**使用 `std::jthread` 替代 `std::thread`（C++20，自动 join + 支持取消）：
  ```cpp
  std::jthread worker([this](std::stop_token st) {
      while (!st.stop_requested()) { /* ... */ }
  });
  ```
- **禁止**使用裸 `std::thread`（忘记 join/detach 会导致 terminate）

### 16.3 原子操作与内存序
- 共享数据**必须**使用原子操作或锁保护
- **必须**使用 `std::atomic` 替代简单标志位的 volatile
- **必须**正确选择内存序：
  - 默认 `memory_order_seq_cst`（顺序一致，最安全）
  - 仅在性能剖析确认需要时使用 `memory_order_acquire` / `memory_order_release` / `memory_order_relaxed`
  - **禁止**无依据地使用 `memory_order_relaxed`

### 16.4 线程局部存储与通信
- 线程私有状态**必须**使用 `thread_local`：
  ```cpp
  thread_local context_t tls_ctx;
  ```
- 线程间通信**必须**使用消息队列、条件变量或 `std::promise` / `std::future`
- **禁止**使用条件变量的虚假唤醒：**必须**用谓词形式 `wait(lk, predicate)`：
  ```cpp
  std::unique_lock lk(mutex_);
  cv_.wait(lk, [this] { return ready_ || cancelled_; });  // 谓词形式
  ```

## 17. 内存管理

- **必须**遵循 RAII 原则
- **禁止**使用裸 `new` / `delete`，**必须**使用智能指针管理
- `new` **必须**立即交给智能指针管理
- **禁止**在类的构造函数中使用 `new` 分配数组
- 内存分配**必须**检查是否成功
- **必须**使用内存池处理高频分配场景

**智能指针选择决策**：
```
需要共享所有权？
├─ 是 → std::shared_ptr<T>（通过 std::make_shared 创建）
└─ 否 → 需要独占所有权？
        ├─ 是 → std::unique_ptr<T>（通过 std::make_unique 创建）
        └─ 否 → 用栈对象或值语义
```

**正例**：
```cpp
auto p = std::make_unique<widget_t>();      // 独占所有权
auto s = std::make_shared<config_t>();      // 共享所有权
```

**反例**：
```cpp
widget_t* p = new widget_t();                // 错！裸 new
std::unique_ptr<widget_t> p(new widget_t()); // 错！未用 make_unique
delete p;                                    // 错！裸 delete
```

## 18. 宏与预处理器规范

- **优先**使用 `const` / `constexpr` / `enum` 替代宏定义常量
- **优先**使用 `inline` / `constexpr` 函数替代宏定义函数
- **必须**使用全大写 + 下划线命名宏：`MAX_ITERATIONS`
- **必须**使用 `do { ... } while(0)` 包裹多语句宏
- **禁止**使用宏实现模板或泛型逻辑

## 19. 安全规范

- **禁止**使用 `strcpy` / `sprintf` / `strncpy`（`strncpy` 不保证 null 终止且有截断问题）
- **优先**使用 `std::string`、`std::string_view`、`std::format`；C 接口处**必须**使用 `snprintf`
- **禁止**缓存溢出，**必须**使用安全的字符串处理函数
- 输入数据**必须**进行有效性校验
- **禁止**在代码中硬编码密钥、密码
- **必须**使用 `sizeof(buffer)` 而非硬编码大小
- **禁止**使用 `rand` 生成安全随机数，**必须**使用 `std::random_device`

### 19.1 `std::string_view` 生命周期安全
- `std::string_view` **必须**视为非所有权视图，**禁止**持有超出源字符串生命周期的 view：

  **正例**：
  ```cpp
  void process(std::string_view sv);             // 函数参数：安全
  std::string s = make_string();
  std::string_view sv = s;                        // 同作用域：安全
  ```

  **反例**：
  ```cpp
  std::string_view get_view() {
      std::string s = make_temp();                // 临时对象
      return s;                                   // 错！返回指向已销毁对象的 view
  }
  std::string_view sv = "hello"s;                 // 错！临时 string 销毁，view 悬垂
  ```
- 类成员**禁止**使用 `std::string_view` 存储字符串，**必须**使用 `std::string`
- **必须**使用 `std::span` 替代 `const T*` + `size_t` 的连续内存参数组合：
  ```cpp
  void process(std::span<const int> data);       // 替代 (const int* p, size_t n)
  ```

---

## 20. 测试规范

- **必须**为新功能编写单元测试
- **必须**保证测试可重复执行
- **必须**使用 mock 对象隔离依赖
- 测试用例**必须**覆盖正常和异常路径
- **禁止**在测试中使用 `sleep` 等待异步完成，应使用条件变量或 `future` 等同步机制

## 21. 编译构建

- **必须**明确指定 C++ 标准（建议 C++20 起步，最低 C++17）
- **必须**使用 CMake 管理项目
- **必须**设置编译选项：`-Wall` / `-Wextra` / `-Wpedantic` / `-Werror`
- **必须**设置调试和发布两种构建配置
- **禁止**修改第三方库源码，**必须**使用补丁或封装
- **必须**在提交前执行完整构建和测试

## 22. 设计模式

- **必须**根据场景选择合适模式，**禁止**过度设计
- 单例模式**必须**使用 `std::call_once` + `std::once_flag`
- 工厂模式**必须**封装对象创建逻辑，隐藏具体类型（对象复用属对象池模式职责，不在此要求）
- **必须**优先使用组合而非继承
- **禁止**在模板中过度使用类型擦除

## 23. Git 规范

- 提交信息**必须**遵循以下格式：

  ```
  [类型]: 简短描述
  - 变更描述01
  - 变更描述02
  ...
  ```

  - 类型：`feat` / `fix` / `docs` / `style` / `refactor` / `test` / `chore`
  - 简短描述**必须**概括本次提交核心目的
  - 变更描述**必须**列出具体改动点，每条以 `- ` 开头
  - 变更描述**必须**说明"改了什么"及"为什么改"
- 分支命名**必须**遵循：`类型/功能描述`
- **必须**在合并前进行代码 review
- **禁止**提交编译产物和临时文件

## 24. 开发七大守则

1. **单一职责**：每个类/函数**必须**只做一件事
2. **开闭原则**：**必须**对扩展开放，对修改关闭
3. **里氏替换**：子类**必须**能替换基类
4. **依赖倒置**：**必须**依赖抽象，而非具体实现
5. **接口隔离**：**必须**保持接口精简，**禁止**庞大接口
6. **DRY 原则**：**禁止**重复代码，**必须**提取公共逻辑
7. **KISS 原则**：**禁止**过度复杂，**必须**保持简单直接

## 25. 文档遵循与建议

1. **必须遵循文档内容**：所有编码实践**必须**严格遵循本规范，**禁止**添油加醋
2. **可以给出建议**：在完成基础要求后，可以提出合理的改进建议供用户参考

## 26. AI 协作规范

### 26.1 需求确认原则
- **必须**在遇到不确定问题时直接向用户询问
- **严禁**基于假设进行实现
- **严禁**默认推断用户需求，所有需求必须以用户明确说明为准

### 26.2 简洁实现原则
- **必须**使用最少代码解决问题，50 行能完成则不使用 200 行
- **禁止**编写未来可能需要的功能（YAGNI 原则）
- 通用 KISS/DRY 原则详见第 24 条，此处不重复

### 26.3 最小变更原则
- **只修改**与任务直接相关的代码
- **严禁**修改无关代码
- **严禁**顺手优化或重构无关代码
- **必须**对每一行代码改动解释修改原因
- **可以**提出优化建议，但**不得**擅自实施

### 26.4 可验证交付原则
- **必须**将任务转化为可验证的具体结果
- **必须**通过实际结果判断任务完成度
- **严禁**仅凭感觉或主观判断认定任务完成
- **必须**提供明确的验证方式或测试用例

### 26.5 文档同步原则
- **必须**在代码更改且通过验证等所有环节后，同步更新相关的技术或维护文档
- 若该项变更不涉及任何现有文档，且无需新建文档，可以忽略此项要求

### 26.6 AI 反模式清单（禁止行为）

以下为 AI 编码时高频出现的错误，**必须**避免：

- **禁止**顺手重构无关代码（即使"看起来不顺眼"）
- **禁止**为未来需求设计抽象（YAGNI）
- **禁止**添加未要求的错误处理、日志、注释、类型标注
- **禁止**修改未触及代码的格式或命名
- **禁止**在修复 bug 时"顺便"优化周边代码
- **禁止**使用双下划线 `__` 前缀（C++ 保留标识符，UB）
- **禁止**捕获过宽异常类型（如 `catch(...)` / `catch(std::exception&)`）
- **禁止**使用裸 `new` / `delete`
- **禁止**使用 `typedef`（必须用 `using`）
- **禁止**使用 `NULL` 或 `0` 表示空指针（必须用 `nullptr`）
- **禁止**使用 `enum`（必须用 `enum class`）
- **禁止**在头文件中使用 `using namespace`
- **禁止**在源文件中使用 `using namespace std`
- **禁止**使用 `strcpy` / `sprintf` / `strncpy`
- **禁止**使用 `rand` 生成安全随机数

---

## 附录 A：关键词速查索引

| 关键词 | 相关章节 |
|--------|---------|
| 智能指针选择 / unique_ptr / shared_ptr | 17 |
| 错误处理 / 异常 vs 错误码 | 14 |
| 模板实现放哪里 / .inl | 11 |
| 私有成员命名（变量/函数） | 2.4, 2.5 |
| Rule of 0/3/5 | 7.1, 8 |
| 列表初始化 / 圆括号初始化 | 7.1 |
| enum class | 2.2 |
| 双下划线 / 保留标识符 | 2.5, 26.6 |
| noexcept | 7.1, 8 |
| override / const / constexpr | 12 |
| using namespace | 9 |
| 头文件引用顺序 | 1.3 |
| 类成员组织顺序 | 4 |
| 接口设计 / i_ 前缀 | 6 |
| 并发 / 锁 / 死锁 | 16 |
| jthread / shared_mutex / scoped_lock | 16 |
| 内存序 / memory_order | 16.3 |
| thread_local | 16.4 |
| RAII | 16, 17 |
| 日志格式 | 15 |
| 安全字符串 / strcpy / strncpy | 19 |
| string_view 生命周期 / 悬垂 | 19.1 |
| span / 连续内存参数 | 19.1 |
| concept / C++20 约束 | 10.3 |
| [[nodiscard]] / 属性 | 10.4 |
| 结构化绑定 / if constexpr | 10.2 |
| auto 反模式 | 10.1 |
| RVO / NRVO / 返回值优化 | 13.2 |
| 容器选择 / vector / list / map | 13.3, 附录 B |
| reserve / emplace_back | 13.3 |
| 函数设计 / 长度 / 参数 / 提前返回 | 7.3 |
| C++ 标准版本 / CMake / 编译选项 | 21 |
| 单例模式 | 22 |
| 工厂模式 | 22 |
| Git 提交格式 | 23 |
| AI 反模式 / 禁止行为 | 26.6 |
| 最小变更 / 顺手重构 | 26.3, 26.6 |
| YAGNI / KISS / DRY | 24, 26.2 |

---

## 附录 B：容器选择决策树

```
需要存储键值对？
├─ 是 → 需要有序遍历？
│       ├─ 是 → std::map / std::set（红黑树，O(log n)）
│       └─ 否 → std::unordered_map / std::unordered_set（哈希，O(1) 平均）
└─ 否 → 需要顺序访问？
        ├─ 是 → 需要头尾高效增删？
        │       ├─ 是 → std::deque（双端队列）
        │       └─ 否 → std::vector（默认首选，缓存友好）
        └─ 否 → 需要频繁中间增删？
                ├─ 是 → 重新评估设计（优先 vector + erase/remove）
                └─ 否 → std::vector
```

**容器选择原则**：
- **默认首选** `std::vector`：缓存友好、内存连续、随机访问 O(1)
- **禁止**默认使用 `std::list`：缓存不友好、内存碎片、随机访问 O(n)
- **仅当**频繁中间增删**且**不需要随机访问时才考虑 `std::list`
- 字符串容器**必须**使用 `std::vector<std::string>` 或 `std::string`，**禁止** `const char*` 容器

**常见场景对照**：

| 场景 | 推荐容器 | 原因 |
|------|---------|------|
| 动态数组 | `std::vector` | 默认首选 |
| 固定大小数组 | `std::array` | 栈分配，无开销 |
| 哈希查找 | `std::unordered_map` | O(1) 查找 |
| 有序遍历 | `std::map` | O(log n)，按键排序 |
| 栈结构 | `std::vector` + `push_back`/`pop_back` | 比适配器更灵活 |
| 队列结构 | `std::deque` | 头尾 O(1) |
| 优先队列 | `std::priority_queue` | 堆实现 |
