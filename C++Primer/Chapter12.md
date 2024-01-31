## Chapter12 动态内存

- 动态内存与智能指针

  - **shared_ptr** 允许多个指针指向同一个对象

  - **unique_ptr** 独占所指向的对象
  - **weak_ptr** 一种弱引用，指向 shared_ptr 所管理的对象

- 智能指针 shared_ptr 与 unique_ptr 均支持的操作

  - 定义

    ```cpp
    shared_ptr<T> sp;
    unique_ptr<T> up;
    ```

  - 解引用

  - get 返回 p 中保存的指针

  - 交换指针

    ```cpp
    swap(p, q);
    p.swap(q);
    ```

- **shared_ptr** 独有的操作

  - 使用一个对象初始化一个 shared_ptr

    ```cpp
    make_shared<T>(args);
    ```

  - 拷贝 shared_ptr

    ```cpp
    shared_ptr<T> p(q);
    ```

  - 赋值，两个智能指针保存的指针必须可以互相转换，并且被赋值的智能指针会减少引用计数，赋值的智能指针会增加引用计数

    ```cpp
    p = q; // reference count of p will decrease and which of q will increase. 
    ```

  - unique
  
    ```cpp
    p.unique(); // if use count of p is 1, return true otherwise will return false.
    ```
  
  - use_count
  
    ```cpp
    p.use_count(); // it will return shared pointer count.
    ```
  
- make_shared

  安全的分配和使用动态内存的方法是调用 make_shared 函数，该函数类似顺序容器中的 emplace 成员，make_shared 用参数来构造给定类型的对象。可以使用 auto 来定义一个对象保存 make_shared 的结果。

- 拷贝一个 shared pointer 会增加 use_count 计数，释放会减少 use_count 计数，当 use_count 为零时会自动释放。

- 直接管理内存，new 和 delete 管理和释放内存。

- 默认初始化与值初始化

  ```cpp
  string *str = new string(); // Value Construct.
  string *str = new string; // Default Construct.
  ```

- 如果提供了一个用括号包围的初始化器，则可以使用 auto 来通过此初始化器推断想要分配的类型。

  ```cpp
  auto p = new auto(obj); // p point to a object same as obj and use obj to initialize this object.
  ```

- 当内存耗尽时，new 表达式会失败，并抛出一个 bad_alloc 的异常。可以通过改变 new 的使用方式来阻止其抛出异常。

  ```cpp
  int *p = new (nothrow) int; // if new can not allocate memory, it will return a nullptr.
  ```

  这种 new 称之为定位 new。

-  delete 指针之后需要重置为 nullptr，否则为空悬指针。

- shared_ptr 与 new 结合使用，但不要混合使用，一旦裸指针交付给智能指针管理，裸指针就不应该在使用了。也不要用 get 来获取裸指针以初始化另一个智能指针。

- 智能指针的构造函数为 explicit，显式接受由 new 返回的指针，但不能隐式将内置指针转换为智能指针。

  ```cpp
  auto p = shared_ptr<int>(new int(5)); // Right
  shared_ptr<int> p_ = new int(5); // Error
  ```

- 应使用 unique 检查该智能指针是否为唯一用户。

- 如果智能指针是唯一指向其对象的 shared_ptr，reset会释放这个对象。

- 智能指针也可以自己定义删除器来管理内存，使得一些必须在析构时进行的操作可以通过智能指针来管理。当使用智能指针管理的资源不是 new 分配的内存需要传递一个删除器。

- **unique_ptr** 拥有它所指向的对象，同一时间只能有一个 unique_ptr 指向一个给定的对象。

- 在定义 unique_ptr 时，需要将其绑定到一个 new 返回的指针上

  ```cpp
  unique_ptr<int> p(new int(42));
  ```

  unique_ptr 拥有它指向的对象，因此 unique_ptr 不支持普通的拷贝或赋值操作。但是作为返回值时会进行一种特殊的拷贝。
  
- 可以向 unique_ptr 传递删除器

- **weak_ptr** 是一种不控制所指向对象生存周期的智能指针，它指向一个 shared_ptr 对象。不会改变 shared_ptr 的引用计数。 

- weak_ptr 支持的操作

  - 使用 shared_ptr 或者其他 weak_ptr 

    ```cpp
    w = p; // p can be a shared pointer or a weak pointer.
    ```

  - 返回 shared_ptr 的引用计数

    ```cpp
    w.use_count();
    ```

  - 返回 shared_ptr 是否存在

    ```cpp
    w.expired(); // if use_count == 0, return true, else return false.
    ```

  - 返回 shared_ptr 指针

    ```cpp
    w.lock(); // if expired == true, return null shared_ptr, else return shared_ptr.
    ```

- 动态数组

  - C++ 语言与标准库提供了两种一次分配一个对象数组的方法。C++ 定义了另一种 new 表达式语法，可以分配并初始化一个对象数组。标准库中包含一个名为 allocator 的类，允许我们将分配与初始化分离。

    ```cpp
    int *p = new int[10](); // p point to a array of 10 int and each element will be initialized with 0.
    ```

  - 甚至可以使用列表初始化

    ```cpp
    int *p = new int[2]{ 1, 2 };
    ```

  - 当分配的大小为0，代码仍能正常工作，会返回一个合法的非空指针，，可以向使用尾后迭代器一样使用这个指针，但不能解引用。

  - 这种分配方式应使用对应的 delete 来删除

    ```cpp
    delete [] p; // p = new int[10];
    ```

  - 标准库提供了用于管理动态数组的 unique_ptr 版本

    ```cpp
    unique_ptr<int[]> up(new int[10]);
    up.release(); // call delete [];
    ```

  - shared_ptr 不支持直接管理动态数组（C++17之前），目前如下代码可以正常编译通过。（之前需要在构造时传入删除器）

    ```cpp
    shared_ptr<int[]> sp(new int[10]); // after C++17
    shared_ptr<int> sp(new int[10], [](int *p){ delete [] p; }); // before C++17
    ```

- allocator 类

  - allocator 是一个模板类，可以分配原始的，未经构造的内存。

    ```cpp
    allocator<string> alloc;
    auto const p = alloc.allocate(n);
    ```

  - deallocate，construct，destroy

    ```cpp
    allocator<T> a;
    a.deallocator(p, n); // p should be destroyed before deallocator.
    a.construct(p, args); // construct will call constructor.
    a.destroy(p); // destroy will call destructor.
    ```

  - allocator 类的伴随算法

    ```cpp
    uninitialized_copy(b, e, b2); // copy elements to b2, between iterator b and iterator e.
    uninitialized_copy_n(b, n, b2); // copy n elements to b2 from iterator b.
    uninitialized_fill(b, e, t); // fill t to uninitialized memory between b and e.
    unintialized_fill_n(b, n, t); // fill n*t to uninitialized memory from b.
    ```

    

