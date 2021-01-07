---
layout: post
title: springboot日志体系---log4j2
description: springboot下log4j2使用整理；maven依赖管理
category: blog
---

## springboot下log4j2日志的使用

1. 将springboot内置日志去掉，并引入log4j2  


    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-log4j2</artifactId>
    </dependency>

2. 在resource下面添加log4j2.xml配置文件   
如果你不添加，springboo会提示你没有对应文件，并使用默认的配置文件，这个时候级别可以在application.properties中配置  
logging.level.root=error  
默认的log4j2配置:  


    <?xml version="1.0" encoding="UTF-8"?>  
    <configuration status="OFF">  
      <appenders>  
        <Console name="Console" target="SYSTEM_OUT">  
          <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>  
        </Console>  
      </appenders>  
      <loggers>  
        <root level="error">  
          <appender-ref ref="Console"/>  
        </root>  
      </loggers>  
    </configuration>  

## 实战 

### 名词解释
appenders里设置日志的输出方式、级别和格式  
loggers里设置全局的级别和绑定appenders里的name  

File 日志输出到文件，可配置覆盖还是追加  
RollingFile “滚动文件”可作为按日输出日志的方式  
Console 控制台日志  
PatternLayout 格式化输出日志  
ThresholdFilter“阈值筛选器” 可单独设置appender的输出级别  
loggers里需要匹配每个appender的名称 name  

详细参见官网：https://logging.apache.org/log4j/2.x/manual/configuration.html#AutomaticConfiguration  

### 需求：打印到控制台的日志级别为Error，日志文件里保存的是INFO级别的日志

    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration status="OFF">
        <Appenders>
            <Console name="Console" target="SYSTEM_OUT">
                <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
                <ThresholdFilter level="ERROR" onMatch="ACCEPT" onMismatch="DENY"/>
                <PatternLayout pattern="%d{yyyy.MM.dd 'at' HH:mm:ss z} %-5level %class{36} %M() @%L - %msg%n"/>
            </Console>
            <File name="ERROR" fileName="logs/error.log" append="false">
                <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
                <PatternLayout pattern="%d{yyyy.MM.dd 'at' HH:mm:ss z} %-5level %class{36} %M() @%L - %msg%n"/>
            </File>
            <!--这个会打印出所有的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
            <RollingFile name="RollingFile" fileName="logs/app.log"
                         filePattern="log/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log.gz">
                <PatternLayout pattern="%d{yyyy.MM.dd 'at' HH:mm:ss z} %-5level %class{36} %M() @%L - %msg%n"/>
                <SizeBasedTriggeringPolicy size="5MB"/>
            </RollingFile>
        </Appenders>
        <Loggers>
            <Root level="INFO">
                <appender-ref ref="ERROR" />
                <appender-ref ref="RollingFile"/>
                <appender-ref ref="Console"/>
            </Root>
        </Loggers>
    </Configuration>

分析一下就很清晰了：  
三个appender：Console、File、RollingFile  
- Console 通过ThresholdFilter过滤规则只输出ERROR级别的错误(onMatch=”ACCEPT” onMismatch=”DENY” 匹配到的接受，没有匹配的走人)  
- File 也通过ThresholdFilter的方式输出到日志，当然了append=”false” 会在服务每次启动的时候清空日志(覆盖)  
- RollingFile 因为日志全局设置的为INFO，所以不需要ThresholdFilter,这里只需要指定filePattern和SizeBasedTriggeringPolicy就行了  

## 总结

### 可能遇到的问题：其他依赖下引入默认日志依赖

#### maven解释  
- exclusions：排除传递依赖  
在mavenB项目中引入mavenA项目依赖，通过依赖传递，会将mavenA中的jar包传递进来,如果B中不需要A中的某个jar包就可以使用以下标签   


    <exclusions>
        <exclusion>
        <groupId></groupId>
        <artifactId></artifactId>
        </exclusion>
    </exclusions>

