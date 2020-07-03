#### 首先使用管理员账号来创建一个路径，并将这个路径进行授权给相应的用户。

```
CREATE OR REPLACE directory file_dir AS 'D:\test'; --windows系统路径
grant READ, WRITE ON directory file_dir TO customerUser; --路径授权，添加对路径读、写权限
grant EXECUTE ON utl_file TO customerUser; --utl_file包授权，添加执行权限
```

#### 保存数据到文件 
分三步：打开文件，写入数据，关闭文件

```
--====保存数据到文件====--
DECLARE
  v_file  utl_file.file_type;
  v_input CLOB := '我是要保存到文件里面的内容';
BEGIN
  --打开文件
  v_file := utl_file.fopen('FILE_DIR', 'data.txt', 'w');
  --将数据写入到文件中
  utl_file.put_line(v_file, v_input);
  --关闭文件
  utl_file.fclose(v_file);
EXCEPTION
  WHEN utl_file.access_denied THEN
    dbms_output.put_line('拒绝访问!');
  WHEN OTHERS THEN
    dbms_output.put_line('SQLERRM: ' || SQLERRM);
END;
```
打开模式有三种：
W：是写文件，文件不存在则创建文件，如果文件存在则覆盖之前文件的内容 
A：是追加，若文件不存在则创建文件，文件存在，则在文件结束后面追加内容 
还有一种在读文件的时候会使用 
R：读文件，文件不存在报错，文件存在则将文件内容读出来。

#### 读取文件中的数据 

```
--====读取文件中的数据====--
DECLARE
  v_file   utl_file.file_type;
  v_output CLOB := '';
BEGIN
  --打开文件
  v_file := utl_file.fopen('FILE_DIR', 'data.txt', 'r');
  --将文件的数据写入到变量中
  utl_file.get_line(v_file, v_output);
  --关闭文件
  utl_file.fclose(v_file);
  --测试输出文件内容
  dbms_output.put_line(v_output);
EXCEPTION
  WHEN utl_file.access_denied THEN
    dbms_output.put_line('拒绝访问!');
  WHEN OTHERS THEN
    dbms_output.put_line('SQLERRM: ' || SQLERRM);
END;
```

#### 拷贝文件中的数据待另外一个文件 
```
--====从一个文件拷贝数据到另外一个文件====--
BEGIN
  utl_file.fcopy(src_location  => 'FILE_DIR',
                 src_filename  => 'src.txt',
                 dest_location => 'FILE_DIR',
                 dest_filename => 'dst.txt',
                 start_line    => '1',
                 end_line      => '4');
EXCEPTION
  WHEN utl_file.access_denied THEN
    dbms_output.put_line('拒绝访问!');
  WHEN OTHERS THEN
    dbms_output.put_line('SQLERRM: ' || SQLERRM);
END;
```

