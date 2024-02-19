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

    