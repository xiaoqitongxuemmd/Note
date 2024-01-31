## Chapter 11 使用关联容器

- 关联容器和顺序容器有着根本的不同，关联容器中的元素是按关键字来保存和访问的。关联容器支持高效的关键字查找和访问。两个主要的关联容器 map 和 set

- 标准库提供了8个关联容器，体现在三个维度

  - 或者是一个 set，或者是一个 map

  - 或者要求不重复的关键字，或者允许重复关键字

  - 按照顺序保存，或者无需保存

​	multimap 和 map 定义在头文件 map 中， set 和 multiset 定义在头文件 set 中，无序容器则定义在头文件 unordered_map 和 unordered_set 中

- 如果需要在 multiset 中定义自已的比较类型操作，需要在定义时在传入比较操作类型

  ```cpp
  multiset<string, decltype(compare_function) *> myset(compare_function);
  ```

- pair 类型，pair 定义在头文件 utility 中

- 通常不对关联容器使用泛型算法，由于关键字为 const 类型

- 关联容器的 insert 会返回一个 pair，pair 的 first 是一个迭代器指向给定关键字的元素，pair 的 second 是一个bool 值， 指出元素是插入成功还是已经存在存在于容器中

- 关联容器的 find 与 count，查找元素是否在容器内应使用 find 而非下标操作。

- multimap 和 multiset 中相同关键字的元素会相邻储存

- 如果关键字还在容器中，low_bound 返回的迭代器将指向第一个具有给定关键字的元素，而 upper_bound 将指向最后一个具有给定关键字的元素。equal_range 则直接返回一个 pair，pair 的 first 指向第一个匹配关键字，pair 的 second 指向最后一个匹配的关键字。

- 无需容器需要定义自己的 hash 模板与 ==，这些可以在定义 map 和 set 时作为模板传入。

  ```cpp
  unorder_multimap<myclass, decltype(hasher) *, decltype(equaler) *>
  ```

  

