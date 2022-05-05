### 课时： 4学时

***
# 0、实验环境
    Win10 + VirtualBox +Ubuntu20.04 + jdk1.8/jdk11 + SSH + Vim + Hadoop3.3.1
# 1、教学目标 
    熟悉Hadoop环境搭建步骤，掌握Hadoop搭建流程，熟练Linux环境下常用命令的使用，能够独立部署Hadoop平台。
# 2、教学内容：
    主要内容为Hadoop平台的搭建，中间会穿插入一些Linux命令使用比如Vim编辑器的使用，以及网络的相关知识，比如ping命令以及SSH。搭建过程大致分为一下五个步骤：
## 2.1 新增Hadoop用户
```
    sudo useradd -m hadoop -s /bin/bash //新增hadoop用户
    sudo passwd hadoop // 为"hadoop"用户设置密码;
    sudo adduser hadoop sudo //为"hadoop"用户添加管理员权限;
    注销当前账户，使用hadoop账户登录

```

## 2.2 虚拟机网络配置
     在VirtualBox环境下，可提供给虚拟机的Network方式主要是NAT、Bridged Adapter、Internal、Host-only Adapter四种方式；

    首先我们需要在Ubuntu安装net工具，执行 sudo apt install net-tools，之后可以使用ifconfig命令，查看当前节点的网络网卡信息；
    如果执行 install net-tools失败，可以先执行sudo apt-get update
`
sudo apt install net-tools
`

    网络配置的目的？
    一是在部署过程中，我们需要虚拟机连接外网；NAT
    二是我们需要集群中的节点通过网络互联；网桥模式

    网络模式选择好后，我们需要配置host
    比如：
    10.130.224.185 master
    10.130.224.184 slave01
    10.130.224.183 slave02
    
    配置过程需要通过vim编辑器对host文件进行修改
`
sudo apt-get install vim
`

    vim的基本使用：
    打开或者创建文件
    vim /.../xxx.txt
    编辑
    英文输入法，i，进入编辑模式；
    保存
    编辑模式 ->esc wq(保存退出)

    测试网络互通：
    ping 命令在master节点ping slave01 ,slave02;

## 2.3 Java环境配置
    两种方式，这里我们使用第一种，通过浏览器下载jdk，解压后，添加到系统path中。
```
cd /usr/lib
sudo mkdir jvm
cd /home/hadoop/Download
sudo tar -xzvf jdkxxxxxxx.tar.gz -C /usr/lib/jvm    //解压jdk到/usr/lib/jvm目录下


vim ~/.bashrc 
export JAVA_HOME=/usr/lib/jvm/jdkxxxxx
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH     //将jdk目录下可执行文件加入到系统PATH中
```
    在线安装DefaultJDK 方式
```
sudo apt-get install default-jre default-jdk// 在/usr/lib/jvm目录下执行命令

export JAVA_HOME=/usr/lib/jvm/default-java //配置环境变量
```
    通过java -version 以及javac 命令测试Java是否安装成功
    
## 2.4 SSH配置 （Secure Shell）
     SSH（Secure Shell）为建立在应用层基础上的安全协议，SSH 是较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题；

    了实现集群中，节点之间的信息交互，我们采用SSH的方式，可以让master节点无密码登录其他slave节点上。这里面关键的一步就是基于RSA的非对称加密，非对称加密，简单来说就是使用公钥加密，私钥解密，私钥保存在本地，公钥可以在网络上流传。下面是SSH配置流程，实现master节点免密登录slave节点；   

### 2.4.1 安装SSH测试本地
```
    sudo apt-get install openssh-server    //安装ssh-server
    ssh localhost //验证是否可以登录本地
```

### 2.4.2 master节点生成公钥
```
cd ~/.ssh          //如果提示找不到文件或路径，先执行一次ssh localhost
 rm ./id_rsa*       //如果有，则删除之前的公钥
ssh-keygen -t rsa  //一路回车生成公钥

cat ./id_rsa.pub >> ./authorized_keys  //将公钥放到autiorized_keys中
```

    将公钥传到slave01、slave02、slave03等节点上，执行如下命令，这里会提示输入slave节点到登录密码：
```
scp ~/.ssh/id_rsa.pub hadoop@slave03:/home/hadoop   //将meter节点到公钥复制到slave03节点中

mkdir ~/.ssh //slave节点创建.ssh文件夹
cat ~/id_rsa.pub>>~/.ssh/authorized_keys//将从master传来的公钥放置在authorized_keys中
rm ~/id_rsa.pub //删除传来的公钥
```
    同样的操作，对所有slave节点进行SSH设置

***
# 3、教学重点
    Hadoop节点网络，Java环境以及SSH的配置。

# 4、教学难点
    SSH的配置过程，理解非对称加密；
# 5、教学手段
    多媒体教学；
# 6、教学方法
    启发式教学，演示法；

# 7、课后作业
###    在Ubuntu环境下，配置成功Java环境，并实现节点之间的互通，在master节点可访问slave节点。
# 8、参考资料
# 9、课后小结