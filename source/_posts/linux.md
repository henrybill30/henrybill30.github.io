---
title: linux命令总结
date: 2021-09-15 11:25:29
tags: [Linux命令]
categories: Linux
---
# Linux命令

## ls  ——— 展示所有指定目录下所有文件或文件夹

- -a  展示所有
- -l   按长模式展示
- -t   按时间顺序排序
- -r   倒序
- -h  与-l配合使用，适合人类阅读
- -S  按大小排序

## cd ——— 切换目录

## pwd ——— 当前目录路径

## less ——— 查看某个文本文件内容

- Page Up或 b  向上一页
- Page Down或空格  向下一页
- up arrow 向上一行
- down arrow 向下一行
- /字符  向前查找某字符
- n 查找上面指定的下一个字符
- h 显示帮助屏幕
- q 推出
- G 移动到最后一行
- g 移动到第一行

![Linux%E5%91%BD%E4%BB%A4%207f7dde58bce74b60a400633056adc75e/Untitled.png](Untitled.png)

![Linux%E5%91%BD%E4%BB%A4%207f7dde58bce74b60a400633056adc75e/Untitled%201.png](Untitled1.png)

![Linux%E5%91%BD%E4%BB%A4%207f7dde58bce74b60a400633056adc75e/Untitled%202.png](Untitled2.png)

## 通配符

- *  匹配任意个字符（0～无穷）
- ?  匹配任意一个字符
- [字符]  匹配任意一个中括号内字符
- [!字符]  匹配任意一个非中括号内字符
- [[:class:]]  匹配任意一个指定字符类的字符
    - [:alnum:]  任意一个字母或数字
    - [:alpha:]  任意一个字母
    - [:digit:]  任意一个数字
    - [:lower:]  任意一个小写字母
    - [:upper:]  任意一个大写字母
