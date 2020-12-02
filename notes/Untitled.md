### DBI

### 基本流程

```flow
op1=>operation: 创建数据源
op2=>operation: 创建数据集
op3=>operation: 创建图表
op4=>operation: 拼装看板
op1(right)->op2(right)->op3(right)->op4
```

#### 数据源

1. 类型: MySQL、Oracle、SQL Server
2. 名称

#### 数据集

> **数据集**类似 OLAP 分析的 Cube，可以提前定义 `聚合`、`聚合表达式`、`动态时间漏斗`。在用户*数据模型比较稳定*的前提下，可以减少相同数据集下不同报表设计时重复的填写查询脚本、新建聚合表达式工作。

任何一条简单的查询输出都可以当做一个 cube 来用，cube 修改避免了常规数据建模、修改、发布、测试的繁琐过程

1. 建模

   数据集的模型定义包含 `维度列`、`度量列`、`聚合表达式`、`预定义漏斗`

   * 选择数据源，填写对应的查询脚本
   * 读取数据成功后，选择列作为维度或度量
   * 一个列可在不同的维度层级下重复使用，如：年-月-日，年-周-日
   * 维度列在图表设计时只能拖拽到维度栏；指标（度量）列只能拖拽到指标栏，聚合函数需要在设计时选定

#### 图表



#### 看板