- maven依赖树查看：   
mvn dependency:tree  
如果要输出到文件，找到pom文件的位置 进入命令行  
输入命令：   
mvn dependency:tree >/Users/raines/IdeaProjects/JavaPro/raines-learn/src/main/resources/tree.txt   
只查看关系的jar包  
mvn dependency:tree -Dverbose -Dincludes=groupId:artifactId:version:artifactId:version  
输入命令：  
mvn dependency:tree -Dverbose -Dincludes=org.springframework:spring-tx  

#### 以引入spring-boot-starter-data-redis为例

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    
去掉spring-boot-starter-web下的依赖后，发现还是无法使用引入的log4j2，遂查看maven依赖树，如下：

    [INFO] +- org.springframework.boot:spring-boot-starter-data-redis:jar:2.1.7.RELEASE:compile
    [INFO] |  +- org.springframework.boot:spring-boot-starter:jar:2.1.7.RELEASE:compile
    [INFO] |  |  +- org.springframework.boot:spring-boot:jar:2.1.7.RELEASE:compile
    [INFO] |  |  +- org.springframework.boot:spring-boot-autoconfigure:jar:2.1.7.RELEASE:compile
    [INFO] |  |  +- org.springframework.boot:spring-boot-starter-logging:jar:2.1.7.RELEASE:compile
    [INFO] |  |  |  +- ch.qos.logback:logback-classic:jar:1.2.3:compile
    [INFO] |  |  |  |  \- ch.qos.logback:logback-core:jar:1.2.3:compile
    [INFO] |  |  |  +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.11.2:compile
    [INFO] |  |  |  |  \- org.apache.logging.log4j:log4j-api:jar:2.11.2:compile
    [INFO] |  |  |  \- org.slf4j:jul-to-slf4j:jar:1.7.26:compile
    [INFO] |  |  \- javax.annotation:javax.annotation-api:jar:1.3.2:compile
    [INFO] |  +- org.springframework.data:spring-data-redis:jar:2.1.10.RELEASE:compile
    [INFO] |  |  +- org.springframework.data:spring-data-keyvalue:jar:2.1.10.RELEASE:compile
    [INFO] |  |  |  \- org.springframework.data:spring-data-commons:jar:2.1.10.RELEASE:compile
    [INFO] |  |  +- org.springframework:spring-tx:jar:5.1.9.RELEASE:compile
    [INFO] |  |  +- org.springframework:spring-oxm:jar:5.1.9.RELEASE:compile
    [INFO] |  |  \- org.springframework:spring-context-support:jar:5.1.9.RELEASE:compile
    [INFO] |  \- io.lettuce:lettuce-core:jar:5.1.8.RELEASE:compile

发现spring-boot-starter-data-redis依赖中，再次引入了spring-boot-starter-logging的依赖，遂再次删除  

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

删除后再次查看maven依赖树发现已无spring-boot-starter-logging依赖：  

    [INFO] +- org.springframework.boot:spring-boot-starter-data-redis:jar:2.1.7.RELEASE:compile
    [INFO] |  +- org.springframework.boot:spring-boot-starter:jar:2.1.7.RELEASE:compile
    [INFO] |  |  +- org.springframework.boot:spring-boot:jar:2.1.7.RELEASE:compile
    [INFO] |  |  +- org.springframework.boot:spring-boot-autoconfigure:jar:2.1.7.RELEASE:compile
    [INFO] |  |  \- javax.annotation:javax.annotation-api:jar:1.3.2:compile
    [INFO] |  +- org.springframework.data:spring-data-redis:jar:2.1.10.RELEASE:compile
    [INFO] |  |  +- org.springframework.data:spring-data-keyvalue:jar:2.1.10.RELEASE:compile
    [INFO] |  |  |  \- org.springframework.data:spring-data-commons:jar:2.1.10.RELEASE:compile
    [INFO] |  |  +- org.springframework:spring-tx:jar:5.1.9.RELEASE:compile
    [INFO] |  |  +- org.springframework:spring-oxm:jar:5.1.9.RELEASE:compile
    [INFO] |  |  \- org.springframework:spring-context-support:jar:5.1.9.RELEASE:compile
    [INFO] |  \- io.lettuce:lettuce-core:jar:5.1.8.RELEASE:compile

至此，已删除全部的依赖，使用log4j2成功













