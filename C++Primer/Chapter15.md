## Chapter 15 面向对象程序设计

- 多态传入的不一定是指针，也可以是对象的引用，究其根本，即对象是什么，就调用什么版本的函数。多态为**运行时绑定**。

- 基类通常都需要定义一个虚析构函数。

- virtual 关键字只能在类内的函数声明中使用，而不能在类外的函数定义中使用。

- 函数在基类中声明为虚函数，则在派生类中也隐式为虚函数。

- 公有派生中，基类的公有成员也是派生类接口的组成部分，并且能将公有派生类型的对象绑定到基类指针或引用上。

- 派生类中包含基类的私有成员（通过 sizeof 可以判断大小），但是不能直接访问。派生类对象中包含其基类对应的组成部分。

- 构造从基类到派生类，析构函数从派生类到基类。

- 如果基类定义了一个静态成员，则在整个继承体系中只存在该成员的唯一定义。

- 可以在定义类时添加 final 阻止继承发生。

  ```cpp
  class A final {
      ...
  }
  class B : public A { // Error, class A is final and can not be derived.
      ...
  }
  ```

- 不存在从基类到派生类的隐式类型转换。并且即使是一个指向派生类的基类指针也不能直接转换为一个派生类指针。需要使用 dynamic_cast 进行运行时检查。当然如果已知一个基类指针向派生类指针转换是安全的，也可以使用 static_cast 覆盖这种检查。

  ```cpp
  B b;
  A *pa = &b;
  B *pb = pa; // Error, can not transfer from base class to derive class.
  B *pb = dynamic_cast<B *>(pa);
  B *pb = static_cast<B *>(pa); // we should ensure this transfer is safe.
  ```

- 通常情况下，虚函数的派生类版本函数形式应与基类一致，除了返回值就是本身类指针或引用的情况，这种情况下允许基类版本返回基类指针，派生类版本返回派生类指针。

- 可以通过对虚函数加入 final 标记来阻止派生类重写该函数。

- 虚函数的默认实参调用由本次调用的静态类型决定。

- 可以通过在函数调用时加入类作用域来强行调用基类版本而拒绝动态绑定。

- 我们可以为纯虚函数提供定义，但是函数体定义必须在类外，即使提供了函数体定义依然不能实例化对象。

- 派生类只初始化直接基类。

- 受保护的成员对于类用户来说是不可访问的，派生类成员或友元只能通过派生类对象来访问基类的受保护成员。

- 派生访问说明符对于**派生类的成员**能否访问其直接基类的成员没有影响，派生类访问说明符的目的是控制**派生类用户**对于基类成员的访问权限。派生访问说明符还可以控制继承自派生类的新类的访问权限。

  ```cpp
  class A {
      int fun(); // member of class.
  }
  A a; // user of class.
  ```

- 只有当派生类公有的继承于基类时，用户才能使用派生类向基类的转换。

- 无论派生类以什么方式继承于基类，成员函数与友元都能使用派生类向基类的转换。

- **对于基类成员访问说明符与派生列表成员访问说明符的个人理解**：类的成员访问说明符声明了类中变量的属性，派生类中又是包含基类的，因此可以想象派生类中有一个基类变量，那么派生列表成员访问说明符其实可以理解为派生类的中基类变量的说明符。而派生类使用基类成员可以理解为两步，首先使用基类对象，在使用基类对象的成员。那么派生类成员访问类内的变量无论是什么类型均可以访问，因此派生列表访问说明符并不影响类内对基类成员的访问。对于用户来说，则需要受到派生列表说明符的控制。

  ```cpp
  class B : public A {
      
  }
  // it means 
  class B {
  public:
      A a;
  }
  ```

- 友元关系不能继承，也不能被传递。

- 可以通过 using 声明改变派生类继承的个别变量的访问级别。

  ```cpp
  class A {
  public:
      int a;
  }
  class B : private A {
  public:
      using A::a;
  }
  int main() {
      B b;
      b.a; // even if derive class is private inheritance, we can use member "a" directly because of "using" declaration.
  }
  ```

- 派生类成员将隐藏同名的基类成员。

- 如果希望基类函数的所有重载版本对派生类来说都是可见的，则需要覆盖所有版本或者全部都不覆盖。但是也可以通过 using 声明在只覆盖一部分，其他调用基类。

  ```cpp
  class A {
  public:
      int func();
      int func(int, int);
  }
  class B : public A {
  public:
      using A::func;
      int fun();
  }
  int main() {
      B b;
      b.func(1, 2); // if do not have "using A::func", this sentence is wrong.
  }
  ```

- 为了能正确的析构指针的动态类型，需要定义虚析构函数。

- 如果一个类定义了析构函数，则编译器不会为这个类合成移动操作。

- C++11认为合成拷贝函数是危险的，但是为了兼容老代码做出了妥协，如果在类中定义了移动赋值或者构造，则不会自动合成拷贝函数。

- 派生类的拷贝构造在调用基类拷贝构造时可以直接传入派生类引用。

  ```cpp
  class D : class B {
      D(const D &d) : B(d) {}
  }
  ```

- 如果构造函数或者析构函数调用了虚函数，则我们应该执行与构造函数或析构函数所属类型相对应的虚函数版本。

- 派生类继承基类构造函数的方式是提供一条基类名的 using 声明

  ```cpp
  class B : class A {
  public:
      using A::A;
  }
  ```

- 通常情况下，using 声明语句只是令某个名字在当前作用域下可见，当作用于构造函数时，using 声明语句将令编译器产生如下代码，形参列表与构造函数一致。

  ```cpp
  derived(params) : base(args) {
  }
  ```

- 构造函数的 using 声明不会改变其访问级别，也无法加入 explicit 和 constexpr。

- 容器中存放派生对象时应采用间接储存的方式（存放基类指针）。

- 智能指针行为与普通裸指针一样，具有动态功能。