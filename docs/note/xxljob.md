## 1.下载管理端源码，解压后导入IDEA

码云地址：http://gitee.com/xuxueli0323/xxl-job/releases
github 地址：https://github.com/xuxueli/xxl-job/releases
文档地址：https://www.xuxueli.com/xxl-job/

## 2.导入sql文件

导入 doc/db下 的 sql 文件，数据库用户和密码修改成自己的配置

```properties
### xxl-job, datasource
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=admin
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

## 3.启动管理端

http://localhost:8080/xxl-job-admin 

默认用户/密码：admin/123456

## 4.项目配置xxlJob

### 4.1pom导入依赖

```xml
        <!-- xxl-job-core 2021.9-->
        <dependency>
            <groupId>com.xuxueli</groupId>
            <artifactId>xxl-job-core</artifactId>
            <version>2.3.0</version>
        </dependency>
```
### 4.2config类

```java
@Configuration
public class XxlJobConfig {
    private Logger logger = LoggerFactory.getLogger(XxlJobConfig.class);

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.accessToken}")
    private String accessToken;

    @Value("${xxl.job.executor.appname}")
    private String appname;

    @Value("${xxl.job.executor.address}")
    private String address;

    @Value("${xxl.job.executor.ip}")
    private String ip;

    @Value("${xxl.job.executor.port}")
    private int port;

    @Value("${xxl.job.executor.logpath}")
    private String logPath;

    @Value("${xxl.job.executor.logretentiondays}")
    private int logRetentionDays;

    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        logger.info(">>>>>>>>>>> xxl-job config init.");
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appname);
        xxlJobSpringExecutor.setAddress(address);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);

        return xxlJobSpringExecutor;
    }
}
```
### 4.3配置

```properties
### 调度中心部署跟地址 [选填]：如调度中心集群部署存在多个地址则用逗号分隔。执行器将会使用该地址进行"执行器心跳注册"和"任务结果回调"；为空则关闭自动注册；和管理端配置的地址和端口号一致
xxl.job.admin.addresses=http://127.0.0.1:8080/xxl-job-admin
### 执行器通讯TOKEN [选填]：非空时启用；
xxl.job.accessToken=
### 执行器AppName [选填]：执行器心跳注册分组依据；为空则关闭自动注册，可自定义
xxl.job.executor.appname=xxl-job-executor-sample
### 执行器注册 [选填]：优先使用该配置作为注册地址，为空时使用内嵌服务 ”IP:PORT“ 作为注册地址。从而更灵活的支持容器类型执行器动态IP和动态映射端口问题。
xxl.job.executor.address=
### 执行器IP [选填]：默认为空表示自动获取IP，多网卡时可手动设置指定IP，该IP不会绑定Host仅作为通讯实用；地址信息用于 "执行器注册" 和 "调度中心请求并触发任务"；
xxl.job.executor.ip=
### 执行器端口号 [选填]：小于等于0则自动获取；默认端口为9999，单机部署多个执行器时，注意要配置不同执行器端口；
xxl.job.executor.port=9999
### 执行器运行日志文件存储磁盘路径 [选填] ：需要对该路径拥有读写权限；为空则使用默认路径；
xxl.job.executor.logpath=/data/applogs/xxl-job/jobhandler
### 执行器日志文件保存天数 [选填] ： 过期日志自动清理, 限制值大于等于3时生效; 否则, 如-1, 关闭自动清理功能；
xxl.job.executor.logretentiondays=30
```
### 4.4定义job方法

```java
@Component
public class XxlJobTest {

    @XxlJob("testJobHanlder")
    public void xxlJobTestMission(){
        SimpleDateFormat dateFormat=new SimpleDateFormat("yy-MM-dd HH:mm:ss");
        System.out.println("现在在执行一次定时任务，现在的时间是："+ dateFormat.format(System.currentTimeMillis()));
    }
    
}
```

## 5. 管理端配置定时任务

### 5.1新增执行器

![image-20210928100127396](C:\Users\Mg\AppData\Roaming\Typora\typora-user-images\image-20210928100127396.png)

AppName 要和配置的 AppName 一致（4.3），名称可自定义。机器地址手动录入填写格式为：http://localhost:9999，9999 为默认端口（配置见 4.3）

### 5.2 新增任务

JobHandler 填写定时任务上注解 @XxJob 定义的名称，Cron 配置定时间隔，保存后可查看任务执行

调度日志可以查看任务执行情况