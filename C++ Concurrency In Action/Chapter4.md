## Chapter 4 并发操作的同步

---

C++ 为线程同步提供了条件变量和 future，以及线程闩和线程卡

### 等待事件或等待其他条件

---

- 凭借条件变量等待条件成立

  - C++ 标准库在头文件 <condition_variable>  中声明了 std::condition_variable 和 std::condition_variable_any 两种条件变量。两者都需要配合互斥，condition_variable 仅限于与 std::mutex 一起使用。std::condition_variable_any 则是某个类如果复合成为互斥的最低标准，则可以与之配合使用。

    ```cpp
    std::mutex mutex;
    std::queue<data> data_queue;
    std::condition_variable data_cond;
    void data_preparation_thread() {
        while (more_data_to_prepare()) {
            data const d = prepare_data();
            {
                std::lock_guard<std::mutex> lk(mutex);
                data_queue.push(data);
            }
            data_cond.notify_one();
        }
    }
    void data_processing_thread() {
        while(true) {
            std::unique_lock<std::mutex> lk(mutex);
            data_cond.wait(
                lk, []{ return !data_queue.empty(); }
            );
            data d = data_queue.front();
            data_queue.pop();
            lk.unlock();
            process(d);
            if (is_last(d)) {
                break;
            }
        }
    }
    ```

    在 wait 调用期间可以多次查验条件，并且互斥会被锁住。如果线程执行线程重新获得互斥并查验条件，而这一行为却不是直接响应准备线程的通知，称之为伪唤醒。因此如果传入 wait 函数的判定函数有副作用，则不建议使用其作为判别条件。

- 利用条件变量构建线程安全的队列

  - 需要 front 和 pop 合并成一个函数以消除固有的条件竞争。

  - 提供 try_pop 和 wait _and_pop 两个 pop 函数的变体，一个是立即返回，通过返回值来确认是否返回失败，另一个是一直等待到有数据。

    ```cpp
    #include <memory>
    template<typename T>
    class threadsafe_queue {
    public:
        threadsafe_queue() = default;
        threadsafe_queue(const threadsafe_queue &);
        threadsafe_queue &operator=(const threadsafe &);
        
        void push(T new_value);
        bool try_pop(T &value);
        std::shared_ptr<T> try_pop();
        void wait_and_pop(T &value);
        std::shared_ptr<T> wait_and_pop();
        bool empty() const;
    private:
        std::mutex mutex;
        std::queue<T> data_queue;
        std::condition_variable cond;
    }
    ```

    互斥需要用 mutable 修饰以针对 const 对象，准许其数据成发生变动。

  - notify_once 会随机触发一个线程让他从 wait 中返回。notify_all 则会通知当前所有执行 wait 而等待的线程，让他们去查验等待条件。

### 使用 future 等待一次性事件发生

---

若线程需等待某个特定的一次性事件发生，则会以恰当的方式取得一个 future，代表目标事件。接着该线程就能一边执行其他任务，一边在 future 上等待。同时，它以短暂的间隔反复查验目标事件是否已经发生。一旦目标事件发生，future 即进入就绪状态，无法重置。

- C++ 标准程序库有两种 future，独占 future (std::future)  和共享 future (std::shared_future)。同一事件仅仅允许关联唯一一个 std::future 实例，但可以关联多个 std::shared_future 实例。只要目标事件发生，与后者关联的所有实例就会同时就绪。并且，他们全都可以访问与该目标事件关联的任何数据。

- future 可以用于进程间通信，但 future 对象本身不提供同步访问。

