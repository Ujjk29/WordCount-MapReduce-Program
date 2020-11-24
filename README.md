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
