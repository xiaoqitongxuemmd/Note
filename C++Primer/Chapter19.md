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
    
  - 对于 operator new 函数或者 operator new [] 函数的返回值类型必须是 void *，size_t 接受储存指定类型对象所需的字节数。
  
  - 理论上用户可以自定义重载任何参数形式的 new 运算符，但是如下形式无法重载，只能被标准库调用
  
    ```cpp
    void operator new(size_t, void *);
    ```
  
  - delete 函数的返回值类型必须是 void，第一个形参的类型必须是 void *
  
  - new 表达式和 operator new 函数
  
    operator new 重载并没有重载 new 表达式，这句话的含义是 `operator new` 和 ` new` 不一样，其他的运算符则不会，例如 `operator+` 和 `+` 是一样的，我们可以 `operator+(a, b)` 这样和 `a + b` 等价。
  
    我们只能重载 operator new 而不能自定义 new 表达式的行为，一条 new 表达式的执行总是先调用 operator new 以获取内存空间，然后在内存空间中构造对象，而 delete 则相反，总是先销毁对象，然后调用 operator delete 释放空间。
  
    这也说明重载 operator new 只会改变内存的分配方式，而不能改变 new 表达式的含义。
  
  - malloc 和 free 从系统分配空间和释放空间到系统。
  
  - placement new 表达式
  
    ```cpp
    new (placement_address) type;
    
    void *p_addr = operator new (sizeof(int));
    int *a = new (p_addr) int;
    ```
  
  - 显式调用析构函数
  
    ```cpp
    string *ps = new string("");
    ps->~string();
    ```
  
- 运行时类型识别。

  - typeid 运算符，用于返回表达式的类型

    `typeid(e)` 当传入表达式的类型为含有虚函数的左值时，typeid 结果会在运行时计算。

  - dynamic_cast 将基类指针或引用安全的转换成派生类的指针或引用。

    - 指针转换可以直接判断
    - 引用转换会抛出异常

  - type_info 类，创建 type_info 对象的唯一途径是使用 typeid 运算符。
  
- 枚举类型

  - 枚举成员是 const 因此在初始化枚举成员时提供的初始值必须是常量表达式
  - 限定作用域的枚举默认类型是 int，而不限定作用域的枚举类型不存在默认类型，只知道成员的潜在类型足够大。

- 类成员指针

  - 成员指针指示的是类的成员而非类的对象。如下声明表示指针 pa 可以指向 A 类的一个 int 类型成员。

    ```cpp
    const int A::*pa;
    ```

    当我们初始化一个成员指针或为成员指针赋值时，该指针并没有指向任何数据，成员指针指定了成员而非该成员所属的对象，只有当解引用成员指针时我们才提供对象的信息。

    ```cpp
    class A {
        int _a;
        
        void func() {
            const int A::*pdata = &A::_a;
        	A a;
        	A *pa = &a;
        	int data_a = a.*pdata;
        	int data_a_ = pa->*pdata;
            
        }
    }
    ```

    类的访问控制对类成员指针依然有效，上述中 pdata 必须位于类 A 内部，由于 _a 为类 A 的私有成员。

  - 成员函数指针

    - 并非可调用对象，必须绑定到一个具体对象解引用后才能调用。

    - 使用 function 可以生成一个可调用对象，mem_fn 也可以实现，或者 bind。

      ```cpp
      function<void(int)> func_1 = &A::func;
      auto func_2 = mem_fn(&A::func);
      auto func_3 = bind(&A::func, _1);
      ```

  - 成员指针函数表
  
- 嵌套类

  在嵌套类的对象中不包含任何外层类定义的成员。同样，外层类的对象中也不包含任何嵌套类定义的成员。

- union

  - union 不能包含引用类型的成员。
  - 默认情况下，union 中的对象都是公有的。
  - 为 union 的一个数据成员赋值会使其他成员变为未定义的状态。
  - 对于 union 来说想要构造或销毁类类型的成员必须执行非常复杂的操作，因此通常将含有类类型成员的 union 内嵌在另一个类当中，这个类可以管理并控制与 union 的类类型成员有关的状态转换。

- 固有的不可移植的特性

  指因机器而异的特性。

  - 位域

    取地址运算符不能作用于位域。

  - violate

    violate 和 const 很像，不能将非 violate 对象绑定到 violate 对象上。

  - 链接指示 extern "C"

    C++ 调用非 C++ 代码。例如如下声明一个指向 C 函数的指针

    ```cpp
    extern "C" void (*pf)(int);
    ```

    