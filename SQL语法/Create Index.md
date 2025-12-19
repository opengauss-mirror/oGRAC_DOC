## 创建索引

### 语法描述
#### stmt
```
CREATE [UNIQUE] INDEX [IF NOT EXISTS] [schema_name.]index_name ON table_index_clause
    [CRMODE {PAGE|ROW}]
    [PARALLEL n]
    [REVERSE]
    [NOLOGING]
```

**table_index_clause**
```
    [schema_name.]table_name ({column_name | column_expr [ASC|DESC]}[,...]) index_attr_clause
```

**index_attr_clause**
```
    [[physical_attr_clause] [TABLESPACE tablespace_name] [index_partition_clause] [ONLINE]]
```

**physical_attr_clause**
    INITRAINS int

**index_partition_clause**
```
    LOCAL [({PARTITION partition_name [TABLESPACE tablespace_name] [physical_attr_clause] [COMPRESS]}[,...])]
```
### 示例
```SQL
-- 在employees表的last_name列上创建普通索引
CREATE INDEX idx_emp_lastname ON employees(last_name);

-- 在departments表的department_name列上创建唯一索引
CREATE UNIQUE INDEX idx_dept_name ON departments(department_name);

-- 在orders表上创建客户和日期的复合索引
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- 指定排序方向（常用于范围查询优化）
CREATE INDEX idx_orders_date_status ON orders(order_date DESC, status ASC);

-- 创建行模式CR索引并禁用日志
CREATE INDEX idx_emp_dept ON employees(department_id)
CRMODE ROW
NOLOGGING
PARALLEL 4;

-- 创建反向键索引，减少索引块争用
CREATE INDEX idx_emp_id_reverse ON employees(manager_id) REVERSE;

-- 在分区表sales上创建本地分区索引
CREATE INDEX idx_sales_date_local ON sales(sale_date) LOCAL;

-- 指定每个分区的表空间和属性
CREATE INDEX idx_sales_product_local ON sales(product_id, sale_date) LOCAL
(
    PARTITION p1_2023 TABLESPACE idx_ts1 COMPRESS,
    PARTITION p2_2023 TABLESPACE idx_ts2,
    PARTITION p3_2023 TABLESPACE idx_ts3,
    PARTITION p4_2023 TABLESPACE idx_ts3,
    PARTITION p5_future TABLESPACE idx_ts3
);

-- 在表达式上创建索引（需语法支持column_expr）
CREATE INDEX idx_emp_upper_name ON employees(UPPER(last_name));


-- 大型表的并行索引创建
CREATE INDEX idx_logs_timestamp ON application_logs(log_timestamp, user_id)
PARALLEL 8
NOLOGGING
TABLESPACE logs_index_ts
CRMODE PAGE;
```