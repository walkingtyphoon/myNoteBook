# Flink学习笔记-Flink本地模式部署

 Flink本地模式部署，解压简单配置就可以运行

## 环境
1. 环境通常使用`Linux`或者`Unix`

2. `JDK 1.8`

3. 下载`Flink`

   1. [官网下载链接](https://flink.apache.org/zh/downloads.html)

   2. 进入后直接选择最新版[Apache Flink 1.16.1 for Scala 2.12](https://dlcdn.apache.org/flink/flink-1.16.1/flink-1.16.1-bin-scala_2.12.tgz)，你可以直接点击蓝色的链接下载

   3. 下载后选择一个适当的地方，推荐英文目录

   4. 直接解压文件

      ```shell
      tar -zxvf flink-1.16.1-bin-scala_2.12.tgz
      ```


## 运行

1. 直接进入`Flink`目录的`bin`目录下，执行启动命令

   ```shell
   ./start-cluster.sh 
   ```

2. 启动后，你将看到如下节点

   ```shell
   14865 Jps
   14149 TaskManagerRunner
   13898 StandaloneSessionClusterEntrypoint
   ```

3. 关闭则直接使用

   ```shell
   ./stop-cluster.sh
   ```

   

## 访问`Flink`的`Web`页面

```http
http://你的主机名:8081
```

如果你的访问没有成功，请检查你的防火墙配置。

如果还有其他问题或者这个帖子已经过时，请您留言，谢谢



