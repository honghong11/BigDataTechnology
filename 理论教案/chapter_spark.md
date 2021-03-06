### 课时： 3+4 学时

***
# 1、教学目标 
    掌握Spark的特点，Spark与Hadoop平台的对比；
    了解Scala语言；
    掌握Spark的生态环境，清楚Spark各核心模块及用途；
    学习掌握Spark的系统架构运行原理；
    掌握RDD的设计过程及运行原理；

# 2、教学内容：
## 2.1 Spark简介
    基于内存的大数据并行计算框架；
    运行速度快； 
    容易使用；
    通用性好；
    运行模式多；
## 2.2 Scala语言简介
    多范式编程语言，集成了面向对象和函数式语言的特性；
    兼容Java；
## 2.3 Spark和Hadoop的对比
    Spark编程模型比MapReduce灵活；
    Spark提供内存计算； 
    Spark基于DAG的任务调度机制，优于MapReduce的迭代执行；
## 2.4 Spark生态系统及运行架构
### 2.4.1 Spark生态模块组成
    Spark core
    Spark SQL
    Spark Streaming
    MLlib等
### 2.4.2 Spark架构设计
    RDD、DAG、Executor、应用、任务、作业、阶段;
    集群管理器、工作节点、任务控制节点、负责具体任务的执行进程executor;
### 2.4.3 Spark运行流程
### 2.4.4 RDD的设计与运行原理
    RDD 弹性分布式数据集，本质上是一个只读的分区记录集合。
    RDD的操作分为两种：转换和行动
        转换操作：一个RDD可以通过确定的转换操作得到新的RDD，比如groupBy操作；
        行动操作：接受一个RDD返回一个值或者结果（非RDD）
    Spark的核心建立在统一的抽象RDD之上，使得Spark的各个组件可以无缝地进行集成；
#### 2.4.4.1 RDD概念
    RDD的操作分为两种：转换和行动
        转换操作：一个RDD可以通过确定的转换操作得到新的RDD，比如groupBy操作；
        行动操作：接受一个RDD返回一个值或者结果（非RDD）
    RDD的“血缘关系”
    RDD构成的DAG图；
#### 2.4.4.2 RDD特性
    高效容错
    中间结果持久化道内存
    可存放Java对象，避免序列化
#### 2.4.4.3 RDD之间的依赖关系
    宽依赖
    窄依赖
    Shuffle操作
#### 2.4.4.4 阶段的划分
    根据宽窄依赖划分stage

## 2.5 Spark的部署及应用
# 3、教学重点
    Spark生态环境
    Spark特点
    Spark的系统结构
    Spark的运行原理
    RDD
***

# 4、教学难点
    Spark的运行原理
    RDD

# 5、教学手段
    多媒体教学；
# 6、教学方法
    启发式教学、讨论、课堂提问等方式；

# 7、课后作业
# 8、参考资料
# 9、课后小结
