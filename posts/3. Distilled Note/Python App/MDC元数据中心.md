#TODO [[数据权限]]待设计

# Meta Data Center
---
## 一、底层功能设计

元数据中心是所有数仓应用的核心，有了它，就有了之后的可能性。

```sql
CREATE SCHEMA mdc -- Meta Data Center
```

**目的：**
统一性：易分析平台是一款产品，以后所有项目都应用该产品，且该产品可以用最小修改量，对所有项目进行迭代更新（意思是产品只负责功能，对每个项目没有任何单独的写死的代码在里面。如果有，对该部分代码单独放在一个文件夹进行管控与调用）

效率：简化一切从项目开始到结束，中间可被自动化的工作。目标人群不仅包括开发人员，还有BA，架构等，一切参与人。

可配置性：功能可前台配置，项目参与人可随时调整

版本记录：所有数据可记录版本，（数据版本管理 或 数据库备份）


### 1.1 主题
---
>[!info] mdc.themes
>用来记录主题，以及每一个主题相对应的数据及时性要求

```sql
CREATE TABLE mdc.themes (
    theme_id INT identity(1, 1) NOT NULL PRIMARY KEY -- 主题ID
    , theme_name VARCHAR(50) NOT NULL -- 主题
    , theme_eng_name VARCHAR(50) NOT NULL -- 英文主题
    , stored_procedure_name AS CONCAT (
        'pipeline_'
        , theme_eng_name
        ) -- 存储过程名称
    , refresh_frequency VARCHAR(50) NULL -- 数据更新频率
    , refresh_time VARCHAR(50) NULL -- 数据更新时间点
    , parameters VARCHAR(50) NULL -- 传参要求
    )
```

#ToDo 重复验证设计

- theme id：自增
- theme name：前台填写
- stored procedure name：自动生成
- refresh frequency：前台选择 （每天，每周几，每月几号）
- refresh time：前台选择

**ETL**
脚本按照主题生成ETL Pipeline，包含：
1. Python ETL Pipeline
	- ODS层数据获取的**Python脚本**
	- 主题级别的**存储过程**
2. Python Django定时执行计划

**易分析数据生成**
1. 自动生成"数据生成界面"，每一个主题生成一个

**易分析项目管理**
增删改查表

### 1.2 表
---
>[!info] mdc.tables
>用来记录数仓分层中，每一张数据表的属性，对应的主题，以及历史数据更新要求

配置项：用户密码库，在代码层放到txt文档

```sql
CREATE TABLE mdc.tables (
    table_id INT NOT NULL PRIMARY KEY IDENTITY(1, 1) -- 表ID
    , db_name VARCHAR(50) NOT NULL -- 数据库名称（记录跨库数据源！！）
    , schema_name VARCHAR(50) NOT NULL -- 模式（联合主键）
    , table_name VARCHAR(50) NOT NULL -- 表名（联合主键）
    , table_description VARCHAR(50) NOT NULL -- 表描述（与表的注释进行同步）
    -- , business_nature VARCHAR(50) NOT NULL -- 业务过程
    , theme_id INT NOT NULL -- 主题（每个主题保存一个存储过程）
    , table_model_type VARCHAR(50) NOT NULL -- 表在模型中的类型（暂时分为事实表和维度表）
    , table_source_type VARCHAR(50) NOT NULL -- 数据源的类型（暂时分类为系统表何上传表）#TODO 写4种处理方式
    , refresh_method VARCHAR(50) NULL -- 全量，增量
    , incremental_date_col VARCHAR(50) NULL -- 增量刷新日期字段
    , incremental_period VARCHAR(50) NULL -- 一个月，两个月，还是每周等等
    FOREIGN KEY (theme_id) REFERENCES mdc.themes(theme_id)
    )
```

- table id：自增
- database name：前台选择
- schema name：前台选择
- table name：前台填写
- table description：前台填写
- business nature：前台填写
- theme：前台选择，每个主题生成一个SP，所以DIM自动生成一个统一主题，事实表分类存储主题
- table model type：前台选择（事实表，维度表 #ToDo 拆分更细）
- table source type：前台选择（数仓表，其他系统表，上传表）
- refresh method：刷新方式（全量，增量）事实表默认增量，维度表默认全量， #ToDo 渐变维度处理方式
- incremental date column：前台选择，标注用来做增量的日期列
- incremental period：前台选择

