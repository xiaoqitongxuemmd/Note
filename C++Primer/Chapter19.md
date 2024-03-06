## Chapter 19 特殊工具与技术

- 控制内存分配

  - 重载 new 和 delete

    标准库定义了 operator new 函数和 operator delete 函数的8个重载版本

    ```cpp
    void *operator new(size_t);
    void *operator new[](size_t);
    void *operator delete(void *) noexcept;
    void *operator delete[](void *) noexcept;
    
    void *operator new(size_t, nothrow_t&);
    void *operator new[](size_t, nothrow_t&);
    void *operator delete(void *, nothrow_t&) noexcept;
    void *operator delete[](void *, nothrow_t&) noexcept;
    ```

    nothrow 为 new 定义的 const 对象，可以用于申请不抛出 bad_alloc 异常。