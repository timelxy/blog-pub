# 使用Scala编写Spark UDF（Integration with Hive）

## Part1: 与Java编写UDF的异同
1. 本质是一样的，scala和java都是建立在JVM之上。目标都是编译代码，打包UDF需要的jar；
2. 区别主要在于环境和工具。Java惯用的IDEA + Maven。本文Scala我们使用的是VSCode + Maven，主要是Maven，VSCode仅用来编写代码。

## Part2: 步骤
1. 创建Scala工程。这里参考Scala官方文档的Maven指引，使用Maven的archetype直接构建
    ```
    mvn archetype:generate \
        -DarchetypeGroupId=net.alchim31.maven \
        -DarchetypeArtifactId=scala-archetype-simple
    ```

2. 配置pom.xml
* Scala Maven Plugin引入。参考：https://github.com/scala/scala-module-dependency-sample/blob/master/maven-sample/pom.xml
* 核心知识点：plugins的作用，为phase提供goal。这里是为打包jar提供compile scala代码的能力

    > Plugins are artifacts that provide goals to Maven
    > -- https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html

    下面示例列出了核心的\<dependencies>、\<build>
    ```
    <dependencies>
        <dependency>
        <groupId>org.scala-lang</groupId>
        <artifactId>scala-library</artifactId>
        <version>${scala.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>3.1.2</version>
        </dependency>
    </dependencies>

    
    
    <build>
        <finalName>UDF.udfscala.4</finalName>
        <sourceDirectory>src/main/scala</sourceDirectory>
        <testSourceDirectory>src/test/scala</testSourceDirectory>
        
        
        <plugins>
        
        <!-- 主要是这个plugin，提供了compile scala代码等goal -->
        <plugin>
            <groupId>net.alchim31.maven</groupId>
            <artifactId>scala-maven-plugin</artifactId>
            <version>4.5.6</version>
            <executions>
            <execution>
                <goals>
                <goal>compile</goal>
                <!-- <goal>testCompile</goal> -->
                </goals>
            </execution>
            </executions>
        </plugin>
        
        
        <!-- 用于本地测试 -->
        <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>1.2.1</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>java</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <mainClass>com.demo.ScalaUDFTest</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
    ```

3. 编写UDF Main Class代码 
    ```
    package com.demo

    import org.apache.hadoop.hive.ql.exec.UDF

    class ScalaUDFTest extends UDF {
        // 测试，为输入拼接前缀
        def evaluate(input: String): String = {
            return "UDF Scala: " + input
        }
    }
    ```

4. 打包jar
    ```
    # 命令行执行maven打包语句。jar输出在target目录下
    mvn package
    ```

5. 本地测试
    ```
    // 编写main函数
    object ScalaUDFTest{
        def main(args: Array[String]) {
            var obj = new ScalaUDFTest();
            println(obj.evaluate("xxx"));
        }
    }

    # 命令行执行运行
    mvn package exec:java -Dexec.mainClass:com.demo.ScalaUDFTest
    ```

6. Spark SQL中引用UDF
    ```
    CREATE TEMPORARY FUNCTION ScalaUDFTest AS 'com.demo.ScalaUDFTest';

    select ScalaUDFTest(xxx) from table1;
    ```

## Part3：Spark和Hive UDF的关系
Spark（2.4.3） HiveSupport的使用示例
> Spark SQL also supports reading and writing data stored in Apache Hive
Configuration of Hive is done by placing your hive-site.xml, core-site.xml (for security configuration), and hdfs-site.xml (for HDFS configuration) file in conf/
https://spark.apache.org/docs/2.4.3/sql-data-sources-hive-tables.html

Spark具体实现的兼容Hive的内容。     针对已经存在的Hive Store，Spark可以通过Thrift JDBC server与之交互
> Spark SQL is designed to be compatible with the Hive Metastore, SerDes and UDFs. Currently, Hive SerDes and UDFs are based on Hive 1.2.1, and Spark SQL can be connected to different versions of Hive Metastore
Deploying in Existing Hive Warehouses
The Spark SQL Thrift JDBC server is designed to be “out of the box” compatible with existing Hive installations. You do not need to modify your existing Hive Metastore or change the data placement or partitioning of your tables.
https://spark.apache.org/docs/2.4.3/sql-migration-guide-hive-compatibility.html

再想深入了解调用机理，就得读源码啦

## Part4：错误排查
```
case1：udf function本地构造用例测试成功，提交集群后运行失败
排查：根据日志，定位到是环境问题，本地使用的scala 2.12，集群使用的2.11。更新版本后成功
经验：任何spark任务失败，都应该细看报错的log，包括driver、executor的log
```               

## 参考
1. 如何在centos上安装scala：https://www.helplib.cn/xn_warm/how-to-install-scala-on-centos-7
2. Scala Maven：https://docs.scala-lang.org/tutorials/scala-with-maven.html
3. Scala Maven Plugin：https://davidb.github.io/scala-maven-plugin/index.html