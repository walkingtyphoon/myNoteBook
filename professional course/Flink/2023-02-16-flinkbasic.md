# Flink学习笔记-流式处理案例

一个Flink流式计算的案例，使用最新且简单的方式


## 环境

1. `pom文件`:

   ```xml
       <dependencies>
           <dependency>
               <groupId>org.apache.flink</groupId>
               <artifactId>flink-java</artifactId>
               <version>1.16.1</version>
           </dependency>
           <dependency>
               <groupId>org.apache.flink</groupId>
               <artifactId>flink-streaming-java</artifactId>
               <version>1.16.1</version>
           </dependency>
           <dependency>
               <groupId>org.apache.flink</groupId>
               <artifactId>flink-clients</artifactId>
               <version>1.16.1</version>
           </dependency>
       </dependencies>
   ```

   导入后会提示Flink的依赖项存在漏洞，这是因为`commons-collections:commons-collections:3.2.2`这个包存在漏洞，不过我们做测试，这并不影响我们学习和正常运行。

## 代码

1. 数据传输对象

   ```java
   package top.istyphoon.streaming;
   
   /**.
    * 数据传输对象
    */
   public class WordWithCount {
   
     public String word;
   
     public long count;
   
     public String getWord() {
       return word;
     }
   
     public WordWithCount() {
     }
   
     public WordWithCount(String str, long l) {
       this.word = str;
       this.count = l;
     }
   
     @Override
     public String toString() {
       return String.format("WordWithCount{word='%s', count=%d}", word, count);
     }
   }
   ```

2. 主程序类

   ```java
   import org.apache.flink.api.common.functions.FlatMapFunction;
   import org.apache.flink.api.common.typeinfo.TypeInformation;
   import org.apache.flink.api.java.utils.ParameterTool;
   import org.apache.flink.streaming.api.datastream.DataStream;
   import org.apache.flink.streaming.api.datastream.DataStreamSource;
   import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
   import org.apache.flink.streaming.api.windowing.assigners.TumblingProcessingTimeWindows;
   import org.apache.flink.streaming.api.windowing.time.Time;
   
   /**
    * . 从网络端口获取数据并进行统计
    */
   public class SocketWindowWordCountJava {
   
     /**
      * . 程序的主函数
      *
      * @param args 主函数参数
      * @throws Exception 可能存在的异常
      */
     public static void main(String[] args) throws Exception {
       // 声明我们使用的端口
       int port;
   
       try {
         // 设置获取配置的参数位置，从主函数参数中
         ParameterTool parameterTool = ParameterTool.fromArgs(args);
         // 获取并初始化我们需要使用的端口
         port = parameterTool.getInt("port");
       } catch (Exception e) {
         // 没有获取到端口打印提示信息
         System.out.println("no port set. use default port 9000-java");
         // 设置端口为9000
         port = 9000;
       }
       // 获取flink运行环境
       StreamExecutionEnvironment executionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment();
       // 设置Flink节点的名称，因为我们并未使用集群，此处直接使用本季
       String hostname = "localhost";
       // 设置定界符
       String delimiter = "\n";
       // 链接socket获取输入的数据
       DataStreamSource<String> stringDataStreamSource = executionEnvironment.socketTextStream(
           hostname, port, delimiter);
       // 遍历数据并将起逐一累加
       DataStream<WordWithCount> count = stringDataStreamSource.flatMap(
               (FlatMapFunction<String, WordWithCount>) (value, out) -> {
                 String[] s = value.split("\\s");
                 for (String str : s) {
                   out.collect(new WordWithCount(str, 1L));
                 }
                 // 设置数据的类型
               }).returns(TypeInformation.of(WordWithCount.class))
           // 设置获取数据的方式
           .keyBy(WordWithCount::getWord)
           // 指定时间窗口大小为两秒
           .window(TumblingProcessingTimeWindows.of(Time.seconds(2)))
           // 统计求和总次数
           .sum("count");
       // 将数据打印到控制台并且设置并行度
       count.print().setParallelism(1);
       // 执行程序
       executionEnvironment.execute("Socket window count");
     }
   }
   
   ```

## 执行

1. 此时我们需要在终端上监控`9000`这个端口

   ```shell
   nc -l 9000
   ```

2. 然后继续在终端输入单词或者若干个字符

   ```shell
   a
   b
   a
   c
   d
   d
   d
   ```

3. 然后运行我们的程序，你就可以看到

   ```shell
   WordWithCount{word='b', count=1}
   WordWithCount{word='a', count=2}
   WordWithCount{word='d', count=3}
   WordWithCount{word='c', count=1}
   ```

此时我们的流式计算体验程序就结束了，当然此处也会存在些问题，比如`keyBy(String string)`的用法被弃用，`timeWindow(Time.secondes(Number number),Time.secondes(Number number))`。此处我们更新了较新的方法，只是希望大家可以体验一下流式计算的魅力，如果使用过程中出现了什么问题，欢迎交流