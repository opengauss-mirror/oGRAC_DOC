## 删除索引

### 语法描述
#### stmt
```
DROP INDEX [IF EXISTS] [schema_name.]index_name ON [schema_name.]table_name
```


### 示例
```SQL
--  删除索引并指定表名
DROP INDEX idx_test1_name ON test_table1;

-- 带schema的删除
DROP INDEX sys.idx_test1_email ON sys.test_table1;
```