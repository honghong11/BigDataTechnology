### 课时： 2学时

***
# 0、实验环境
    Win10 + VirtualBox +Ubuntu20.04 + jdk1.8/jdk11 + SSH + Vim + Hadoop3.3.1 + Eclipse
# 1、教学目标 
    通过练习掌握，HDFS常用的Shell命令以及HDFS的常用Java Api调用。
# 2、教学内容：
    主要包括HDFS shell命令的使用，HDFS Web管理页面的查看使用以及HDFS Java API的练习使用。
## 2.1 HDFS shell命令的使用
    查看hdfs相关shell命令
    hdfs dfs 
    hdfs dfs -help put #查看具体某个命令的使用方法
    创建用户目录
        cd /usr/local/hadoop
        hdfs dfs -mkdir -p /user/hadoop
    查看目录
        hdfs dfs -ls/
    本地文件系统文件上传至HDFS中
        hdfs dfs -put /本地文件系统文件的存储路径 /hdfs中某目录
    HDFS中的文件下载到本地
        hdfs dfs -get /hdfs某目录中的某文件 /本地文件系统存储路径
    HDFS中的文件从一个目录下复制到另个HDFS文件目录下
        hdfs dfs -cp /hdfs某目录下文件 /hdfs另一目录文件

## 2.2 HDFS Web页面的使用
    master:9780

## 2.3 HDFS Java API调用
    http:hadoop.apache.org/docs/stable/api/ 提供了完整的Hadoop API文档
    准备：
    eclipse下载安装/或者vs code；
    右键项目，选择build path 选择 Configure build path  选择Libraries 选择bbbbb    classpath 右侧add external jars 
    添加HDFS相关jar包：
        /usr/local/hadoop/share/hadoop/common 目录下的所有jar包，注意不包括子级目录中的jar包；
        /usr/local/hadoop/share/hadoop/common/lib 目录下所有的jar包；
        /usr/local/hadoop/share/hadoop/hdfs 目录下的所有jar包，注意不包括子级目录中的jar包；
        /usr/local/hadoop/share/hadoop/hdfs/lib目录下所有jar包；
### 2.3.1 HDFS文件查找
```
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;


    public class HDFSFileIfExist {
        public static void main(String[] args){
            try{
                String fileName = "10.txt";
                Configuration conf = new Configuration();
                conf.set("fs.defaultFS", "hdfs://master:9000"); //core-site.xml中
                conf.set("fs.hdfs.impl", "org.apache.hadoop.hdfs.DistributedFileSystem");
                FileSystem fs = FileSystem.get(conf);
    //            System.out.println(fs.getHomeDirectory());
    //            fs.create(new Path("/user/hadoop/3334.txt"));
                Path path = new Path(fileName);
                System.out.println(path.toString());
                if(fs.exists(path)){
                    System.out.println("文件存在");
                }else{
                    System.out.println("文件不存在");
                }

            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
```
    默认的查找方法：fs.exits(path) 会默认在用户目录下查找 即 /user/hadoop

### 2.3.2 HDFS文件融合
    整体流程：
        创建文件过滤器；
        HDFS的初始化过程；
        获取指定目录下过滤后的文件信息；
        读取文件信息，并将内容输出到同一个文件内；


    正则表达式：
    表达式 .* 就是单个字符匹配任意次，即贪婪匹配。
```
import java.io.IOException;
import java.io.PrintStream;
import java.net.URI;
 
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.*;
 
/**
 * 过滤掉文件名满足特定条件的文件 
 */
class MyPathFilter implements PathFilter {
     String reg = null; 
     MyPathFilter(String reg) {
          this.reg = reg;
     }
     @Override
     public boolean accept(Path path) {
        if (!(path.toString().matches(reg)))
            return true;
        return false;
    }
}
/***
 * 利用FSDataOutputStream和FSDataInputStream合并HDFS中的文件
 */
public class MergeFile {
    Path inputPath = null; //待合并的文件所在的目录的路径
    Path outputPath = null; //输出文件的路径
    public MergeFile(String input, String output) {
        this.inputPath = new Path(input);
        this.outputPath = new Path(output);
    }
    public void doMerge() throws IOException {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS","hdfs://master:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        FileSystem fsSource = FileSystem.get(URI.create(inputPath.toString()), conf);
        FileSystem fsDst = FileSystem.get(URI.create(outputPath.toString()), conf);
                
        //下面过滤掉输入目录中后缀为.abc的文件
        FileStatus[] sourceStatus = fsSource.listStatus(inputPath,
                new MyPathFilter(".*\\.txt")); 
        FSDataOutputStream fsdos = fsDst.create(outputPath);
        PrintStream ps = new PrintStream(System.out);
        //下面分别读取过滤之后的每个文件的内容，并输出到同一个文件中
        for (FileStatus sta : sourceStatus) {
            //下面打印后缀不为.abc的文件的路径、文件大小
            System.out.print("路径：" + sta.getPath() + "    文件大小：" + sta.getLen()
                    + "   权限：" + sta.getPermission() + "   内容：");
            FSDataInputStream fsdis = fsSource.open(sta.getPath());
            byte[] data = new byte[1024];
            int read = -1;
 
            while ((read = fsdis.read(data)) > 0) {
                ps.write(data, 0, read);
                fsdos.write(data, 0, read);
            }
            fsdis.close();          
        }
        ps.close();
        fsdos.close();
    }
    public static void main(String[] args) throws IOException {
        MergeFile merge = new MergeFile(
                "hdfs://master:9000/user/hadoop/",
                "hdfs://master:9000/user/hadoop/merge.txt");
        merge.doMerge();
    }
}
```

### 2.3.3 课后实验
    留给大家

***
# 3、教学重点
    HDFS的shell命令
    HDFS Java API调用

# 4、教学难点
    代码编写
# 5、教学手段
    多媒体教学；
# 6、教学方法
    演示法；

# 7、课后作业
    书后习题
# 8、参考资料
# 9、课后小结