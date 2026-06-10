---
title: Python基础
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

# Python基础
- 特性
    - 面向对象 -- 一切皆对象
    - 解释型语言 -- 无需编译
    - 跨平台性
    - 本身解释运行速度较慢
- 基础功能
    -  [Python 面向对象]("https://mubu.com/app/edit/home/47cl6eAb100")
    - 判断
        - is 判断地址
        - == 判断值
    - 赋值
        - 赋值
        - 浅拷贝
        - 深拷贝
    - 语法糖
        - 装饰器 @
        - 迭代器 iterator
            - 一种机制，实现迭代协议的两个方法的类对象可以使用
            - __iter__方法
            - __next__方法
        - 生成器 generator
            - 本质是迭代器通过函数实现
            - yield关键字类似return
            - 只从头到尾迭代一次，若找不到下一个yield会报StopIteration异常
            - 需先执行得到生成器然后赋值给变量，用next()执行
        - 匿名函数 lambda
    - 函数
        - 参数
            - 位置参数
                - *args 元组
            - 关键字参数
                - **kwargs 字典
        - 参数传递
            - 在 Python 中，本质都是拷贝对象的引用（指针）传递
    - 字符串处理
    - 列表处理
        - 切片[start:end:step]
            - 包含start不包含end
            - 默认[0:length:1]
            - 负数索引表示倒数第几个元素
            - 负数步长表示从尾向头