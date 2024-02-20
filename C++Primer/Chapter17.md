## Chapter 17 标准库特殊设施

- tuple 类型

  tuple 是一个类似 pair 的模板，每个 pair 的成员类型都不相同，但每个 pair 都恰好有两个成员。而 tuple 可以有任意数量的成员。

  - tuple 常用操作

    ```cpp
    tuple<T1, T2, ..., Tn> t;
    tuple<T1, T2, ..., Tn> t(v1, v2, ..., vn);
    make_tuple(v1, v2, ..., vn);
    t1 == t2;
    t1 != t2;
    t1 relop t2; // relop means relation operator
    get<i>(t);
    tuple_size<tupleType>::value; // return tuple size
    tuple_element<i, tupleType>::type;
    ```

- bitset 类型

  使位运算的使用更加容易。bitset 类是一个类模板，类似 array 类。

  ```cpp
  bitset<32> bitvec(1u); // 32 bit, and the low order is 1, template must be a const exp.
  ```
  
  - bitset 初始化方法
  
    ```cpp
    bitset<n> b;
    bitset<n> b(u);
    bitset<n> b(s, pos, m, zero, one); // default value: pos = 0, m = string::npos, zero = 0, one = 1;
    bitset<n> b(cp, pos, m, zero, one); 
    ```
  
  - unsigned 值初始化 bitset
  
    ```cpp
    bitset<12> b(0xbeef);
    ```
  
  - string 初始化 bitset
  
    ```cpp
    bitset<12> b("1001");
    ```
  
  - bitset 操作
  
    ```cpp
    bitset<n> b;
    b.any(); // check if has 1.
    b.all(); // check if all is 1.
    b.none(); // check if all is 0.
    b.count(); // return all 1 count.
    b.size(); // return n, constexpr.
    b.test(pos); // check if pos is 1.
    b.set(pos, v);  // pos = v, v defualt == 1.
    b.set(); // set all 1.
    b.reset(pos); // set pos = 0.
    b.reset(); // set all 0.
    b.flip(pos); // reverse pos.
    b.flip(); // reverse all.
    b[pos]; // return pos value.
    b.to_ulong(); // change b to an unsigned long.
    b.to_ullong(); // change b to an unsigned long long.
    b.to_string(zero, one); // change b to string.
    os << b;
    is >> b;
    ```
  
- 正则表达式

  - 正则表达式组件

    ```html
    regex 表示有一个正则表达式的类
    regex_match 将一个字符序列与一个正则表达式匹配
    regex_search 寻找第一个与正则表达式匹配的子序列
    regex_replace 使用给定格式替换一个正则表达式
    sregex_iterator 迭代器适配器，调用 regex_search 来遍历一个 string 中所有匹配的子串
    smatch 容器类，保存在 string 中搜索的结果
    ssub_match string 中匹配的子表达式的结果
    ```

  - 正则表达式库的使用

    ```cpp
    string pattern("[^c]ei");
    pattern = "[[:alpha:]]*" + pattern + "[[:alpha:]]*";
    regex r(pattern);
    smatch results;
    string test_str = "receipt freind theif receive";
    if (regex_search(test_str, results, r)) {
        cout << results.str() << endl;
    }
    ```

  - 正则表达式语法是否正确是在**运行时**解析的

  - sregex_iterator

    ```cpp
    sregex_iterator it(b, e, r);
    sregex_iterator end;
    *it;
    it->;
    ++it;
    it++;
    it1 == it2;
    it1 != it2;
    ```

    ```cpp
    string pattern("[^c]ei");
    pattern = "[[:alpha:]]*" + pattern + "[[:alpha:]]*";
    regex r(pattern, regex::icase);
    for (sregex_iterator it(file.begin(), file.end(), r), end; it != end; ++it) {
        cout << it->str() << endl;
    }
    ```

- 随机数

  - C++ 程序不应该使用库函数 rand，而应使用 default_random_engine 类和恰当的分布类对象。

    default_random_engine 定义了调用运算符来获取下一个随机数。

    ```cpp
    default_random_engine e;
    for (size_t i = 0; i < 10; ++i) {
        cout << e() << endl;
    }
    ```

  - 标准库定义了多个随机数引擎类，区别在于性能和随机性质量不同，每个编译器都会指定其中一个作为 default_random_engine 类型，此类型一般具有最常用的特性。

  - 随机数引擎操作

    ```cpp
    Engine e;
    Engine e(s); // s is seed.
    e.seed(s); // use seed s to reset random engine.
    e.min(); // this engine min value.
    e.max(); // this engine max value.
    Engine::result_type; // result type.
    e.discard(u); // push u step of engine.
    ```

  - 分布类型和引擎

    ```cpp
    uniform_int_distribution<unsigned> u(0, 9);
    default_random_engine e;
    for (size_t i = 0; i < 0; ++i) {
        cout << u(e) << endl;
    } // generate 0 - 9 random number.
    ```

    分布类型也是函数对象类，分布类型定义了一个调用运算符，它接受一个随机数引擎作为参数，分别对象使用引擎参数生成随机数，并将其映射到指定的分布。

  - 随机数引擎定义为 static 才能保持状态使得每次调用的随机序列不同。通过设置种子可以使每次运行结果不同。

  - 其他随机数分布

    ```cpp
    uniform_real_distribution<double> u(0, 1); // 0 - 1 real number.
    normal_distribution<> n(4, 1.5); // normal distribution.
    bernoulli_distribution b; // 0.5 true, and 0.5 false.
    ```

- IO 库再探

  - 操作符改变格式状态

    ```cpp
    boolalpha <=> noboolalpha // change bool value 0, 1 to false, true.
    default, oct, hex, dec // change scale,
    showbase <=> noshowbase // show base like 0xff, 023.
    uppercase <=> nouppercase // change 0xff to 0XFF
    ```

    类似操作还有很多，可以控制浮点精度，输出格式，输入格式等。

  - 未格式化的输入输出操作

    标准库还提供了一组低层操作，支持未格式化 IO，这些操作允许我们将一个流当作一个无解释的字节序列来处理。

  - 流随机访问

    各种流类型通常都支持对流中数据的随机访问，我们可以重定位流，使之跳过一些数据，首先读取最后一行，然后读第一行等等。标准库提供了一对函数，来定位 seek，到流中给定的位置，以及告诉 tell 我们当前的位置。

    虽然标准库为所有流类型都定义了 seek 和 tell 函数，但它们是否会做有意义的事情依赖于流绑定到什么设备， 绑定到 cin、cout 等流的不支持随机访问，因为没有意义。

    以下内容针对 fstream 和 sstream。

    为了支持随机访问，IO 类维护了一个标记来确定下一个读写操作要在哪里进行。标准库提供了两个函数，一个函数通过将标记 seek 到一个给定位置来重定位它，另一个函数 tell 我们标记的当前位置。

    ```cpp
    tellg(); // input stream.
    tellp(); // output stream.
    seekg(pos); // input stream.
    seekp(pos); // output stream, pos is usually the value get from tell function,
    seekg(off, from); // change flag to from + offset.
    seekp(off, from);
    ```

    