PS：其中增量日期字段，需要字段填写后才能进行选择。

该Table用于生成表，并把每个表与主题进行关联，根据不同的数据源类型，在另外关联的系统数据表和上传数据表中进行更多的信息填写。

#### 1.2.1 系统数据

>[!info] mdc.ods_sys_tables
>用来记录ODS层中，来自于系统、数据库等的数据的获取方式

```sql
CREATE TABLE mdc.tables (
	table_id -- 表ID
	, source_database -- SQLServer，MySQL，Oracle，SAP Hana等，不同系统应用不同的获取方式
	, ip
	, database
	, account
	, password -- 加密处理
)	
```


**ETL**
自动生成python ETL脚本，处理数仓外部的数据获取
#ToDo python ETL包，实现性能优化，如多线程、切片入库等功能

**易分析接口**
自动配置数据库信息并接入


#### 1.2.2 上传数据

#ToDo 小明已经有部分架构在这里

**易分析Excel上传**
自动配置Excel上传信息

**易分析项目管理**
增删改查表

### 1.3 数据字段
---

```sql
CREATE TABLE mdc.columns (
	table_id
	, column_name
	, data_type
	, description
)	
```



#ToDo 小明已经有部分架构在这里

**易分析项目管理**
增删改查表

### 1.4 数据血缘
---
> [!info] mdc.table_relationships

```sql
CREATE TABLE mdc.table_relationships (
    id INT NOT NULL PRIMARY KEY identity(1, 1)
    , source_schema_name VARCHAR(50) NOT NULL -- 源表的模式
    , source_table_name VARCHAR(50) NOT NULL -- 源表的表名
    , target_schema_name VARCHAR(50) NOT NULL -- 目标表的模式
    , target_table_name VARCHAR(50) NOT NULL -- 目标表的表名
    , etl_job_type VARCHAR(50) NULL -- 存储过程，或ETL脚本
    , etl_job_name VARCHAR(50) NULL -- 名称
    )

```

*如果脚本在客户服务器上，提供远程链接信息，可能还要在服务器单独部署一个API*

**易分析数据血缘应用**
思考：
如何快速将数据血缘匹配上呢？
- 情况1：当数据项目已经做完时，有了完整的表和存储过程，那么通过SQL的数据血缘可以反查出该数据，需要大量开发
- 情况2：当数据项目刚开始时，刚有表，可以手填，也可以不填
- 情况3：当需要出血缘，又不想填，又没有no.1的开发时，用模糊匹配来找到相应的数据，现在就可以开发。 

情况3Fuzzy Lookup代码一览：
```python
import pandas as pd  
from fuzzywuzzy import process  
  
df_tables = pd.read_excel('data/tables.xlsx')  
sp_tables = pd.read_excel('data/stored_procedures.xlsx')  
  
sp_name = sp_tables['ROUTINE_NAME']  
  
df_tables['sp_match'] = df_tables['table_name'].apply(lambda x: process.extractOne(x, sp_name.tolist())[0])  
df_tables.to_excel('data/table_output.xlsx', index=False)
```

数据血缘可视化一览：
![[Pasted image 20230425182517.png]]

数据血缘可视化代码一览：
```python
import plotly.graph_objects as go
import pandas as pd

df = pd.read_excel('data/table_output.xlsx', sheet_name='Sheet1')
df_label = pd.read_excel('data/table_output.xlsx', sheet_name='Sheet2')
source = df['source_id'].to_list()
target = df['target_id'].to_list()
value = df['value'].to_list()
color = df_label['color'].to_list()
label = df_label['name'].to_list()
x = df_label['x'].to_list()


fig = go.Figure(data=[go.Sankey(
    node=dict(
        pad=15,
        thickness=20,
        line=dict(color="black", width=0.5),
        label=label,
        color=color,
        x=x,
        y=x
    ),
    link=dict(
        source=source,  # indices correspond to labels, eg A1, A2, A1, B1, ...
        target=target,
        value=value
    ))])

fig.update_layout(title_text="Basic Sankey Diagram", font_size=10)
fig.show()

```

