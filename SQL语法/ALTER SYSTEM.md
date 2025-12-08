# ALTER SYSTEM

#### 功能描述

修改数据库系统参数。

#### 注意事项

执行该语句的用户需要有ALTER SYSTEM系统权限。

#### 语法格式

ALTER SYSTEM

 { DUMP DATAFILE file_id PAGE page_id

 | SWITCH LOGFILE

 | SET parameter_name = parameter_value [ SCOPE = { MEMORY | PFILE | BOTH } ]

 | SET REPLICATION { ON '[IP]:PORT' | OFF }

 | LOAD DICTIONARY FOR [ schema_name.]object_name

 | INIT DICTIONARY | RELOAD {HBA | PBL} CONFIG

 | REFRESH SYSDBA PRIVILEGE

 | KILL SESSION 'session_id,serial'

 | RESET STATISTIC | CHECKPOINT

 |{ ADD | DELETE } LSNR_ADDR LISTENING_IP

 |{ ADD | DELETE } HBA ENTRY hba_conf_entry

 | FLUSH {BUFFER | SQLPOOL}

 | DUMP CTRLFILE

 | DEBUG MODE debug_parameter_name = debug_parameter_value

 | STOP BUILD

 | DUMP CATALOG { TABLE table_name | USER user_name } [ TO 'folders' ]

 | RECYCLE SHAREDPOOL [FORCE]

 | REPAIR CATALOG }

#### 参数说明

- **DUMP DATAFILE** ***file_id*** **PAGE** ***page_id***

dump指定数据文件page页。

- ***file_id***

文件编号，取值范围[0, 2147483648)。可通过关联FILE_NAME查询DBA视图ADM_DATA_FILES中的FILE_ID字段获取。

- ***page_id***

页编号，正整数，必须小于文件使用页数的高水位线，取值范围[1, 2147483648)。

高水位线可通过查询动态性能视图DV_DATA_FILES的HIGH_WATER_MARK字段获取。

- **SWITCH LOGFILE**

切换LOGFILE。

- **SET** ***parameter_name*** **=** ***parameter_value*** **[ SCOPE = { MEMORY | PFILE | BOTH } ]**

设置系统参数。SCOPE为可选参数，指定参数写入范围。SCOPE指定为PFILE和BOTH时，参数将被保存在Zenith.ini配置文件中。

- MEMORY：只在内存上修改，立即生效，但重启后将不再生效。此修改方式只适用于动态参数，不允许静态参数使用此模式设置。
- PFILE：此更改写入初始化参数文件，更改将在下次启动时生效。动态参数与静态参数都一样可以。也是静态参数唯一可以使用的方式。
- BOTH：既写入到初始化参数文件，也在内存上修改，立即生效。同样也只适用于动态参数，静态参数则不允许。

如果不设置SCOPE选项，默认为BOTH。

- **SET REPLICATION { ON '[IP]:PORT' | OFF }**

设置复制监听IP地址和端口，动态生效。

- ON ':PORT'
  
  - 在原有IP地址，指定端口PORT上开启监听，端口值不得小于1024。
  
  - 如果端口值为0，则表示关闭监听。

- ON 'IP:PORT'
  
  在指定的IP地址，端口PORT上开启监听。IP地址和冒号之间不允许出现空格，否则会报错。
  
  IP最多可以指定8个，不同IP之间用逗号分隔，且不允许出现空格。
  
  修改监听IP地址和端口生效后，需要同步修改主备的复制链路信息，才能使主备链路功能正常。当备机的复制监听IP和端口为*ip1:port1*时，修改主机的复制监听IP和端口为ip:port，需要在主机和备机上执行如下操作：
  
  主机:

```
ALTER SYSTEM SET LOG_ARCHIVE_DEST_STATE_2=DEFER; 
ALTER SYSTEM SET LOG_ARCHIVE_DEST_2 = 'LOCAL_HOST=ip SERVICE=ip1:port1 PRIMARY_ROLE AFFIRM'; 
ALTER SYSTEM SET LOG_ARCHIVE_DEST_STATE_2=ENABLE;
```

       备机：

```
ALTER SYSTEM SET LOG_ARCHIVE_DEST_STATE_2=DEFER;
ALTER SYSTEM SET LOG_ARCHIVE_DEST_2 = 'LOCAL_HOST=ip1 SERVICE=ip:port PRIMARY_ROLE AFFIRM';
ALTER SYSTEM SET LOG_ARCHIVE_DEST_STATE_2=ENABLE;
```

        OFF：关闭监听。

- LOAD DICTIONARY FOR [*****schema_name*****].*****object_name***

加载对象到数字字典中。

- **INIT DICTIONARY**

加载除系统表以外的其余类型entry（系统视图，动态视图，sequence，role等）。

执行条件：进入restricted模式，且确保所有系统表已经通过“ALTER SYSTEM LOAD DICTIONARY FOR [schema_name].object_name;”语句加载。

