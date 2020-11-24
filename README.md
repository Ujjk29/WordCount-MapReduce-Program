# WordCount-MapReduce-Program
We are given with the assignment with following objectives:
1. Understand the MapReduce programming model by implementing WordCount program.
2. Setting up Hadoop and running a MapReduce program in Hadoop environment.

## System
* MacBook Air
* Processor 1.8GHz Intel Core i5
* Memory 8GB 1600 MHz DDR3

## Java Version
```console
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```

## Downloading and Installing Hadoop on MacOS
The following command will download and install all the dependencies required for hadoop. Now hadoop is installed at path /usr/local/Cellar. Version 3.2.1_1
```console
$ brew install hadoop
```

## Configuring Hadoop
I had updated environment variable settings at hadoop-env.sh with the following code.
```console
$ cd /usr/local/cellar/hadoop/3.2.1_1/libexec/etc/hadoop
$ open hadoop-env.sh
```

Now I wrote the following line on hadoop-env.sh
```sh
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home"
export PATH=${JAVA_HOME}/bin:${PATH}
export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true -Djava.security.krb5.realm= -Djava.security.krb5.kdc="
export HADOOP_OS_TYPE=${HADOOP_OS_TYPE:-$(uname -s)}
```

After this I had made changes to the following files:
1. Core files
   ```console
   $ open core-site.xml
   ```
   In that file, I had added the following lines:
   ```xml
   <configuration>
      <property>
         <name>fs.defaultFS</name>
         <value></value>
      </property>
   </configuration>
   ```

2. hdfs file
   ```console
   $ open hdfs-site.xml
   ```
   In that file, I had added the following lines:
   ```xml
   <configuration>
      <property>
         <name>dfs.replication</name>
         <value>1</value>
      </property>
   </configuration>
   ```

3. Mapred file
   ```console
   $ open mapred-site.xml
   ```
   In that file, I had added the following lines:
   ```xml
   <configuration>
      <property>
         <name>mapreduce.framework.name</name>
         <value>yarn</value>
      </property>
      <property>
         <name>mapreduce.application.classpath</name>
         <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOM E/share/hadoop/mapreduce/lib/*</value>
      </property>
   </configuration>
   ```

4. Yarn File
   ```console
   $ open yarn-site.xml
   ```
   In that file, I had added the following lines:
   ```xml
   <configuration>
      <property>
         <name>yarn.nodemanager.aux-services</name>
         <value>mapreduce_shuffle</value>
      </property>
      <property>
         <name>yarn.nodemanager.env-whitelist</name>
         <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CON F_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED _HOME</value>
      </property>
   </configuration>
   ```

Now I run the following command to format namenode.
```console
$ cd /usr/local/Cellar/hadoop/3.2.1_1/libexec/bin
$ hdfs namenode -format
```

## Running the Hadoop
To run the Hadoop environment, write the following command.
```console
$ cd /usr/local/Cellar/hadoop/3.2.1/libexec/sbin
$ ./start-all.sh
```

## Input Text
I have used the book "Pride and Prejudice".

## WordCount Java Code Snippet
```java
import java.io.IOException; import java.util.StringTokenizer;
import org.apache.hadoop.conf.Configuration; import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable; import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat; import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
public class WordCount {
   public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable>{
      private final static IntWritable one = new IntWritable(1); 
      private Text word = new Text();
      public void map(Object key, Text value, Context context ) throws IOException, InterruptedException {
         StringTokenizer itr = new StringTokenizer(value.toString()); while (itr.hasMoreTokens()) {
            word.set(itr.nextToken());
            context.write(word, one); 
         }
      } 
   }
   public static class IntSumReducer extends Reducer<Text,IntWritable,Text,IntWritable> {
      private IntWritable result = new IntWritable();
      public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException { 
         int sum = 0;
         for (IntWritable val : values) { 
            sum += val.get();
         }
         result.set(sum); context.write(key, result);
      } 
   }
   public static void main(String[] args) throws Exception { 
      Configuration conf = new Configuration();
      Job job = new Job(conf,”wordcount”);
      job.setJarByClass(WordCount.class); 
      job.setMapperClass(TokenizerMapper.class); 
      job.setCombinerClass(IntSumReducer.class); 
      job.setReducerClass(IntSumReducer.class); 
      job.setOutputKeyClass(Text.class); 
      job.setOutputValueClass(IntWritable.class); 
      FileInputFormat.addInputPath(job, new Path(args[0])); 
      FileOutputFormat.setOutputPath(job, new Path(args[1])); 
      System.exit(job.waitForCompletion(true) ? 0 : 1);
   }
}
```
I had saved this java program at path ```/usr/local/Cellar/hadoop/3.2.1_1/bin``` with name ```WordCount.java```
Now to compile this program I used the following line of code
```console
$ cd /usr/local/Cellar/hadoop/3.2.1_1/bin
$ hadoop com.sun.tools.javac.Main WordCount.java
```
Now WordCount.java is compiled and three classes are created. And I now create a jar file with all the classes in it.
```console
$ jar cf wordcount.jar WordCount*.class
```
Now our jar file is created that contains all the WordCount* classes.

