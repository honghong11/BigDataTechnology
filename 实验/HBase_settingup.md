### 课时： 4-6学时

***
# 0、实验环境
    Win10 + VirtualBox +Ubuntu20.04 + jdk1.8/jdk11 + SSH + Vim + Hadoop2.8.5 + 
    + HBase2.4.11 + Zookeeper(HBase自带) 
# 1、教学目标 
    通过练习让同学掌握HBase在Hadoop平台上的安装，HBas实践。
# 2、教学内容：
    HBase在Hadoop平台上的安装，HBas实践
``` 
    准备工作：

    HBase官网下载HBase2.4.1 binary版本：https://hbase.apache.org 
    下载版本2.4.11 
    将二进制包解压到/usr/local目录下
    sudo tar -zxf ~/Downloads/hbasexxx.tar.gz -C /usr/local
    修改名称为hbase
    sudo mv /usr/local/hbasexxx-2.4.11 /usr/local/hbase
```

```
    配置环境变量
    vim ~/.bashrc
    添加hbase相关路径
    将/usr/local/hbase/bin追加到PATH路径下
    执行source ~/.bashrc 使命令生效
```

```
    添加用户权限
    cd /usr/local
    sudo chown -R hadoop ./hbase
    查看hbase的版本信息
    /usr/local/hbase/bin/hbase version 
    可查看到hbase的版本信息则说明，hbase安装成功
```

## 2.1 HBase 单机版安装
    配置hbase-env.sh文件
    编辑 /usr/local/hbase/conf/hbase-env.sh
    export JAVA_HOME=/jdk路径
    export HBASE_MANAGES_ZK=true  //使用hbase自带的zookeeper


    配置hbase-site.xml文件
    编辑 /usr/local/hbase/conf/hbase-env.sh
```
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>file:///usr/local/habse/hbase-tmp</value>
  </property>
</configuration>
```

    运行hbase
    start-hbase.sh
    进入hbase的命令行模式 hbase shell
    停止hbase
    stop-hbase.sh

## 2.2 HBase 完全分布式安装
    首先，需要安装hadoop2.8.5版本，下载hadoop2.8.5版本，配置core-site.xml，hdfs-site.xml，yarn-site.xml，marped-site.xml，workers文件。因为之前已经安装好3.x.x版本，所以，hadoop相关的jdk、ssh等模块不需要重新安装，需要将3.x.x版本的Hadoop中上述5个配置文件拷贝到2.8.5版本中，但是需要注意里面的一些参数路径。比如core-site.xml文件中的hadoop.tmp.dir对应的value，需要修改为file:/usr/local/hadoop2.8.5(hadoop2.8.5的存储路径)/tmp。
    环境变量~/.bashrc文件中，也需要注意修改hadoop的安装路径，修改为2.8.5版本hadoop安装路径
    同时注意一点，hadoop2.8.5中没有workers文件，而是对应为slaves文件。
    


    配置hbase-env.sh文件
```
export JAVA_HOME=/usr/lib/jvm/java8/jdk1.8.0_181  //Java路径
export HBASE_CLASSPATH=/usr/local/hbase/conf  //hbase路径
export HBASE_MANAGES_ZK=true  //是否使用HBase自带的zookeeper

```

    配置hbsae-site.xml文件
```
<property>
        <name>hbase.cluster.distributed</name>
    <value>true</value> //是否采用分布式
  </property>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://master:9000/hbase</value>  //hbase在hdfs中的路径
  </property>
  <property>
	  <name>hbase.zookeeper.quorum</name>
          <value>master,slave01</value> //zookeeper配置
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
  <property>
	<name>hbase.master.info.port</name>
	<value>60010</value> //hbase web页面，可在浏览器上通过master:60010 查看
  </property>
</configuration>
```
    配置regionservers文件
 ```
    master
    slave01
```
    将/usr/local/hadoop/etc/hadoop/下的core-site.xml和hbase-site.xml文件拷贝到/usr/local/hbase/conf目录下

