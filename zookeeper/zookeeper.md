# zookeeper 安装
```sh
https://dlcdn.apache.org/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz

tar -xvzf apache-zookeeper-3.7.0-bin.tar.gz

```

# 单机模式

```sh
cd apache-zookeeper-3.7.0-bin

cat << EOF > zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/itsok/Downloads/apache-zookeeper-3.7.0-bin/data
clientPort=2181
EOF

# 使用帮助命令查看
./zkServer.sh --help
#./zkServer.sh [--config <conf-dir>] {start|start-foreground|stop|version|restart|status|print-cmd}
#如果想要配置启动内存等相关参数可以在配置文件下创建java.env文件进行配置配置,通过阅读启动脚本获得在脚本zkEnv.sh
#exprot JAVA_HOME=/var/appinstall/jdk
#export JVMFLAGS="-Xms128m -Xmx128m $JVMFLAGS"

./bin/zkServer.sh --config ./config start

./zkCli.sh  # 链接本地客户端2181
# ./zkCli.sh -server host:port 链接到远程
[zk: localhost:2181(CONNECTED) 0] help
ZooKeeper -server host:port -client-configuration properties-file cmd args
        addWatch [-m mode] path # optional mode is one of [PERSISTENT, PERSISTENT_RECURSIVE] - default is PERSISTENT_RECURSIVE 
        addauth scheme auth
        close
        config [-c] [-w] [-s]
        connect host:port
        create [-s] [-e] [-c] [-t ttl] path [data] [acl]
        delete [-v version] path
        deleteall path [-b batch size]
        delquota [-n|-b|-N|-B] path
        get [-s] [-w] path
        getAcl [-s] path
        getAllChildrenNumber path
        getEphemerals path
        history
        listquota path
        ls [-s] [-w] [-R] path
        printwatches on|off
        quit
        reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
        redo cmdno
        removewatches path [-c|-d|-a] [-l]
        set [-s] [-v version] path data
        setAcl [-s] [-v version] [-R] path acl
        setquota -n|-b|-N|-B val path
        stat [-w] path
        sync path
        version
        whoami
Command not found: Command not found help
```

# 集群模式

```sh
# 分别创建zk1 zk2 zk3,其中myid文件内容分别为1、2、3 用于标注zk id
mkdir -p ./zk1/conf
mkdir -p ./zk1/data
echo "1" ./zk1/data/myid
mkdir -p ./zk1/log

mkdir -p ./zk2/conf
mkdir -p ./zk2/data
echo "2" ./zk2/data/myid
mkdir -p ./zk2/log

mkdir -p ./zk3/conf
mkdir -p ./zk3/data
echo "2" ./zk3/data/myid
mkdir -p ./zk3/log

#每个conf目录下均包含zoo.cfg文件，配置不同的日志路径和数据存储路径，增加副本配置信息
cat <<EOF >zk1/conf/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/itsok/Downloads/apache-zookeeper-3.7.0-bin/zk1/data
dataLogDir=/home/itsok/Downloads/apache-zookeeper-3.7.0-bin/zk1/log
clientPort=2381 #端口配置差异
server.1=127.0.0.1:2888:3888
server.2=127.0.0.1:2788:3788
server.3=127.0.0.1:2688:3688
EOF

#分别启动
./bin/zkServer.sh --config zk1/conf start
./bin/zkServer.sh --config zk2/conf start
./bin/zkServer.sh --config zk3/conf start

# 链接到zk1 然后创建 key
./bin/zkCli.sh -server 127.0.0.1:2181
create /testreplication zk1
ls /
#链接到zk3查询在zk1创建的 key
./bin/zkCli.sh -server 127.0.0.1:2381
get /testreplication
```
# 问题解决
1. 启动报错
    zkServer.sh --config configdir start-foreground 查看启动日志
2. 创建集群时提示myid file missing
    手动在配置数据目录下创建myid文件内容为数字
    官方描述：The myid file consists of a single line containing only the text of that machine's id. So myid of server 1 would contain the text "1" and nothing else. The id must be unique within the ensemble and should have a value between 1 and 255.