- 从后台任务返回值

  - 只要不急需线程运算的值，就可以使用 std::async 按异步方式启动任务，从 std::async 函数处获得 std::future 对象。运行函数一旦完成，其返回值就由该对象最后持有。若要用到这个值只需要在 future 对象上调用 get 函数，当前线程就会阻塞，以便 future 准备妥当并返回该值。

    ```cpp
    #include <future>
    #include <iostream>
    int find_the_answer_to_ltuae();
    void do_other_stuff();
    int main() {
        std::future<int> the_answer = std::async(find_the_answer_to_ltuae);
        do_other_stuff();
        the_answer.get();
    }
    ```

  - 默认情况下 std::async 的具体实现会自行决定，等待 future 时是启动新线程还是同步执行任务，也可以给 std::async 补充一个参数已指定采用哪种运行方式，参数类型为 std::launch，其值可以是  std::launch::deferred 或 std::launch::async。前者指定在当前线程上延后调用任务函数，等到在 future 上调用 wait 或 get 后任务函数才会执行，后者指定必须另外开启专属线程在其上运行任务函数。也可以是 std::launch::deferred | std::launch::async 表示自行选择运行方式。若延后调用任务函数则有可能永远不会运行。
  
- 关联 future 实例和任务

  std::packaged_task<> 连结了 future 对象与函数，在执行任务时会调用关联函数，并把返回值保存在 future 的内部数据，并令 future 准备就绪。可用作线程池的构件单元以及其他任务管理方案。

  - 模板参数是函数名，例如 std::packaged_task<void()>

  - 类模板 std::packaged_task<> 具有成员函数 get_future 返回 future 实例。future 的参数类型取决于函数的返回值类型。

  - packaged_task 对象是可调用对象，具有函数调用调用运算符。可以包装在 std::function 对象内，当作线程池函数传递给 std::thread 对象。

  - 在线程间传递任务

    ```cpp
    #include <deque>
    #include <mutex>
    #include <future>
    #include <thread>
    #include <utility>
    
    std::mutex m;
    std::deque<std::packaged_task<void()>> tasks;
    bool gui_shutdown_message_received();
    void get_and_process_gui_message();
    void gui_thread() {
        while(!gui_shutdown_message_received()) {
            get_and_process_gui_message();
            std::packaged_task<void()> task;
            {
                std::lock_guard<std::mutex> lk(m);
                if (tasks.empty()) {
                    continue;
                }
                task = std::move(tasks.front());
                tasks.pop_front();
            }
            task();
        }
    }
    std::thread gui_bg_thread(gui_thread);
    template<typename Func>
    std::future<void()> post_task_for_gui_thread(Func f) {
        std::packaged_task<void()> task(f);
        std::future<void> res = task.get_future();
        std::lock_guard<std::mutex> lk(m);
        tasks.push_back(std::move(task));
        return res;
    }
    ```

- 创建 std::promise

  std::promise\<T\> 给出了一种异步求值的方法。配对的 std::promise 和 std::future 可实现下面的工作机制。等待数据的线程在 future 上阻塞，而提供数据的线程利用相配的 promise 设定关联的值，使 future 准备就绪。

  - 调用 std::promise 的成员函数 get_future 即可。promise 的值通过成员函数 set_value 设置，只要设置好，future 即准备就绪，如果在 promise 销毁时仍未设置该值，保存的数据则由异常代替。

    ```cpp
    #include <future>
    void process_connections(connection_set &connection) {
        while (!done(connection)) {
            for (connection_iterator connection = connections.begin(), end = connections.end(); connection != end; ++connection) {
                if (connection->has_incoming_data()) {
                    data_packet data = connection->incoming();
                    std::promise<payload_type> &p = connection->get_promise(data.id);
                    p.set_value(data.payload);
                }
                if (connection->has_outgoing_data()) {
                    outgoing_packet data = connection->top_of_outgoing_queue();
                    connection->send(data.payload);
                    data.promise.set_value(true);
                }
            }
        }
    }
    ```

