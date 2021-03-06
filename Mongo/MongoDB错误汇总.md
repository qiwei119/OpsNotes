---
title: "MongoDB错误汇总"
date: "2020-10-13"
categories:
    - "技术"
tags:
    - "MongoDB"
toc: false
indent: false
original: true
draft: true
---

## 复制集

``` zsh
rs0:PRIMARY> rs.status()
{
    "members" : [
        {
            "_id" : 0,
            "name" : "192.168.100.226:27017",
            "ip" : "192.168.100.226",
            "health" : 1,
            "state" : 1,
            "stateStr" : "PRIMARY",
            "uptime" : 55198,
            "optime" : {
                "ts" : Timestamp(1602552745, 1),
                "t" : NumberLong(30)
            },
            "optimeDate" : ISODate("2020-10-13T01:32:25Z"),
            "syncingTo" : "",
            "syncSourceHost" : "",
            "syncSourceId" : -1,
            "infoMessage" : "",
            "electionTime" : Timestamp(1602498153, 1),
            "electionDate" : ISODate("2020-10-12T10:22:33Z"),
            "configVersion" : 1,
            "self" : true,
            "lastHeartbeatMessage" : ""
        },
        {
            "_id" : 1,
            "name" : "192.168.100.227:27017",
            "ip" : "192.168.100.227",
            "health" : 0,
            "state" : 8,
            "stateStr" : "(not reachable/healthy)",
            "uptime" : 0,
            "optime" : {
                "ts" : Timestamp(0, 0),
                "t" : NumberLong(-1)
            },
            "optimeDurable" : {
                "ts" : Timestamp(0, 0),
                "t" : NumberLong(-1)
            },
            "optimeDate" : ISODate("1970-01-01T00:00:00Z"),
            "optimeDurableDate" : ISODate("1970-01-01T00:00:00Z"),
            "lastHeartbeat" : ISODate("2020-10-13T01:32:29.748Z"),
            "lastHeartbeatRecv" : ISODate("2020-10-12T10:48:20.816Z"),
            "pingMs" : NumberLong(167),
            "lastHeartbeatMessage" : "Error connecting to 192.168.100.227:27017 :: caused by :: Connection refused",
            "syncingTo" : "",
            "syncSourceHost" : "",
            "syncSourceId" : -1,
            "infoMessage" : "",
            "configVersion" : -1
        },
        {
            "_id" : 2,
            "name" : "192.168.100.228:27017",
            "ip" : "192.168.100.228",
            "health" : 1,
            "state" : 7,
            "stateStr" : "ARBITER",
            "uptime" : 55196,
            "lastHeartbeat" : ISODate("2020-10-13T01:32:28.094Z"),
            "lastHeartbeatRecv" : ISODate("2020-10-13T01:32:28.572Z"),
            "pingMs" : NumberLong(0),
            "lastHeartbeatMessage" : "",
            "syncingTo" : "",
            "syncSourceHost" : "",
            "syncSourceId" : -1,
            "infoMessage" : "",
            "configVersion" : 1
        }
    ],
    "ok" : 1,
    "$clusterTime" : {
        "clusterTime" : Timestamp(1602552745, 1),
        "signature" : {
            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
            "keyId" : NumberLong(0)
        }
    },
    "operationTime" : Timestamp(1602552745, 1)
}
rs0:PRIMARY>
```

``` log
2020-10-13T13:59:37.238+0800 I  CONNPOOL [Replication] Connecting to 192.168.100.227:27017
2020-10-13T13:59:38.086+0800 I  REPL_HB  [replexec-17] Heartbeat to 192.168.100.227:27017 failed after 2 retries, response status: HostUnreachable: Error connecting to 192.168.100.227:27017 :: caused by :: Connection refused
```

### 解决问题

