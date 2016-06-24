目录

Maven介绍
Maven安装(win)
用Maven构建Hadoop环境
MapReduce程序开发
模板项目上传github

1. Maven介绍
Apache Maven，是一个Java的项目管理及自动构建工具，由Apache软件基金会所提供。基于项目对象模型（缩写：POM）概念，Maven利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。曾是Jakarta项目的子项目，现为独立Apache项目。

2. Maven安装(win)
下载Maven：http://maven.apache.org/download.cgi
下载最新的apache-maven-3.3.9-bin.zip文件，
在win上解压到 E:\software\apache-maven-3.3.9

并把maven添加到环境变量PATH中：
E:\software\apache-maven-3.3.9\bin

然后，打开命令行输入mvn，我们会看到mvn命令的运行效果
C:\Users\Polaris>mvn
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 0.086 s
[INFO] Finished at: 2016-06-24T14:33:03+08:00
[INFO] Final Memory: 5M/149M
[INFO] ------------------------------------------------------------------------
[ERROR] No goals have been specified for this build. You must specify a valid lifecycle phase or a goal in the format <plugin-prefix>:<goal> or <plugin-group-id>:<plugin-artifact-id>[:
<plugin-version>]:<goal>. Available lifecycle phases are
: validate, initialize, generate-sources, process-sources, generate-resources, process-resources, compile, process-classes, generate-test-sources, process-test-sources, generate-test-resources, process-test-resources, test-compile, process-test-classes, test, prepare-package, package, pre-integration-test, integration-test, post-integration-test, verify, install, deploy, pre-clean, clean, post-clean, pre-site, site, post-site, site-deploy. -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/NoGoalSpecifiedException

Maven的Eclipse插件配置
 


3. 用Maven构建Hadoop环境

1). 用Maven创建一个标准化的Java项目
F:\workspace>mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetyp
es -DgroupId=org.conan.myhadoop.mr -DartifactId=myhadoop -DpackageName=org.conan
.myhadoop.mr -Dversion=1.0-SNAPSHOT -DinteractiyeMode=false




F:\workspace>cd myhadoop
F:\workspace\myhadoop>mvn clean install


2). 导入项目到eclipse
我们创建好了一个基本的maven项目，然后导入到eclipse中。 
 


3). 增加hadoop依赖
这里我使用hadoop-1.0.3版本，修改文件：pom.xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:
xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.conan.myhadoop.mr</groupId>
  <artifactId>myhadoop</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>myhadoop</name>
  <url>http://maven.apache.org</url>
  <dependencies>

  <dependency>
  	<groupId>org.apache.hadoop</groupId>
  	<artifactId>hadoop-core</artifactId>
  	<version>1.0.3</version>
  </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    
    <dependency>
    	<groupId>jdk.tools</groupId>
    	<artifactId>jdk.tools</artifactId>
    	<version>1.6</version>
    	<scope>system</scope>
    	<systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>
    </dependency>

  </dependencies>
</project>

pom.xml报错时添加jdk.tools的部分。

4). 下载依赖
下载依赖：
F:\workspace\myhadoop>mvn clean install



5). 从Hadoop集群环境下载hadoop配置文件
core-site.xml
hdfs-site.xml
mapred-site.xml

查看core-site.xml
<property>
<name>fs.default.name</name>
<value>hdfs://master:9000</value>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>/cloud/hadoop-1.0.3/tmp</value>
</property>
<property>
<name>io.sort.mb</name>
<value>256</value>
</property>
查看hdfs-site.xml
<property>
<name>dfs.data.dir</name>
<value>/cloud/hadoop-1.0.3/data</value>
</property>
<property>
<name>dfs.replication</name>
<value>1</value>
</property>
<property>
<name>dfs.permissions</name>
<value>false</value>
</property>


查看mapred-site.xml
<property>
<name>mapred.job.tracker</name>
<value>hdfs://master:9001</value>
</property>



保存在src/main/resources/hadoop目录下面
删除原自动生成的文件：App.java和AppTest.java
 




6).配置本地host，增加master的域名指向
c:/Windows/System32/drivers/etc/hosts

