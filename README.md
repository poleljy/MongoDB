# MongoDB

#### RDBMS vs NoSQL

>RDBMS
- 高度组织化结构化数据
- 结构化查询语言（SQL） (SQL)
- 数据和关系都存储在单独的表中。
- 数据操纵语言，数据定义语言
- 严格的一致性
- 基础事务

>NoSQL
- 代表着不仅仅是SQL
- 没有声明性查询语言
- 没有预定义的模式
-键 - 值对存储，列存储，文档存储，图形数据库
- 最终一致性，而非ACID属性
- 非结构化和不可预知的数据
- CAP定理
- 高性能，高可用性和可伸缩性

>  CAP定理（CAP theorem）

在计算机科学中, CAP定理（CAP theorem）, 又被称作 布鲁尔定理（Brewer's theorem）, 它指出对于一个分布式计算系统来说，不可能同时满足以下三点:

    一致性(Consistency) (所有节点在同一时间具有相同的数据)
    可用性(Availability) (保证每个请求不管成功或者失败都有响应)
    分隔容忍(Partition tolerance) (系统中任意信息的丢失或失败不会影响系统的继续运作)

CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。

因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：

    CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
    CP - 满足一致性，分区容忍性的系统，通常性能不是特别高。
    AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。


#### MongoDB 简介
`MongoDB` 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。


#### 默认文件路径
config: "/etc/mongod.conf"
pidFilePath: "/var/run/mongodb/mongod.pid"
dbPath: "/var/lib/mongo"
systemLog: "/var/log/mongodb/mongod.log"


#### 集权搭建

服务器179	                    服务器189                          服务器199                                   端口
mongos 	                        mongos                             	mongos                                      20000
config server 	               config server       	              config server                              21000
shard server1 主节点       shard server1 副节点 	      shard server1 仲裁                      27001
shard server2 仲裁 	      shard server2 主节点 	    shard server2 副节点                  27002
shard server3 副节点 	    shard server3 仲裁 	        shard server3 主节点                  27003


./mongod -f /usr/local/mongodb/conf/config.conf

./mongo --port 21000
config = { _id : "configs", members:[ {_id:0,host :"192.168.1.179:21000"}, {_id:1,host :"192.168.1.189:21000"}, {_id:2,host :"192.168.1.199:21000"} ] }
rs.initiate(config)

---------------------------------------------------------------------------------------------------------------------------------------
./mongod -f /usr/local/mongodb/conf/shard1.conf
./mongod -f /usr/local/mongodb/conf/shard2.conf
./mongod -f /usr/local/mongodb/conf/shard3.conf

./mongo --port 27001
use admin
config = { _id : "shard1", members:[ {_id:0,host:"192.168.1.179:27001",priority:1}, {_id:1,host:"192.168.1.189:27001",priority:2}, {_id:2,host:"192.168.1.199:27001",arbiterOnly:true} ] }
rs.initiate(config);

./mongo --port 27002
use admin
config = { _id : "shard2", members:[ {_id:0,host:"192.168.1.179:27002",arbiterOnly:true}, {_id:1,host:"192.168.1.189:27002",priority:1}, {_id:2,host:"192.168.1.199:27002",priority:2} ] }
rs.initiate(config);

./mongo --port 27003
use admin
config = { _id : "shard3", members:[ {_id:0,host:"192.168.1.189:27003",arbiterOnly:true}, {_id:1,host:"192.168.1.199:27003",priority:1}, {_id:2,host:"192.168.1.179:27003",priority:2} ] }
rs.initiate(config);

./mongos -f /usr/local/mongodb/conf/mongos.conf

./mongo --port 20000
use admin
sh.addShard("shard1/192.168.1.179:27001,192.168.1.189:27001,192.168.1.199:27001")
sh.addShard("shard2/192.168.1.189:27002,192.168.1.179:27002,192.168.1.199:27002")
sh.addShard("shard3/192.168.1.199:27003,192.168.1.189:27003,192.168.1.179:27003")
sh.status()

        {  "_id" : "shard1",  "host" : "shard1/192.168.1.179:27001,192.168.1.189:27001",  "state" : 1 }
        {  "_id" : "shard2",  "host" : "shard2/192.168.1.189:27002,192.168.1.199:27002",  "state" : 1 }
        {  "_id" : "shard3",  "host" : "shard3/192.168.1.179:27003,192.168.1.199:27003",  "state" : 1 }



#### 启动顺序
./mongod -f /usr/local/mongodb/conf/config.conf
./mongod -f /usr/local/mongodb/conf/shard1.conf
./mongod -f /usr/local/mongodb/conf/shard2.conf
./mongod -f /usr/local/mongodb/conf/shard3.conf
./mongos -f /usr/local/mongodb/conf/mongos.conf

#### 参考资料
[mongodb 3.4 集群搭建：分片+副本集](http://www.ityouknow.com/mongodb/2017/08/05/mongodb-cluster-setup.html)

[mongodb 3.4 集群搭建升级版 五台集群](http://www.cnblogs.com/ityouknow/p/7566682.html)

http://blog.csdn.net/wangshubo1989/article/details/75105397?locationNum=4&fps=1