**易分析数据生成**
自动查找ODS层依赖，进行校验

**易分析项目管理**
增删改查表

### 1.5 查询视图
---
查询表信息：
```sql
CREATE VIEW [mdc].[view_tables]
AS
WITH tb
AS (
    SELECT thm.theme_name
        , theme_eng_name
        , s.name AS schema_name
        , t.name AS table_name
        , tb.table_description
        , tb.table_model_type
        , tb.table_source_type
    FROM sys.tables AS t
    LEFT JOIN sys.schemas s
        ON t.schema_id = s.schema_id
    LEFT JOIN mdc.tables AS tb
        ON s.name = tb.schema_name
            AND t.name = tb.table_name
    LEFT JOIN mdc.themes AS thm
        ON thm.theme_id = tb.theme_id
    WHERE s.name IN ('ads', 'dim', 'dwd', 'ods', 'tmp', 'ts')
    )
SELECT *
FROM tb
```


查询表字段信息：
#ToDo 此处的字段信息暂时来源于SQL Server自带信息。需要增加字段信息表，再以该表进行同步
```sql
CREATE VIEW [mdc].[view_columns]
AS
SELECT S.name AS [SchemaName]
    , T.name AS [TableName]
    , C.name AS [ColumnName]
    , E.value AS [Description]
    , TYPE_NAME(C.user_type_id) AS [DataType]
FROM sys.schemas S
INNER JOIN sys.tables T
    ON S.schema_id = T.schema_id
INNER JOIN sys.columns C
    ON T.object_id = C.object_id
LEFT OUTER JOIN (
    SELECT *
    FROM sys.extended_properties
    WHERE name = 'MS_Description'
    ) AS E
    ON C.object_id = E.major_id
        AND C.column_id = E.minor_id
```


递归查询依赖表信息：
```sql
DECLARE @Target NVARCHAR(255) = 'ads_psi';

WITH RecursiveAncestors
AS (
    -- Base case: select the direct parent of the given child
    SELECT source_table_name
        , target_table_name
    FROM mdc.table_relationships
    WHERE target_table_name = @Target
    
    UNION ALL
    
    -- Recursive step: find parent of the current parent
    SELECT pcr.source_table_name
        , ra.target_table_name
    FROM mdc.table_relationships AS pcr
    INNER JOIN RecursiveAncestors AS ra
        ON pcr.target_table_name = ra.source_table_name
    )
-- SELECT *
-- FROM RecursiveAncestors


SELECT 
    target_table_name
    , STUFF((
            SELECT ', ' + source_table_name
            FROM RecursiveAncestors AS inner_t
            FOR XML PATH('')
            ), 1, 2, '') AS [source_table_name]
FROM RecursiveAncestors AS main_t
    GROUP BY [target_table_name];
```

**易分析项目管理**
查询视图，可视化等

### 1.6 导出Excel

**易分析项目管理**
导出文档式Excel