- 将异常保存到 future 中

  若经由 std::async 调用的函数抛出异常则会被保存到 future 中，代替本该设定的值，future 随即进入就绪状态，等到其成员函数 get 被调用，储存在内的异常即被重新抛出。

  - 对于 promise 如果不想保存值，只想保存异常，则不调用 set_value 而调用 set_exception。若算法的并发实现会抛出异常，则该函数通常可用于其 catch 块中，捕获异常并装填 promise。

    ```cpp
    extern std::promise<double> some_promise;
    try {
        some_promise.set_value(calculate_value());
    }
    catch (...) {
        some_promise.set_exception(std::current_exception());
    }
    ```

    std::current_exception 用于捕获抛出的异常，也可以用 std::make_exception_ptr 直接保存新异常，而不触发抛出行为。

    ```cpp
    some_promise.setexception(std::make_exception_ptr(std::logic_error("foo "))); 
    ```

    如果能预知异常的类型，则相较于 try/catch 块，后面的方法不仅简化了代码也更有利于编译器优化代码。

  - 直接销毁 packaged_task 和 promise 也会将异常保存在与之关联的 future 中。

- 多个线程一起等待

  std::future 只能处理一对一在线程间传递数据的情况。std::future 是对异步结果的独占行为。 get 仅能被有效调用一次。shared_future 的实例则能复制出副本，因此可以持有该类的多个对象，并全部指向同一异步任务的结果。

  - 使用 shared_future 同一个对象的成员依然没有同步，如果从多个线程访问同一个对象，则必须采用锁保护以避免数据竞争。因此首选向每个线程传递 shared_future 对象的副本，被每个线程独自占有，因此这些副本则会由标准库正确的同步。
  - future 和 promise 都具备成员函数 valid，用于判断异步状态是否有效。
  - shared_future 实例依据 future 实例构造而得，使用 std::move 转移所有权。

### 限时等待

---

- 有两种超时机制可以选用

  - 迟延超时：线程根据指定的时长而继续等待。函数以 _for 结尾
  - 绝对超时：某一时间点来临之前线程一直等待。函数以 _until 结尾

- 时钟类

  每种始终都是一个类，提供四项关键信息

  - 当前时刻：now()

    ```cpp
    chrono::system_clock::now(); // return current time.
    ```

  - 时间值的类型

    - 每种时钟类都具有 time_point 的成员类型，now 的返回值类型就是 time_point。

  - 该时钟的计时单元长度

    - 时钟类的计时单元属于 period 的成员类型，表示为秒的分数形式。若时钟每秒计数 25 次，计时单元为 std::ratio<1, 25>，每 2.5 秒计时一次，表示为 std::ratio<5, 2> 计时单元并不保证其长短与运行期实际观测值一样。

  - 计时速率是否恒定（恒稳时钟）

    - 通过 is_steady 判断时钟类是否为恒稳时钟。

- 时长类

  std::chrono::duration<> 类具有两个模板参数，一个是使用何种类型表示计时单元的数量（int，short，long），另一个是表示该类中每个计时单元代表多少秒。例如使用 short 值计数的分钟时长类是 std::chrono::duration<short, std::ratio<60,  1>>，采用 double 值计数的毫秒类为 std::chrono::duration<double, std::ratio<1, 1000>>

  - 标准库预先 typedef 了 nanoseconds, microseconds, milliseconds, seconds, minutes, hours。

  - std::chrono_literals 中预定义了一些后缀运算符

    ```cpp
    using namespace chrono_literals;
    auto half_hour = 30min;
    ```

  - 时长类支持算术运算。

  - 计时单元的数量可以通过 count 函数获取。

    ```cpp
    std::chrono::millisecond(1234).count(); // return 1234.
    ```

  - 迟延超时实现

    ```cpp
    std::future<int> f = std::async(some_task);
    if (f.wait_for(std::chrono::milliseconds(35) == std::future_status::ready) {
        do_something(f.get());
    }
    ```

