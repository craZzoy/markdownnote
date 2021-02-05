cd - 上一次目录
cd ~用户主目录（回到桌面）
cd ..
cd /



# Python

在linux和mac中是py文件变成可执行文件：chmod a+x hello.py



python数据类型

- int
- float
- 布尔值：True/False
- 空值None



输入函数：input()

```python
>>> age = input('my age is:')
my age is:20
>>> print(age)
20
```



format函数

1. 插入数据

   ```python
   >>> name = input('name:')
   name:tom
   >>> age = input('age:')
   age:30
   >>> print('你叫{}，今年{}岁了'.format(name,age))
   你叫tom，今年30岁了
   >>> print('your name is {},you are {} years old'.format(name,age))
   your name is tom,you are 30 years old
   ```

2. 数字格式化

   ```python
   >>> print("{:.2f}".format(3.1415146))
   3.14
   >>> print("{:.1f}".format(3.1415146))
   3.1
   >>> print("{:.0f}".format(3.1415146))
   3
   ```

3. 2





算术运算符

+、-、*、/、%（取模）、**（幂）、//（商向下取整）



逻辑运算符

and

or

not





条件语句if/elif/else

```python
a = int(input("Please enter a number: ")) 
## 让用户输入一个数字，用变量 a 接受
if a > 10:   ##如果 a > 10，则执行内部的代码，反之跳过
    print('a > 10')
    ## 请特别注意语句块内的空格缩进，请使用 4 个空格缩进。
    ## 请特别注意语句块内的空格缩进，请使用 4 个空格缩进。
    ## 请特别注意语句块内的空格缩进，请使用 4 个空格缩进。
elif a == 10:
    print('a == 10')
else:
    print('a < 10')
```

pass：跳过代码块

```python
>>> a = 3
>>> if a<1:
...     print("a<1")
... else:
...     pass
...
>>> #程序没有报错
```



for循环：

```python
for a in range(1,11):
	print('书桓走的第{}天，想他'.format(a))
```





驱动：

谷歌：ChromeDriver

firefox：GeckoDriver







- dto：data transfer object
- domain：domain object，可认为是do（与数据库对应）
- vo：view object
- query：query object





2020/12/25

周志明架构探索 https://icyfenix.cn/exploration/guide/quick-start.html

go语言学习 https://www.runoob.com/go/go-constants.html

service mesh https://www.servicemesher.com/istio-handbook/





轻流：

https://www.kdocs.cn/view/l/cobifGYeAFeq

https://qingflow.com/help/docs/5f48701721f11600160916b7







# Rust



# GoLang

golang实现sock5代理服务器





# EL-ADMIN

- 安全框架：Spring Security + Jwt Token
- Jpa QueryCriteria 
- 代码自动生成
- mapstruct







开源cms系统

- JEECMS：非开源





servlet filter

filter在spring mvc中的实现、调用过程

rpc mock数据