## RDD

### RDD创建

#### 从集合（内存）中创建

从集合中创建 RDD，Spark 主要提供了两个方法：parallelize 和 makeRDD

```scal
import org.apache.spark.{SparkConf, SparkContext}

object CreateRDDFromCollection {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("create RDD from collection")
    val sparkContext = new SparkContext(sparkConf)
    val rdd1 = sparkContext.parallelize(List(1, 2, 3, 4))
    val rdd2 = sparkContext.makeRDD(List(1, 2, 3, 4, 5))
    rdd1.collect().foreach(println)
    rdd2.collect().foreach(println)
    sparkContext.stop()
  }
}

```

```java

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class CreateRDDFromCollectionJava {
    public static void main(String[] args) {
        SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("create RDD from collection");
        JavaSparkContext sparkContext = new JavaSparkContext(sparkConf);
        JavaRDD<Integer> rdd1 = sparkContext.parallelize(Arrays.asList(1,2,3,4,5),2);
        List<Integer> collect = rdd1.collect();
        for (Integer str : collect){
            System.out.println(str);
        }

        sparkContext.stop();
    }
}

```

从底层代码实现来讲，makeRDD 方法其实就是 parallelize 方法

```scal
def makeRDD[T: ClassTag](
 seq: Seq[T],
 numSlices: Int = defaultParallelism): RDD[T] = withScope {
 parallelize(seq, numSlices)
}
```

#### 从外部存储系统的数据集创建

由外部存储系统的数据集创建RDD包括：本地的文件系统，还有所有Hadoop支持的数据集，比如HDFS、HBase等

```sc
import org.apache.spark.{SparkConf, SparkContext}

object CreateRDDFromFiles {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("create RDD from files")
    val sc = new SparkContext(sparkConf)
    val fileRDD = sc.textFile("datas/wordcount/")
    val wordRdd = fileRDD.flatMap(_.split(" "))
    wordRdd.collect().foreach(println)
    sc.stop()
  }
}
```

```java
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;

import java.util.Arrays;

public class CreateRDDFromFilesJava {
    public static void main(String[] args) {
        SparkConf sparkConf = new SparkConf().setAppName("create RDD from files").setMaster("local[*]");
        JavaSparkContext javaSparkContext = new JavaSparkContext(sparkConf);
        JavaRDD<String> lineJavaRDD = javaSparkContext.textFile("datas/wordcount/");
        JavaRDD<String> wordJavaRDD = lineJavaRDD.flatMap(line -> Arrays.asList(line.split(" ")).iterator());
        wordJavaRDD.collect().forEach(System.out::println);
    }
}
```



### **RDD** 并行度与分区

默认情况下，Spark 可以将一个作业切分多个任务后，发送给 Executor 节点并行计算，而能够并行计算的任务数量我们称之为并行度。这个数量可以在构建 RDD 时指定。记住，这里的并行执行的任务数量，并不是指的切分任务的数量，不要混淆了。

```scala
val sparkConf =
 new SparkConf().setMaster("local[*]").setAppName("spark")
val sparkContext = new SparkContext(sparkConf)
val dataRDD: RDD[Int] =
 sparkContext.makeRDD(
 List(1,2,3,4),
 4)
val fileRDD: RDD[String] =
 sparkContext.textFile(
 "input",
 2)
fileRDD.collect().foreach(println)
sparkContext.stop()
```

读取内存数据时，数据可以按照并行度的设定进行数据的分区操作，数据分区规则的Spark 核心源码如下：

```scala
def positions(length: Long, numSlices: Int): Iterator[(Int, Int)] = {
 (0 until numSlices).iterator.map { i =>
 val start = ((i * length) / numSlices).toInt
 val end = (((i + 1) * length) / numSlices).toInt
 (start, end)
 }
}
```

读取文件数据时，数据是按照 Hadoop 文件读取的规则进行切片分区，而切片规则和数据读取的规则有些差异，具体 Spark 核心源码如下

```java
public InputSplit[] getSplits(JobConf job, int numSplits)
 throws IOException {
 long totalSize = 0; // compute total size
 for (FileStatus file: files) { // check we have valid files
 if (file.isDirectory()) {
 throw new IOException("Not a file: "+ file.getPath());
 }
 totalSize += file.getLen();
 }
 long goalSize = totalSize / (numSplits == 0 ? 1 : numSplits);
 long minSize = Math.max(job.getLong(org.apache.hadoop.mapreduce.lib.input.
 FileInputFormat.SPLIT_MINSIZE, 1), minSplitSize);
 
 ...
 
 for (FileStatus file: files) {
 
 ...
 
 if (isSplitable(fs, path)) {
 long blockSize = file.getBlockSize();
 long splitSize = computeSplitSize(goalSize, minSize, blockSize);
 ...
 }
 protected long computeSplitSize(long goalSize, long minSize,
 long blockSize) {
 return Math.max(minSize, Math.min(goalSize, blockSize));
 }
```



### **RDD** **转换算子**

RDD 根据数据处理方式的不同将算子整体上分为 Value 类型、双 Value 类型和 Key-Value

#### map

函数签名

def map[U: ClassTag](f: T => U): RDD[U]

将处理的数据逐条进行映射转换，这里的转换可以是类型的转换，也可以是值的转换

```scala
import org.apache.spark.{SparkConf, SparkContext}

object Spark01_RDD_Operator_Transform {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("create RDD from files")
    val sc = new SparkContext(sparkConf)
    val rdd = sc.makeRDD(List(1, 2, 3, 4, 5, 6))
    // val rdd2 = rdd.map(num => num * 2 )
    val rdd2 = rdd.map(_ * 2 )
    rdd2.collect().foreach(println)
    sc.stop()
  }

}
```



```java
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;

import java.util.Arrays;

public class Spark01RDDOperatorTransform {
    public static void main(String[] args) {
        SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("Spark01RDDOperatorTransform");
        JavaSparkContext sc = new JavaSparkContext(sparkConf);
        JavaRDD<Integer> parallelize = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5));
        JavaRDD<Integer> mappedRDD = parallelize.map(num -> num * 2);
        mappedRDD.collect().forEach(System.out::println);
        sc.stop();
    }
}

```

#### **mapPartitions**

![image-20241025165628278](images/image-20241025165628278.png)





