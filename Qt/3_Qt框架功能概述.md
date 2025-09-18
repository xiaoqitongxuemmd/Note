# Chapter 3 Qt 框架功能概述

- Qt 框架中的模块分为两大类
  - Qt Essentials：Qt 框架的基础模块
  - Qt Addons：Qt 框架的附加模块，常见的有
    - Qt Charts：绘制各种统计图表
    - Qt 3D：3D 图形与渲染支持
    - Qt Multimedia：多媒体处理（音频、视频、相机）
    - Qt XML：XML 解析
    - Qt WebEngine：基于 Chromium 的浏览器引擎
    - Qt Data Visualization：3D 数据可视化
    - Qt Remote Objects：远程对象通信
- Qt 编译时使用 lib 其实是 import library（导入库），这个 lib 只提供符号表，告诉编译器某个函数在某个 dll 中

## 元对象系统

Qt 的元对象系统的功能建立在以下 3 个方面
- QObject 类是所有使用元对象系统的类的基类
- 必须在一个类的开头部分插入宏 Q_OBJECT
- MOC 为每个 QObject 的子类提供必要的代码来实现元对象系统的特性

## 信号与槽

- 如果我们要自定义一个操作，需要自定义信号和槽，信号是通过重载父类事件处理函数实现发射与信号传递
  ```C++
  class MyButton : public QPushButton {
      Q_OBJECT
  
  public:
      MyButton(QWidget *parent = nullptr) : QPushButton(parent) {}
  
  signals:
      void clicked(int x, int y); // 自定义信号，传递点击位置的坐标
  
  protected:
      void mousePressEvent(QMouseEvent *event) override {
          QPushButton::mousePressEvent(event); // 调用父类的处理逻辑
          emit clicked(event->pos().x(), event->pos().y()); // 触发信号    并传递参数
      }
  };
  ```
- connect 函数最后都有一个参数 type，表示信号与槽的关联方式，有几种取值
  - AutoConnection：如果信号的接收者与发射者在同一个线程，就使用 DirectConnection 的连接方式，
  否则使用 QueuedConnection 的连接方式。
  - DirectConnection：信号被发射时槽函数立即运行，槽函数与信号在同一个线程中。
  - QueuedConnection：在事件循环回到接收者线程后运行槽函数，槽函数与信号在不同的线程中
  - BlockingQueuedConnection：与 QueuedConnection 类似，区别是信号线程会阻塞直到槽函数运行完毕

- 槽函数中可以使用 sender() 获取发射对象

## 容器类

- 顺序容器：
  - QList/QVector/QStack/QQueue
  - Qt6 中 QVector 就是 QList，而 QList 的底层实现采用了 Qt5 中 QVector 的机制
- 关联容器：
  - QSet/QMap/QMultiMap/QHash/QMultiHash
  - QSet 基于哈希表的集合模板类，储存顺序不确定
- Qt 的很多容器类和值类采用隐式共享的优化机制（implicit sharing），在多个对象共享同一份数据时，避免不必要的内存复制，提高效率；只有当某个对象要修改数据时，才真正拷贝一份副本。

## 其他常用基础类

- QVariant 是 Qt 中的一种万能数据类型，可以存储任何类型的数据。
- QFlags<Enum> 是一个模板类，其中 Enum 是枚举类型，QFlags 用于定义枚举值的或运算组合
- QRandomGenerator 是随机数发生器，