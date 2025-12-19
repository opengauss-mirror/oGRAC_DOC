## 创建表

### 语法描述
#### stmt
```
CREATE [[GLOBAL] TEMPORARY] TABLE [IF NOT EXISTS] [schema_name.]table_name
    {({column_def_clause}[,...] [external_constraint][,...])} | {AS query}
    [ON COMMENT {DELETE|PRESERVE} ROWS]
    [physical_properties_clause]
    [table_attr_clause]
    [CRMODE {PAGE|ROW}]
```
- column_def_clause: 列定义
- external_constraint: 表约束
- query: 查询子句
- physical_properties_clause: 物理存储属性
- table_attr_clause：表级别属性
- CRMODE
    - PAGE: 页级MVCC
    - ROW: 行级MVCC

**column_def_clause:**
```
    column_name {datatype | SERIAL}
    [DEFAULT expr [ON UPDATE expr]]
    [AUTO INCREMENT]
    [COMMENT 'comment_str']
    [COLLATE collation_name]
    [internal_constraint][...]
```
- internal_constraint: 列约束

**internal_constraint:**
```
    CONSTRAINT constraint_name {[NOT] NULL
                                | UNIQUE
                                | PRIMARY KEY
                                | CHECK(expr)
                                | refenence_clause }[...]
```
- refenence_clause: 外键关联

**refenence_clause:**
```
    REFRENCES [schema_name.]table_name[(column_name)] [ON DELETE {CASCADE | SET NULL}]
```

**external_constraint:**
```
    CONSTRAINT constraint_name {UNIQUE(column_name[,...]) [using_index_clause]
                                | PRIMARY KEY(column_name[,...]) [using_index_clause]
                                | CHECK(expr)
                                | FOREIGN KEY(column_name[,...]) refenence_ex_clause}
```
- using_index_clause: 指定索引
- refenence_ex_clause: 外键关联

**using_index_clause:**
```
    USE INDEX [ INITRANS int
                | TABLESPACE tablespace_name
                | LOCAL
              ]
```

**refenence_ex_clause:**
```
    REFRENCES [schema_name.]table_name[(column_name[,...])] [ON DELETE {CASCADE | SET NULL}]
```

**physical_properties_clause**
```
    segment_attr_clause
    | ORGANZATION EXTERNAL external_table_clause
    | FORMAT row_format_clause
```

**segment_attr_clause**
```
    { physical_attr_clause | TABLESPACE tablespace_name}[ ...]
```

**physical_attr_clause**
```
    {
        PCTFREE int
        | INITRANS int
        | MAXTRANS int
        | storage_clause
    }[ ...]
```

**storage_clause**
```
    STORAGE ({INITIAL int [K|M|G|T] | MAXSIZE {UNLIMITED | int [K|M|G|T]}}[ ...])
```

**external_table_clause**
```
    ([TYPE {LOADER|DATAPUMP}] external_data_properties)
```

**external_data_properties**
```
    DIRECTORY directory_name
    [ACCESS PARAMETERS (opaque_format_spec)]
    LOCATION location_name
```

**opaque_format_spec**
```
    RECORDS DELIMITED BY records_delimiter FIELDS TERMINATED BY fields_term
```

**row_format_clause**
```
    ({ASF|CSF})
```

**table_attr_clause**
```
    [column_attr_clause]
    [AUTO_INCREMENT [=] value]
    [AS subqery]
```

**column_attr_clause**
```
    [LOB_storage_clause]
    [APPENDONLY {ON|OFF}]
```

**LOB_storage_clause**
```
    LOB (LOB_item) STORE AS {[(LOB_parameters)]}
```

**LOB_parameters**
```
    [TABLESPACE tablespace_name | {ENABLE | DISABLE} STORAGE IN ROW][ ...]
```


### 示例
```SQL
-- 1. 简单员工表
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    salary DECIMAL(10,2) CHECK (salary > 0),
    dept_id INT
);

-- 2. 带外键和注释
CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(50) COMMENT '部门名称',
    manager_id INT
);

-- 全局临时表
CREATE GLOBAL TEMPORARY TABLE temp_data (
    session_id VARCHAR(50)
) ON COMMIT DELETE ROWS;

-- 会话级临时表
CREATE TEMPORARY TABLE #session_cache (
    key VARCHAR(100) PRIMARY KEY,
    value TEXT
) ON COMMIT PRESERVE ROWS;

-- 列级约束
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE,
    age INT CHECK (age >= 0),
    status VARCHAR(10) DEFAULT 'ACTIVE'
);

-- 表级外键
CREATE TABLE orders (
    order_id SERIAL,
    user_id INT,
    CONSTRAINT pk_order PRIMARY KEY (order_id),
    CONSTRAINT fk_user FOREIGN KEY (user_id) 
        REFERENCES users(user_id) ON DELETE CASCADE
);

-- 指定表空间和存储
CREATE TABLE large_logs (
    log_id SERIAL PRIMARY KEY,
    log_time TIMESTAMP,
    message TEXT
) TABLESPACE log_ts
  PCTFREE 10
  STORAGE (INITIAL 100M MAXSIZE 2G);

-- 带BLOB的表
CREATE TABLE documents (
    doc_id SERIAL PRIMARY KEY,
    doc_content BLOB
) LOB (doc_content) STORE AS (TABLESPACE lob_ts);

-- CATS
CREATE TABLE sales_summary
AS
SELECT product_id, SUM(quantity) as total_qty
FROM sales
GROUP BY product_id;

-- 创建行级MVCC表（默认）
CREATE TABLE oltp_table (
    account_id SERIAL PRIMARY KEY,
    balance DECIMAL(15,2),
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) CRMODE ROW;

-- 创建页级MVCC表
CREATE TABLE page_mvcc_table (
    id SERIAL PRIMARY KEY,
    log_data TEXT,
    created_time TIMESTAMP
) CRMODE PAGE;

-- 仅指定物理属性，使用默认表空间
CREATE TABLE logs (
    log_id SERIAL PRIMARY KEY,
    log_time TIMESTAMP
)
PCTFREE 5
INITRANS 2
MAXTRANS 255
STORAGE (INITIAL 100M MAXSIZE 1G);

-- 使用ASF格式创建表
CREATE TABLE user_profiles (
    user_id SERIAL PRIMARY KEY,
    profile_data JSONB,
    preferences JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
FORMAT ASF;

-- 只指定DIRECTORY和LOCATION
CREATE TABLE external_simple (
    id INT,
    name VARCHAR(50)
)
ORGANIZATION EXTERNAL (
    DIRECTORY data_dir
    LOCATION 'simple.csv'
);
```

CREATE TABLESPACE lob_ts
    DATAFILE '/home/ogracdba/data/tbs11111.dbf' SIZE 1G;