``` zsh
# 226
rs.remove("192.168.100.227:27017")

# 227
cd /ahdata/mongo
mv rs0{,.bak}
mkdir rs0
>rs0.log
tail -f rs0.log

cd /opt/mongodb-linux-x86_64-rhel70-4.2.2/
bin/mongod -f conf/rs0.yaml

# 226
rs.add("192.168.100.227:27017")


# 运行一会儿后
2020-10-13T16:22:31.238+0800 I  CONNPOOL [Replication] Connecting to 192.168.100.227:27017
2020-10-13T16:22:31.882+0800 I  REPL     [replexec-24] Member 192.168.100.227:27017 is now in state STARTUP
2020-10-13T16:22:31.882+0800 I  NETWORK  [listener] connection accepted from 192.168.100.227:48762 #591 (7 connections now open)
2020-10-13T16:22:31.882+0800 I  NETWORK  [conn591] received client metadata from 192.168.100.227:48762 conn591: { driver: { name: "NetworkInterfaceTL", version: "4.2.2" }, os: { type: "Linux", name: "CentOS Linux release 7.6.1810 (Core) ", architecture: "x86_64", version: "Kernel 3.10.0-957.el7.x86_64" } }
2020-10-13T16:22:31.884+0800 I  NETWORK  [listener] connection accepted from 192.168.100.227:48764 #592 (8 connections now open)
2020-10-13T16:22:31.885+0800 I  NETWORK  [conn592] end connection 192.168.100.227:48764 (7 connections now open)
2020-10-13T16:22:32.425+0800 W  STORAGE  [FlowControlRefresher] Flow control is engaged and the sustainer point is not moving. Please check the health of all secondaries.
2020-10-13T16:22:35.882+0800 I  REPL     [replexec-23] Member 192.168.100.227:27017 is now in state STARTUP2
2020-10-13T16:22:42.425+0800 W  STORAGE  [FlowControlRefresher] Flow control is engaged and the sustainer point is not moving. Please check the health of all secondaries.

# 查看复制集状态
rs.status()
{
    "members" : [
        {
            "_id" : 0,
            "name" : "192.168.100.226:27017",
            "ip" : "192.168.100.226",
            "health" : 1,
            "state" : 1,
            "stateStr" : "PRIMARY",
            "uptime" : 72256,
            "optime" : {
                "ts" : Timestamp(1602569803, 1),
                "t" : NumberLong(30)
            },
            "optimeDate" : ISODate("2020-10-13T06:16:43Z"),
            "syncingTo" : "",
            "syncSourceHost" : "",
            "syncSourceId" : -1,
            "infoMessage" : "",
            "electionTime" : Timestamp(1602498153, 1),
            "electionDate" : ISODate("2020-10-12T10:22:33Z"),
            "configVersion" : 3,
            "self" : true,
            "lastHeartbeatMessage" : ""
        },
        {
            "_id" : 2,
            "name" : "192.168.100.228:27017",
            "ip" : "192.168.100.228",
            "health" : 1,
            "state" : 7,
            "stateStr" : "ARBITER",
            "uptime" : 72254,
            "lastHeartbeat" : ISODate("2020-10-13T06:16:47.115Z"),
            "lastHeartbeatRecv" : ISODate("2020-10-13T06:16:47.118Z"),
            "pingMs" : NumberLong(0),
            "lastHeartbeatMessage" : "",
            "syncingTo" : "",
            "syncSourceHost" : "",
            "syncSourceId" : -1,
            "infoMessage" : "",
            "configVersion" : 3
        },
        {
            "_id" : 3,
            "name" : "192.168.100.227:27017",
            "ip" : "192.168.100.227",
            "health" : 1,
            "state" : 5,
            "stateStr" : "STARTUP2",
            "uptime" : 4,
            "optime" : {
                "ts" : Timestamp(0, 0),
                "t" : NumberLong(-1)
            },
            "optimeDurable" : {
                "ts" : Timestamp(0, 0),
                "t" : NumberLong(-1)
            },
            "optimeDate" : ISODate("1970-01-01T00:00:00Z"),
            "optimeDurableDate" : ISODate("1970-01-01T00:00:00Z"),
            "lastHeartbeat" : ISODate("2020-10-13T06:16:47.118Z"),
            "lastHeartbeatRecv" : ISODate("2020-10-13T06:16:47.677Z"),
            "pingMs" : NumberLong(320),
            "lastHeartbeatMessage" : "",
            "syncingTo" : "",
            "syncSourceHost" : "",
            "syncSourceId" : -1,
            "infoMessage" : "",
            "configVersion" : 3
        }
    ],
    "ok" : 1,
    "$clusterTime" : {
        "clusterTime" : Timestamp(1602569803, 1),
        "signature" : {
            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
            "keyId" : NumberLong(0)
        }
    },
    "operationTime" : Timestamp(1602569803, 1)
}
```

``` log
# 226 又连接不上227了
2020-10-13T16:25:52.425+0800 W  STORAGE  [FlowControlRefresher] Flow control is engaged and the sustainer point is not moving. Please check the health of all secondaries.
2020-10-13T16:25:53.571+0800 I  NETWORK  [conn591] end connection 192.168.100.227:48762 (9 connections now open)
2020-10-13T16:25:53.571+0800 I  NETWORK  [conn594] end connection 192.168.100.227:48932 (8 connections now open)
2020-10-13T16:25:53.571+0800 I  NETWORK  [conn620] Error sending response to client: HostUnreachable: Connection reset by peer. Ending connection from 192.168.100.227:50728 (connection id: 620)
2020-10-13T16:25:53.571+0800 I  NETWORK  [conn620] end connection 192.168.100.227:50728 (7 connections now open)
2020-10-13T16:25:53.931+0800 I  CONNPOOL [replexec-23] dropping unhealthy pooled connection to 192.168.100.227:27017
2020-10-13T16:25:53.931+0800 I  CONNPOOL [Replication] Connecting to 192.168.100.227:27017
2020-10-13T16:25:53.933+0800 I  REPL_HB  [replexec-23] Heartbeat to 192.168.100.227:27017 failed after 2 retries, response status: HostUnreachable: Error connecting to 192.168.100.227:27017 :: caused by :: Connection refused
```

rs0:PRIMARY> db.printReplicationInfo()
configured oplog size:   2048MB
log length start to end: 81861secs (22.74hrs)
oplog first event time:  Mon Oct 12 2020 18:12:35 GMT+0800 (CST)
oplog last event time:   Tue Oct 13 2020 16:56:56 GMT+0800 (CST)
now:                     Tue Oct 13 2020 16:57:04 GMT+0800 (CST)

rs0:PRIMARY> rs.printSlaveReplicationInfo()
source: 192.168.100.227:27017
	syncedTo: Thu Jan 01 1970 08:00:00 GMT+0800 (CST)
	1602579446 secs (445160.96 hrs) behind the primary

mongodb statestr状态大全及功能解释  
STARTUP：刚加入到复制集中，配置还未加载  
STARTUP2：配置已加载完，初始化状态  
RECOVERING：正在恢复，不适用读  
ARBITER: 仲裁者  
DOWN：节点不可到达  
UNKNOWN：未获取其他节点状态而不知是什么状态，一般发生在只有两个成员的架构，脑裂  
REMOVED：移除复制集  
ROLLBACK：数据回滚，在回滚结束时，转移到RECOVERING或SECONDARY状态  
FATAL：出错。查看日志grep “replSet FATAL”找出错原因，重新做同步  
PRIMARY：主节点  
SECONDARY：备份节点  

> 参考文档：
> 1、[mongodb statestr状态大全及功能解释](http://www.75271.com/20225.html)
>