- **RELOAD {HBA | PBL} CONFIG**

在线加载zhba.conf文件，从而使用户白名单生效。

在线加载pbl.conf文件，从而使用户弱口令生效。

- **REFRESH SYSDBA PRIVILEGE**

在线刷新sysdba免密登录的密文、加密密钥。不会影响当前连接的客户端，其他客户端连接会使用新的密钥进行免密登录认证。

- **KILL SESSION** ***'******session_id******,******serial******'***

kill会话，session_id是会话ID，serial是序列号ID。

- **RESET STATISTIC**

清理动态视图DV_SYS_STATS下的计数。

- **CHECKPOINT**

为当前实例执行检查点，确保将已提交事务所做的所有更改写入磁盘的数据文件。

- **{ ADD | DELETE } LSNR_ADDR** ***LISTENING_IP***

增加或者删除一个监听IP地址，IP地址需要使用引号标识。该配置立即生效。当前Gauss100 OLTP最大支持8个监听IP地址。

增加一个不存在网卡IP地址作为浮动监听IP的时候，直接返回报错。

删除一个正在使用的监听IP地址的时候，通过该IP地址建立的会话连接会被断连，其业务会做回滚。

- **{ ADD | DELETE } HBA ENTRY** ***hba_conf_entry***

向用户白名单文件zhba.conf中增加或删除一条用户白名单。

- ***hba_conf_entry***

格式为'type user address'，参数说明请参考表1。

| 参数      | 描述                                | 备注                                                                                                                                                                                                                                                             |
| ------- | --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| type    | 建立连接的类型。                          | 可配置为host或者hostssl。host支持普通TCP或SSL连接。hostssl是指SSL连接，如果服务端开启SSL，但是客户端没有配置ssl，此时服务端会拒绝连接。                                                                                                                                                                         |
| user    | 允许访问数据库的指定用户。如果user是“*”，则表示所有用户。  | 如果user中包含特殊字符（例如#、*、TAB键等特殊字符）需要写成“user”格式。例如：host "#abc" 127.0.0.1、host "abc" 127.0.0.1 表示将双引号中的字符串整体作为user。单行只能指定一个用户。                                                                                                                                       |
| address | 允许访问数据库的指定用户的IP地址范围。可以英文逗号分隔声明多个。 | IP地址支持IPV4、IPV6地址、或指定子网掩码长度表示一个子网网段。如下均为合法格式：<br><br>- *.*.*.*和0.0.0.0/0均表示全网段主机。<br>- 192.168.3.222 表示一个IPV4主机。<br>- 192.168.3.0/24 表示一个IPV4子网网段192.168.3.0所有IP。<br>- 20AB::9217:acff:feab:fcd0 表示一个IPV6主机。<br>- 20AB::9217:acff:feab:fcd0/64 表示前缀为64的IPV6网段。 |

- **FLUSH BUFFER**

清空数据库的缓存数据。

- **FLUSH SQLPOOL**

清空SQLPOOL的缓存数据。

- **DUMP CTRLFILE**

dump控制文件信息。

- **DEBUG MODE** ***debug_parameter_name*** **=** ***debug_parameter_value***

设置开发调试参数。立即生效，所有debug参数不会写入配置文件，仅保存在内存中，重启后可还原。

注意：调试参数仅限开发调试使用，用户禁止修改，否则会造成数据库异常。

- **DUMP CATALOG { TABLE** ***table_name*** **| USER** ***user_name*** **} [ TO** ***'folder*****s' ]**

- DUMP CATALOG TABLE *table_name*

DUMP表的数据字典里的内存信息和表对应的索引的内存信息。

- DUMP CATALOG USER *user_name*

DUMP用户数据字典的内存信息。

- [ TO '*folder*s']

DUMP出来的文件指定输出到某文件夹，不指定时默认存放在trc文件夹里。

- DUMP出来的文件不能超过10M，如果超过10M，就会报错，需要将文件删除后，重新进行DUMP操作。

- SYS用户可以DUMP所有用户的信息；普通用户只可以DUMP自己的信息；DBA用户可以DUMP普通用户和其他DBA用户下的信息。

- **STOP BUILD**

停止备机或级联备机重建基线。

- 当备机（或级联备机）在做BUILD DATABASE时，如果处于主机（或备机）发送数据阶段，在主机（或备机）上执行该命令，可停止build。

- 当主机（或备机）已完成数据发送，此时在主机（或备机）上执行该命令，不能停止备机（或级联备机）上的build。

- **RECYCLE SHAREDPOOL [FORCE]**

回收DC POOL/SQL POOL到SHARED AREA。

FORCE表示会强制把SQL POOL里面所有软解析全部置位FALSE。

- **REPAIR CATALOG**

修复场景：升级后数据库二进制记录的核心系统表的列，和data中核心系统表的列不相等的场景。

#### 示例

- dump数据文件的指定页。

```
--删除表空间。
DROP TABLESPACE video_space;
```

