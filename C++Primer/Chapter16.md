## Chapter 16 模板与泛型编程

- 模板定义以关键字 template，后跟一个模板参数列表。

- 调用函数模板时，编译器通常根据函数实参来推断模板实参。

- **非类型参数**表示一个值而非一个类型。非类型参数被一个用户提供的或编译器推断出的值所代替，这个值必须是常量表达式。

  ```cpp
  template<unsigned N, unsigned M>
  int func(const char (&p)[N], const char (&q)[M]) {
      // to do something.
  }
  ```

- 编写泛型代码的两个重要原则

  - 模板中的函数参数是 const 引用
  - 函数体中的条件判断仅使用 < 比较运算（为了可移植性，可以使用 less）

- 与非模板代码不同，模板的头文件通常既包括声明也包括定义(函数模板)。

- 编译器可以为函数推导模板参数类型但不能为类推导模板参数类型。

- typename 的作用是告诉编译器 typename 后面的字符串是一个类型名称而非一个成员函数或者成员变量。T::const_iterator 依赖于模板参数T，模板中依赖于模板参数的名称称为从属名称，当一个从属名称嵌套在一个类中时，称为嵌套从属名称。嵌套从属名称需要使用 typename 声明。

  ```cpp
  template<typename T>
  void fun(const T &a, typename T::const_iterator b);
  ```

- 与其他类相同，既可以在类模板内部，也可以在类模板外部定义成员函数。**定义在类模板内部的成员函数被隐式声明为内联函数**。

- 类模板的成员函数都具有和模板相同的模板参数，因此定义在类模板之外的成员函数就必须以 template 开始，后接类模板参数列表。

- 即使一个类模板被实例化也并非将所有成员函数都实例化，而是只实例化使用到的成员函数，因此在模板参数不能完全满足类模板的要求时，有时也可以使用。

- 在类模板作用域中可以直接使用模板名而不提供实参。

- 类模板与友元，可以建立一对一的友好关系，也可以建立多对一或者一对多的友好关系，取决于友元如何声明。

- 可以令模板类型参数声明为友元

  ```cpp
  template <typename Type>
  class A {
      friend Type;
  }
  ```

- 新标准允许为类模板定义一个类型别名

  ```cpp
  template <typename T>
  using twin = Pair<T, T>;
  twin<string> a;
  ```

- 模板类的 static 成员在每一个实例化版本都有一个。

- 函数参数是指向模板类型参数的右值引用，则对应实参的 const 属性和左右值属性将得到保持。

- 使用 `std::forward` 保持类型信息。`std::forward` 必须通过显式模板实参来调用，显式返回显式实参类型的右值引用。

- 使用 `std::move` 与 `std::forward` 最好不要使用 using 声明

- 函数模板可以被另一个模板或一个普通非模板函数重载。

- 当有多个重载模板对一个调用提供同样好的匹配时，应选择更特例化的版本

  ```cpp
  template <typename T> func(const T &);
  template <typename T> func(T *);
  int main() {
      int *a = nullptr;
      func(a); // it will choose second vision.
  }
  ```

- 多个可以匹配的候选函数，如有有非模板函数则优先选择非模板版本。

- 可变参数模板就是一个接受可变数目参数的模板函数或模板类。可变数目的参数称为参数包，存在两种参数包，模板参数包（表示零个或多个模板参数）和函数参数包（表示零个或多个函数参数）。`typename...` 指出接下来的参数表示零个或多个类型的列表。在函数参数列表中如果一个参数的类型是一个模板参数包，则此参数也是一个函数参数包

  ```cpp
  template <typename T, typename... Args>
  void fun(const T &t, const Args& ... rest);
  ```

- 使用 `sizeof...` 获取参数包中有多少元素

- 递归打印所有参数

  ```cpp
  template <typename T>
  ostream &print(ostream &os, const T &t) {
      return os << t;
  } // final
  
  template <typename T, typename... Args>
  ostream &print(ostream &os, const T &t, const Args &...rest) { // first expand
      os << t << ", ";
      return print(os, rest...); // second expand
  }
  ```

- 对于一个参数包除了获取大小之外，唯一能做的事是扩展。当扩展一个包时，需要提供每个扩展元素的模式，扩展一个包就是将其分解为构成的元素。对每个元素应用模式，获得扩展后的列表。通过在模式右边放置一个省略号（...）来触发扩展。

  上述例子中第一次扩展扩展模板参数包，第二次扩展函数参数包。

  `print(cout, "a", "b", "c")` 如此调用则第一次扩展为 `ostream &print(ostream &os, const char *&, const char *&, const char *&)` 第二次扩展为 `print(os, "b", "c")` 

- 上述用例中仅仅将包扩展为其构成元素，C++允许更为复杂的扩展模式 

  - `func(rest)...` 对包中每个元素执行 `func`  函数

- 转发参数包，可以使用可变参数模板与 `std::forward` 机制来编写函数实现将实参不变的传递给其他函数。例如 `emplace_back` 可变参数版本的实现

  ```cpp
  template <typename... Args>
  inline void emplace_back(Arg &&...args) {
      alloc.construct(loc, std::forward<Arg>(arg)...);
  }
  ```

- 模板特例化

  ```cpp
  template<typename T>
  int compare(const T &lhs, const T &rhs) { ... } // template
  
  template<>
  int compare(const char * const &lhs, const char * const &rhs) { ... } // special template for const char *
  ```

  - 定义特例化版本时，函数参数必须与一个先声明的模板中对应的类型匹配。

  - 模板及其特例化版本应该声明在同一个头文件中，所有同名模板的声明应该放在前面，而后是这些模板的特例化版本。
  - 类模板中，template <> 表明全特例化版本，类模板也可以部分特例化。
