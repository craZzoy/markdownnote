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