## 2.3 Hbase的启动停止
    首先需要启动hadoop
        start-dfs.sh
        start-yarn.sh
        这个时候master节点的进程：
            NameNode
            SecondaryNameNode
            ResourceManager
        slave01节点的进程：
            DataNode
            NodeManager

    接着启动hbase
        start-hbase.sh
        这时候master节点的进程：
            NameNode
            SecondaryNameNode
            ResourceManager
            HQuorumPeer //zookeeper进程
            HRegionServer 
            HMaster
        slave01节点进程：
            DataNode
            NodeManager
            HQuorumPeer
            HRegionServe
    停止Hbase
    命令执行顺序如下：
        stop-hbase.sh  
            hbaes-daemon.sh stop master //关闭当前节点hmaster进程
            hbase-daemons.sh stop master //关闭所有节点hmaster进程
            hbase-daemon.sh stop regionserver //关闭当前节点的regionserver进程
            hbase-daemons.sh stop regionserver //关闭所有节点的regionserver进程
        stop-yarn.sh
        stop-dfs.sh
    
## 2.4 Hbase shell命令使用
    Hbase单机、全分布式的shell命令使用都一样：

    进入hbase shell页面：
    hbase shell
    表创建
    create 'student','Sname','Ssex','Sage','Sdept','course'  通常会把第一个属性作为行键

    添加
    put
    一次只能为一个单元格添加一个数据
    put 'student','95001','Sname','LiYing'
    put 'student','95001','Sage','22'
    put 'student','95001','Sdept','cs'
    put 'student','95001','course:math','80'

    查看
    get 查看表的某个单元格数据
    get 'student', '95001'
    scan 查看某个表的所有数据
    scan 'student'


    删除
    delete 删除一个单元格数据
    delete 'student', '95001','Ssex'
    deleteall 删除一行数据
    deleteall 'student', '95001'


    删除表
    删除表分两步，第一步让该表不可用，第二步删除表
    disable 'student' 
    drop 'student'

    查询历史数据
    hbase存在时间戳的概念，在创建表的时候，需要指定版本数
    create 'teacher',{NAME=>'username',VERSION=>5}
    put 'teacher','91001','username','Mary'
    put 'teacher','91001','username','Mary1'
    put 'teacher','91001','username','Mary2'
    put 'teacher','91001','username','Mary3'
    put 'teacher','91001','username','Mary4'
    put 'teacher','91001','username','Mary5'

    查看历史数据：
    get 'teacher','91001',{COLUME=>'username',VERSION=>3}

    退出hbase shell 
    exit

## 注 问题处理
    0、首先需要注意，完全分布式的hbase是基于Hadoop的，所以一定要把hadoop先起来。
    1、运行eclipse的时候，master节点最好可以多分配一些内存资源
    2、提示/hbase/master出错，但是查看jps发现各个进程都是对着的，可以到hdfs文件系统中删除 /hbase文件夹，重新启动下整个系统。

## 2.4 完全分布式Hbase环境下调用hbase Java api 对Hbase进行操作

    这里还是使用eclipse，首先需要添加Hbase相关的jar包：
        hbaes目录下/usr/local/hbase/lib目录下的jar包以及
        /usr/local/hbase/lib/client-facing-thirdparty目录中的jar包添加到Java工程的classpath中

    hbase api  https://hbase.apache.org/apidocs/index.html
```

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;
 
import java.io.IOException;
public class ExampleForHBase {
    public static Configuration configuration;
    public static Connection connection;
    public static Admin admin;
    public static void main(String[] args)throws IOException{
        init();
        createTable("student",new String[]{"score"});
        insertData("student","zhangsan","score","English","69");
        insertData("student","zhangsan","score","Math","86");
        insertData("student","zhangsan","score","Computer","77");
        getData("student", "zhangsan", "score","English");
        close();
    }
 
 
    /**
    * 初始化Hbase 
    */
    public static void init(){
        configuration  = HBaseConfiguration.create();
        configuration.set("hbase.rootdir","hdfs://localhost:9000/hbase");
        try{
            connection = ConnectionFactory.createConnection(configuration);
            admin = connection.getAdmin();
        }catch (IOException e){
            e.printStackTrace();
        }
    }
 
    public static void close(){
        try{
            if(admin != null){
                admin.close();
            }
            if(null != connection){
                connection.close();
            }
        }catch (IOException e){
            e.printStackTrace();
        }
    }
 
    public static void createTable(String myTableName,String[] colFamily) throws IOException {
        TableName tableName = TableName.valueOf(myTableName);
        if(admin.tableExists(tableName)){
            System.out.println("talbe is exists!");
        }else {
            TableDescriptorBuilder tableDescriptor = TableDescriptorBuilder.newBuilder(tableName);
            for(String str:colFamily){
                ColumnFamilyDescriptor family = 
ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes(str)).build();
                tableDescriptor.setColumnFamily(family);
            }
            admin.createTable(tableDescriptor.build());
        } 
    }
 
    public static void insertData(String tableName,String rowKey,String colFamily,String col,String val) throws IOException { 
        Table table = connection.getTable(TableName.valueOf(tableName));
        Put put = new Put(rowKey.getBytes());
        put.addColumn(colFamily.getBytes(),col.getBytes(), val.getBytes());
        table.put(put);
        table.close(); 
    }
 
    public static void getData(String tableName,String rowKey,String colFamily, String col)throws  IOException{ 
        Table table = connection.getTable(TableName.valueOf(tableName));
        Get get = new Get(rowKey.getBytes());
        get.addColumn(colFamily.getBytes(),col.getBytes());
        Result result = table.get(get);
        System.out.println(new String(result.getValue(colFamily.getBytes(),col==null?null:col.getBytes())));
        table.close(); 
    }
}

```


