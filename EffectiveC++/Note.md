## Effective C++

#### 01 视 C++ 为一个语言联邦

- 对内置类型通常值传递比引用传递更高效

#### 02 尽量以 const enum inline 替换 \#define

- enum hack 

  理论基础：一个枚举类型的数值可以权充 int 被使用

  ```cpp
  class A {
      enum { _Num = 5 };
      // old complier do not support define value when declare a varible. like
      // static const _Num = 5;
      int _mem[_Num];
  }
  ```

- 变量使用 const 或 enum 代替 \#define，函数使用 inline 代替 \#define

#### 03  尽可能使用 const

- 使用 mutable 释放 const 的约束

- 当定义一个函数需要一个 const 版本和一个非 const 版本时，考虑到代码维护等问题，非 const 应调用 const 版本并移除常量性

  ```cpp
  class A {
  public:
      const int &operator[](std::size_t pos) const {
          ... // check
          return _text[pos];
      }
      int &operator[](std::size_t pos) {
          return const_cast<int &>(static_cast<const A &>(*this)[pos]);
      }
  private:
      int *_text;
  }
  ```

  static_cast 避免了一个非 const A 对象递归调用自身。

  使用 const 调用 non-const 版本以避免重复是一种错误行为。

#### 04 确定对象被使用前已经先被初始化

- C++ 对象成员变量的初始化发生在进入构造函数本体之前

- 总是在初始值列中列出所有成员变量。

- class 的成员变量总是以其声明次序被初始化

- C++ 对于定义于不同的编译单元内的 non-local（函数内的 static 对象为 local，其他为 non-local）static 对象的初始化顺序并无明确定义，但保证 local static 对象会在函数调用期间，首次遇上该对象之定义式时被初始化。所以，以**函数调用的方式（返回 reference 指向 local static 对象）**替换**直接访问 non-local 对象**。

  ```cpp
  class A {}
  A &ra() {
      static A a;
      return a;
  }
  ```

  多线程环境下 static 变量（无论 local 还是 non-local）都会存在问题。

#### 05 了解 C++ 默默编写并调用那些函数

#### 06 若不想使用编译器自动生成的函数，就该明确拒绝

- 使用 final 拒绝拷贝构造函数和拷贝复制运算符的编译器自生成行为（旧版本的操作为声明为 private 而不实现）

#### 07 为多态基类声明 virtual 析构函数

- 将基类析构函数声明为  virtual 可以正确析构基类指针，即使它具有动态特性而指向了派生类对象。
- 如果一个类不包含 virtual 对象，不应该将析构函数声明为 virtual，会消耗额外的内存。多态的实现方式：
  - 每个包含虚函数的类都有一个虚函数列表
  - 虚函数表的指针存在对象实例最前面的位置
  - 派生类虚表的虚函数地址排列顺序与基类中虚函数地址排列顺序完全一致。如果子类中包含有自身虚函数，会排列在后面
  - 虚表可以继承。如果子类没有重写虚函数，那么子类虚表中仍然会有该函数的地址，只不过这个地址指向的是基类的虚函数实现。如果重写了相应的虚函数，那么虚表中的地址就会改变，指向自身的虚函数实现。如果派生类有自己的虚函数，那么虚表中就会添加该项
- 抽象类应将析构函数声明为 virtual，因为抽象类通常都是用于继承的。

#### 08 别让异常逃离析构函数

- C++ 不禁止析构函数抛异常，但是不鼓励。所以析构函数绝对不要抛异常，即使可能抛出异常，也应该阻止异常传播。

#### 09 绝不在构造和析构过程中调用 virtual 函数

- base class 构造期间 virtual  class 不是 virtual class。

#### 10 令 operator= 返回一个 reference to *this

- 为了实现连续赋值

#### 11 在 operator= 中处理自赋值

- 具备异常安全性的自赋值处理

  ```cpp
  A &A::operator=(const A &rhs) {
      B *porig = pb;
      pb = new B(*rhs.pb);
      delete porig;
      return *this;
  }
  ```

#### 12 复制对象时勿忘其每一个部分

- 任何时候只要需要为派生类编写 copying 函数，必须很小心的复制基类部分。派生类应调用相应的基类 copying 函数。

  ```cpp
  class B {}
  class D : public B {}
  D::D(const D &rhs) : B(rhs) {
  }
  D &D::operator=(const D &rhs) {
      B::operator=(rhs);
      return *this;
  }
  ```

- 拷贝构造和拷贝赋值运算符在实现上均不应该互相调用。

#### 13 以对象管理资源

- 把资源放进对象内，可以依赖 C++ 的析构函数自动调用机制确保资源被释放（智能指针策略）。
- 手动释放资源也可以自行设计资源管理类。

