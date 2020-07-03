#### 查询表所有字段
```
select ',' || lower(column_name)
  from USER_tab_columns
 where table_name = upper('TB_MD_PRICE_PARAM')
 order by column_id;
```
#### 查询快码

```
select DICT_CODE  from TB_SM_DICTIONARY
where DICT_TYPE = 'CALC_CODE' and DICT_TITLE = '****'
```

#### 切分出数字列表
```
select REGEXP_SUBSTR(str, '[^/*&!+-]+', 1, LEVEL) STR
  from (select '*1+2/3-4' str from dual)
CONNECT BY LEVEL <= REGEXP_COUNT(str, '[^/*&!+-]+');
```

#### 找到第几个出现的字符
```
SELECT REGEXP_SUBSTR('*1-9+9/0+5','[\+\-\*/]',1,1) FROM DUAL
```


#### 解决死锁

```
select a.sid, a.serial#, a.username, b.owner, b.object, b.type
from   v$session a, v$access b
where  a.sid = b.sid and b.OBJECT like '%cust%';

alter system kill session   '3379,1'

select * from V$LOCKED_OBJECT lo,dba_objects do
where lo.OBJECT_ID = do.OBJECT_ID
```

1. 查找死锁的进程：

```
SELECT s.username,l.OBJECT_ID,l.SESSION_ID,s.SERIAL#,
l.ORACLE_USERNAME,l.OS_USER_NAME,l.PROCESS 
FROM V$LOCKED_OBJECT l,V$SESSION S WHERE l.SESSION_ID=S.SID;
```
2. kill掉这个死锁的进程：
```
alter system kill session ‘sid,serial#’;
```
3. 如果还不能解决：
```
select pro.spid from v$session ses,v$process pro where ses.sid=XX and ses.paddr=pro.addr;
```
（其中sid用死锁的sid替换，其中spid是这个进程的进程号。）


strsplit和


#### **oracle**分页语句
常用格式：
1. 效率高的格式：

```
SELECT * FROM  
(  
SELECT A.*, ROWNUM RN  
FROM (SELECT * FROM TABLE_NAME) A  
WHERE ROWNUM <= 40  
)  
WHERE RN >= 21
```
原因：这是由于CBO 优化模式下，Oracle可以将外层的查询条件推到内层查询中，以提高内层查询的执行效率。对于这个查询语句，第二层的查询条件WHERE ROWNUM <= 40就可以被Oracle推入到内层查询中，这样Oracle查询的结果一旦超过了ROWNUM限制条件，就终止查询将结果返回了。

2. 效率低的格式：

```
SELECT * FROM  
(  
SELECT A.*, ROWNUM RN  
FROM (SELECT * FROM TABLE_NAME) A  
)  
WHERE RN BETWEEN 21 AND **40**
```

###创建系列 
create sequence TB_MD_ORDER_HEAD_sequence
increment by 1
start with 1
maxvalue 999999999


#### 快速建表

```
select 'private String '||lower(column_name)||';'  from USER_tab_columns where table_name = upper('TB_BPM_warn_Setting') order by column_id ;
```



mybaties 模糊查找
```
CONCAT(CONCAT('%', #{period_name}), '%')
```



#### 查取重复数据
```
SELECT *
  FROM t_info a
 WHERE ((SELECT COUNT(*) FROM t_info WHERE Title = a.Title) > 1)
 ORDER BY Title DESC
```

```
select a.loginname from ad_allperson_info a
group by a.loginname
having count(1)>1;
```


#### 查询sql历史记录

```
select * from v$sql
```

#### oracle删除重复数据
1. 查找表中多余的重复记录，重复记录是根据单个字段（Id）来判断

```
select * from 表 where Id in (select Id from 表 group byId having count(Id) > 1)
```

2. 删除表中多余的重复记录，重复记录是根据单个字段（Id）来判断，只留有rowid最小的记录

```
DELETE from 表 WHERE (id) IN ( SELECT id FROM 表 GROUP BY id HAVING COUNT(id) > 1) AND ROWID NOT IN (SELECT MIN(ROWID) FROM 表 GROUP BY id HAVING COUNT(*) > 1);
```

