## 关于${}和#{}传参
1. #{} 传参,sql语句解析会加上""。
如：

```
select * from report where orgname= #{orgname}
```
结果：

```
select * from report whereorgname= ‘花果山’
```

2. ${} 传参,mybatis不会修改或转义字符串。
如：

```
select * from report  order by  ${sortField}
```
结果：

```
select * fromreport  order by  orgname
```
