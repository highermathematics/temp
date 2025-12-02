# Spark实验演示脚本（含讲解+指令行）
## 一、实验平台版本
- 操作系统：Ubuntu 24.04.2 LTS
- Hadoop版本：3.3.5
- JDK版本：1.8.0_371
- Spark版本：3.4.0

接下来我会按步骤演示每个环节的操作及结果验证

## 二、第一部分：Spark安装及环境配置
### 讲解要点
1. 采用aria2c多线程下载Spark安装包，提升下载速度；
2. 解压安装后配置环境变量，关联Hadoop类路径；
3. 通过运行官方示例程序SparkPi验证安装成功（输出π近似值）。

### 指令行操作（按顺序执行）
```bash
# 1. 多线程下载Spark安装包（-x 16设置最大连接数，-s 16设置最大分片数）
aria2c -x 16 -s 16 https://archive.apache.org/dist/spark/spark-3.4.0/spark-3.4.0-bin-hadoop3.tgz

# 2. 解压到/usr/local目录（假设下载到桌面）
sudo tar -zxvf ~/桌面/spark-3.4.0-bin-hadoop3.tgz -C /usr/local/

# 3. 重命名为spark（简化路径）
sudo mv /usr/local/spark-3.4.0-bin-hadoop3 /usr/local/spark

# 4. 配置Spark环境变量（关联Hadoop类路径）
echo "export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath)" >> /usr/local/spark/conf/spark-env.sh

# 5. 验证安装（运行SparkPi示例）
cd /usr/local/spark
bin/run-example SparkPi
```

### 预期结果
输出π的近似值（如：3.1394956974784876），同时日志显示SparkContext正常启动并停止，无报错。

## 三、第二部分：熟悉常用SparkShell命令
### 讲解要点
1. SparkShell是交互式命令行工具，支持Scala语法，自动创建SparkContext（sc）和SparkSession（spark）；
2. 演示RDD的核心操作：加载文件、计数、取首行、过滤、转换计算等；
3. 所有操作基于弹性分布式数据集（RDD），体现Spark的惰性计算特性。

### 指令行操作（按顺序执行）
```bash
# 1. 启动SparkShell（local[*]表示使用本地所有CPU核心）
spark-shell
# 启动SparkShell，设置日志级别为WARN（减少冗余日志，只看关键信息），使用本地所有CPU核心
spark-shell --master local[*] --conf spark.driver.extraJavaOptions="-Dlog4j.configuration=file:///usr/local/spark/conf/log4j2.properties"
```

- 启动成功后，控制台会显示：`Spark context available as 'sc'` 和 `Spark session available as 'spark'`，说明环境初始化完成。

进入Scala交互环境后，执行以下命令：
```scala
# 1. 加载Spark自带的README.md文件（本地路径，前缀file://不可省略），创建文本RDD
val textFile = sc.textFile("file:///usr/local/spark/README.md")
# 输出提示：textFile: org.apache.spark.rdd.RDD[String] = file:///usr/local/spark/README.md MapPartitionsRDD[1] at textFile at <console>:23

# 2. 查看RDD的分区数（默认与CPU核心数一致，本地模式下为local[*]的核心数）
textFile.getNumPartitions
# 预期输出：Int = 2

# 3. 统计RDD的行数（count操作触发计算）
textFile.count()  # 预期输出：125

# 4. 获取RDD的第一行内容
textFile.first()  # 预期输出：# Apache Spark

# 5. 过滤出包含"Spark"的行，返回新RDD并统计行数
val sparkLines = textFile.filter(line => line.contains("Spark"))
sparkLines.count()  # 预期输出：包含"Spark"的行数（约30+）

# 6. 计算每行的单词数，找出最大值
textFile.map(line => line.split(" ").length).reduce((a, b) => math.max(a, b))  # 预期输出：每行最大单词数（约20+）

# 7. 退出SparkShell
:quit
```

### 行动操作：触发计算，获取结果（体现惰性计算）
行动操作会触发之前所有转换操作的执行，返回具体结果或写入外部存储。
```scala
# 1 统计RDD总行数（基础行动操作，触发计算）
val totalLines = textFile.count()
# 预期输出：totalLines: Long = 125（与实验文档一致，确认文件加载正确）

# 2 获取RDD的前5行内容（抽样查看，避免打印所有数据）
textFile.take(5)
# 预期输出：
# Array(
#   "# Apache Spark", 
#   "", 
#   "Spark is a unified analytics engine for large-scale data processing. It provides", 
#   "high-level APIs in Scala, Java, Python, and R, and an optimized engine that", 
#   "supports general computation graphs for data analysis."
# )

# 3 检查是否包含指定内容（判断RDD中是否有含"Python"的行）
val hasPython = textFile.filter(line => line.contains("Python")).isEmpty
# 预期输出：hasPython: Boolean = false（说明有含"Python"的行）
textFile.filter(line => line.contains("Python")).take(1)  # 查看其中一行
# 预期输出：Array("high-level APIs in Scala, Java, Python, and R, and an optimized engine that")
```


