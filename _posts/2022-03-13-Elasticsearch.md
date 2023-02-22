## 启动单节点

### windeows

cd elasticsearch\bin

.\elasticsearch -d



![image-20220318154058332](F:\202109-pro\jacob-yang.github.io\images\posts\2022-03-13-Elasticsearch\image-20220318154058332.png)



## 2.开启mysql主备模式

### 配置mysql

### windows

1.打开 my.ini

```
#开启主从模式后每个MySql节点的id
server_id = 1
#选择存储binlog日志方式为ROW
binlog_format=ROW
#bin-log的存储位置
log-bin=mysql-bin
```

2.重启mysql

3.验证是否成功

```
show variables like 'log_bin'
```

