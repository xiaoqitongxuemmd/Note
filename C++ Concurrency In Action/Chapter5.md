## Chapter 5 C++ 内存模型和原子操作

---

### 内存模型基础

---

内存模型涉及两个方面：基本结构和并发。

- 对象和内存区域

  不论对象属于什么类型，都会储存在一个或多个内存区域中，如果使用到了位域，尽管相邻的位域分属不同对象，但是同样属于同一内存区域。

  - 0 位域

    - 0 位域必须匿名
    - 0 位域可以分割变量位域，使其下一个变量位域一个内存区域的起点

    ```cpp
    class A {
        unsigned _a : 2;
        unsigned _b : 4;
    }
    class B {
        unsigned _a : 2;
        unsigned : 0;
        unsigned _b : 4;
    }
    
    int main() {
        sizeof(A); // 4 == unsigned
        sizeof(B); // 8 == unsigned * 2
    }
    ```

- 对象、内存区域和并发

  如果两个线程访问同一内存区域，只读数据无需保护或同步，如果更新内存则需要采取响应措施。即使使用互斥锁定，每次仅允许一个线程访问内存区域，这种通常也无从预知具体哪个访问在前。因此可以利用原子操作的的同步性，强制两个线程遵从一定的访问次序。
  
- 改动序列

  在 C++ 程序中，每个对象都具有一个改动序列，它由所有线程在对象上的全部写操作构成，其中第一个写操作即为对象的初始化。大部分情况下，这个序列会随着程序的多次运行而发生变化，但是在程序的任意一次运行中，所有线程都会形成相同的改动序列（这里的含义是每个变量的改动序列在不同线程都已一样的）。如果多个线程共同操作某一对象，但是该对象不属于原子类型，则需要自行负责同步操作，进而确保每个线程形成相同的改动序列。

  - 变量的值会随着时间推移形成了一个序列，在不同线程上观察属于同一个变量的序列，如果所见不同，说明出现了数据竞争和未定义行为。
  - 如果采用原子操作，则编译器有责任保证必要的同步操作有效。
  - 为了实现上述保障，需要禁止投机并行（speculative execution， 是一类底层优化技术，包括分支预测、数值预测、预读内存和预读文件等，目的是在多级 CPU 上提高指令的并发程度。做法是提前执行指令而不考虑是否必要，如果完成后发现没有必要，则抛弃或修正与执行的结果）。只要某个线程看到过某个对象，则该线程的后续读操作必须获得相对新进的值，并且线程就同一对象的写操作必然出现在改动序列的后方。另外如果某线程先向一个对象写数据过后在读取他，则必须读取它前面写的值。总结如下
    - 序列中每个值由写操作提供
    - 序列中的第一个值就是原子变量初始化时提供的值
    - 如果线程的读操作得到某个值，那么这个线程此后读到的结果要么是这个值，要么是序列中比这个值更靠后的值。
    - 如果线程进行写操作后接读操作，那么线程读操作的值要么是刚才写操作的值，要么序列中更新的值。
    - 本质上原子操作 atomic::load 函数就是读取改动序列中某个下标的值。

### C++ 中的原子操作及其类别

---

原子操作是不可分割的操作，在系统的任一线程内，都不会观察到这种操作处于半完成状态，它或者完全做好，或者完全没做。在读取某个对象的过程，假如其内存加载行为属于原子操作并且该对象的全部修改行为也都是原子操作，则通过内存加载行为就可以得到该对象的初始值，或某次修改而完整存入的值。

非原子操作在完成到一半的时候有可能被另一线程所见。

