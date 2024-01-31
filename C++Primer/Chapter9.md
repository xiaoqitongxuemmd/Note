## Chapter 9 顺序容器

- emplace_front & emplace & emplace_back

​	这些操作与push_front & insert & push_back的区别在于上述函数执行构造，下面的函数执行拷贝。即emplace的相关函数允许如下操作：
```cpp
v.emplace_back(a, b) // a, b is the argument of constructor.
```

> argument & parameter 区别在于 argument 为实参，paramenter 为形参。

- auto 只能推导出不加 const 与引用的数据类型，而 decltype 可以识别 const 与引用***（AutoAndDecltype）***

​	因此在使用 auto 定义 const 与引用类型时需要显式声明如下：

```cpp
int a = 0;
const auto &cra = a;
```

- 如果需要同时遍历容器并增删内容时，可以使用迭代器。**容器操作可能使迭代器失效，需要在每层循环中都更新迭代器，而非预先保存迭代器**。
- forward_list 为单向链表，增删操作与常规顺序容器不同

​	forward_list 仅提供 insert_after、emplace_after、erase_after 操作。

- vector 与 string 提供了一些容器大小管理操作

  - **shrink_to_fit** 含以上为capacity 减小为与 size 一致，但标准库实现上不一定会归还内存，即 shrink_to_fit 可能不会减小 capacity。***(TestShrinkToFit)***

    本地尝试后发现策略是**会将 capacity 减小为与 size 一致**。

    ```cpp
    void TestShrinkToFit() {
        vector<int> v;
        for (vector<int>::size_type i = 0; i < 17; ++i) {
            v.push_back(i);
        }
        cout << "Capacity before shrink_to_fit: " << v.capacity() << endl;
        v.shrink_to_fit();
        cout << "Capacity after shrink_to_fit: " << v.capacity() << endl;
    }
    
    Result:
    Capacity before shrink_to_fit: 19
    Capacity after shrink_to_fit: 17
    ```

  - **capacity** 不重新分配内存的条件下，可以保存的元素数量。

  - **reserve(n)** 分配至少能容纳n个元素的内存空间，但是当 n < capacity 时不会归还空间，即reserve不会减少内存空间占用。

- 构造 string 的字符数组必须以 '\0' 结尾，或者给构造函数传入一个计数值。

- string 支持 substr 来获取子串。

- assign 主要是将一个容器中的元素全部复制到另一个容器中，**会清除掉这个容器之前的内容**，函数原型：***(TestAssign)***

> assign 在计算机科学中可以翻译为：（在计算机内存中）为......赋值。

- string 支持 append 与 replace 函数
  - **s.append(args)** 将 args 追加到 s 并返回一个指向 s 的引用。
  - **s.replace(range, args)** 删除 range 内的字符，替换为 args 指定的字符。range 为一个下标和一个长度，或者一对指向 s 的迭代器，返回指向 s 的引用。
- assign 总是替换 string 中的所有内容，append 总是在 string 的末尾添加。
- istream_iterator 为读取元素的迭代器
  - 可以从输入流（如cin）中读取连续的元素。
  - 如果定义 istream_iterator 时不指定 istream 对象，则代表了 end-of-stream。
- string 的搜索函数
  - **s.find(args)** 查找 s 中 args 第一次出现的位置。
  - **s.rfind(args)** 查找 s 中 args 最后一次出现的位置。
  - **s.find_first_of(args)** 在 s 中查找 args 中任何一个字符第一次出现的位置。
  - **s.find_last_of(args)** 在 s 中查找 args 中任何一个字符最后一次出现的位置。
  - **s.find_first_not_of(args)** 在 s 中查找第一个不在 args 中的字符。
  - **s.find_last_not_of(args)** 在 s 中查找最后一个不在args中的字符。
- string 的数值转换
  - 整数转换为字符表示形式 to_string
  - 字符转换为数字（包括不同类型的整数与浮点数）：stoi、stol、stoul、stoll、stoull、stof、stod、stold
- **适配器（adaptor）**是标准库中的一个通用概念，本质上一个适配器是一种机制，能使某种事物的行为看起来像另外一种事物一样。标准库定义了三个顺序容器适配器：stack，queue 和 priority_queue。例如 stack 适配器接受一个顺序容器（除 array 和 forward_list 外），并使其操作起来像一个 stack 一样。

```cpp
stack<string vector<string>> // Implement a empty stack based on vector.
```

由于不同适配器需要的操作不同，因此可以用来构造适配的类型也不同。