```
--创建表空间。
CREATE TABLESPACE video_space DATAFILE 'video_dfile1' SIZE 32M;
```

```
--查询FILE_ID。
SELECT FILE_NAME,FILE_ID FROM ADM_DATA_FILES WHERE FILE_NAME='/opt/ograc/data/data/video_dfile1';
```

```
--查询数据文件的高水位线。
SELECT * FROM DV_DATA_FILES WHERE FILE_NAME='/opt/ograc/data/data/video_dfile1';
```

```
--导出数据文件第一页，17为数据文件的FILE_ID。
ALTER SYSTEM DUMP datafile 17 PAGE 1;
```

切换LOGFILE。

```
ALTER SYSTEM SWITCH LOGFILE;
```

- 修改参数UNDO_RETENTION_TIME的值为1200秒，只在内存上修改，立即生效。

```
--查询参数UNDO_RETENTION_TIME的当前值。
SHOW PARAMETER UNDO_RETENTION_TIME
```

```
--修改参数UNDO_RETENTION_TIME的值。
ALTER SYSTEM SET UNDO_RETENTION_TIME=1200 SCOPE=MEMORY;
```

- 设置复制监听。

```
--关闭监听。
ALTER SYSTEM SET REPLICATION OFF;
```

```
--开启监听。
ALTER SYSTEM SET REPLICATION ON '127.0.0.1:1611';
```

```
--当监听IP地址和端口生效后，需要同步修改主备的复制链路信息，才能使主备链路功能正常。
--假设备机的复制监听IP和端口为127.0.0.1:1611时，修改主机的复制监听IP和端口为10.10.10.10:1888。
--主机需执行如下操作。
ALTER SYSTEM SET LOG_ARCHIVE_DEST_STATE_2=DEFER;
ALTER SYSTEM SET LOG_ARCHIVE_DEST_2 = 'LOCAL_HOST=10.10.10.10 SERVICE=127.0.0.1:1611 PRIMARY_ROLE AFFIRM';
ALTER SYSTEM SET LOG_ARCHIVE_DEST_STATE_2=ENABLE;
--备机需执行如下操作。
ALTER SYSTEM SET LOG_ARCHIVE_DEST_STATE_2=DEFER;
ALTER SYSTEM SET LOG_ARCHIVE_DEST_2 = 'LOCAL_HOST=127.0.0.1 SERVICE=10.10.10.10:1888 PRIMARY_ROLE AFFIRM';
ALTER SYSTEM SET LOG_ARCHIVE_DEST_STATE_2=ENABLE;
```

- 加载表education到数据字典中。

```
--删除表education。
DROP TABLE IF EXISTS education;
--创建表education。
CREATE TABLE education(staff_id INT, highest_degree CHAR(8) NOT NULL, graduate_school VARCHAR(64), graduate_date DATETIME, education_note VARCHAR(70));
--加载表education到数据字典中。
ALTER SYSTEM LOAD DICTIONARY FOR education;
```

- 加载除系统表以外的其余类型entry（系统视图，动态视图，sequence，role等）。

首先进入restricted模式，确保所有系统表已经通过ALTER SYSTEM LOAD DICTIONARY FOR [schema_name].object_name 语句加载后才能执行此操作。

```
ALTER SYSTEM INIT DICTIONARY;
```

在线加载zhba.conf文件。

```
ALTER SYSTEM RELOAD HBA CONFIG;
```

增加一个监听IP地址。

```
ALTER SYSTEM ADD LSNR_ADDR '10.253.49.29';
```

- 删除一个监听IP地址。
  
  ```
  ALTER SYSTEM DELETE LSNR_ADDR '10.253.49.29';
  ```

- 在线刷新sysdba免密登录的密文、加密密钥。
  
  ```
  ALTER SYSTEM REFRESH SYSDBA PRIVILEGE;
  ```

- 清理动态视图DV_SYS_STATS下的计数。
  
  ```
  ALTER SYSTEM RESET STATISTIC;
  ```

- 为当前事务设置检查点。
  
  ```
  ALTER SYSTEM CHECKPOINT;
  ```

- 清空缓存数据。
  
  清空数据库缓存数据。

```
ALTER SYSTEM FLUSH BUFFER;
```

        清空SQLPOOL缓存数据。

```
ALTER SYSTEM FLUSH SQLPOOL;
```

- DUMP控制文件信息。
  
  ```
  ALTER SYSTEM DUMP CTRLFILE;
  ```

- 修改数据库调试参数。
  
  ```
  ALTER SYSTEM DEBUG MODE _MRP_RES_LOGSIZE = 1G;
  ```

- DUMP 数据字典信息。

```
ALTER SYSTEM DUMP CATALOG TABLE TEST;
ALTER SYSTEM DUMP CATALOG USER TEST;
```

- 停止重建基线。
  
  ```
  ALTER SYSTEM STOP BUILD;
  ```