- 标准原子类型

  标准原子类型位于头文件 \<atomic\> 这些类型的操作全是原子化的。实际上，可以凭借互斥保护模拟出标准的原子类型，他们几乎全部都具有 is_lock_free 允许使用者判定某一给定类型上的操作是能由原子指令直接实现。还是要借助编译器和程序库的内部锁来实现。
  
  ```cpp
  std::atomic<T>::is_lock_free;
  ```
  
  原子操作的关键用途是取代需要互斥的同步方式。如果原子操作本身也在内部使用互斥，就很可能无法达到所期望的性能提升。从 C++17 全部原子类型 都含有一个静态常量表达式成员变量，`T::is_always_lock_free` 功能是考察**编译**生成的一个特定版本的程序，当且仅当在所有支持该程序运行的硬件上，原子类型 T 全都以无锁结构实现。（这里的含义主要是确定这个原子类型是不是在任意给定的目标硬件都可以以无锁的形式实现，则该函数返回 true，即在编译期可以确定，而如果该原子类型的无锁情况依赖于硬件，则无法在编译期确定遂返回 false）
  
  ```cpp
  std::atomic<T>::is_always_lock_free;
  ```
  
  - 只有一个原子类型不提供 is_lock_free 成员函数，std::atomic_flag，它是简单的布尔标志，因此必须采取无锁操作，利用这种简单的无锁布尔标志可以实现一个简单的锁，进而基于该锁实现其他所有原子类型。std::atomic_flag 在初始化的时候清零，随后通过成员函数    test_and_set 查值并设置成立，或者由 clear 清零，并没有其他操作。
  - 其余原子类型都是通过 std::atomic<> 特化而来，但不一定是无锁结构。
  - 由于不具备拷贝构造函数或拷贝赋值运算符，因此传统上，标准的类型对象无法复制也无法赋值，然而，它们其实可以接受内建类型赋值。也支持隐式转换为内建类型。
  - 整型和指针都支持 ++ 和 -- 运算，这些操作都有对应的具名函数 fetch_add，fetch_sub 等，逻辑操作也有函数例如 fetch_and，fetch_or，fetch_xor，但是操作运算符返回存入的值，而具名函数返回操作前的值。（如果返回原子类型引用，在需要单独的读取操作来获取存入的值，赋值与读取之间存在间隙则有可能导致条件竞争）。
  - 类模板 std::atomic<> 并不局限于上述特化类型，而具有泛化模板，可依据用户自定义类型创建原子类型的变体，但仅提供 load、store、exchange、campare_exchange_weak、compare_exchange_strong。
  - 对于原子类型上的每一种操作，都可以提供额外的参数，从枚举类 std::memory_order 取值，用于设定所需的内存次序语义
    - std::memory_order_relaxed
    - std::memory_order_acquire
    - std::memory_order_consume
    - std::memory_order_acq_rel
    - std::memory_order_release
    - std::memory_order_seq_cst
  - 操作的类别决定了内存次序所准许的取值，如果没有显式设定内存次序，则默认按照最严格的内存次序来（std::memory_order_seq_cst），而操作被划分为三类
    - 存储操作：memory_order_relaxed、memory_order_release、memory_order_seq_cst
    - 载入操作：memory_order_relaxed、memory_order_consume、memory_order_acquire 和 memory_order_seq_cst
    - 读改写操作：memory_order_acq_rel 以及上述五种 memory_order 都可以选用。[浅谈C++内存模型 | Caturra's Blog (bluepuni.com)](https://www.bluepuni.com/archives/cpp-memory-model/) 中并不建议使用 memory_order_consume。
  
- 操作 std::atomic_flag

  atomic_flag 是一个布尔标志，只有两种状态，成立或置零。其唯一的用途适用于充当构建单元。

  - 初始化由宏 ATOMIC_FLAG_INIT，并处于置零状态（只有 atomic_flag 需要采用这种特殊处理）

    ```CPP
    std::atomic_flag = ATOMIC_FLAG_INIT;
    ```

  - 唯一保证无锁的原子类型。

  - 如果 std::atomic_flag 对象具有静态的储存周期，他就会保证以静态方式初始化，从而避免初始化次序的问题。

  - atomic_flag 初始化后只能对其进行三种操作。

    - 销毁 -> 析构函数
    - 置零 -> clear （可以指定内存次序，只能使用存储操作的内存次序）
    - 读取原有值并设置标志位成立 -> test_and_set （可以指定内存次序）

    ```cpp
    f.clear(memory_order_relaxed);
    bool x = f.test_and_set();
    ```

  - 无法从 atomic_flag 对象拷贝构造出另一个对象，也无法向另一个对象拷贝赋值，这两个限制在所有原子类型上均有（由于拷贝赋值和拷贝构造都涉及两个对象，而牵涉两个不同对象的单一操作无法原子化）。

  - 实现自旋互斥

    ```cpp
    class spinlock_mutex {
    public:
        spinloc_mutex()
            : _flag(ATOMIC_FLAG_INIT) {}
        void lock() {
            while (_flag.test_and_set(std::memory_order_acquire));
        }
        void unlock() {
            _flag.clear(std::memory_order_release);
        }
    private:
        std::atomic_flag _flag;
    }
    ```

- 插入一个问题 [浅谈C++内存模型 | Caturra's Blog (bluepuni.com)](https://www.bluepuni.com/archives/cpp-memory-model/) （引一下大佬博客）

  下面这段代码断言是否会触发。

  ```cpp
  int val = 0;
  bool flag = false;
  void write() {
      val = 1;
      flag = true;
  }
  void read() {
      while (!flag);
      assert(val == 1);
  }
  int main() {
      thread t1(write), t2(read);
      t1.join();
      t2.join();
  }
  ```

  依据博客描述，这里的断言是有可能会触发。这段代码的逻辑非常简单，write 通过一个 flag 来确认 val 值已经被写入，而 read 函数中先自旋等待，flag 置为 true 后说明 val 已经更新完成，进行断言判定。在弱内存模型下，对于 read 线程中，是有可能先感知到 flag 的变化，后感知到 val 的变化，因此则会有可能触发断言，就是在 write 函数中，对 val 的赋值可能会被重排到 flag 赋值之后。（了解了后续内容后，发现这里的理解有点问题，针对每个线程自己来说，是存在先行关系的，即对于 t1 来说，一定是先执行 val = 1， 后执行 flag = true。线程 t2 来说一定是先执行 while(!flag)，后执行 assert(val == 1)，但是无论 flag 变量，还是 val 变量，读写之间都缺乏同步关系，因此，对于 t2 来说，读取 val 可能是旧值，产生这个的原因是 val 的读取是从内存中读取，但是 val 新写入的值可能还在 cpu 缓存中，并未写入内存。）

  解决这个问题就涉及 release-acquire ）

  - acquire 语义

    acquire 操作 A 之后的原子或非原子 load/store 操作不能被重排到 A 之前

  - release 语义

    release 操作 A 之前的原子或非原子 load/store 操作不能被重拍到 A 之后

  因此上述代码应修改为

  ```cpp
  void write() {
      val = 1;
      flag.store(true, memory_order_release);
  }
  void read() {
      while (!flag.load(memory_order_acquire));
      assert(val == 1);
  }
  ```

- 操作 std::atomic\<bool\>

  是基于整数的最基本原子类型，并且比 std::atomic_flag 功能更加齐全，其可以依据非原子布尔量创建其对象（atomic_flag 只能通过宏初始化），并可以接受非原子布尔量的赋值。

  ```cpp
  std::atomic<bool> b(true);
  b = false;
  ```

  同样受限于原子操作，该赋值返回的是赋予的布尔值而非原子类型的引用，如果代码依赖赋值操作的结果，则必须显式加载该结果的值。

  - 写操作调用 store
  - test_and_set 替换为 exchange
  - 支持单纯读取，可以隐式将实例转换为普通布尔值，显式做法是调用 load

  ```cpp
  std::atomic<bool> b;
  bool x = b.load(std::memory_order_acquire);
  b.store(true);
  x = b.exchange(false, std::memory_order_acq_rel);
  ```

  - 同时也支持一个新操作，可以依据当前对象的值来决定是否保存新值，compare_exchange（比较-交换），使用者给定期望值，相等就存入另一既定的值，不等就更新期望所属变量，向其赋予原子变量的值。

    - compare_exchange_weak 即使原子变量的值与期望值相等，仍可能保存失败，并维持原值不变。（这种失败是由比较交换操作在一些处理器向并没有对应的指令，不能以原子化的形式实现，只能用多条指令实现，然而在线程多于处理器数量时，线程有可能执行中途因系统调度而切出导致执行失败）。这种称为佯败，因此使用时通常配合循环。

      ```cpp
      bool expected = false;
      extern atomic<bool> b;
      while (!b.compare_exchange_weak(excepted, true) && !excepted);
      ```

    - compare_exchange_strong 当原子变量的值不符合预期时，compare_exchange_strong 才返回 false。

    - weak 版本允许偶然出乎意料的返回（字段值与期待值一致时的返回）。但是在循环算法中是可以接受并相较于 strong 有着更好的性能。
  
- std::atomic<T *>

  指针原子类型的操作与布尔类型类似。新操作是提供算术形式的指针运算。

- 操作标准整数原子类型

  整数原子类型上可以执行

  - 常用原子操作
  - 原子计算
  - 复合赋值计算（+=）

  缺少乘除运算和位移。

- 泛化的 std::atomic<> 类模板

  对于某个自定义类型要满足一定条件才能具现化出 std::atomic\<UDT\>

  - 必须具备平实拷贝赋值操作符
  - 不含有虚函数
  - 不从虚基类派生得出
  - 由编译器代其隐式生成拷贝赋值运算符

  这种种限制就是希望可以使得原子操作的比较交换操作可以等效 memcmp 函数。

  内建浮点类型虽然可以使用原子类模板，但是在调用 compare_exchange_strong 时可能结果出人意料，即使两个浮点型的值相同，也可能比较失败（由于 IEEE 的浮点型标准，允许采用多种形式表示同一数值）

- 原子操作的非成员函数

  标准库中还定了大量的非成员函数原子操作，并冠以前缀 atomic。后缀上也有区分，带 explicit 的版本接受内存次序参数。
  
  - C++ 标准的设计，这些非成员函数都要兼容 C 语言，因此全部都只接收指针。
  - C++ 标准库还提供了非成员函数，按原子化形式访问的 std::shared_ptr 实例

### 同步操作和强制次序

---

- 两种内存模型关系

  - 先行（before-happens）和同步（synchronizes-with）

  - 同步关系只存在于原子类型的操作之间，基本思想：对变量 x 执行原子写操作 W 和原子读操作 R，且两者都有适当的标记，只要满足以下一点即为同步

    - R 读取了 W 直接存入的值
    - W 所属线程随后还执行了另一原子写操作，R 读取了后面存入的值
    - 任意线程执行一连串 “读-改-写” 操作，而第一个操作读取的值由 W 写出。

  - 先行关系和严格先行关系是在程序中确立操作次序的基本要素；用于界定哪些操作能看见其他操作产生的结果。

    - 同一语句内出现多个操作，则他们之间通常不存在先行关系。

      但并不是所有，例如逗号表达式、一个语句结果是另一个语句的参数。

    - 假设在某线程上改动多个数据，如果需要令这些改动为另一线程的后续操作所见，仅需建立一次同步关系。

- 内存次序

  - 先后一致次序（memory_order_seq_cst）

    这种内存模型无法重新编排操作次序，如果一个线程内某项操作先于另一项发生，则其他线程所见先后次序都必须如此。从同步的角度的角度考虑，同一个变量发生了存储和载入的操作，若它们都保持先后一致次序，则读取的值即为写出的值。同时先后一致次序还具有更强有力的功能，如果在载入操作后，程序又执行了别的原子操作而他们之间的次序保持先后一致，如果在系统的其他线程上，所采取的原子操作也保持先后一致，则这些操作也必须在该储存操作之后发生。<font color=red>只要服从该次序，全部线程所见的一切操作都必须服从相同次序。</font>

    若某项操作标记为 memory_order_seq_cst，则编译器和 CPU 需严格遵循源码逻辑流程的先后顺序，在相同的线程上，以该项操作为界，其后方的任何操作都不得重新编排到它的前面，而前方的任何操作不得重新编排到它后面，<font color=red>这里的任何是指任何带有内存标记的任何变量之上的任何操作。</font>

    - 但是在弱序内存模型架构的则会严重损失性能
  
  - 非先后一致顺序
  
    不同线程看到的同一组操作的次序和效果可能出现差异，同一份代码在多个线程下运行，可能由于某些线程上的操作没有显式的次序约束，因此他们有可能无法就多个事件的发生达成一致，并且在不同的 CPU 缓存和内部缓存钟，同一份内存数据也可能有不同的值。
  
  - 宽松次序
  
    采用宽松次序，原子类型上的操作不再存在同步关系，在单一线程内，同一变量上的操作仍服从先行次序，但不要求线程间存在任何次序关系。<font color=red>该内存次序的唯一要求就是在一个线程内，对相同变量的访问次序不得重新编排。</font>
  
    不同变量上的宽松原子操作可以自由的重新排列，前提是这些操作受到限定而须服从先行关系，不会产生同步关系。
  
    非必要并不建议使用宽松原子次序。
    
  - 获得-释放次序
  
    该内存模型中，原子化载入即为获取操作（memory_order_acquire），原子化的载入即为释放操作（memory_order_release），而原子化的 “读-改-写” 操作（fetch_add 和 exchange）则为获取或释放操作，或者二者皆是（memory_order_acq_rel）。这种次序在成对的读写线程之间起到同步作用，释放与获取操作构成同步关系，前者写出的值由后者获取。
  
    以下为个人理解
  
    ```cpp
    #include <thread>
    #include <atomic>
    #include <iostream>
    
    std::atomic<bool> a;
    std::atomic<int> b;
    
    void write() {
        b.store(1, std::memory_order_release); // in real code, we may do something to set b value.
        a.store(true, std::memory_order_release); // if b has updated, set a true as a flag.
    }
    
    void read() {
        while (!a.load(std::memory_order_acquire)) {}
        std::cout << b.load(std::memory_order_acquire);
    }
    
    int main() {
        a = false;
        b = 0;
        std::thread t1(write);
        std::thread t2(read);
        t1.join();
        t2.join();
        return 0;
    }
    ```
  
    这里代码模拟了一个场景，例如我们需要更新 b 的值，并将 a 作为一个标志，表明 b 已经更新完成。这里使用 `std::memory_order_release` 保证 a 更新之前，b 一定已经更新。load 操作时使用 `std::memory_order_acquire` 保证 b 的 load 操作一定在自旋等待之后执行。
  
    - 这里认为 b 其实可以设置为宽松次序，假设现在还有 c 需要更新，a 表明两个都更新完成，但是 b、c 之前其实没有先后次序，完成可以都用宽松次序实现。
    - 比较神奇的点是，将 a 的读写均改为宽松次序后，预期时有可能 b 的输出并不为 1，但是多次执行也并不能在 VS Studio 中复现这个现象。
    
    这里在对获得-释放做一些深入的说明
    
    - 当一个线程对共享变量执行写操作并使用 memory_order_release 作为内存顺序时，它保证了
      - 释放操作：该写操作之前的所有写操作，包含写入其他内存的操作，都将对其他线程可见，这表明使用该内存次序的写操作为一个同步点，它确保了之前所有写入操作都已经完成，并对其他线程可见。
    - 当一个线程对共享变量执行读操作并使用 memory_order_acquire 作为内存顺序时
      - 保证可见性：使用 memory_order_acquire 的读操作保证了其他线程中使用 memory_order_release 写入的值对当前线程可见。
      - 不保证立即性：虽然 memory_order_acquire 确保了内存顺序和可见性，但并不保证可以立即看到最新写入的值。
    - 当一个线程对共享变量执行 memory_order_release 写操作时，它释放了一个同步点，确保了所有在该写操作之前的写入操作都对其他线程可见。
    - 当另一个线程使用 memory_order_acquire 的读操作读取同一个共享变量时，他获取一个同步点，确保所有在该读操作之后的读取操作都将看到写入线程中 memory_order_release 写操作之前的所有写入。
    
  - 内存顺序整体的深入理解
  
    - 内存序需要完整标记，如果有一个 writer thread 的原子变量的写操作施放 `seq_cst` 而对应 reader thread 同一原子变量的读操作却只施放 `relaxed`，那么在语言层面上并不提供同步和全局顺序一致性的保证。
    - RMW 操作（read-modify-write，比如 `atomic::exchange()` 函数）总是能读到改动序列中最新的值。
    
  - 线程间的先行关系具有<font color=red>传递性</font>
  
  - 再探著名的 CAS 操作 `compare_exchange_strong`/`compare_exchange_weak` （ref：[cpp_reference](https://en.cppreference.com/w/cpp/atomic/atomic/compare_exchange)）
  
    不考虑内存顺序的基本函数原型
  
    ```cpp
    bool compare_exchange_strong(T &expected, T desired, some_memory_order);
    ```
  
    这个函数本质就是用于检查当前原子变量是否符合预期，如果符合预期则将 desired 赋值给他，如果不符合预期，说明预期已经陈旧，因此需要更新预期值。
  
  - memory_order_consume 的内存顺序较为特殊，暂时不考虑使用。
  
- 释放序列与同步关系

  针对同一个原子变量，可以在线程甲上对其执行储存操作，在线程乙上对其执行载入操作，从而构成同步关系，即使之前存在任意多个 “读-改-写” 操作，这成立的前提条件是，所有操作都采用合适的内存次序标记，储存与载入操作相扣成链，每次载入的值都源自前面的储存操作，则该操作链由一个<font color=red>释放序列组成</font>。操作链上可以有任意多的 “读-改-写” 操作，并且每个都有对应的 store 与之同步，他们可以是不同类型的操作，并且以不同内存次序语义标记。

- 栅栏

  栅栏用于强制施加内存次序，却无需改动任何数据。通常，他们与服从 memory_order_relaxed 次序的原子操作组合使用。栅栏操作全部通过全局函数执行，当线程运行至栅栏处时，便对线程中其他原子操作的次序产生作用。常被称之为 “内存卡” 和 “内存屏障”。

  ```cpp
  #include <atomic>
  #include <thread>
  #include <assert.h>
  
  atomic<bool> x,y;
  atomic<int> z;
  
  void write_x_then_y() {
      x.store(true, std::memory_order_relaxed);
      std::atomic_thread_fence(std::memory_order_release);
      y.store(true, std::memory_order_relaxed);
  }
  
  void read_y_then_x() {
      while (!y.load(std::memory_order_relaxed)) {}
      std::atomic_thread_fence(std::memory_order_acquire);
      if (x.load(std::memory_order_relaxed))
      {
          ++z;
      }
  }
  
  int main() {
      x = false;
      y = false;
      z = 0;
      std::thread t1(write_x_then_y);
      std::thread t2(read_y_then_x);
      t1.join();
      t2.join();
      assert(z != 0);
  }
  ```

  由于栅栏的存在，上述断言并不会触发。

- 强制非原子操作服从内存次序

  基本原理如下，如果非原子操作甲在原子操作乙之前执行，而另一线程上执行了原子操作丙，并且操作乙在操作丙之前发生，则甲也在丙之前发生。本质互斥锁的作用就是上述，通过原子操作的同步顺序关系来控制非原子的线程间同步。
