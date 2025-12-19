## 删除表/释放存储

### 语法描述
#### stmt
```
TRUNCATE TABLE [schema_name.]table_name [PURGE] [{DROP|REUSE} STORAGE]
```

### 示例
```SQL
-- 清空表
TRUNCATE TABLE test_data;

-- 使用 PURGE 彻底清空（不进回收站）
TRUNCATE TABLE sensitive_data PURGE;

-- 使用DROP STORAGE清空并释放空间
TRUNCATE TABLE large_table DROP STORAGE;

-- 使用REUSE STORAGE快速清空（保留分配的空间）
TRUNCATE TABLE cache_table REUSE STORAGE;

```