### 转换操作：构建计算逻辑（不触发执行）
转换操作会生成新的RDD，记录与父RDD的依赖关系，不立即执行计算，仅在行动操作触发时才运行。
```scala
# 1 过滤非空行（去除README中的空行，生成新RDD）
val nonEmptyLines = textFile.filter(line => !line.trim.isEmpty)
# 输出提示：nonEmptyLines: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[2] at filter at <console>:25
# 此时未触发计算，需后续行动操作触发

# 2 每行单词拆分（flatMap与map的区别：flatMap会将数组“扁平化”，适合单词拆分）
// map：每行返回一个单词数组，RDD类型为RDD[Array[String]]
val wordArrays = nonEmptyLines.map(line => line.split(" "))
// flatMap：每行的单词数组拆分为单个单词，RDD类型为RDD[String]（最终所有单词在一个RDD中）
val words = nonEmptyLines.flatMap(line => line.split(" "))
# 输出提示：words: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[3] at flatMap at <console>:25

# 3 单词去重（统计不重复的单词数量）
val distinctWords = words.distinct()
# 输出提示：distinctWords: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[4] at distinct at <console>:25
# 打印去重后单词
distinctWords.foreach(println)
```


### 综合操作：转换+行动结合（实战场景）
#### 场景1：单词频次统计（经典案例）
```scala
# 1 统计每个单词出现的次数（步骤：单词→(单词,1)→按单词聚合求和）
  // 统一转为小写，避免"Spark"和"spark"被视为不同单词
  // 按单词（key）聚合，value累加（统计次数）
val wordCounts = words
  .map(word => (word.toLowerCase, 1))
  .reduceByKey((a, b) => a + b)

# 2 查看出现次数前10的单词（按次数降序排序）
val top10Words = wordCounts.sortBy(_._2, ascending = false).take(10)
top10Words.foreach(println)  # 循环打印结果
# 预期输出（示例，具体次数可能略有差异）：
# (the,22)
# (a,18)
# (spark,15)
# (for,12)
# (and,10)
# (to,9)
# (in,8)
# (is,7)
# (data,6)
# (processing,5)
```

## 四、第三部分：基于Spark API的独立应用开发
### 讲解要点
1. 分别使用Java和Scala语言开发Spark独立应用，核心逻辑为统计文件中含"a"和"b"的行数；
2. 使用Maven进行项目构建和打包，解决Spark依赖问题；
3. 通过spark-submit脚本提交应用，验证运行结果。

### 模块1：Java独立应用（SparkApp2）
#### 步骤1：创建项目结构及代码
```bash
# 1. 创建项目目录结构
mkdir -p ./sparkapp2/src/main/java

# 2. 编写Java应用代码（SimpleApp.java）
vim ./sparkapp2/src/main/java/SimpleApp.java
```
将以下代码复制到文件中：
```java
/*** SimpleApp.java ***/
import org.apache.spark.api.java.*;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.SparkConf;
 
public class SimpleApp {
    public static void main(String[] args) {
        String logFile = "file:///usr/local/spark/README.md"; // Should be some file on your system
        SparkConf conf=new SparkConf().setMaster("local").setAppName("SimpleApp");
        JavaSparkContext sc=new JavaSparkContext(conf);
        JavaRDD<String> logData = sc.textFile(logFile).cache(); 
        long numAs = logData.filter(new Function<String, Boolean>() {
            public Boolean call(String s) { return s.contains("a"); }
        }).count(); 
        long numBs = logData.filter(new Function<String, Boolean>() {
            public Boolean call(String s) { return s.contains("b"); }
        }).count(); 
        System.out.println("Lines with a: " + numAs + ", lines with b: " + numBs);
    }
}
```

#### 步骤2：编写Maven依赖文件（pom.xml）
```bash
# 进入项目根目录
cd ~/sparkapp2
# 创建pom.xml文件
vim pom.xml
```
复制以下依赖内容：
```xml
<project>
    <groupId>cn.edu.xmu</groupId>
    <artifactId>simple-project</artifactId>
    <modelVersion>4.0.0</modelVersion>
    <name>Simple Project</name>
    <packaging>jar</packaging>
    <version>1.0</version>
    <repositories>
        <repository>
            <id>jboss</id>
            <name>JBoss Repository</name>
            <url>http://repository.jboss.com/maven2/</url>
        </repository>
    </repositories>
    <dependencies>
        <dependency> <!-- Spark dependency -->
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>3.4.0</version>
        </dependency>
    </dependencies>
</project>
```