## 二、功能架构设计
---
<iframe frameborder="0" style="width:100%;height:460px;" src="https://viewer.diagrams.net/?tags=%7B%7D&highlight=0000ff&edit=_blank&layers=1&nav=1&title=%E6%98%93%E5%88%86%E6%9E%90%E8%AE%BE%E8%AE%A1#R7Vtbb%2BI4FP41fixK4lwfE5LsPnS0SJ1qdudlZIibZBtiZMwA%2B%2BvXThwgMe2wEtSwJZUq%2B%2FgS%2B3zfOfZxDIDj%2BeY3ihbFF5LhClhGtgEwBpblewb%2FLwTbVmDDoBXktMxakbEXPJX%2F4FZodtJVmeGllLUiRkjFykVfOCN1jWesJ0OUknW%2F2gupsp5ggXLcG4YQPM1QhZVq38qMFXJalreX%2F47LvOjebLpyfnPUVZYdLwuUkfWBCCYAjikhrE3NN2NcCd319ZK%2BUbobGMU1O6WBs918Can5nHx%2Fnj5OJ6k%2FLeIHKMfGtt2EccbnL7OEsoLkpEZVspdGlKzqDIteDZ7b13kkZMGFJhf%2BjRnbSjDRihEuKti8kqV8wHT7p2g%2FcrrsX7K7JhNvermtzLVjFQN8UwVStCQrOsPvzLujEqI5Zu%2FUs3ZAcYJjMsd8PLwdxRVi5c%2F%2BOJCkWr6rt0eDJyQg%2FwEc2e9PVK3km0Big8gHUQSSAIQWCHwFvz4666Jk%2BGmBGmWsuYX2kZAvwJThzfs6VXUgG0DXbpt0Bi7pvt5bi9mZQHFgKa5xIa05Oih9RmpaJ1LT1klN75Mo2dWp5ECrczZ7rnnvqS%2FvnE8Fx9fqnM07Ou%2BiYxpa4dG7s7kBePRubbSs0rcEj9bl3Tqy8%2FRBaILw%2Bjacpqd7xwn1ktkYGTYcENr8FaWb3ATTkqsA0%2FPz3D6R51Arz%2B0jPHdAGIPAA4kLwgREKra6CQ8t3YR3j6jNBYEDIkMk%2FFhornUYPpd4IBqLUDVJm0QILLfiQ4ymlKdykRKtohAEfMDGAxeiuVBjW2Q1PQRR0yfHxAC%2BGn18NCaO51yZE7JUz3xbIZl%2Fosdoj950eQz%2FCPWbMxk%2FbBJjQdDdWikkqUgLq%2BCSVDtxTbvvTBztvLVvaSd46WWzO%2Fb%2B9f5Q68lEN8zhCuAK3yxIH4HIbZZSng5Ewk9F6X6VuB0LMaF2E3HvJjIMXE9ZKBytJqLGUH%2FET7dtCruvcvpCrZs61rm4KZwcZek97FHDrPgb7ykU9sC39jYI0ob0vogAbskeAt32YKqhWPL18UeymYnP8AN1kRWryhqPd5%2FKhdYytCx2KhTaKWfcMtAUVxOyLFlJal42JYyR%2BUGFsCpzUcCEBUVI5mZcj4Lnh2pfFmgh3j%2Ff5OKCwAitl3CE6owSMbHopayqMakIbYYI0zS27VTEfhRlJe%2BuK6tJLUy5EgOL0Ow1b7A%2FaBkEaToenym6GwR30B85CtDQU3G24aVwVuOOibi5MIT4QPHHzGHJKHnFA50OIDAML4jdTtcHJNih%2BxZL5mWWNQ73RJa8kJpJn8tXS%2B6Zj%2FFzyWdQ1vnXxlU%2FuLJZjzLiETVfMZsVXbMB7fgg8RwtRu19j3OQxB6wxLcVijimShEruBRFgjtFrpwiVmfGujhiqXHbnSPXxRHoepo5ol47unNEL0dc1xr4EXU78rEcUYPbO0f0csSCZn87EmimiHM0MomjzxWWpGniBMFFwhLLVCH%2B0KCkuwR9APHzoiIo%2B1wQXzDydLQHntY9qrh2T296mncD8B5VXB1HOt%2FcrRWOpZkj96ji2jkCoXtktflYlqhxxRMjFGc%2FJpTMcLaiKmP%2BB3uLC%2BwNA%2B07B3jsct0N3RIbYPjiiD%2FVettHcU28xG2eM8E7uG%2Bm%2Fw4gVL883R26Xoc%2BPCaCrnM5h86z%2B9%2BuNmUHPwCGyb8%3D"></iframe>

### 2.1 Todo List
---
**Felix:**
- [x] Python: ETL Excel
- [ ] Python: ETL Excel Metadata
- [ ] Python: ETL DB (developing)
- [ ] Python: Stored Procedure

**小明:**
- [ ] 增删改查
- [ ] ......
