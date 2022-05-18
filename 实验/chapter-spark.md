# Spark实验
## spark local安装
    https://mirrors.cloud.tencent.com/apache/spark/
    选择2.4.8-bin-without-hadoop版本

    sudo tar -zxf ~/下载/spark-2.4.0-bin-without-hadoop.tgz -C /usr/local/
    cd /usr/local
    sudo mv ./spark-2.4.0-bin-without-hadoop/ ./spark
    sudo chown -R hadoop:hadoop ./spark          # 此处的 hadoop 为你的用户名 

    //新建spark-env.sh文件，并进行配置
    cd /usr/local/spark
    cp ./conf/spark-env.sh.template ./conf/spark-env.sh  

    //在新建的spark-env.sh文件中第一行添加如下：
    export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath)

    //通过运行Spark自带的示例，验证Spark是否安装成功。
    cd /usr/local/spark
    bin/run-example SparkPi

    //通过grep过滤信息，查看示例结果
    bin/run-example SparkPi 2>&1 | grep "Pi is"

## Spark Shell 练习
    // 进入spark shell
    cd /usr/local/spark
    bin/spark-shell


    //加载text文件
        val textFile = sc.textFile("file:///usr/local/spark/README.md")
        //获取RDD文件textFile的第一行内容
        textFile.first()
        //获取RDD文件textFile所有项的计数
        textFile.count()
        //抽取含有“Spark”的行，返回一个新的RDD
        val lineWithSpark = textFile.filter(line => line.contains("Spark"))
        //统计新的RDD的行数
        lineWithSpark.count()

        //找出文本中每行的最多单词数
        textFile.map(line => line.split(" ").size).reduce((a, b) => if (a > b) a else b)

        :quit //退出spark shell

## Scala语言编写 Spark应用程序
### 使用sbt对Scala独立应用程序进行编译打包
    到“http://www.scala-sbt.org”下载sbt安装文件

    sudo mkdir /usr/local/sbt             　　　# 创建安装目录
    cd ~/Downloads 
    sudo tar -zxvf ./sbt-1.3.8.tgz -C /usr/local 
    cd /usr/local/sbt
    sudo chown -R hadoop /usr/local/sbt  　　 # 此处的hadoop为系统当前用户名
    cp ./bin/sbt-launch.jar ./  #把bin目录下的sbt-launch.jar复制到sbt安装目录下

    vim /usr/local/sbt/sbt
    //sbt 可执行文件中添加如下信息
    SBT_OPTS="-Xms512M -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=256M"
    java $SBT_OPTS -jar `/usr/local/sbt`/sbt-launch.jar "$@"

    chmod u+x /usr/local/sbt/sbt //保存后，还需要为该Shell脚本文件增加可执行权限：

    cd /usr/local/sbt
    ./sbt sbtVersion
    Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=256M; support was removed in 8.0
    [warn] No sbt.version set in project/build.properties, base directory: /usr/local/sbt
    [info] Set current project to sbt (in build file:/usr/local/sbt/)
    [info] 1.3.8
### Scala应用程序代码
    cd ~           # 进入用户主文件夹
    mkdir ./sparkapp        # 创建应用程序根目录
    mkdir -p ./sparkapp/src/main/scala     # 创建所需的文件夹结构

    在 ./sparkapp/src/main/scala 下建立一个名为 SimpleApp.scala 的文件（vim ./sparkapp/src/main/scala/SimpleApp.scala），添加代码如下：

```
    /* SimpleApp.scala */
    import org.apache.spark.SparkContext
    import org.apache.spark.SparkContext._
    import org.apache.spark.SparkConf
    
    object SimpleApp {
            def main(args: Array[String]) {
                val logFile = "file:///usr/local/spark/README.md" // Should be some file on your system
                val conf = new SparkConf().setAppName("Simple Application")
                val sc = new SparkContext(conf)
                val logData = sc.textFile(logFile, 2).cache()
                val numAs = logData.filter(line => line.contains("a")).count()
                val numBs = logData.filter(line => line.contains("b")).count()
                println("Lines with a: %s, Lines with b: %s".format(numAs, numBs))
            }
        }

```
        该程序计算 /usr/local/spark/README 文件中包含 “a” 的行数 和包含 “b” 的行数。代码第8行的 /usr/local/spark 为 Spark 的安装目录，如果不是该目录请自行修改。不同于 Spark shell，独立应用程序需要通过 val sc = new SparkContext(conf) 初始化 SparkContext，SparkContext 的参数 SparkConf 包含了应用程序的信息。

        我们需要通过 sbt 进行编译打包。 在~/sparkapp这个目录中新建文件simple.sbt，命令如下：
        cd ~/sparkapp
        vim simple.sbt

        //在simple.sbt中添加如下内容，声明该独立应用程序的信息以及与 Spark 的依赖关系：
        name := "Simple Project"
        version := "1.0"
        scalaVersion := "2.11.12"
        libraryDependencies += "org.apache.spark" %% "spark-core" % "2.4.0"
        //scalaVersion用来指定scala的版本，sparkcore用来指定spark的版本

### 使用 sbt 打包 Scala 程序
    //查看程序文件结构
    cd ~/sparkapp
    find .

    /usr/local/sbt/sbt package

    最后，我们就可以将生成的 jar 包通过 spark-submit 提交到 Spark 中运行了，命令如下：

    /usr/local/spark/bin/spark-submit --class "SimpleApp" ~/sparkapp/target/scala-2.11/simple-project_2.11-1.0.jar
    # 上面命令执行后会输出太多信息，可以不使用上面命令，而使用下面命令查看想要的结果
    /usr/local/spark/bin/spark-submit --class "SimpleApp" ~/sparkapp/target/scala-2.11/simple-project_2.11-1.0.jar 2>&1 | grep "Lines with a:"