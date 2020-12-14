类比java

- 解释型（Interpreted）
- 提前编译（**ahead-of-time compilation**，AOT）
- 即时编译（**just-in-time compilation**，JIT）

| 语言/比较类型 | 解析类型    | 数据类型 | 垃圾回收 |
| ------------- | ----------- | -------- | -------- |
| Java          | JIT/AOT     | 静态     | 半自动   |
| Python        | Interpreted | 动态     | 自动     |
| C             | AOT         | 静态     | 手动     |



函数

- 第一个函数

  ```python
  >>> def say_hi():
  ...     print('Hi!')
  ...
  >>> say_hi()
  Hi!
  ```

- 缩进(Indentation)：python中用相同的缩进间隔来区分同一个代码块，一般通过Tab键缩进，相当于4个空格

- 形参（Parameters）和实参（Arguments）

  ```python
  >>> def say_hi(name):
  ...     print('Hi', name)
  ...
  >>> say_hi('Erik')
  Hi Erik
  ```

  > name为形参，‘Erik‘为实参

- 变量范围：类比java中的局部变量和成员变量

  ```python
  >>> def say_hi():
  ...    print("Hi", name)
  ...    answer = "Hi"
  ...
  >>> name = 'Erik'
  >>> say_hi()
  Hi Erik
  >>> print(answer)
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  NameError: name 'answer' is not defined
  ```

- 多参数

  ```python
  >>> def welcome(name, location):
  ...     print("Hi", name, "welcome to", location)
  ...
  >>> welcome('erik', 'this tutorial')
  Hi erik welcome to this tutorial
  ```

- 默认值和命名参数

  - 默认参数

    ```python
    >>> def welcome(name='learner', location='this tutorial'):
    ...     print("Hi", name, "welcome to", location)
    ...
    >>> welcome()
    Hi learner welcome to this tutorial
    >>> welcome(name='John')
    Hi John welcome to this tutorial
    >>> welcome(location='this epic tutorial')
    Hi learner welcome to this epic tutorial
    >>> welcome(name='John', location='this epic tutorial')
    Hi John welcome to this epic tutorial
    ```

  - 命名参数：https://www.cnblogs.com/xiaojianliu/articles/10022029.html



布尔值关键字

- True
- False

可比较

```python
1 < 2 < 3
```
# Python基础

## 内置对象类型

- 整数、浮点数

  - 浮点数运算

    ```python
    >>> import decimal
    >>> a = decimal.Decimal('0.1')
    >>> b = decimal.Decimal('0.2')
    >>> a + b
    Decimal('0.3')
    >>> a = 0.1
    >>> b = 0.2
    >>> a + b
    0.30000000000000004
    ```

- 字符和字符串

  - 字符

  - 字符编码

    - unicode

    - utf

      ```python
      >>> ord('a')
      97
      >>> bin(97)
      '0b1100001'
      >>> import sys
      >>> sys.getdefaultencoding()
      'utf-8'
      ```

  - 字符串：‘’，“”包裹的字符

    - 类型转换

    - 系列及其操作

      ```python
      >>> m = 'python'
      >>> n = 'book'
      # 字符串拼接
      >>> m + n
      'pythonbook'
      # 不同类型不能计算
      >>> m + 5
      Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
      TypeError: can only concatenate str (not "int") to str
      >>> m + '5'
      'python5'
      # 重复字符串
      >>> m * 3
      'pythonpythonpython'
      # 查看不同字符数量
      >>> len(m)
      6
      >>> len(n)
      4
      >>> name = "老七"
      >>> len(name)七
        File "<stdin>", line 1
          len(name)七
                   ^
      SyntaxError: invalid syntax
      >>> len(name)
      2
      # 是否包含
      >>> 'p' in m
      True
      ```

    - 索引和切片

- 列表

- 元组

- 字典

- 集合

- 变量与对象  

  python中万物皆是对象，变量是引用

  简单替换变量：

  ```python
  >>> a=2
  >>> b=1
  >>> a
  2
  >>> b
  1
  >>> a,b=b,a
  >>> a
  1
  >>> b
  2
  ```

  

常用内置函数：

- 查看类型：type(1)

- 查看内存地址：id(1)

- 查看对象文档：help(id)

  ```python
  help(math.pi)
  ```

- 查看包内函数：dir(math)



运算

加减乘除：+-*/

取商和余数：divmod(5,2)

```python
>>> divmod(5,2)
(2, 1)
```

取余：//

指数运算：math.pow(2,3)，2**3
