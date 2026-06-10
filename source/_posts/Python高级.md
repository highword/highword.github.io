---
title: Python面向对象
date: 2024-9-27 16:37:40
cover: https://mypersonaldata.oss-cn-shanghai.aliyuncs.com/img/202509272206116.png
categories:
  - Development
  - Python
tags:
  - Computer Science
  - Note
  - Development
  - python
---

# Python高级
- 装饰器语法
- 解包语法
    - a, b, c = list
- 列表推导式语法
    - [expression for item1, item2 in list if condition]
- lambda匿名函数
    - 向函数传入函数或让函数返回函数最终实现代码的解耦合
- 单例模式
    - 让一个类只能创建出唯一的实例
    - 保证一致，状态共享
    - 项目中使用的数据库连接池对象和配置对象通常都是单例
- 元类 Metaclass
    - 用来创建类的类
    - 将元类指定为类的metaclass关键字参数
    - 仅在必要时使用元类。元类是高级编程工具，通常不需要在日常编程中使用。
- 线程安全，进程安全
- 垃圾回收
- 去重
- Python不同的解释器的特性
    - Cpython
        - 性能优化
            - 把频繁使用的整数对象区间[-5, 256]用一个叫 **small_ints** 的对象池缓存起来，一直使用
            - 同一个代码块中已经存在一个值与其相同的整数对象，则直接引用该对象，否则才建新对象
    - PyPy