# ALTER SYSTEM KILL SESSION

#### 功能描述

结束一个会话。

#### 注意事项

- 执行该语句的用户需要有ALTER SYSTEM系统权限。
- 不能kill当前登录会话，不能kill保留会话。

#### 语法格式

```
ALTER SYSTEM KILL SESSION 'session_id,serial#';
```

#### 参数说明

- *session_id*：会话id。
- serial#：会话serial id。

#### 示例

```
--删除用户。
DROP USER DDMUSER CASCADE;
--创建用户。
CREATE USER DDMUSER IDENTIFIED BY password;
--授权。
GRANT CONNECT , RESOURCE  TO DDMUSER;
GRANT SELECT ON DV_SESSIONS TO DDMUSER;
GRANT ALTER SYSTEM TO DDMUSER;
--使用DDMUSER连接数据库进程端口并创建表。
conn DDMUSER/password@127.0.0.1:1611
--查询要结束会话的ID。
select SID,SPID,SERIAL#,USERNAME,CLIENT_IP from DV_SESSIONS where USERNAME='DDMUSER';

SID          SPID        SERIAL#      USERNAME                                                         CLIENT_IP                                                       
------------ ----------- ------------ ---------------------------------------------------------------- ----------------------------------------------------------------
130          639782      185          DDMUSER                                                          127.0.0.1                                                       
174          637288      184          DDMUSER                                                          127.0.0.1                                                       

2 rows fetched.
--结束其他会话。
ALTER system kill session '174,184';

Succeed.
--无法结束当前会话。
ALTER system kill session '130,185';

CT-00684, The current session cannot be killed
```
