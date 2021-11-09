目前比较流行的日志框架有 log4j、logback 等，Logback 是 log4j 的后续版本。另外 slf4j(Simple Logging Facade for Java) 则是一个日志门面框架，提供了日志系统中常用的接口，logback 和 log4j 则对 slf4j 进行了实现。

### 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

但实际开发中不需要添加此依赖。spring-boot-starter 包含了 spring-boot-starter-logging，而 spring-boot-starter-web 包含了 spring-boot-starter，因此直接引入 web 依赖即可。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 默认配置

默认情况下日志输出到控制台，若还想输入到文件，需在 application.properties 中设置 logging.file 或 logging.path 属性。

```xml
注：二者不能同时使用，如若同时使用，则只有logging.file生效
logging.file=文件名
logging.path=日志文件路径
 
logging.level.包名=指定包下的日志级别
logging.pattern.console=日志打印规则
```

### logback-spring.xml 详解

默认配置只能实现基本功能，要想实现定制化功能，需要使用配置文件。

推荐命名 logback-spring.xml，将 xml 放至 src/main/resource 下面。如果是自定义命名，需要到application.properties 指定。

```xml
logging.config=classpath:xxx.xml
```

直接给出示例。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 日志级别从低到高分为 TRACE < DEBUG < INFO < WARN < ERROR < FATAL，比如：如果设置为 WARN，则低于 WARN 的信息都不会输出 -->
<!-- scan：当此属性设置为 true 时，配置文档如果发生改变，将会被重新加载，默认值为 true -->
<!-- scanPeriod：设置监测配置文档是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当 scan 为 true 时，此属性生效。默认的时间间隔为 1 分钟 -->
<!-- debug:当此属性设置为 true 时，将打印出 logback 内部日志信息，实时查看 logback 运行状态。默认值为 false -->
<configuration scan="true" scanPeriod="30 seconds" debug="false">
    <!-- 定义参数,后面可以通过 ${} 使用 -->
    <!-- 定义日志的根目录 -->
    <propert name="LOG_HOME" value="/logs"/>
    <!-- 定义日志文件名称 -->
    <propert name="appname" value="user"/>
    <!-- 定义日志文件大小，默认为 10MB -->
    <propert name="maxFileSize" value="20MB"/>
    <!-- 定义日志格式 -->
            <!-- %37()：如果字符没有 37 个字符长度,则左侧用空格补齐 -->
            <!-- %-37()：如果字符没有 37 个字符长度,则右侧用空格补齐 -->
            <!-- %15.15()：如果记录的线程字符长度小于 15(第一个)则用空格在左侧补齐,如果字符长度大于 15(第二个),则从开头开始截断多余的字符 -->
            <!-- %-40.40()：如果记录的logger字符长度小于 40(第一个)则用空格在右侧补齐,如果字符长度大于 40(第二个),则从开头开始截断多余的字符 -->
            <!-- %msg：日志打印详情 -->
            <!-- %n：换行符 -->
            <!-- %highlight()：转换说明符以粗体红色显示其级别为 ERROR 的事件，红色为 WARN，BLUE 为 INFO，以及其他级别的默认颜色 -->
    <propert name="layoutPattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS} %highlight(%-5level) --- [%15.15(%thread)] %cyan(%-40.40(%logger{40})) : %msg%n"/>
    <propert name="Pattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level --- [%15.15(%thread)] %-40.40(%logger{40}) : %msg%n"/>
    <!-- 定义日志文件保留天数 -->
    <propert name="maxHistory" value="30"/>
    <!-- appender 是 configuration 的子节点，是负责写日志的组件 -->
    
    <!-- ConsoleAppender：把日志输出到控制台 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 默认情况下，每个日志事件都会立即刷新到基础输出流。这种默认方法更安全，因为如果应用程序在没有正确关闭 appender 的情况下退出，则日志事件不会丢失 -->
        <!-- 定义控制台输出格式 -->
        <encoder>
            <pattern>${layoutPattern}</pattern>
            <!-- 控制台也要使用 UTF-8，不要使用 GBK，否则会中文乱码 -->
            <charset>UTF-8</charset>
        </encoder>
    </appender>
 
    <!-- info 日志-->
    <!-- RollingFileAppender：滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->
    <!-- 1.先按日期存日志，日期变了，将前一天的日志文件名重命名为 XXX% 日期 % 索引，新的日志仍然是 info.log -->
    <!-- 2.如果日期没有发生变化，但是当前日志的文件大小超过 10MB 时，对当前日志进行分割重命名 -->
    <appender name="info" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--日志文件路径和名称-->
        <File>logs/info.log</File>
        <!--是否追加到文件末尾,默认为 true-->
        <append>true</append>
        <!-- 滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 文件名：logs/info.2017-12-05.0.log -->
            <!-- 注意：SizeAndTimeBasedRollingPolicy 中，％i 和 ％d 令牌都是强制性的，必须存在，要不会报错 -->
            <fileNamePattern>${Log_Home}/info.%d.%i.log</fileNamePattern>
            <!-- 每个日志文件的保存期限为 30 天,如果定义为 yyyy-MM,则单位为月,默认 yyyy-MM-dd -->
            <maxHistory>${maxHistory}</maxHistory>
            <!-- 每个日志文件到 10mb 的时候开始切分，最多保留 30 天，最大到 20GB，1.1.6 后支持 -->
            <totalSizeCap>20GB</totalSizeCap>
            <maxFileSize>${maxFileSize}</maxFileSize>
        </rollingPolicy>
        <!-- 编码器 -->
        <encoder>
            <!-- pattern 节点，用来设置日志的输入格式.日志文件中不能设置颜色,否则颜色部分会有ESC[0:39em 等乱码 -->
            <pattern>${Pattern}</pattern>
            <!-- 记录日志的编码：此处设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>
        <!-- 过滤器 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- info 级别以下的日志不会输出 -->
            <level>INFO</level>
            <!-- 如果命中 ERROR 就禁止这条日志 -->
            <onMatch>DENY</onMatch>
            <!-- 如果没有命中就使用这条规则 -->
            <onMismatch>ACCEPT</onMismatch>
        </filter>
    </appender>
 
        <!-- error 日志-->
    <appender name="error" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>logs/error.log</File>
        <append>true</append>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${Log_Home}/error.%d.%i.log</fileNamePattern>
            <maxHistory>30</maxHistory>
            <totalSizeCap>20GB</totalSizeCap>
            <maxFileSize>10MB</maxFileSize>
        </rollingPolicy>
        <encoder>
            <pattern>${Pattern}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
    </appender>
 
    <!-- 最多允许一个 root，如果没有定义其他 logger 则默认使用 root -->
    <root level="INFO">
        <appender-ref ref="STDOUT" />
    </root>
 
    <!-- 对于类路径以 com.example.logback 开头的 Logger,输出级别设置为 warn，输出到控制台 -->
    <!-- 未指定 appender，所以继承 root 节点中定义的 appender -->
    <logger name="com.example.logback" level="warn"/>

    <!-- 指定项目中某个包，当有日志操作行为时的日志记录级别 -->
    <!-- 如果 logger 里未指定 appender，它会继承 root 节点中定义的那些 appender -->
    <!-- 级别依次为【从高到低】：FATAL > ERROR > WARN > INFO > DEBUG > TRACE  -->
    <logger name="com.sailing.springbootmybatis" level="INFO">
        <appender-ref ref="info" />
        <appender-ref ref="error" />
    </logger>
 
    <!-- 
    利用 logback 输入 mybatis 的 sql 日志，
    注意：如果不加 additivity="false" 则此 logger 会将输出转发到自身以及祖先的 logger 中，就会出现日志文件中 sql 重复打印
    -->
    <logger name="com.sailing.springbootmybatis.mapper" level="DEBUG" additivity="false">
        <appender-ref ref="info" />
        <appender-ref ref="error" />
    </logger>
 
    <logger name="com.atomikos" level="INFO" additivity="false">
        <appender-ref ref="info" />
        <appender-ref ref="error" />
    </logger>
    
    <!-- 由于设置了 additivity="false" ，所以输出时不会使用 rootLogger 的 appender -->
    <!-- 但是这个 logger 本身又没有配置 appender，所以使用这个 logger 输出日志的话就不会输出到任何地方 -->
    <logger name="mytest2" level="info" additivity="false"/>
 
</configuration>
```