#### 步骤3：打包并执行应用
```bash
# 1. 打包（需提前安装Maven，验证：mvn -v）
cd ~/sparkapp2
find .
cd ~/sparkapp2
/usr/local/maven/bin/mvn package

# 2. 使用spark-submit提交应用
/usr/local/spark/bin/spark-submit --class "SimpleApp" ~/sparkapp2/target/simple-project-1.0.jar 2>&1 | grep "Lines with a"
```

#### 预期结果
输出：`Lines with a: 72, lines with b: 39`

### 模块2：Scala独立应用（SparkApp3）
#### 步骤1：创建项目结构及代码
```bash
# 1. 创建项目目录结构
mkdir -p ./sparkapp3/src/main/scala

# 2. 编写Scala应用代码（SimpleApp.scala）
vim ./sparkapp3/src/main/scala/SimpleApp.scala
```
将以下代码复制到文件中：
```scala
/* SimpleApp.scala */
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf
 
object SimpleApp {
        def main(args: Array[String]) {
            val logFile = "file:///usr/local/spark/README.md" // Should be some file on your system
            val conf = new SparkConf().setAppName("Simple Application")
            val sc = new SparkContext(conf)
            val logData = sc.textFile(logFile, 2).cache()
            val numAs = logData.filter(line => line.contains("a")).count()
            val numBs = logData.filter(line => line.contains("b")).count()
            println("Lines with a: %s, Lines with b: %s".format(numAs, numBs))
        }
    }
```

#### 步骤2：编写Maven依赖文件（pom.xml）
```bash
# 进入项目根目录
cd ~/sparkapp3
# 创建pom.xml文件
vim pom.xml
```
复制以下依赖内容：
```xml
<project>
    <groupId>cn.edu.xmu</groupId>
    <artifactId>simple-project</artifactId>
    <modelVersion>4.0.0</modelVersion>
    <name>Simple Project</name>
    <packaging>jar</packaging>
    <version>1.0</version>
    <repositories>
        <repository>
            <id>jboss</id>
            <name>JBoss Repository</name>
            <url>http://repository.jboss.com/maven2/</url>
        </repository>
    </repositories>
    <dependencies>
        <dependency> <!-- Spark dependency -->
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>3.4.0</version>
        </dependency>
    </dependencies>

  <build>
    <sourceDirectory>src/main/scala</sourceDirectory>
    <plugins>
      <plugin>
        <groupId>org.scala-tools</groupId>
        <artifactId>maven-scala-plugin</artifactId>
        <executions>
          <execution>
            <goals>
              <goal>compile</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <scalaVersion>2.12.17</scalaVersion>
          <args>
            <arg>-target:jvm-1.8</arg>
          </args>
        </configuration>
    </plugin>
    </plugins>
</build>
</project>
```

#### 步骤3：打包并执行应用
```bash
# 1. 打包
cd  ~/sparkapp3
/usr/local/maven/bin/mvn  package
# 2. 提交应用到Spark
/usr/local/spark/bin/spark-submit --class "SimpleApp" ~/sparkapp3/target/simple-project-1.0.jar 2>&1 | grep "Lines with a:"
```

#### 预期结果
输出：`Lines with a: 72, Lines with b: 39`（与Java应用结果一致，验证代码正确性）


### 如何切换运行 Java 或 Scala 程序

#### 🔹 运行 Java 版本
1. 修改 pom 设置 Java MainClass：
```xml
<mainClass>com.example.SparkJavaApp</mainClass>
```

2. 打包
```bash
mvn clean package
```

3. 执行
```bash
spark-submit \
 --class com.example.SparkJavaApp \
 target/spark-maven-app-1.0-SNAPSHOT-jar-with-dependencies.jar \
 file:///usr/local/spark/README.md
```

#### 🔹 运行 Scala 版本
1. 只需修改 pom 中 `<mainClass>` 为 Scala 类：
```xml
<mainClass>com.example.SparkScalaApp</mainClass>
```

2. 重新打包：
```bash
mvn clean package
```

3. 运行方式：
```bash
spark-submit \
 --class com.example.SparkScalaApp \
 target/spark-maven-app-1.0-SNAPSHOT-jar-with-dependencies.jar \
 file:///usr/local/spark/README.md
```

