#### oracle查看计划顺序
先从最开头一直往右看，直到看到最右边的并列的地方，对于不并列的，靠右的先执行：对于并列的，靠上的先执行。 即并列的缩进块，从上往下执行，非并列的缩进块，从下往上执行。

#### 计划详解
1. index unique scan 通过唯一索引条件查找出对应索引的rowid
2. 通过查询的rowid获取获取所要的数据
3. table access full 全表扫描 没有索引的条件列也为全表扫描（花费时间相对较大，应尽量避免此类检索）