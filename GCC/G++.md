## G++ 常用命令

- gcc 与 g++ 区别

  - gcc 将  \*.c 结尾的文件视为 C 语言程序，\*.cpp 结尾的文件视为 C++ 语言程序
  - g++ 将 \*.c 与 \*.cpp 统一视为 cpp 文件编译
  - g++ 会自动连接标准库 STL，而 gcc 不会。因此包含 STL 的代码使用 gcc 会报错。

- `g++ test.cpp`

  生成 a.out 的可执行文件

- `g++ test.cpp -o test.out`

  生成 test.out 的可执行文件

- `g++ -c test.cpp -o test.o`

  生成 test.o 的二进制文件

- `g++ test.o -o test.out`

  生成 test.out 的可执行文件

- `g++ -g test.cpp -o test_debug.out`

  生成 test_debug.out 的 debug 调试文件

## Gdb 常用命令

```javascript
-g          // 使用参数编译可以执行文件，得到调试表
gdb ./a.out // 进入调试
list        // 列出源码
b 20        // 在 20 行设置断点
r           // 运行程序
n           // 下条指令，越过函数（VS F10）
s           // 下条指令，进入函数（VS F11）
p i         // 查看变量 i 的值
continue    // 继续执行断点后续指令
finish      // 结束当前函数调用
quit        // 退出 gdb 调试
```

