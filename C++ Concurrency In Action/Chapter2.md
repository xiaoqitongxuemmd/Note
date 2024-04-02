## Chapter 2 线程管控



### 线程的基本管控

---

每个 C++ 程序都至少含有一个线程，即运行 main 的线程，由 C++ 运行时系统启动，随后，程序可以发起更多线程，这些新线程与起始线程并发运行，当 main 返回时，程序就会退出。

- 发起线程

  线程通过构建 `std::thread` 对象而启动，该对象指明线程要运行的任务，任何可调用类型，都适用于 `std::thread`。
  
  ```cpp
  class A {
      void operator();
  }
  void func() {
      A a;
      std::thread my_thread(a);
  }
  ```
  
  一旦启动了线程，我们就需要明确时要等待它结束（与之汇合），还是由它独立运行。，
  
- `detach` 为设定不等待线程，`join` 为等待线程。

- 只要调用了 `join` 隶属于该线程的任何储存空间即会即会因此清除，并且一个线程只能汇合一次。调用过 `join` 的线程其 `joinable` 函数会返回 false。

- 如果线程启动以后有异常抛出，而 `join` 尚未执行，则该 `join` 的调用会被略过。

  ```cpp
  class A; // a callable object.
  void func() {
      A a;
      std::thread my_thread(a);
      try {
          // do something.
      }
      catch(...){
          my_thread.join();
          throw;
      }
      t.join();
  }
  ```

  上述方案并不是一个好的处理方案。

  ```cpp
  class thread_guard {
  public:
      explicit thread_guard(std::thread &t)
          : _t(t) {}
      thread_guard(const thread_guard &) = delete;
      thread_guard &operator=(const thread_guard &) = delete;
      ~thread_guard() noexcept {
          if (_t.joinable) {
              _t.join();
  		}
      }
      
  private:
      std::thread &_t;
  }
  ```

  这里使用一个 thread_guard 来负责保证线程的汇合，使用 >RAII 的编程模式。

- 调用 std::thread 对象的成员函数 `detach` 会令程序在后台运行，并且无法与之通信。分离的进程通常为守护进程，这种线程往往长时间运行，几乎在应用程序的整个生命周期内，他们都会一直运行，以执行后台任务，例如文件系统监控，优化数据结构等。另有一种模式，就是在分离线程执行启动后可自主完成的任务。

- 只有在 `joinable` 为 true 时才能调用 `detach` 。

- 线程具有内部存储空间，参数会先按照默认的方式复制到该处，新创建的执行线程才能直接访问他们。**即使是引用**，上述依然会发生。如果向线程传入临时变量的指针，很可能导致上述复制行为还未完成，主线程已经析构临时变量，触发未定义的行为。

- 使用 `std::ref` 可以包装变量使其可以引用传入线程。

  ```cpp
  void t_func(int &);
  void func() {
      int a = 0;
      std::thread t(t_func, a);
      t.join();
  }
  ```

  上述代码会编译失败，虽然 `t_func` 期望参数以引用的方式传递，但是线程的建立全然不知，因此仍会拷贝到线程作为右值，而 `t_func` 会因此非 const 引用右值导致编译失败。这时就需要使用 `std::ref`

  ```cpp
  void t_func(int &);
  void func() {
      int a = 0;
      std::thread t(t_func, std::ref(a);
      t.join();
  }
  ```

- thread 线程的参数传入和 bind 类似，函数会被隐式转换为函数指针，但是成员函数不会，需要显式指明，并给出一个类对象作为第一个参数。

  ```cpp
  class A {
      void func();
  }
  
  int main() {
      A a;
      thread t(&A::func, &a);
  }
  ```

- `std::thread` 类的实例可以移动但不能复制。

- 移交线程归属权

  ```cpp
  void func1();
  void func2();
  std::thread t1(func1);
  std::thread t2(func2);
  std::thread t3;
  t3 = std::move(t1); // move t1 to t3.
  t2 = std::move(t3); // it will call std::terminate() and exit the program.
  ```

### 在运行时选择线程数量

---

- `std::thread::hardware_concurrency` 它的返回值是一个指标，表示程序在各次运行中可真正并发的数量。
-  因为无法从线程中**直接**返回值，因此需要向线程传入引用。

### 识别线程

- 线程 id 所属型别是 `std::thread::id`，有两种获取方法，一是在与线程相关的 `std::thread` 对象上调用 `get_id()`，如果该线程没有关联任何对象则返回一个 `std::thread::id` 对象，以默认形式构造表示线程不存在。当前线程的 id 可以通过调用 `std::this_thread::get_id()` 获取。

- C++ 标准库允许任意判断两个线程 id 是否相同。

- 标准库的 hash 模板可以具体化成 std::hash\<std::thread::id\>  