### 课时： 2-3 + 4学时

***
# 1、教学目标 
    让同学理解MapReduce过程，MapReduce框架和传统的分布式计算的区别以及MapReduce的设计理念；
    MapReduce的体系结构；
    MapReduce的工作流程；
        

        Shuffle过程；
            溢写、分区排序、合并；
# 2、教学内容：
## 2.1 MapReduce框架概述

## 2.2 MapReduce体系结构
    mapreduce 主从架构；
    mapreduce 结构：Client、JobTracker（master）、TaskTracker（slave），Task以及各个模块的作用；

## 2.3 MapReduce工作流程
    mapreduce执行过程：
        client提交作业；
        JobTracker 初始化作业、分配作业并与TaskTracker通信并协调整个作业；
        HDFS 共享节点之间的作业文件；
        TaskTracker接收任务后，从HDFS上获取作业资源，包括作业配置和本作业分片的输入；
    MapReduce各个执行阶段，MapReduce的执行流程；
                Split、Map、Shuffle、Reduce；
        
## 2.4 WordCount实例分析
***

# 3、教学重点
    MapReduce 体系结构；
    MapReduce 工作流程；
    MapReduce 编程实践；
***

# 4、教学难点
    MapReduce 工作流程；
    MapReduce 编程实践；

# 5、教学手段
    多媒体教学；
# 6、教学方法
    启发式教学、讨论、课堂提问等方式；

# 7、课后作业

# 8、参考资料
# 9、课后小结
