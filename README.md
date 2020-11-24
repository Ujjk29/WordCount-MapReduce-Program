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