192.168.10.246 master


4. MapReduce程序开发
编写一个简单的MapReduce程序，实现wordcount功能。
新一个Java文件：WordCount.java
package org.conan.myhadoop.mr;
/*
 * 修改之后的core
 */
import java.io.IOException;
import java.util.Iterator;
import java.util.StringTokenizer;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.FileInputFormat;
import org.apache.hadoop.mapred.FileOutputFormat;
import org.apache.hadoop.mapred.JobClient;
import org.apache.hadoop.mapred.JobConf;
import org.apache.hadoop.mapred.MapReduceBase;
import org.apache.hadoop.mapred.Mapper;
import org.apache.hadoop.mapred.OutputCollector;
import org.apache.hadoop.mapred.Reducer;
import org.apache.hadoop.mapred.Reporter;
import org.apache.hadoop.mapred.TextInputFormat;
import org.apache.hadoop.mapred.TextOutputFormat;

public class WordCount {

	public static class WordCountMapper extends MapReduceBase implements Mapper<Object, Text, Text, IntWritable> {
		private final static IntWritable one = new IntWritable(1);
		private Text word = new Text();

		public void map(Object key, Text value, OutputCollector<Text, IntWritable> output, Reporter reporter)
				throws IOException {
			StringTokenizer itr = new StringTokenizer(value.toString());
			while (itr.hasMoreTokens()) {
				word.set(itr.nextToken());
				output.collect(word, one);
			}
		}
	}

	public static class WordCountReducer extends MapReduceBase
			implements Reducer<Text, IntWritable, Text, IntWritable> {
		private IntWritable result = new IntWritable();

		public void reduce(Text key, Iterator<IntWritable> values, OutputCollector<Text, IntWritable> output,
				Reporter reporter) throws IOException {
			int sum = 0;
			while (values.hasNext()) {
				sum += values.next().get();
			}
			result.set(sum);
			output.collect(key, result);
		}
	}

	public static void main(String[] args) throws Exception {
		String input = "hdfs://master:9000/input";
		String output = "hdfs://master:9000/output";

		JobConf conf = new JobConf(WordCount.class);
		conf.setJobName("WordCount");
		conf.addResource("classpath:/hadoop/core-site.xml");
		conf.addResource("classpath:/hadoop/hdfs-site.xml");
		conf.addResource("classpath:/hadoop/mapred-site.xml");

		conf.setOutputKeyClass(Text.class);
		conf.setOutputValueClass(IntWritable.class);

		conf.setMapperClass(WordCountMapper.class);
		conf.setCombinerClass(WordCountReducer.class);
		conf.setReducerClass(WordCountReducer.class);

		conf.setInputFormat(TextInputFormat.class);
		conf.setOutputFormat(TextOutputFormat.class);

		FileInputFormat.setInputPaths(conf, new Path(input));
		FileOutputFormat.setOutputPath(conf, new Path(output));

		JobClient.runJob(conf);
		System.exit(0);
	}
}


启动Java APP.
控制台输出错误
这个错误是win中开发特有的错误，文件权限问题，在Linux下可以正常运行。
解决方法是，修改/hadoop-1.0.3/src/core/org/apache/hadoop/fs/FileUtil.java文件
688-692行注释，然后重新编译源代码，重新打一个hadoop.jar的包。
685 private static void checkReturnValue(boolean rv, File p,
686                                        FsPermission permission
687                                        ) throws IOException {
688     /*if (!rv) {
689       throw new IOException("Failed to set permissions of path: " + p +
690                             " to " +
691                             String.format("%04o", permission.toShort()));
692     }*/
693   }


我们还要替换maven中的hadoop类库。
C:\Users\Polaris\.m2\repository\org\apache\hadoop\hadoop-core\1.0.3\hadoop-core-1.0.3.jar
再次启动Java APP，这次就运行正确了。 
成功运行了wordcount程序。


5. 模板项目上传github
git clone git@github.com:Polaris-zlf/maven-hadoop.git
git clone https://github.com/Polaris-zlf/maven-hadoop.git

