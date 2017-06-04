
# 使用 docker-composer 搭建 一个简单的 MySQL 主从

## 使用说明

### 下载
`git clone https://github.com/cjeruen/docker-compose-mysql-matser-slave.git`

`cd docker-compose-mysql-matser-slave`

### 第一次使用
`docker-compose up -d`

### 停止服务
`docker-compose stop`

### 启动服务
`docker-compose start`

## 配置主从

### 1. 配置 master
```
$ docker exec -it m1 /bin/bash
$ mysql -uroot -p

或者

mysql -h127.0.0.1 -P3310 -uroot -p

```

```
a. 创建一个用户用户复制数据
mysql> GRANT REPLICATION SLAVE ON *.* to 'hui'@'172.18.0.%' identified by 'hui';

b. 查看一些主库的信息 File && Position
mysql> show master status\G
*************************** 1. row ***************************
             File: mysql-bin.000003
         Position: 154
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set:
1 row in set (0.00 sec)

```


### 2. 配置slave


```
连接到 s1 (s2)
$ docker exec -it s1 /bin/bash
$ mysql -uroot -p

或者

mysql -h127.0.0.1 -P3321 -uroot -p

```

```
执行

1. 
change master to 
master_host='m1',
master_port=3306,
master_user='hui',
master_password='hui',
master_log_file='mysql-bin.000003',
master_log_pos=154;


2. 
mysql> start slave;


查看状态

注意下  
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates



mysql> show slave status\G

*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: m1
                  Master_User: hui
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 598
               Relay_Log_File: s2-relay-bin.000002
                Relay_Log_Pos: 764
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 598
              Relay_Log_Space: 968
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 101
                  Master_UUID: 5be8ebcf-48b8-11e7-bdb6-0242ac120002
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)

```


## 测试

```
# m1 (主库中)
mysql> create database mtest;

# s1(s2)
mysql> show databases;

其他自行测试
```