3. 查找表中多余的重复记录（多个字段）

```
select * from 表 a where (a.Id,a.seq) in(select Id,seq from 表 group by Id,seq having count(*) > 1)
```

4. 删除表中多余的重复记录（多个字段），只留有rowid最小的记录

```
delete from 表 a where (a.Id,a.seq) in (select Id,seq from 表 group by Id,seq having count(*) > 1) and rowid not in (select min(rowid) from 表 group by Id,seq having count(*)>1)
```

5. 查找表中多余的重复记录（多个字段），不包含rowid最小的记录

```
select * from 表 a where (a.Id,a.seq) in (select Id,seq from 表 group by Id,seq having count(*) > 1) and rowid not in (select min(rowid) from 表 group by Id,seq having count(*)>1)
```
6.定时任务

```
DELETE FROM QRTZ_BLOB_TRIGGERS;
DELETE FROM QRTZ_CALENDARS;
DELETE FROM QRTZ_CRON_TRIGGERS;
DELETE FROM QRTZ_FIRED_TRIGGERS;
DELETE FROM QRTZ_LOCKS;
DELETE FROM QRTZ_PAUSED_TRIGGER_GRPS;
DELETE FROM QRTZ_SCHEDULER_STATE;
DELETE FROM QRTZ_SIMPLE_TRIGGERS;
DELETE FROM QRTZ_SIMPROP_TRIGGERS;
DELETE FROM QRTZ_TRIGGERS;
DELETE FROM QUARTZ_JOB_MANAGE;
DELETE FROM QRTZ_JOB_DETAILS;

SELECT * FROM QRTZ_BLOB_TRIGGERS;
SELECT * FROM QRTZ_CALENDARS;
SELECT * FROM QRTZ_CRON_TRIGGERS;
SELECT * FROM QRTZ_FIRED_TRIGGERS;
SELECT * FROM QRTZ_LOCKS;
SELECT * FROM QRTZ_PAUSED_TRIGGER_GRPS;
SELECT * FROM QRTZ_SCHEDULER_STATE;
SELECT * FROM QRTZ_SIMPLE_TRIGGERS;
SELECT * FROM QRTZ_SIMPROP_TRIGGERS;
SELECT * FROM QRTZ_TRIGGERS;
SELECT * FROM QUARTZ_JOB_MANAGE;
SELECT * FROM QRTZ_JOB_DETAILS;


select qt.job_name,
       fnd_util.get_lookup_meaning('TRIGGER', qt.job_name) job_description,
       qt.trigger_name,
       qct.cron_expression,
       to_char(qt.next_fire_time / (1000 * 60 * 60 * 24) +
               to_date('1970-01-01 08:00:00', 'yyyy-mm-dd hh24:mi:ss'),
               'yyyy-mm-dd hh24:mi:ss') next_fire_time,
       to_char(qt.prev_fire_time / (1000 * 60 * 60 * 24) +
               to_date('1970-01-01 08:00:00', 'yyyy-mm-dd hh24:mi:ss'),
               'yyyy-mm-dd hh24:mi:ss') prev_fire_time,
       to_char(qt.start_time / (1000 * 60 * 60 * 24) +
               to_date('1970-01-01 08:00:00', 'yyyy-mm-dd hh24:mi:ss'),
               'yyyy-mm-dd hh24:mi:ss') start_time,
       to_char(qt.end_time / (1000 * 60 * 60 * 24) +
               to_date('1970-01-01 08:00:00', 'yyyy-mm-dd hh24:mi:ss'),
               'yyyy-mm-dd hh24:mi:ss') end_time,
       decode(qt.trigger_state,
              'ACQUIRED',
              '正在运行',
              'WAITING',
              '等待',
              'PAUSED',
              '暂停',
              '其他') trigger_state
  from qrtz_triggers qt, qrtz_cron_triggers qct
  where qt.trigger_name = qct.trigger_name
```
