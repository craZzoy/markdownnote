#### 用法DEMO:  

```
DBMS_JOB.SUBMIT(:jobno,//job号   
               'your_procedure;',//要执行的过程   
                trunc(sysdate)+1/24,//下次执行时间
                'trunc(sysdate)+1/24+1'//每次间隔时间);   
     删除job:dbms_job.remove(jobno);   
     修改要执行的操作:job:dbms_job.what(jobno,what);   
     修改下次执行时间：dbms_job.next_date(job,next_date);   
     修改间隔时间：dbms_job.interval(job,interval);   
     停止job:dbms.broken(job,broken,nextdate);   
     启动job:dbms_job.run(jobno);   
```
##### 附：调用语句和参数说明： 

```
dbms_job.submit( job out binary_integer,

what　　　　　　　in　　　archar2,
next_date　　　 　in　　　date，
interval　　　　　in　　　varchar2,
no_parse　　　　　in　　　boolean)

其中：
●    job：输出变量，是此任务在任务队列中的编号；
●    what：执行的任务的名称及其输入参数；
●    next_date：任务执行的时间；
●    interval：任务执行的时间间隔。
```

#### 示例：

```
declare   
                jobid     number;   
                v_sql     varchar2(2000);   
    begin   
                v_sql:='begin   
                                      if     to_char(sysdate,''HH24:MI'')=''15:30''     then   -- 15:30执行
                                            insert     into     rjck.rkjl(cksj)     select    cksj     from     wzcs.ckjl;   
                                            dbms_output.put_line(''inserted     success'');   
```