## Running the application
At path ```/usr/local/Cellar/hadoop/3.2.1_1/bin```, you can write the following line:
```console
$ hadoop jar wordcount.jar WordCount "path_to_input_file" "path_to_output_file"
```
I had writen the following line:
```console
$ hadoop jar wordcount.jar WordCount /Users/ujjk29/Desktop/wordcount/input /Users/ujjk29/wordcount/output
```
Following is the console output:
```console
2020-03-17 22:23:30,146 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
2020-03-17 22:23:30,446 INFO client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
2020-03-17 22:23:31,167 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
2020-03-17 22:23:31,292 INFO input.FileInputFormat: Total input files to process : 1
2020-03-17 22:23:31,364 INFO mapreduce.JobSubmitter: number of splits:1
2020-03-17 22:23:31,584 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1584463806876_0001
2020-03-17 22:23:31,586 INFO mapreduce.JobSubmitter: Executing with tokens: []
2020-03-17 22:23:31,857 INFO conf.Configuration: resource-types.xml not found
2020-03-17 22:23:31,857 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2020-03-17 22:23:32,330 INFO impl.YarnClientImpl: Submitted application application_1584463806876_0001
2020-03-17 22:23:32,372 INFO mapreduce.Job: The url to track the job: http://Ujjawals-MacBook-Air.local:8088/proxy/application_1584463806876_0001/
2020-03-17 22:23:32,372 INFO mapreduce.Job: Running job: job_1584463806876_0001
2020-03-17 22:23:40,554 INFO mapreduce.Job: Job job_1584463806876_0001 running in uber mode : false
2020-03-17 22:23:40,555 INFO mapreduce.Job: map 0% reduce 0%
2020-03-17 22:23:47,647 INFO mapreduce.Job: map 100% reduce 0%
2020-03-17 22:23:53,707 INFO mapreduce.Job: map 100% reduce 100%
2020-03-17 22:23:53,719 INFO mapreduce.Job: Job job_1584463806876_0001 completed successfully
2020-03-17 22:23:53,862 INFO mapreduce.Job: Counters: 44
         File System Counters
                  FILE: Number of bytes read=990912
                  FILE: Number of bytes written=1015201 
                  FILE: Number of read operations=0 
                  FILE: Number of large read operations=0 
                  FILE: Number of write operations=0
         Job Counters
                  Launched map tasks=1
                  Launched reduce tasks=1
                  Rack-local map tasks=1
                  Total time spent by all maps in occupied slots (ms)=3785
                  Total time spent by all reduces in occupied slots (ms)=3585 
                  Total time spent by all map tasks (ms)=3785
                  Total time spent by all reduce tasks (ms)=3585
                  Total vcore-milliseconds taken by all map tasks=3785
                  Total vcore-milliseconds taken by all reduce tasks=3585
                  Total megabyte-milliseconds taken by all map tasks=3875840 
                  Total megabyte-milliseconds taken by all reduce tasks=3671040
         Map-Reduce Framework
                  Map input records=14597
                  Map output records=124714
                  Map output bytes=1210186
                  Map output materialized bytes=205572
                  Input split bytes=114
                  Combine input records=124714
                  Combine output records=13673
                  Reduce input groups=13673
                  Reduce shuffle bytes=205572
                  Reduce input records=13673
                  Reduce output records=13673
                  Spilled Records=27346
                  Shuffled Maps =1
                  Failed Shuffles=0
                  Merged Map outputs=1
                  GC time elapsed (ms)=100
                  CPU time spent (ms)=0
                  Physical memory (bytes) snapshot=0
                  Virtual memory (bytes) snapshot=0
                  Total committed heap usage (bytes)=395837440
         Shuffle Errors 
                  BAD_ID=0
                  CONNECTION=0 
                  IO_ERROR=0 
                  WRONG_LENGTH=0 
                  WRONG_MAP=0 
                  WRONG_REDUCE=0
         File Input Format Counters 
                  Bytes Read=785207
         File Output Format Counters 
                  Bytes Written=153540
```
I got the output in the file [part-r-00000](/part-r-00000)

If you run the following code at path ```/usr/local/Cellar/hadoop/3.2.1_1/bin``` you will get wordcount of words.
```console
$ hadoop fs -cat "path_to_output/part-r-00000"
```
I had written the following code:
```console
$ hadoop fs -cat /Users/ujjk29/Desktop/wordcount/output/part-r-0000
```

## Stopping the Hadoop environment
Write the following code at path ```/usr/local/hadoop/3.2.1_1/sbin```
```console
$ ./stop-all.sh
```
