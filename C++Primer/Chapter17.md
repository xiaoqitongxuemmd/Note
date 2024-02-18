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

  使位运算的使用更加容易。

  - 