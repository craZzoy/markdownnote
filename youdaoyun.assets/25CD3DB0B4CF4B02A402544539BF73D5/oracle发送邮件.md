
```
--邮件发送
create or replace procedure p_send_mail(retcode     out varchar2,
                                        errbuf      out varchar2,
                                        p_recipient varchar2, -- 邮件接收人
                                        p_subject   varchar2, -- 邮件标题
                                        p_message   varchar2 -- 邮件正文
                                        ) is
  --下面四个变量请根据实际邮件服务器进行赋值
  v_mailhost VARCHAR2(30) := 'smtp.163.com'; --SMTP服务器地址
  v_user     VARCHAR2(30) := '18819457414@163.com'; --登录SMTP服务器的用户名
  v_pass     VARCHAR2(20) := 'wz3670793879'; --登录SMTP服务器的密码
  v_sender   VARCHAR2(50) := '18819457414@163.com'; --发送者邮箱，一般与 ps_user 对应

  v_conn utl_smtp.connection; --到邮件服务器的连接
  v_msg  varchar2(4000); --邮件内容
begin
  v_conn := utl_smtp.open_connection(v_mailhost, 25);
  utl_smtp.ehlo(v_conn, v_mailhost); --是用 ehlo() 而不是 helo() 函数
  --否则会报：ORA-29279: SMTP 永久性错误: 503 5.5.2 Send hello first.

  UTL_SMTP.command(v_conn, 'AUTH LOGIN'); -- smtp服务器登录校验
  UTL_SMTP.command(v_conn,
                   UTL_RAW.cast_to_varchar2(UTL_ENCODE.base64_encode(UTL_RAW.cast_to_raw(v_user))));
  UTL_SMTP.command(v_conn,
                   UTL_RAW.cast_to_varchar2(UTL_ENCODE.base64_encode(UTL_RAW.cast_to_raw(v_pass))));
  UTL_SMTP.mail(v_conn, v_sender); --设置发件人
  UTL_SMTP.rcpt(v_conn, p_recipient); --设置收件人

  -- 创建要发送的邮件内容 注意报头信息和邮件正文之间要空一行
  v_msg := 'Date:' || TO_CHAR(SYSDATE, 'dd mon yy hh24:mi:ss') ||
           UTL_TCP.CRLF || 'From: ' || '<' || v_sender || '>' ||
           UTL_TCP.CRLF || 'To: ' || '<' || p_recipient || '>' ||
           UTL_TCP.CRLF || 'Subject: ' || p_subject || UTL_TCP.CRLF ||
           UTL_TCP.CRLF -- 这前面是报头信息
           || p_message; -- 这个是邮件正文

  UTL_SMTP.open_data(v_conn); --打开流
  UTL_SMTP.write_raw_data(v_conn, UTL_RAW.cast_to_raw(v_msg)); --这样写标题和内容都能用中文
  UTL_SMTP.close_data(v_conn); --关闭流
  UTL_SMTP.quit(v_conn); --关闭连接
exception
  when others then
    retcode := 'N';
    errbuf  := sqlcode || sqlerrm;
end p_send_mail;





```
需开通权限：

```
--1.创建访问控制列表（ACL）
BEGIN
  DBMS_NETWORK_ACL_ADMIN.CREATE_ACL(acl         => 'email_server_permissions.xml',
                                    description => 'Enables network permissions for the e-mail server',
                                    principal   => 'BUP',
                                    is_grant    => TRUE,
                                    privilege   => 'connect');
END;

--2.将 ACL 与邮件服务器相关联
BEGIN
  DBMS_NETWORK_ACL_ADMIN.assign_acl(acl        => 'email_server_permissions.xml',
                                    host       => 'smtp.163.com', --SMTP服务器地址
                                    lower_port => 25,
                                    upper_port => NULL);
  COMMIT;
END;

--3.ACL 为用户授与连接邮件服务器的权限

begin
  DBMS_NETWORK_ACL_ADMIN.add_privilege(acl       => 'email_server_permissions.xml',
                                       principal => 'BUP', --授权用户
                                       is_grant  => TRUE,
                                       privilege => 'resolve');
end;

--查询
SELECT host, lower_port, upper_port, acl FROM dba_network_acls;

SELECT acl,
       principal,
       privilege,
       is_grant,
       TO_CHAR(start_date, 'DD-MON-YYYY') AS start_date,
       TO_CHAR(end_date, 'DD-MON-YYYY') AS end_date
  FROM dba_network_acl_privileges;

BEGIN
  --删除acl配置
  DBMS_NETWORK_ACL_ADMIN.drop_acl(acl => 'email_server_permissions.xml');
  --COMMIT;
END;
```

