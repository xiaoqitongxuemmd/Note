## Chapter 10 泛型算法

> 泛型算法 generic algorithm

- 算法头文件 algorithm，同时在 numeric 中定义了一组数值泛型算法。泛型算法一般不操作容器，而是**遍历由两个迭代器指定的一个元素范围**。

- 由于泛型算法运行于迭代器之上，而非操作容器，因此必须假定**算法不会直接增加或者删除元素**。插入功能需要特殊的迭代器 **inserter**。

- 只读算法

  - **accumulate**

    定义在 numeric 头文件中，函数原型如下

    ```cpp
    auto sum = accumulate(v.cbegin(), v.cend(), 0); // calculate sum of vector. 
    ```

  ​	第三个参数的类型决定了函数中使用哪个加法运算符以及返回值的类型。（这种只读取而不改变元素的算法通常使用 cbegin 与 cend)

  - **equal**

    定义在 algorithm 头文件中，函数原型如下

    ```cpp
    auto result = equal(list1.cbegin(), list1.cend(), list2.cbegin()); // suppose the second list is no less than the first one.
    ```

- 写容器元素的算法

  一些算法将新值赋予序列中的元素，这类算法的使用需要保证序列原大小至少不小于我们要求算法写入的元素数目。

  - **fill**

    定义在 algorithm 头文件中，函数原型如下

    ```cpp
    fill(v.begin(), v.end(), 0); // reset all elements to zero.
    ```

- 一种保证算法有足够空间来容纳输出数据的的方法是使用插入迭代器 （insert iterator）

  - **back_inserter**

    定义在 iterator 头文件中，back_inserter 接受一个指向容器的引用，返回一个与该容器绑定的插入迭代器。当我们通过此迭代器赋值时，赋值运算符会调用 push_back 将一个具有给定值的元素添加到容器中。

    ```cpp
    vector<int> vec;
    auto it = back_inserter(vec);
    *it = 42; // call push_back to push 42 into vec.
    ```

  - **copy** 

    函数原型如下 ***(TestCopy)***

    ```cpp
    auto ret = copy(begin(list1), end(list1), begin(list2)); // copy from the first list to the second one, and ret point to the end element of the second list.
    ```

- 重排容器元素的算法

  - **sort**

    函数原型如下
    
    ```cpp
    sort(v.begin(), v.end());
    ```
    
  - **unique**
  
    重拍输入范围，使其中每个只出现一次。函数原型如下，返回指向不重复区域之后一个位置的迭代器。
  
    ```cpp
    auto end_unique = unique(v.begin(), v.end());
    ```
  
  - **erase**
  
     函数原型如下
  
    ```cpp
    v.erase(v.begin(), v.end());
    ```
  
- 定制操作

  通过向算法传递函数来重载泛型算法的的行为（传递函数称之为谓词）

  > predicate 谓词

   谓词为一个可以调用的表达式，其返回值结果为一个能作为条件的值。
  
  - **一元谓词** 只接受单一参数
  - **二元谓词** 接受两个参数
  
- lambda 表达式

  lambda 表达式不能有默认参数，可以通过 lambda 表达式实现向谓词传递多个参数。lambda 表达式在定义时编译器会生成一个与 lambda 表达式对应的类类型。**可以通过在参数列表后加入 mutable 来使值传递的参数改变。**

- for_each 算法

  函数原型如下

  ```cpp
  for_each(v.begin(), v.end(), func()); // the third parameter is a predicate used for transferring a lambda expression.
  ```

- 参数绑定

  bind 函数定义在 functional 头文件中，可以将 bind 函数看作一个通用的函数适配器，它接受一个可调用对象，生成一个新的可调用对象来适应原对象的参数列表。函数原型如下

  ```cpp
  auto newCallable = bind(callable, arg_list); // callable is a function and arg_list is argument list used for this function.
  ```

  placeholder 占位符，表明调用函数的参数，传递引用对象可以使用标准库函数 ref
  
  ```cpp
  auto g = bind(f, a, b, placeholder::_2, c, placeholder::_2);
  // g(X, Y) is equal to f(a, b, Y, c, X);
  ```
  
- 迭代器

  - 插入迭代器（insert iterator）用于向容器中插入元素，back_inserter，front_inserter，inserter

  - 流迭代器（stream iterator）这些迭代器被绑定在输入或者输出流上，可用来遍历所有关联的IO流，istream_iterator，ostream_iterator

  - 反向迭代器（reverse iterator）反向移动迭代器 reverse_iterator

    使用 base 可以将反向迭代器转换为正向迭代器

  - 移动迭代器（move iterator）用于移动而非拷贝