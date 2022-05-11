# Hive 实验

## 实验环境
    Ubuntu20.04 + Hadoop 3.3.1 + jdk1.8 + hive3.1.2 + mysql 8;

    hive官网下载速度较慢，可以使用腾讯的镜像：https://mirrors.cloud.tencent.com/apache/hive  实验室环境是选用的3.1.2
    jdk8下载链接http://www.jdkdownload.com/ 

## 实验目的
    掌握使用mysql作为元数据存储模块的hive的搭建过程；
    练习使用HiveQL语句，对常用的HiveQL语句有一定的掌握；
    学习使用HiveQL编写处理MR程序；

## 环境搭建
    之前我们已经搭建好了Hadoop的环境，这里主要是mysql + hive ，以及对出现的一些问题的解决；
### ubuntu 上构建mysql
    如果本机上之前有安装过mysql并且没有删除完整，可能会对后面的环境搭建造成影响，可以用一下方式进行卸载：
        sudo apt purge mysql-*
        sudo rm -r /etc/mysql
        sudo rm -r /var/lib/mysql
        sudo apt autoremove
        sudo apt autoclean
    安装，这里使用apt的安装方式：
        sudo apt-get install mysql-server -y
        sudo apt install mysql-client -y
        sudo apt install libmysqlclient-dev -y
    检查是否安装成功：
        sudo netstat -tap | grep mysql
    如果出现有mysqld相关信息，则说明安装mysql成功；
    
    本地连接mysql，即本地登录。由于安装过程中没有设置root 用户密码，无法直接登录。可以通过以下方式，使用默认用户名密码登录：
        sudo cat /etc/mysql/debian.cnf
        并使用其中client部分的 user和password进行登录，比如user = debian-sys-maint  password = EFDSFSDdfljs231
        登录方法：
            service  mysql stop // 安装完成mysql后，默认会自启动
            service  mysql start //启动mysql服务
            mysql -u debian-sys-maint -p  //回车
            // 输入密码，进入mysql

        进入到mysql之后，我们可以设置一个hive用户，命令如下：
            use mysql;
            flush privileges; //刷新一下配置
            ALTER USER 'hive'@'localhost' identified with mysql_native_password by '这里输入密码';
            flush privileges;
        之后，就可以使用hive这个账户登录mysql；
### 配置hive
    下载hive后，解压到/usr/local目录下：
        sudo tar -zxvf apache-hive-3.1.2.tar.gz -C /usr/local
        sudo mv apache-hive-3.1.2 hive
        sudo chown -R hadoop:hadoop hive
    配置环境变量：
        vim ~/.bashrc 
        export HIVE_HOME=/usr/local/hive
        export PATH = $PATH:$HIVE_HOME/bin  // 加入到PATH 中
        source ～/.bashrc
    修改配置文件hive-site.xml：

    下载并导入mysql jdbc驱动：
        www.mysql.com/downloads/connector/j  //这里的版本实验环境是用的最新的；
        解压；
        并将其中的jar文件复制到/usr/local/hive/lib 中；
    配置mysql 允许hive接入
        首先进入mysql
        执行：
            grant all on *.* to 'hive'@'localhost' identifid by 'hive'； //这里的密码hive是在hive-site文件中配置的；
            ```
            <?xml version="1.0" encoding="UTF-8" standalone="no"?>
            <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
            <configuration>
            <property>
                <name>javax.jdo.option.ConnectionURL</name>
                <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
                <description>JDBC connect string for a JDBC metastore</description>
            </property>
            <property>
                <name>javax.jdo.option.ConnectionDriverName</name>
                <value>com.mysql.jdbc.Driver</value>
                <description>Driver class name for a JDBC metastore</description>
            </property>
            <property>
                <name>javax.jdo.option.ConnectionUserName</name>
                <value>hive</value>
                <description>username to use against metastore database</description>
            </property>
            <property>
                <name>javax.jdo.option.ConnectionPassword</name>
                <value>hive</value>
                <description>password to use against metastore database</description>
            </property>
            </configuration>
            ```
    启动hive：
        首先启动Hadoop
        之后执行：
            hive
        之后可正常进入hive 命令行内


## HiveQL练习
     
    HiveQL 数据类型
    TINYINT: 1个字节
    SMALLINT: 2个字节
    INT: 4个字节
    BIGINT: 8个字节
    BOOLEAN: TRUE/FALSE
    FLOAT: 4个字节，单精度浮点型
    DOUBLE: 8个字节，双精度浮点型
    STRING 字符串

    ARRAY: 有序字段
    MAP: 无序字段
    STRUCT: 一组命名的字段


    常用HiveQL 语句：
    create database if not exists hive;       #创建数据库
    show databases;                           #查看Hive中包含数据库
    show databases like 'h.*';                #查看Hive中以h开头数据库
    describe databases;                       #查看hive数据库位置等信息

    use hive;                                 #切换到hive数据库下
    drop database if exists hive;             #删除不含表的数据库
    drop database if exists hive cascade;     #删除数据库和它中的表

    //建表
    use hive;
    create table usr(id bigint, name string,age int)// 在hive仓库内创建表usr,并给定三个属性： id（bigint）, name(string), age(int)

    加载数据：
    usr hive;
    create table if not exists stu(id int,name string) 
    row format delimited fields terminated by '\t'; //以 tab作分割，需要设定分隔符，否则会出现null
    load data inpath '/home/hadoop/input/stu1.txt' overwrite into stu;
    select * from stu;


    HiveQL 应用实例： wordcount
    在本地文件系统中，新建两个文件，文件内容为文本，比如 I love Hadoop
    use hive;
    create table docs(line string);
    导入数据 load data inpath 'file:///home/hadoop.....' overwrite into docs;
    切分数据：
    create table words(line string);
    insert into table words select explode(split(line," ")) from docs; //从docs表中，对数据通过空格作切分导入到表words中；
    select * from words;
    累加：
    select line, count(line) from words group by line;


## 相关问题
    1、java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument
    com.google.common.base.Preconditions.checkArgument 这是因为hive内依赖的guava.jar和hadoop内的版本不一致造成的。
    解决办法：
    查看hadoop安装目录下share/hadoop/common/lib内guava.jar版本
    查看hive安装目录下lib内guava.jar的版本 如果两者不一致，删除版本低的，并拷贝高版本的 问题解决

    2、启动Hive时，有可能会出现Hive metastore database is not initialized的错误
    或者 version information not found in metastore错误。
    schematool -dbType mysql -initSchema //hive 提供了一个schematool 命令 可以用来处理初始化相关问题，这句话表示我们对元数据模块使用mysql，并进行初始化

    3、通过hive进行mr过程时，出现：java.lang.NoSuchFieldException:parentOffset，表示当前jdk版本存在问题。目前hive最高适配jdk8，查看是否master节点或者slave节点是高于jdk8的版本；

    4、之前我们在yarn-site.xml 中设置了mr的web端页面，入口为：http://master:8088，可以通过改网页借口，方便查看mr过程出现的错误；

    5、could not find or load main class org.apache.hadoop.mapreduce.v2.app.MRAppMaster错误：
    反射过程，存在类无法找到或者加载；
    在所有节点的yarn-site.xml文件中加入hadoop classpath
    在命令行中 hadoop classpath 获取hadoop classpath
    在yarn-site.xml中添加
        <property>
            <name>yarn.application.classpath</name>
            <value>复制hadoop classpath内容</value>
        </property>

