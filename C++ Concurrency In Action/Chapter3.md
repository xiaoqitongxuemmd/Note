## Chapter 3 在线程间共享数据

### 线程间共享数据的问题

---

- 多线程共享数据的问题多由数据改动引发。
- 防止数据恶性竞争最简单的就是采用保护措施包装数据结构，确保不变量被破坏时，中间状态只对执行改动的线程可见。
- 防止恶性竞争的另一种方式是通过将修改数据结构及其不变量的设计改变为由一连串不可拆分的改动完成数据变更，称为无锁编程。
- 还有一种防止恶性竞争的方法是将修改数据结构当作事务，类似于数据库在一个事务内完成更新，把需要执行的数据读写操作视为一个完整序列，先用事务日志储存记录，再把序列当成单一步骤提交运行，若别的线程改动了数据而令提交无法完整执行，则事务重新开始，称为软件事务内存（STM）。

### 用互斥保护共享数据

---

- 用互斥保护共享数据，访问一个数据结构前，先锁住与数据相关的互斥，访问结束后在解锁互斥。

- C++ 线程库保证了，一旦有线程锁住了某个互斥，若其他线程试图在给它加锁则需等待，直到最初成功枷锁的线程把该互斥解锁。

- 互斥是 C++ 最通用的共享数据保护措施之一，但非万能的。

- C++ 中通过 `std::mutex` 创建互斥，调用成员函数 `lock` 对其加锁，`unlock` 对其解锁，但并不适合直接使用，C++ 标准库提供了类模板 `std::lock_guard` 针对互斥类融合实现了 RAII 手法：构造加锁，析构解锁。

  ```cpp
  #include <list>
  #include <mutex>
  std::list<int> a_list;
  std::mutex a_mutex;
  void add_to_list(int &new_val) {
      std::lock_guard<std::mutex> a_guard(a_mutex);
      a_list.push_back(new_val);
  }
  ```

- 正常情况下会将互斥与受保护的数据组成一个类。

- 如果成员函数返回受保护数据的指针或者引用，即使锁定互斥也可以访问和修改数据。这种情况只能由程序员自己保证，不得向锁所在的作用域之外传递指针和引用，包括返回值和作为参数传递。

- stack 和 queue 将 pop 函数与 top 函数分开是为了处理系统负载过重导致的内存分配失败问题，如果将 top 和 pop 函数合并为一个操作，则有可能会在返回时分配失败，而这时无论是返回值还是 stack 与 queue 本身都再也找不到数据。

- 线程安全的 stack 定义

  ```cpp
  #include <exception>
  #include <memory>
  struct empty_stack : std::exception {
      const char *what() const noexcept;
  }
  template<typename T>
  class threadsafe_stack {
  public:
      threadsafe_stack();
      threadsafe_stack(const threadsafe_stack &);
      threadsafe_stack &operator=(const threadsafe_stack &) = delete;
      void push(T new_value);
      std::shared_ptr<T> pop();
      void pop(T &value);
      bool empty() const;
  }
  ```

- 类的构造函数在函数体内进行复制操作，确保互斥的锁定会横跨整个复制过程。

- 控制加锁的粒度防止粒度过小导致的需要保护的操作没有完全覆盖而出现的恶性条件竞争。如果锁的粒度太大，则会导致并发的性能下降。

- 如果对某项操作锁住多个互斥，则会导致死锁，正好是条件竞争的反面。

- 对于死锁的防范建议通常是始终按照相同顺序对两个互斥加锁。但是该建议并不总是有效的。C++ 给出 `std::lock` 函数，可以同时锁住多个互斥而没有发生死锁的风险。

  ```cpp
  class some_big_object;
  void swap(some_big_object &lhs, some_big_object &rhs);
  class X {
  public:
      friend void swap(some_big_object &lhs, some_big_object &rhs);
      
      X(const some_big_object &sd)
          : some_detail(sd) {}
      void swap(X &lhs, X &rhs) {
          if (&lhs == &rhs) {
              return;
          }
          std::lock(lhs.m, rhs.m);
          std::lock_guard<std::mutex> lock_a(lhs.m, std::adopt_lock);
          std::lock_guard<std::mutex> lock_b(rhs.m, std::adopt_lock);
          // std::scoped_lock guard(lhs.m, rhs.m); // implicit template parameter deduction.
          swap(lhs.some_detail, rhs.some_detail);
      }
  private:
      some_big_object some_detial;
      std::mutex m;
  }
  ```

- scoped_lock 可以管理多个互斥量并处理死锁问题，内部调用 std::lock，通过死锁避免算法策略实现。

