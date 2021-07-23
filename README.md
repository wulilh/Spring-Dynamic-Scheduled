# 序言

Spring 3.0 版本之后自带定时任务，提供了@EnableScheduling注解和@Scheduled注解来实现定时任务功能。

使用SpringBoot创建定时任务非常简单，目前主要有以下三种创建方式：

## 1. 基于注解（@Scheduled）

> @Scheduled注解和@EnableScheduling注解的使用，基于注解@Scheduled默认为串行执行，默认只有一个单例化的线程池（始终只有一个线程）去处理定时任务;多个任务时，任务的执行时机会受上一个任务执行时间的影响。
> > 在线生成Cron表达式的工具：http://cron.qqe2.com/ 来生成自己想要的表达式。

@EnableScheduling注解： 在配置类上使用，开启计划任务的支持（类上）。

@Scheduled注解： 来声明这是一个任务，包括 cron，fixDelay，fixRate 等类型（方法上，需先开启计划任务的支持）。


    String cron() default "";         cron表达式
 
    String zone() default "";         时区，接收一个 java.util.TimeZone#ID。cron表达式会基于该时区解析。默认是一个空字符串，即取服务器所在地的时区。比如我们一般使用的时区Asia/Shanghai。该字段我们一般留空。
 
    long fixedDelay() default -1L;      上一次执行完毕时间点之后多长时间再执行。
 
    String fixedDelayString() default "";       与 fixedDelay 意思相同，只是使用字符串的形式。唯一不同的是支持占位符。
 
    long fixedRate() default -1L;       上一次开始执行时间点之后多长时间再执行。
 
    String fixedRateString() default "";        与 fixedRate 意思相同，只是使用字符串的形式。唯一不同的是支持占位符。
 
    long initialDelay() default -1L;        第一次延迟多长时间后再执行。
 
    String initialDelayString() default "";     与 initialDelay 意思相同，只是使用字符串的形式。唯一不同的是支持占位符。


## 2. 基于接口（SchedulingConfigurer）
>  前者相信大家都很熟悉，但是实际使用中我们往往想从数据库中读取指定时间来动态执行定时任务，这时候基于接口的定时任务就派上用场了。

创建数据表
```sql
CREATE TABLE `tool_job` (
    `job_id` varchar(30) NOT NULL,
    `cron` varchar(30) NOT NULL COMMENT 'cron表达式',
    `job_name` varchar(255) DEFAULT NULL,
    `job_bean_name` varchar(255) DEFAULT NULL,
    `job_class_name` varchar(255) DEFAULT NULL,
    `job_method_params` varchar(255) DEFAULT NULL,
    `job_method_name` varchar(255) DEFAULT NULL,
    `job_status` tinyint(1) DEFAULT '1' COMMENT '任务状态（1正常 0暂停）',
    `job_exec_status` tinyint(1) DEFAULT '0' COMMENT '任务执行状态（ 0已停止 1执行中 ）',
    `remark` varchar(255) DEFAULT NULL,
    `create_time` datetime DEFAULT NULL,
    `update_time` datetime DEFAULT NULL,
    `create_by` varchar(255) DEFAULT NULL,
    `update_by` varchar(255) DEFAULT NULL,
    PRIMARY KEY (`job_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `db_test`.`tool_job`(`job_id`, `cron`, `job_name`, `job_bean_name`, `job_class_name`, `job_method_params`, `job_method_name`, `job_status`, `job_exec_status`, `remark`, `create_time`, `update_time`, `create_by`, `update_by`) VALUES ('2021072313468793', '0/5 * * * * ?', NULL, NULL, 'top.b0x0.scheduled.task.MyDynamicTask', NULL, 'execute', 1, 0, NULL, '2021-07-23 13:59:36', NULL, NULL, NULL);
INSERT INTO `db_test`.`tool_job`(`job_id`, `cron`, `job_name`, `job_bean_name`, `job_class_name`, `job_method_params`, `job_method_name`, `job_status`, `job_exec_status`, `remark`, `create_time`, `update_time`, `create_by`, `update_by`) VALUES ('2021072313468794', '0/5 * * * * ?', NULL, NULL, 'top.b0x0.scheduled.task.MyDynamicTask', NULL, 'sayHello', 1, 0, NULL, '2021-07-23 13:59:36', NULL, NULL, NULL);


```

## 3. 并行执行定时任务

配置ScheduledConfig  (定时器线程池（ScheduledThreadPoolExecutor）)