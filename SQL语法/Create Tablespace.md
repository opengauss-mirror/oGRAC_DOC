## 创建表空间

### 语法描述
#### stmt
```grammar
CREATE [UNDO] TABLESPACE name
    DATAFILE {datafile_clause [, ...]} 
    [NOLOGGING]
    [autooffline_clause]
    [EXTENT AUTOALLOCATE]
```
- UNDO: 创建UNDO表空间
- datafile_clause: 表空间数据文件描述，多个逗号分割
- NOLOGGING: 表空间为NOLOGGING类型
- autooffline_clause：表空间离线配置
- EXTENT AUTOALLOCATE: 表空间extent size字段扩展


**datafile_clause:**
```grammar
    name SIZE size_clause
    [COMPRESS]
    [autoextend_clause]
```
- name：数据文件路径。支持绝对路径和相对路径
- size_clause: 数据文件大小描述
- COMPRESS: 表压缩特性,集群模式不适用
- autoextend_clause: 表空间扩展设置

**autoextend_clause:**
```grammar
    AUTOEXTEND {OFF 
                |   ON [NEXT size_clause] [MAXSIZE {size_clause | UNLIMITED}]
                }
```
- OFF: 关闭自动扩展
- ON: 开启自动扩展
    - NEXT: 自动扩展的大小
    - MAXSIZE: 自动扩展的上限
    - UNLIMITED: 无限扩展

**autooffline_clause:**
```grammar
    AUTOOFFLINE [ON|OFF]
```
- ON: 开启表空间自动离线。启动时生效，文件打开失败自动离线。
- OFF: 关闭表空间自动离线。

**size_clause:**
```grammar
    size [K|M|G]
```
- K: KB
- M: MB
- G: GB
### 示例
```SQL
-- 1. 创建基础表空间
CREATE TABLESPACE tbs1
    DATAFILE '/home/ogracdba/data/tbs1.dbf' SIZE 100M;

-- 2. 创建带自动扩展的表空间
CREATE TABLESPACE tbs2
    DATAFILE '/home/ogracdba/data/tbs2.dbf' SIZE 200M
    AUTOEXTEND ON NEXT 50M MAXSIZE 1G;

-- 3. 创建无限自动扩展的表空间
CREATE TABLESPACE tbs3
    DATAFILE '/home/ogracdba/data/tbs3.dbf' SIZE 500M
    AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED;

-- 4. 创建多个数据文件的表空间
CREATE TABLESPACE tbs4
    DATAFILE 
        '/home/ogracdba/data/tbs4_1.dbf' SIZE 100M,
        '/home/ogracdba/data/tbs4_2.dbf' SIZE 200M
    EXTENT AUTOALLOCATE;

-- 5. 创建UNDO表空间
CREATE UNDO TABLESPACE undotbs
    DATAFILE '/home/ogracdba/data/undotbs.dbf' SIZE 2G
    AUTOEXTEND ON NEXT 500M MAXSIZE 10G;

-- 6. 创建NOLOGGING表空间
CREATE TABLESPACE tbs_nolog
    DATAFILE '/home/ogracdba/data/tbs_nolog.dbf' SIZE 300M
    NOLOGGING;

-- 7. 创建压缩表空间
CREATE TABLESPACE tbs_compress
    DATAFILE '/home/ogracdba/data/tbs_compress.dbf' SIZE 500M COMPRESS
    AUTOEXTEND ON NEXT 100M;

-- 8. 创建带自动离线功能的表空间
CREATE TABLESPACE tbs_autooffline
    DATAFILE '/home/ogracdba/data/tbs_autooffline.dbf' SIZE 200M
    AUTOOFFLINE ON;

-- 9. 组合多种特性的表空间
CREATE TABLESPACE tbs_complex
    DATAFILE 
        '/home/ogracdba/data/tbs_complex1.dbf' SIZE 300M COMPRESS,
        '/home/ogracdba/data/tbs_complex2.dbf' SIZE 300M
    NOLOGGING
    AUTOOFFLINE ON
    EXTENT AUTOALLOCATE
    AUTOEXTEND ON NEXT 100M MAXSIZE 2G;

-- 10. 使用相对路径的表空间（相对于数据库数据目录）
CREATE TABLESPACE tbs_relative
    DATAFILE './tbs_relative.dbf' SIZE 100M;

-- 11. 关闭自动扩展的表空间
CREATE TABLESPACE tbs_noextend
    DATAFILE '/home/ogracdba/data/tbs_noextend.dbf' SIZE 500M
    AUTOEXTEND OFF;

-- 12. 创建自动离线关闭的表空间
CREATE TABLESPACE tbs_nooffline
    DATAFILE '/home/ogracdba/data/tbs_nooffline.dbf' SIZE 200M
    AUTOOFFLINE OFF;

```

