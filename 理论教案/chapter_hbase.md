### 课时： 4学时理论+4学时实践

***
# 1、教学目标 
    什么是Hbase?
    Hbase和传统关系型数据库有什么区别？
    Hbase的特性？
    Hbase的数据模型？Hbase数据模型中的相关概念：行键、表、列族、列限定符、时间戳
    如何理解Hbase是面向列的数据库？
    如何理解Hbase的存储方式？
    Hbase的架构组成？
    Hbase为什么要采用这样的架构方式？
    Hbase架构中各个模块的作用？master的作用？regionserver的作用？zookeeper组件在Hbase架构中的作用？
    Hbase的运行流程？
    Hbase regionserver的运行原理？store、storefile、memstore、hlog等结构等作用？
    Hbase regionserver storefile 合并、拆分过程？
    Hbase的安装及编程实践


# 2、教学内容：
## HBase的概念及HBase的发展史
    
## HBase与传统关系型数据库之间的对比
    HBase数据库的特点；
    
## HBase的访问接口
## HBase的数据模型
    HBase数据模型的相关概念：表、行键、列族、列限定符、单元格、时间戳
    数据坐标
    概念视图
    物理视图
    理解什么是面向列的存储；

## HBase的实现原理
    CS 架构 master服务器负责管理，regionserver服务器负责数据的存储及数据的读写；
    zookeeper组件负责协调，regionserver和master需要在zookeeper上注册，region server需要一直发送心跳包到zookeeper，zookeeper负责监听整个系统，并把异常region server信息发送给master。
    
    zookeeper可以理解为班委，master是老师，regionserver是学生，班委帮助老师实时汇报班级情况。
## HBase的运行机制
    Hbase由Client、master、regionserver组成，并需要zookeeper组件的协助；
    读过程：
        Client通过zookeeper获取数据对应存储的region server，并开启与region server的交互，去region server上进行数据查询；
            regionserver 存在一个缓冲层，memstore是一个内存缓冲层，存放较近时间写入的数据，优先会在memstore中查找，如果memstore中没有目标数据则会到storefile中查找，storefile是磁盘存储。
        找到数据后，region server将结果返回到client；
    写过程：
        Client通过zookeeper获取到应该写入数据的region server，这个region server信息的返回实质上是有master服务器决定。获取到目标region server后，Client通过RPC将数据写入region server的合适的region中。首先是把数据直接写到memstore中，memstore内容足够多后，会把数据flush到store file中也即是磁盘中，完成持久化操作。对于storefile，如果region中storefile较多，storefile会进行合并操作，如果storefile过大，又会执行分裂操作，这个分裂会把一个父region分裂为两个子region。
        这里涉及到一个log的预写过程，对于HBase，为了保证数据的正确性以及意外情况下memstore数据未持久化的情况，所有Hbase的操作都是先写日志后进行操作。

## HBase编程实践
***
# 3、教学重点
    HBase的数据模型
    HBase的实现原理
    HBase的运行原理
    HBase的编程实践
***

# 4、教学难点
    HBase的数据模型
    HBase的实现原理
    HBase的运行原理
    HBase的编程实践

# 5、教学手段
    多媒体教学；
# 6、教学方法
    启发式教学、讨论、课堂提问等方式；

# 7、课后作业   
    主要是实验课作业，包括Hbase的搭建、Hbase相关shell命令的使用以及Hbase对应java API的调用；
# 8、参考资料
# 9、课后小结