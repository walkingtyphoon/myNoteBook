# Flink学习笔记-批处理案例

一个Flink批处理计算的案例，使用最新且简单的方式

## 环境
1. `pom`文件
   ```xml
      <dependencies>
        <dependency>
          <groupId>org.apache.flink</groupId>
          <artifactId>flink-java</artifactId>
          <version>1.16.1</version>
        </dependency>
        <dependency>
          <groupId>org.apache.flink</groupId>
          <artifactId>flink-clients</artifactId>
          <version>1.16.1</version>
        </dependency>
      </dependencies>
   ```

## 代码
1. 主函数
    ```java
    import org.apache.flink.api.java.DataSet;
    import org.apache.flink.api.java.ExecutionEnvironment;
    import org.apache.flink.api.java.operators.DataSource;
    import org.apache.flink.api.java.tuple.Tuple2;
    
    class BatchWordCountJava {
    
        public static void main(String[] args) throws Exception {
            // 设置输入路径
            String inputPath = "input/";
            // 设置输出路径
            String outputPath = "output/";
            // 获取执行环境
            ExecutionEnvironment executionEnvironment = ExecutionEnvironment.getExecutionEnvironment();
            //读取文件中的内容
            DataSource<String> stringDataSource = executionEnvironment.readTextFile(inputPath);
            // 统计字符个数
            DataSet<Tuple2<String, Integer>> sum = stringDataSource.flatMap(new Tokenizer())
            .groupBy(0).sum(1);
            // 将统计后的数据写入到文件中
            sum.writeAsCsv(outputPath, "\n", " ").setParallelism(1);
            // 执行任务
            executionEnvironment.execute("batch word count");
        }
    }
    ```
   
2. 数据传输对象
   ```java
    import org.apache.flink.api.common.functions.FlatMapFunction;
    import org.apache.flink.api.java.tuple.Tuple2;
    import org.apache.flink.util.Collector;
    
   /**.
    * 数据传输对象
    */
    class Tokenizer implements FlatMapFunction<String, Tuple2<String, Integer>> {
   
        public Tokenizer() {
        }
        
       /**
        * The core method of the FlatMapFunction. Takes an element from the input data set and transforms
        * it into zero, one, or more elements.
        *
        * @param value The input value.
        * @param out   The collector for returning result values.
        * @throws Exception This method may throw exceptions. Throwing an exception will cause the
        *                   operation to fail and may trigger recovery.
        */
        @Override
        public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
        String[] split = value.toLowerCase().split("\\W+");
        
            for (String s : split) {
              if (s.length() > 0) {
                out.collect(new Tuple2<>(s, 1));
              }
            }
        }
    }
   ```
## 执行
  我们需要在当前项目的根目录下创建对应的目录(`input`)然后在下边加入我们数据文件,下面我举例我是用的
  ```text
  hello a hello b hello c helo a
  ```
  当然此处我们可以有很多个文件，此处我们指定的`output`为输出的文件名。执行后直接观察就可以找到。
  如果你在使用的过程中遇到问题 ，欢迎给我留言。

