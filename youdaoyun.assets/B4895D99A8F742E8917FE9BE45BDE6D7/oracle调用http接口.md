
设置acl权限：
```
--定义ACL,若没有ACL，则无法访问网络. 取名：httprequestpermission.xml
BEGIN
  dbms_network_acl_admin.create_acl(acl         => 'httprequestpermission.xml',
                                    DESCRIPTION => 'Normal Access',
                                    principal   => 'BUP',
                                    is_grant    => TRUE,
                                    PRIVILEGE   => 'connect',
                                    start_date  => NULL,
                                    end_date    => NULL);
END;

--查看ACL是否增加成功
SELECT any_path FROM resource_view where any_path like '/sys/acls/%.xml';

--给用户增加acl权限，这里是 SD_JY 注意是大写，小写不识别
begin
  dbms_network_acl_admin.add_privilege(acl        => 'httprequestpermission.xml',
                                       principal  => 'BUP',
                                       is_grant   => TRUE,
                                       privilege  => 'connect',
                                       start_date => null,
                                       end_date   => null);
end;

--添加对应主机 ，将对应主机和端口添加到ACL。这里是 192.168.0.156 和 8080 ，这个ip和端口要和上面存储过程中定义的地址一致
begin
  dbms_network_acl_admin.assign_acl(acl        => 'httprequestpermission.xml',
                                    host       => '172.17.4.6',
                                    lower_port => 8090, 
                                    upper_port => NULL);
end;

SELECT acl,
       principal,
       privilege,
       is_grant,
       TO_CHAR(start_date, 'DD - MON - YYYY') AS start_date,
       TO_CHAR(end_date, 'DD - MON - YYYY') AS end_date
  FROM dba_network_acl_privileges;


BEGIN
  --删除acl配置
  DBMS_NETWORK_ACL_ADMIN.drop_acl(acl => 'httprequestpermission.xml');
  --COMMIT;
END;

```
