## Chapter 18 用于大型程序的工具

- 异常处理

  - 沿着调用链的函数可能会提早退出
  - 一旦程序开始执行异常处理代码，则沿着调用链创建的对象都将被销毁

- 抛出指向局部对象的的指针是几乎肯定错误的行为

- 如果 throw 表达式解引用一个基类指针，而该指针实际指向派生类，则抛出对象将被切掉一部分，只有基类部分被抛出。

  ```cpp
  class A {
      int a;
  };
  
  class B : public A {
      int b;
  };
  
  int main() {
      try {
          A *a = new B();
          throw *a;
      }
      catch (const B &b) {
          cout << "b" << endl;
      }
      catch (const A &a) {
          cout << "a" << endl;
      }
      catch (...) {
          cout << "c" << endl;
      }
  } // result: a
  ```

- 捕获异常类型必须是完全类型，它可以是左值引用，但不能是右值引用

  > 完整类型是指在编译期，编译器能计算出一个类型的 size。

- throw; 空的 throw 表达式只能出现在 catch 语句或 catch 语句直接或间接调用的函数之内。如果在处理代码之外的区域遇到了空 throw 语句，编译器将调用 terminate。

  - 一个重新抛出的语句并不指定新的表达式，而是将当前异常对象沿着调用链向上传递。

- 当 catch 语句改变了参数内容并重新抛出，只有在参数类型为引用类型时，改动才会继续传播。

- `catch(...)` 可以捕获所有类型的异常。

- 处理初始值列表抛出的异常

  ```cpp
  class A {
  public:
      A(int a) try : _a(a) {} catch(...){}
  private:
      int _a;
  }
  ```

- noexcept 异常说明

  - 函数指针只能指向具有相同异常说明的函数
  - 虚函数基类如果声明不抛出异常，则派生类不能抛出异常。基类如果声明允许抛出异常，则派生类既可以抛出异常，也可以不抛出异常。
  
- 命名空间结束后无需分号，命名空间可以是不连续的。

- 内联命名空间

  - 内联命名空间中的名字可以被外层命名空间直接使用。

    ```cpp
    inline namespace A{
        // ...
    }
    ```

- 未命名的命名空间

  - 未命名的命名空间中定义的变量具有静态的声明周期，在第一次使用前创建，直到程序结束后才销毁。
  - 一个未命名的命名空间可以在某个给定的文件内不连续，但是不能跨越多个文件，每个文件定义自己的未命名的命名空间。

- 命名空间的别名

  ```cpp
  namespace A = AAA;
  ```

- using 指示

  using 指示使得命名空间中所有内容都是可见的，它将具有命名空间成员提升到包含命名空间本身和 using 指示最近作用域的能力。using 指示通常不在头文件中使用，而在源文件中使用。

  ```cpp
  using namespace std;
  ```

  ```cpp
  namespace A {
      int i, j;
  }
  
  void f() {
      using namespace A;
      cout << i * j << endl; // for function f, variable i and j seem like just in the global scope.
  }
  ```

- 当我们给函数传递一个类类型的对象时，除了在常规的作用域查找外，还会在查找实参所属的命名空间，这一例外对传递类的引用与指针同样有效。

- move 与 forward 建议使用时结合作用域使用 std::move 与 std::forward，以减少冲突。

- 实参查找甚至可以找到类中的友元声明

  ```cpp
  namespace A {
  class C {
      friend void f1();
      friend void f2(const C &);
  }
  void f1() {}
  void f2(const C &) {}
  }
  int main() {
      A::C obj;
      f1(); // fail to find function f1 defination.
      f2(obj); // success.
  }
  ```

- using 声明的是名字而非是函数，使用 using 声明，则同名函数的所有版本均会被引入到**当前作用域**中，using 指示则会将命名空间的成员提升到**外层作用域**中，因此当命名空间中包括一个与本作用域形参列表相同的同名函数时，使用 using 声明引入函数会引发 using 错误，而使用 using 指示则不会（**如果上层作用域已经是全局而不是其他命名空间则会存在问题**），并且会优先使用本作用域版本，如果需要使用进入命名空间的版本，则需要加入作用域说明符。

  ```cpp
  namespace A {
  void func(int i) { std::cout << 2; }
  }
  
  namespace B {
  void func(int i) { std::cout << 1; }
  using namespace A;
  }
  
  int main() {
      B::func(1); // 1
  }
  ```

  ```cpp
  namespace A {
  void func(int i) { std::cout << 2; }
  }
  
  namespace B {
  // void func(int i) { std::cout << 1; }
  using namespace A;
  }
  
  int main() {
      B::func(1); // 2
  }
  ```

  ```cpp
  namespace A {
  void func(int i) { std::cout << 2; }
  }
  
  void func(int i) { std::cout << 1; }
  using namespace A;
  
  int main() {
      func(1); // fail to compile.
  }
  ```

  ```cpp
  namespace A {
  void func(int i) { std::cout << 2; }
  }
  
  void func(int i) { std::cout << 1; }
  using A::func;
  
  int main() {
      func(1); // fail to compile.
  }
  ```

- 多重继承中，虚继承的目的是令某个类做出声明，承诺愿意共享它的基类。其中，共享的基类子对象称为虚基类，在这种机制下，不论虚基类对象在继承体系中出现了多少次，在派生类中，都只包含唯一一个共享的虚基类子对象。

- 虚派生只影响从指定了虚基类的派生类中进一步派生出来的类，它不会影响派生类本身。

- 虚基类的构造由最底层的派生类初始化的。