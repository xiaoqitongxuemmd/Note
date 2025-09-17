# Chapter 2 GUI 程序设计基础
- 使用 Qt Creator 可以可视化创建 widget.ui，这个是如何被引入 Qt 程序的
  - 当你构建（build）项目时，Qt 的构建系统（qmake / CMake）会调用 uic 工具，把 widget.ui 自动转化为一个 C++ 头文件，名字一般是：ui_widget.h，这个头文件定义了一个 Ui::Widget 类
  - C++ 通过类似如下代码，创建 ui 窗体
    ```C++
    // widget.cpp
    #include "widget.h"
    #include "ui_widget.h"   // 这里就是引入自动生成的头文件

    Widget::Widget(QWidget *parent)
        : QWidget(parent), ui(new Ui::Widget)
    {
        ui->setupUi(this);  // 把 UI 元素加载到当前窗口
    }
    ```
- 目前 Qt 程序推荐使用 CMake 构建项目
- Qt 整体是**事件驱动**的，依赖事件循环和信号完成对事件的响应
- 使用可视化创建的 ui 文件同样可以在代码中访问其中成员，修改其参数
- qrc 是 qt 的资源管理文件，可以通过设置 prefix 实现对资源的分组
- 伙伴关系是指界面上一个 label 和一个具有输入焦点的组件相关联，利用伙伴关系，可以设置通过快捷键来快速将输入焦点切换到某个组件。
- Tab 顺序编辑状态则是可以指定在程序运行时，按 Tab 的组件切换顺序，**只有具有输入焦点的组件才有 Tab 顺序**
- 信号与槽（signal & slot）
  - 信号是在特定情况下被发射的通知，GUI 设计工作的内容就是对界面上各组件的信号进行响应
  - 槽就是对信号进行响应的函数
  - 一个信号可以连接多个槽，多个信号也可以连接同一个槽
  - 信号可以连接另一个信号
  - **在使用信号与槽的类中，必须在类的定义中插入宏 Q_OBHJECT**
- Qt 提供了元对象编译器（MOC）构建项目时，项目中的头文件会被 MOC 预编译
  - 窗口 UI 文件会被 UIC 编译
  - 资源文件会被 RCC 编译