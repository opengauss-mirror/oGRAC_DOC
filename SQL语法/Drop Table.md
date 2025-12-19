## 删除表

### 语法描述
#### stmt
```
DROP [TEMPORARY] TABLE [IF EXISTS] [schema_name.]table_name [CASCADE CONSTRAINTS] [PURGE]
```

### 示例
```SQL
-- 删除单个表
DROP TABLE employees;

-- 删除指定模式下的表
DROP TABLE sys.session_cache;

-- 删除普通临时表
DROP TEMPORARY TABLE #session_cache;

-- 删除表并级联删除外键约束
DROP TABLE departments CASCADE CONSTRAINTS;

-- 彻底删除表，不进回收站
DROP TABLE temporary_data PURGE;

```

