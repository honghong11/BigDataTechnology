### 课时： 2学时

***
# 1、教学目标 
    通过分析介绍第二章Hadoop1.0存在的问题，引入Hadoop2.0，分析总结Hadoop2.0的改进与提升。使得同学们对Hadoop的改进过程有深刻的理解，能够形成一种对已有的知识体系批判改进的思维。
    学习掌握 HDFS HA 、HDFS 联邦 、YARN;
    了解Pig、Tez、Kafka等功能组件；
# 2、教学内容：
## 2.1 Hadoop1.0的局限与不足
## 2.2 针对Hadoop1.0的改进与提升

## 2.3 Hadoop2.0新特性
### 2.3.1 HDFS HA 
    两个名称节点解决Hadoop1.0存在的单点故障问题；
    HDFS HA的架构
### 2.3.2 HDFS 联邦
    HDFS 联邦的设计思路：设计了多个相互独立的名称节点，使得HDFS的命名空间可以横向扩展，提供各进程之间的隔离性；
    HDFS 联邦的访问方式；
    HDFS 联邦对比Hadoop1.0的优势；
## 2.4 资源管理调度框架YARN
### 2.4.1 MapReduce1.0的缺陷
    单点故障、JobTracker需要同时负责资源调度和作业调度、资源划分不合理等； 
### 2.4.2 YARN的设计思路
### 2.4.3 YARN的体系结构
    ResourceManager
    ApplicationMaster
    NodeManager
### 2.4.4 YARN的工作流程；
### 2.4.5 YARN框架与MapReduce1.0对比
### 2.4.6 YARN的发展目标

## 2.5 pig
    允许用户通过编写简单的脚本来实现复杂的数据分析；
    Pig Latin 脚本；
## 2.6 Tez 
## 2.7 Kafka
    一种高吞吐的分布式发布/订阅消息系统

# 3、教学重点
    HDFS HA
    HDFS 联邦
    YARN
    pig，Tez，Kafka
***

# 4、教学难点
    YARN 工作流程；
    HDFS 联邦 文件访问；
    Tez组件；

# 5、教学手段
    多媒体教学；
# 6、教学方法
    启发式教学、讨论、课堂提问等方式；

# 7、课后作业

# 8、参考资料
# 9、课后小结