- 避免死锁的补充准则

  - 避免嵌套锁。如果已经持有锁就不要尝试获取第二个锁。万一需要多个锁，应采用 `std::lock` 函数。
  - 一旦持锁，就须避免调用由用户提供的程序接口。
  - 依从固定顺序获取锁。如果多个锁是绝对必要的，却无法通过 `std::lock` 在一步操作中全部获取，我们只能退而求其次，在每个线程内部依据固定顺序获取这些锁。
  - 按层级加锁。
  - 尽量避免嵌套锁，如果要等待线程，需要对线程规定层级，使得每个线程仅等待层级更低的线程。
  
- 使用 `std::unique_lock` 灵活加锁。

  `std::unique_lock` 放宽了不变量的成立条件，该对象不一定始终占用与之关联的互斥

  - 其构造函数可以传入 `std::adopt` 实例，借此指明对象管理互斥上的锁，可以传入 `std::defer_lock` 实例，使互斥在构造完成时处于无锁状态。

  - unique_lock 中还有一个内部标志，表明关联的互斥是否正被该类占据，用于保证析构可以正常 unlock，具有所有权的 unqiue_lock 在释放时必须解锁，反之则不能。

  - unique_lock 会比 lock_guard 有轻微的性能损失，因此通常在两个场景下使用。延迟加锁与转移锁的所有权。

  - 在不同作用域之间转移互斥所有权

    ```cpp
    std::unique_lock<std::mutex> get_lock() {
        extern std::mutex some_mutex;
        std::unique_lock<std::mutex> lk(some_mutex);
        prepare();
        return lk;
    }
    void process() {
        std::unique_lock<std::mutex> lk(get_lock());
        do_something();
    }
    ```

    该例子的 process 中调用了 get_lock 能直接接收锁的所有权。这时准备工作已经完成，do_something 可以进一步操作并且其他线程也不能改动数据。

### 保护共享数据的其他工具

---

- C++ 提供了一套机制，仅在初始化过程中保护共享数据。C++标准库中提供了 std::once_flag 类和 std::call_once 函数，以专门处理该情况。上述代码先锁住互斥，再显式检查指针。所有线程共同调用 std::call_once 函数，确保调用返回时，指针初始化由其中某个线程安全且唯一完成。同步数据由 std::once_flag 储存。

  ```cpp
  std::shared_ptr<some_resource> resource_ptr;
  std::once_flag resource_flag;
  void init_resource() {
      resource_ptr.reset(new some_resource);
  }
  void func() {
      std::call_once(resource_flag, init_resource);
      resource_ptr->do_something();
  }
  ```

  - std::call_once 接收可调用对象。

  - std::once_flag 的实例既不可复制也不可移动。

  - 如果把局部变量声明为静态数据，那样便有可能让初始化过程出现条件竞争。C++11 规定初始化只会在某个线程单独发生，在初始化完成前，其他线程不会越过静态变量的声明而继续运行。

- 保护甚少更新的数据结构，需要采用一种新的互斥称为读写互斥：允许单独一个写线程进行完全排他的访问，也允许多个读线程共享数据或并发访问。

  - C++17 提供两种新的互斥，std::shared_mutex 和 std::shared_timed_mutex。

  - 更新操作可用 std::lock_guard\<std::shared_mutex\> 和 std::unique_lock\<std::shared_mutex\> 锁定，保证了访问的排他性。

  - 无需更新数据的线程可以改用共享锁，std::shared_lock\<std::shared_mutex\> 锁定，共享锁如果已经被某些线程所持有，若别的线程试图获取排他锁，就会发生阻塞，直到那些线程全部释放该共享锁，反之，如果任一线程持有排他锁，则其他线程全都无法获取共享锁和排他锁。直到持有排他锁的线程释放。

    ```cpp
    #include <map>
    #include <string>
    #include <mutex>
    #include <shared_mutex>
    class dns_entry;
    class dns_cache {
    public:
        dns_entry find_entry(const std::string &domain) const {
            std::shared_lock<std::shared_mutex> lk(entry_mutex);
            std::map<std::string, dns_entry>::const_iterator const it = entries.find(domain);
            return (it == entries.end()) ? dns_entry() : it->second();
        }
        void update_or_add_entry(const std::string &domain, const dns_entry &dns_details) {
            std::lock_guard<std::shared_mutex> lk(entry_mutex);
            entries[domain] = dns_details;
        }
    private:
        std::map<std::string, dns_entry> entries;
        mutable std::shared_mutex entry_mutex;
    }
    ```

- 递归加锁

  - 标准库提供了 std::recursive_mutex 允许同一线程对某一互斥的同一实例多次加锁。同时，调用几次加锁就需要调用几次解锁后才能被其他线程加锁。
  - **递归锁并不是好的设计选择**，大多数情况下需要使用递归锁的场景都是设计本身存在问题。
