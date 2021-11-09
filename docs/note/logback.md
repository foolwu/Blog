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

推荐命名 logback-spring.xml，将 xml 放至 src/main/resource 下面。如果是自定义命名，需要到application.properties指定。

```xml
logging.config=classpath:xxx.xml
```

直接给出示例。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，比如: 如果设置为WARN，则低于WARN的信息都不会输出 -->
<!-- scan:当此属性设置为true时，配置文档如果发生改变，将会被重新加载，默认值为true -->
<!-- scanPeriod:设置监测配置文档是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。 -->
<!-- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。 -->
<configuration scan="true" scanPeriod="30 seconds" debug="false">
    <!--定义参数,后面可以通过${}使用-->
    <!-- 定义日志的根目录 -->
    <propert name="LOG_HOME" value="/logs"/>
    <!-- 定义日志文件名称 -->
    <propert name="appname" value="user"/>
    <!-- 定义日志文件大小，默认为10MB -->
    <propert name="maxFileSize" value="20MB"/>
    <!-- 定义日志格式 -->
            <!-- %37():如果字符没有37个字符长度,则左侧用空格补齐 -->
            <!-- %-37():如果字符没有37个字符长度,则右侧用空格补齐 -->
            <!-- %15.15():如果记录的线程字符长度小于15(第一个)则用空格在左侧补齐,如果字符长度大于15(第二个),则从开头开始截断多余的字符 -->
            <!-- %-40.40():如果记录的logger字符长度小于40(第一个)则用空格在右侧补齐,如果字符长度大于40(第二个),则从开头开始截断多余的字符 -->
            <!-- %msg：日志打印详情 -->
            <!-- %n:换行符 -->
            <!-- %highlight():转换说明符以粗体红色显示其级别为ERROR的事件，红色为WARN，BLUE为INFO，以及其他级别的默认颜色。 -->
    <propert name="layoutPattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS} %highlight(%-5level) --- [%15.15(%thread)] %cyan(%-40.40(%logger{40})) : %msg%n"/>
    <propert name="Pattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level --- [%15.15(%thread)] %-40.40(%logger{40}) : %msg%n"/>
    <!-- 定义日志文件保留天数 -->
    <propert name="maxHistory" value="30"/>
    <!-- appender是configuration的子节点，是负责写日志的组件。 -->
    
    <!-- ConsoleAppender：把日志输出到控制台 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 默认情况下，每个日志事件都会立即刷新到基础输出流。 这种默认方法更安全，因为如果应用程序在没有正确关闭appender的情况下退出，则日志事件不会丢失。
        <!--定义控制台输出格式-->
        <encoder>
            <pattern>${layoutPattern}</pattern>
            <!-- 控制台也要使用UTF-8，不要使用GBK，否则会中文乱码 -->
            <charset>UTF-8</charset>
        </encoder>
    </appender>
 
    <!-- info 日志-->
    <!-- RollingFileAppender：滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->
    <!-- 1.先按日期存日志，日期变了，将前一天的日志文件名重命名为XXX%日期%索引，新的日志仍然是info.log -->
    <!-- 2.如果日期没有发生变化，但是当前日志的文件大小超过10MB时，对当前日志进行分割 重命名-->
    <appender name="info" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--日志文件路径和名称-->
        <File>logs/info.log</File>
        <!--是否追加到文件末尾,默认为true-->
        <append>true</append>
        <!-- 滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 文件名：logs/info.2017-12-05.0.log -->
            <!-- 注意：SizeAndTimeBasedRollingPolicy中 ％i和％d令牌都是强制性的，必须存在，要不会报错 -->
            <fileNamePattern>${Log_Home}/info.%d.%i.log</fileNamePattern>
            <!-- 每个日志文件的保存期限为30天,如果定义为yyyy-MM,则单位为月,默认yyyy-MM-dd-->
            <maxHistory>${maxHistory}</maxHistory>
            <!-- 每个日志文件到10mb的时候开始切分，最多保留30天，最大到20GB，1.1.6后支持 -->
            <totalSizeCap>20GB</totalSizeCap>
            <maxFileSize>${maxFileSize}</maxFileSize>
        </rollingPolicy>
        <!--编码器-->
        <encoder>
            <!-- pattern节点，用来设置日志的输入格式.日志文件中不能设置颜色,否则颜色部分会有ESC[0:39em等乱码-->
            <pattern>${Pattern}</pattern>
            <!-- 记录日志的编码:此处设置字符集 - -->
            <charset>UTF-8</charset>
        </encoder>
        <!-- 过滤器 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- info级别以下的日志不会输出 -->
            <level>INFO</level>
            <!-- 如果命中ERROR就禁止这条日志 -->
            <onMatch>DENY</onMatch>
            <!-- 如果没有命中就使用这条规则 -->
            <onMismatch>ACCEPT</onMismatch>
        </filter>
    </appender>
 
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
 
    <!--给定记录器的每个启用的日志记录请求都将转发到该记录器中的所有appender以及层次结构中较高的appender（不用在意level值）。
    换句话说，appender是从记录器层次结构中附加地继承的。
    例如，如果将控制台appender添加到根记录器，则所有启用的日志记录请求将至少在控制台上打印。
    如果另外将文件追加器添加到记录器（例如L），则对L和L'子项启用的记录请求将打印在文件和控制台上。
    通过将记录器的additivity标志设置为false，可以覆盖此默认行为，以便不再添加appender累积-->
    <!-- configuration中最多允许一个root，别的logger如果没有设置级别则从父级别root继承 -->
    <root level="INFO">
        <appender-ref ref="STDOUT" />
    </root>
 
    <!-- 指定项目中某个包，当有日志操作行为时的日志记录级别 -->
    <!-- 级别依次为【从高到低】：FATAL > ERROR > WARN > INFO > DEBUG > TRACE  -->
    <logger name="com.sailing.springbootmybatis" level="INFO">
        <appender-ref ref="info" />
        <appender-ref ref="error" />
    </logger>
 
    <!-- 利用logback输入mybatis的sql日志，
    注意：如果不加 additivity="false" 则此logger会将输出转发到自身以及祖先的logger中，就会出现日志文件中sql重复打印-->
    <logger name="com.sailing.springbootmybatis.mapper" level="DEBUG" additivity="false">
        <appender-ref ref="info" />
        <appender-ref ref="error" />
    </logger>
 
    <!-- additivity=false代表禁止默认累计的行为，即com.atomikos中的日志只会记录到日志文件中，不会输出层次级别更高的任何appender-->
    <logger name="com.atomikos" level="INFO" additivity="false">
        <appender-ref ref="info" />
        <appender-ref ref="error" />
    </logger>
 
</configuration>
```