- 时间点类

  时间点类由类模板 std::chrono::time_point<> 表示，第一个模板参数指明所参考的时钟，第二个模板指明计时单元。时间点是一个时间跨度，始于一个称为时钟纪元的特殊时刻，且无法查询，时钟纪元有多种可能，可能多个时钟共享一个时钟纪元，也可能分别有各自的时钟纪元。虽然时钟纪元的时刻无法得知，但是可以在给定的时间点上调用， time_since_epoch，返回一个时长对象，表示时钟纪元到该时间点的长度。

  - 可以使用 std::chrono::time_point<std::chrono::system_clock, std::chrono::minutes> 指定一个时间点。

  - 可以使用时间点加减时长来获取新的时间点。

  - 若两个时间点共享同一个时钟，其可以相减来得到两个时间点之间的时长。

    ```cpp
    auto start = std::chrono::high_resolution_clock::now();
    do_something();
    auto end = std::chrono::high_resolution_clock::now();
    cout << std::chrono::duration<double, std::chrono::seconds>(end - start).count();
    ```

  - 条件变量实现限时等待

    ```cpp
    #include <condition_variable>
    #include <mutex>
    #include <chrono>
    
    using namespace std;
    
    condition_variable cv;
    bool done = false;
    std::mutex m;
    bool wait_loop() {
        auto const time_out = std::chrono::steady_clock::now() + std::chrono::milliseconds(500);
        std::unique_lock lk(m);
        while (!done) {
            if (cv.wait_until(lk, time_out) == std::cv_status::timeout) {
                break;
            }
        }
        return done;
    }
    ```

- 接受超时时限的函数

  超时时限的最简单用途是推迟线程的处理过程，推迟处理时间可以使用 this_thread::sleep_for 和 this_thread::sleep_until。休眠并非唯一能处理超时的工具，超时时限可以配合条件变量和 future 使用，甚至 timed_mutex 和 recursive_timed_mutex 也能设定超时时限。这两种锁都含有成员函数 try_lock_for 和 try_lock_unti。

  C++ 库中接受超时时限的函数（以 for 为后缀的接受 std::chrono::duration 时长，以 until 为后缀的接受 std::chrono::time_point 时间点）

  - this_thread
    - sleep_for
    - sleep_unit
  - condition_variable, condition_variable_any 
    - wait_for
    - wait_unti
  - timed_mutex, recursive_time_mutex, shared_time_mutex
    - try_lock_for
    - try_lock_until
    - try_lock_shared_for (shared_time_mutex)
    - try_lock_shared_until (shared_time_mutex)
  - unique_lock
    - try_lock_for
    - try_lock_until
  - shared_lock
    - wait_for
    - wait_until

### 运用同步简化代码

---

- 利用 future 进行函数式编程

  函数式编程是一种编程风格，函数调用的结果完全取决于参数而不依赖于外部状态， future 对象可以在线程之间传递，所以一个任务可以依赖另一个任务的结果而不必显式的访问共享数据。
  
   ```cpp
   template<typename Func, typename... Args>
   std::future<std::invoke_result_t<Func, Args...>> spawn_task(Func &&func, Args&&... args) {
       using result_type = std::invoke_result_t<Func, Args...>;
       std::packaged_task<result_type(Args...)> task(std::move(func));
       std::future<result_type> res(task.get_future());
       std::thread t(std::move(task), std::move(args)...);
       t.detach();
       return res;
   }
   ```
  
- 使用消息传递进行同步

  通信式串行进程（CSP），CSP 线程互相完全隔离，没有共享数据，采用通信管道传递消息。CSP 的理念，假设不存在共享数据，线程只接受消息，单纯的依据其反应行为，独立的对线程进行完整的逻辑判断。因此每个 CSP 线程实际上都与状态机等效，从原始状态起步，只要收到消息就按某种方式更新自身状态。

  真正的 CSP 模型没有共享数据，全部通信都经由消息队列传递，但是 C++ 线程共享地址空间。除了作为线程间通信的唯一途径，消息队列必须共享，但是细节要由程序库封装并隐藏。
  
- 符合并发技术规约的后续风格并发

  并发技术规约在命名空间 std::experimental 内

  **并发技术规约相关内容后续学习**