```
package cn.swordfall.hbaseOnJava;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.filter.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @Author: Yang JianQiu
 * @Date: 2019/1/22 17:46
 *
 * HBaseAdmin用于创建表，table用于增删改查数据
 */

public class HBaseJavaApiDemo {

    /**
     * 声明静态配置
     */
    static Configuration conf = null;
    static Connection conn = null;
    static {
        conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum", "hadoop01,hadoop02,hadoop03");
        conf.set("hbase.zookeeper.property.client", "2181");
        try{
            conn = ConnectionFactory.createConnection(conf);
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    /**
     * 创建只有一个列簇的表
     * @throws Exception
     */
    public static void createTable() throws Exception{
        Admin admin = conn.getAdmin();
        if (!admin.tableExists(TableName.valueOf("test"))){
            TableName tableName = TableName.valueOf("test");
            //表描述器构造器

            TableDescriptorBuilder tdb = TableDescriptorBuilder.newBuilder(tableName);
            //列族描述器构造器
            ColumnFamilyDescriptorBuilder cdb = ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes("user"));
            //获得列描述器
            ColumnFamilyDescriptor cfd = cdb.build();
            //添加列族
            tdb.setColumnFamily(cfd);
            //获得表描述器
            TableDescriptor td = tdb.build();
            //创建表
            admin.createTable(td);

        }else {
            System.out.println("表已存在");
        }
        //关闭连接
    }

    /**
     * 创建表（包含多个列族）
     * @param tableName
     * @param columnFamilys
     * @throws Exception
     */
    public static void createTable(TableName tableName, String[] columnFamilys) throws Exception{
        Admin admin = conn.getAdmin();
        if (!admin.tableExists(tableName)){
            //表描述构造器
            TableDescriptorBuilder tdb = TableDescriptorBuilder.newBuilder(tableName);
            //列族描述构造器
            ColumnFamilyDescriptorBuilder cdb;
            //获得列描述器
            ColumnFamilyDescriptor cfd;
            for (String columnFamily: columnFamilys){
                cdb = ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes(columnFamily));
                cfd = cdb.build();
                //添加列族
                tdb.setColumnFamily(cfd);
            }
            //获得表描述器
            TableDescriptor td = tdb.build();
            //创建表
            admin.createTable(td);
        }else {
            System.out.println("表已存在！");
        }
        //关闭链接
    }

    /**
     * 添加数据（多个rowKey，多个列族，适合由固定结构的数据）
     * @param tableName
     * @param list
     * @throws Exception
     */
    public static void insertMany(TableName tableName, List<Map<String, Object>> list) throws Exception{
        List<Put> puts = new ArrayList<Put>();
        //Table负责跟记录相关的操作如增删改查等
        Table table = conn.getTable(tableName);
        if (list != null && list.size() > 0){
            for (Map<String, Object> map: list){
                Put put = new Put(Bytes.toBytes(map.get("rowKey").toString()));
                put.addColumn(Bytes.toBytes(map.get("columnFamily").toString()), Bytes.toBytes(map.get("columnName").toString()), Bytes.toBytes(map.get("columnValue").toString()));
                puts.add(put);
            }
        }
        table.put(puts);
        table.close();
        System.out.println("add data Success!");
    }

    /**
     * 添加数据（多个rowKey，多个列族）
     * @throws Exception
     */
    public static void insertMany() throws Exception{
        Table table = conn.getTable(TableName.valueOf("test"));
        List<Put> puts = new ArrayList<Put>();
        Put put1 = new Put(Bytes.toBytes("rowKey1"));
        put1.addColumn(Bytes.toBytes("user"), Bytes.toBytes("name"), Bytes.toBytes("wd"));

        Put put2 = new Put(Bytes.toBytes("rowKey2"));
        put2.addColumn(Bytes.toBytes("user"), Bytes.toBytes("age"), Bytes.toBytes("25"));

        Put put3 = new Put(Bytes.toBytes("rowKey3"));
        put3.addColumn(Bytes.toBytes("user"), Bytes.toBytes("weight"), Bytes.toBytes("60kg"));

        Put put4 = new Put(Bytes.toBytes("rowKey4"));
        put4.addColumn(Bytes.toBytes("user"), Bytes.toBytes("sex"), Bytes.toBytes("男"));

        puts.add(put1);
        puts.add(put2);
        puts.add(put3);
        puts.add(put4);
        table.put(puts);
        table.close();
    }

    /**
     * 根据RowKey , 列簇， 列名修改值
     * @param tableName
     * @param rowKey
     * @param columnFamily
     * @param columnName
     * @param columnValue
     * @throws Exception
     */
    public static void updateData(TableName tableName, String rowKey, String columnFamily, String columnName, String columnValue) throws Exception{
        Table table = conn.getTable(tableName);
        Put put1 = new Put(Bytes.toBytes(rowKey));
        put1.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(columnName), Bytes.toBytes(columnValue));
        table.put(put1);
        table.close();
    }

    /**
     * 根据rowKey删除一行数据
     * @param tableName
     * @param rowKey
     * @throws Exception
     */
    public static void deleteData(TableName tableName, String rowKey) throws Exception{
        Table table = conn.getTable(tableName);
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        table.delete(delete);
        table.close();
    }

    /**
     * 删除某一行的某一个列簇内容
     * @param tableName
     * @param rowKey
     * @param columnFamily
     * @throws Exception
     */
    public static void deleteData(TableName tableName, String rowKey, String columnFamily) throws Exception{
        Table table = conn.getTable(tableName);
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        delete.addFamily(Bytes.toBytes(columnFamily));
        table.delete(delete);
        table.close();
    }

    /**
     * 删除某一行某个列簇某列的值
     * @param tableName
     * @param rowKey
     * @param columnFamily
     * @param columnName
     * @throws Exception
     */
    public static void deleteData(TableName tableName, String rowKey, String columnFamily, String columnName) throws Exception{
        Table table = conn.getTable(tableName);
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        delete.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(columnName));
        table.delete(delete);
        table.close();
    }

    /**
     * 删除某一行某个列簇多个列的值
     * @param tableName
     * @param rowKey
     * @param columnFamily
     * @param columnNames
     * @throws Exception
     */
    public static void deleteData(TableName tableName, String rowKey, String columnFamily, List<String> columnNames) throws Exception{
        Table table = conn.getTable(tableName);
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        for (String columnName: columnNames){
            delete.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(columnName));
        }
        table.delete(delete);
        table.close();
    }

    /**
     * 根据rowKey查询数据
     * @param tableName
     * @param rowKey
     * @throws Exception
     */
    public static void getResult(TableName tableName, String rowKey) throws Exception{
        Table table = conn.getTable(tableName);
        //获得一行
        Get get = new Get(Bytes.toBytes(rowKey));
        Result set = table.get(get);
        Cell[] cells = set.rawCells();
        for (Cell cell: cells){
            System.out.println(Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength()) + "::" +
            Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength()));
        }
        table.close();
    }

    /**
     * 全表扫描
     * @param tableName
     * @throws Exception
     */
    public static void scanTable(TableName tableName) throws Exception{
        Table table = conn.getTable(tableName);
        Scan scan = new Scan();
        ResultScanner rscan = table.getScanner(scan);
        for (Result rs : rscan){
            String rowKey = Bytes.toString(rs.getRow());
            System.out.println("row key :" + rowKey);
            Cell[] cells = rs.rawCells();
            for (Cell cell: cells){
                System.out.println(Bytes.toString(cell.getFamilyArray(), cell.getFamilyOffset(), cell.getFamilyLength()) + "::"
                        + Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength()) + "::"
                        + Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength()));
            }
            System.out.println("-------------------------------------------");
        }
    }

    //过滤器 LESS <  LESS_OR_EQUAL <=   EQUAL =   NOT_EQUAL <>   GREATER_OR_EQUAL >=   GREATER >   NO_OP 排除所有

    /**
     * rowKey过滤器
     * @param tableName
     * @throws Exception
     */
    public static void rowKeyFilter(TableName tableName) throws Exception{
        Table table = conn.getTable(tableName);
        Scan scan = new Scan();
        //str$ 末尾匹配，相当于sql中的 %str  ^str开头匹配，相当于sql中的str%
        RowFilter filter = new RowFilter(CompareOperator.EQUAL, new RegexStringComparator("Key1$"));
        scan.setFilter(filter);
        ResultScanner scanner = table.getScanner(scan);
        for (Result rs: scanner){
            String rowKey = Bytes.toString(rs.getRow());
            System.out.println("row key: " + rowKey);
            Cell[] cells = rs.rawCells();
            for (Cell cell: cells){
                System.out.println(Bytes.toString(cell.getFamilyArray(), cell.getFamilyOffset(), cell.getFamilyLength()) + "::"
                        + Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength()) + "::"
                        + Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength()));
            }
            System.out.println("--------------------------------------------");
        }
    }

    /**
     * 列值过滤器
     * @param tableName
     * @throws Exception
     */
    public static void singColumnFilter(TableName tableName) throws Exception{
        Table table = conn.getTable(tableName);
        Scan scan = new Scan();
        //下列参数分别为列族，列名，比较符号，值
        SingleColumnValueFilter filter = new SingleColumnValueFilter(Bytes.toBytes("author"), Bytes.toBytes("name"),
                CompareOperator.EQUAL, Bytes.toBytes("spark"));
        scan.setFilter(filter);
        ResultScanner scanner = table.getScanner(scan);
        for (Result rs: scanner){
            String rowKey = Bytes.toString(rs.getRow());
            System.out.println("row key: " + rowKey);
            Cell[] cells = rs.rawCells();
            for (Cell cell: cells){
                System.out.println(Bytes.toString(cell.getFamilyArray(), cell.getFamilyOffset(), cell.getFamilyLength()) + "::"
                        + Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength()) + "::"
                        + Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength()));
            }
            System.out.println("-----------------------------------------------");
        }
    }

    /**
     * 列名前缀过滤器
     * @param tableName
     * @throws Exception
     */
    public static void columnPrefixFilter(TableName tableName) throws Exception{
        Table table = conn.getTable(tableName);
        Scan scan = new Scan();
        ColumnPrefixFilter filter = new ColumnPrefixFilter(Bytes.toBytes("name"));
        scan.setFilter(filter);
        ResultScanner scanner = table.getScanner(scan);
        for (Result rs: scanner){
            String rowKey = Bytes.toString(rs.getRow());
            System.out.println("row key:" + rowKey);
            Cell[] cells = rs.rawCells();
            for (Cell cell : cells){
                System.out.println(Bytes.toString(cell.getFamilyArray(),cell.getFamilyOffset(),cell.getFamilyLength())+"::"+Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength())+"::"+
                        Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength()));
            }
            System.out.println("-----------------------------------------");
        }
    }

    /**
     * 过滤器集合
     * @param tableName
     * @throws Exception
     */
    public static void filterSet(TableName tableName) throws Exception{
        Table table = conn.getTable(tableName);
        Scan scan = new Scan();
        FilterList list = new FilterList(FilterList.Operator.MUST_PASS_ALL);
        SingleColumnValueFilter filter1 = new SingleColumnValueFilter(Bytes.toBytes("author"), Bytes.toBytes("name"),
                CompareOperator.EQUAL, Bytes.toBytes("spark"));
        ColumnPrefixFilter filter2 = new ColumnPrefixFilter(Bytes.toBytes("name"));
        list.addFilter(filter1);
        list.addFilter(filter2);

        scan.setFilter(list);
        ResultScanner scanner = table.getScanner(scan);
        for (Result rs: scanner){
            String rowkey = Bytes.toString(rs.getRow());
            System.out.println("row key :"+rowkey);
            Cell[] cells  = rs.rawCells();
            for(Cell cell : cells) {
                System.out.println(Bytes.toString(cell.getFamilyArray(),cell.getFamilyOffset(),cell.getFamilyLength())+"::"+Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength())+"::"+
                        Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength()));
            }
            System.out.println("-----------------------------------------");
        }

    }

    public static void main(String[] args) throws Exception{
        //创建表（只有一个列簇）
//        createTableOne();

        // 创建表(包含多个列簇)
        TableName tableName = TableName.valueOf("test");
        String[] columnFamilys = { "article", "author" };
//        createTable(tableName, columnFamilys);

        //添加数据
        List<Map<String,Object>> listMap=new ArrayList<Map<String, Object>>();
        Map<String,Object> map1=new HashMap<String,Object>();
        map1.put("rowKey","ce_shi1");
        map1.put("columnFamily","article");
        map1.put("columnName","title");
        map1.put("columnValue","Head First HBase");
        listMap.add(map1);
        Map<String,Object> map2=new HashMap<String,Object>();
        map2.put("rowKey","ce_shi1");
        map2.put("columnFamily","article");
        map2.put("columnName","content");
        map2.put("columnValue","HBase is the Hadoop database");
        listMap.add(map2);
        Map<String,Object> map3=new HashMap<String,Object>();
        map3.put("rowKey","ce_shi1");
        map3.put("columnFamily","article");
        map3.put("columnName","tag");
        map3.put("columnValue","Hadoop,HBase,NoSQL");
        listMap.add(map3);
        Map<String,Object> map4=new HashMap<String,Object>();
        map4.put("rowKey","ce_shi1");
        map4.put("columnFamily","author");
        map4.put("columnName","name");
        map4.put("columnValue","nicholas");
        listMap.add(map4);
        Map<String,Object> map5=new HashMap<String,Object>();
        map5.put("rowKey","ce_shi1");
        map5.put("columnFamily","author");
        map5.put("columnName","nickname");
        map5.put("columnValue","lee");
        listMap.add(map5);
        Map<String,Object> map6=new HashMap<String,Object>();
        map6.put("rowKey","ce_shi2");
        map6.put("columnFamily","author");
        map6.put("columnName","name");
        map6.put("columnValue","spark");
        listMap.add(map6);
        Map<String,Object> map7=new HashMap<String,Object>();
        map7.put("rowKey","ce_shi2");
        map7.put("columnFamily","author");
        map7.put("columnName","nickname");
        map7.put("columnValue","hadoop");
        listMap.add(map7);
//        insertMany(tableName,listMap);

        //根据RowKey，列簇，列名修改值
        String rowKey="ce_shi2";
        String columnFamily="author";
        String columnName="name";
        String columnValue="hbase";
//        updateData(tableName,rowKey,columnFamily,columnName,columnValue);


        String rowKey1="ce_shi1";
        String columnFamily1="article";
        String columnName1="name";
        List<String> columnNames=new ArrayList<String>();
        columnNames.add("content");
        columnNames.add("title");
        //删除某行某个列簇的某个列
//        deleteData(tableName,rowKey1,columnFamily1,columnName1);
        //删除某行某个列簇
//        deleteData(tableName,rowKey1,columnFamily1);
        //删除某行某个列簇的多个列
//        deleteData(tableName,rowKey1,columnFamily1,columnNames);
        //删除某行
//        deleteData(tableName,rowKey1);
        //根据rowKey查询数据
//        getResult(tableName,"rowKey1");

        //全表扫描
        scanTable(tableName);

        //rowKey过滤器
//        rowkeyFilter(tableName);

        //列值过滤器
//        singColumnFilter(tableName);

        //列名前缀过滤器
//        columnPrefixFilter(tableName);

        //过滤器集合
//        filterSet(tableName);
    }
}

```
***
# 3、教学重点
    HBase环境搭建
    HBase shell 命令
    HBase java api调用

# 4、教学难点
    HBase完全分布式环境搭建，Java API的调用
# 5、教学手段
    多媒体教学；
# 6、教学方法
    演示法；

# 7、课后作业
    书后习题
# 8、参考资料
# 9、课后小结