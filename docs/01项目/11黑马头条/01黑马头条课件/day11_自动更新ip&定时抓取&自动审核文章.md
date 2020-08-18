# 自动更新ip&定时抓取&自动审核文章

## 目标

- 能够实现逆向爬取文章
- 能够动态设置ip代理池的更新
- 能够完成定时爬取文章
- 能够完成爬虫文章的自动审核

# 1 爬虫系统完善

## 1.1 增加代理下载

配置CrawlerConfig

com.heima.crawler.config.CrawlerConfig

```java
     @Autowired
    private CrawlerIpPoolService crawlerIpPoolService;

  @Bean
    public CrawlerProxyProvider getCrawlerProxyProvider() {
        CrawlerProxyProvider crawlerProxyProvider = new CrawlerProxyProvider();
        crawlerProxyProvider.setUsedProxyIp(isUsedProxyIp);
        //使用动态代理
        crawlerProxyProvider.setProxyProviderCallBack(new ProxyProviderCallBack() {

            @Override
            public List<CrawlerProxy> getProxyList() {
                return getCrawlerProxyList();
            }

            @Override
            public void unvailable(CrawlerProxy proxy) {
                unvailableProxy(proxy);
            }
        });
        crawlerProxyProvider.updateProxy();
        return crawlerProxyProvider;
    }


    /**
     * 获取初始化的Ip列表
     * 从代理Ip数据库中查询，响应时间小于5s 并且是可用的
     * @return
     */
    private List<CrawlerProxy> getCrawlerProxyList() {
        List<CrawlerProxy> crawlerProxyList = new ArrayList<CrawlerProxy>();
        ClIpPool clIpPool = new ClIpPool();
        //查询链接时长 <= 5s 的代理IP
        clIpPool.setDuration(5);
        List<ClIpPool> clIpPoolList = crawlerIpPoolService.queryAvailableList(clIpPool);
        if (null != clIpPoolList && !clIpPoolList.isEmpty()) {
            for (ClIpPool ipPool : clIpPoolList) {
                if (null != ipPool) {
                    crawlerProxyList.add(new CrawlerProxy(ipPool.getIp(), ipPool.getPort()));
                }
            }
        }
        return crawlerProxyList;
    }

    /**
     * 代理IP不可用处理方法
     * @param proxy
     */
    private void unvailableProxy(CrawlerProxy proxy) {
        if (null != proxy) {
            crawlerIpPoolService.unvailableProxy(proxy, "自动禁用");
        }
    }
```

## 1.2 设置逆向爬取文章

修改ProcessingFlowManager  添加以下方法

```java
/**
     * 逆向处理
     */
public void reverseHandel() {
    List<ParseItem> parseItemList = crawlerNewsAdditionalService.queryIncrementParseItem(new Date());
    handelReverseData(parseItemList);
    log.info("开始进行数据反向更新，增量数据数量：{}", parseItemList.size());
    if (null != parseItemList && !parseItemList.isEmpty()) {
        startTask(parseItemList, CrawlerEnum.HandelType.REVERSE);
    } else {
        log.info("增量数据为空不进行增量数据更新");
    }
}

/**
     * 反向同步数据处理
     *
     * @param parseItemList
     */
public void handelReverseData(List<ParseItem> parseItemList) {
    if (null != parseItemList && !parseItemList.isEmpty()) {
        for (ParseItem item : parseItemList) {
            item.setDocumentType(CrawlerEnum.DocumentType.PAGE.name());
            item.setHandelType(CrawlerEnum.HandelType.REVERSE.name());
        }
    }
}
```



# 2 代理IP更新

crawler.properties 配置

```properties
proxy.get.url=http://www.89ip.cn/tqdl.html?api=1&num=50&port=&address=&isp=
proxy.validate.url=http://myip.kkcha.com/
```

## 2.1 ProxyIpManager类

> 代理IP管理类

com.heima.crawler.proxy.ProxyIpManager

```java
/**
 * 动态代理IP管理类
 */
@Component
@Log4j2
public class ProxyIpManager {

    private static final ResourceBundle resourceBundle = ResourceBundle.getBundle("crawler");
    //获取代理IP配置的URL
    private static final String proxyGetUrl = resourceBundle.getString("proxy.get.url");
    /**
     * 抓取IP的正则表达式 预编译模式
     */
    Pattern proxyIpParttern = Pattern.compile("(\\d+\\.\\d+\\.\\d+\\.\\d+)\\:(\\d+)");

    @Autowired
    private CrawlerProxyProvider crawlerProxyProvider;

    @Autowired
    private CrawlerIpPoolService crawlerIpPoolService;

    @Autowired
    private SeleniumClient seleniumClient;

    /**
     * 校验动态代理IP
     */
    public void validateProxyIp() {
        List<ClIpPool> clIpPoolList = crawlerIpPoolService.queryList(new ClIpPool());
        if (null != clIpPoolList && !clIpPoolList.isEmpty()) {
            for (ClIpPool clIpPool : clIpPoolList) {
                boolean odlEnable = clIpPool.getEnable();
                //如果状态是可用
                validateProxyIp(clIpPool);
                //如果原始状态以及当前状态都是不可用，则判断是废弃的代理，删除
                if (!odlEnable && !clIpPool.getEnable()) {
                    crawlerIpPoolService.delete(clIpPool);
                    log.info("删除代理IP" + clIpPool.getIp() + ":" + clIpPool.getPort());
                } else {
                    crawlerIpPoolService.updateCrawlerIpPool(clIpPool);
                    log.info("更新代理IP" + clIpPool.getIp() + ":" + clIpPool.getPort());
                }
            }
        }
    }

    /**
     * 更新动态代理IP
     */
    public void updateProxyIp() {
        List<ClIpPool> clIpPoolList = getGrabClIpPoolList();
        if (null != clIpPoolList && !clIpPoolList.isEmpty()) {
            for (ClIpPool clIpPool : clIpPoolList) {
                validateProxyIp(clIpPool);
                if (clIpPool.getEnable()) {
                    boolean isExis = crawlerIpPoolService.checkExist(clIpPool.getIp(), clIpPool.getPort());
                    if (!isExis) {
                        crawlerIpPoolService.saveCrawlerIpPool(clIpPool);
                        log.info("插入代理IP:" + clIpPool.getIp() + ":" + clIpPool.getPort());
                    }
                }
            }
        }
    }

    /**
     * 抓取获取的动态代理IP
     *
     * @return
     */
    private List<ClIpPool> getGrabClIpPoolList() {
        List<ClIpPool> clIpPoolList = new ArrayList<ClIpPool>();
        //使用SeleniumUtils的方式获取代理IP数据
        CrawlerHtml crawlerHtml = seleniumClient.getCrawlerHtml(proxyGetUrl, crawlerProxyProvider.getRandomProxy(), "yd_cookie");
        if (null != crawlerHtml && StringUtils.isNotEmpty(crawlerHtml.getHtml())) {
            //通过正则表达式来获取代理IP数据
            Matcher matcher = proxyIpParttern.matcher(crawlerHtml.getHtml());
            while (matcher.find()) {
                String host = matcher.group(1);
                String port = matcher.group(2);
                ClIpPool clIpPool = new ClIpPool();
                clIpPool.setSupplier("89免费代理");
                clIpPool.setIp(host);
                clIpPool.setCreatedTime(new Date());
                clIpPool.setPort(Integer.parseInt(port));
                clIpPoolList.add(clIpPool);
            }
        }
        return clIpPoolList;
    }

    /**
     * 校验IP是否可用
     *
     * @param clIpPool
     */
    private void validateProxyIp(ClIpPool clIpPool) {
        clIpPool.setEnable(false);
        ProxyValidate proxyValidate = new ProxyValidate(clIpPool.getIp(), clIpPool.getPort());
        try {
            ProxyIpUtils.validateProxyIp(proxyValidate);
        } catch (Exception e) {
            log.error(e.getMessage());
        }
        if (proxyValidate.getReturnCode() == 200) {
            clIpPool.setEnable(true);
        }
        clIpPool.setCode(proxyValidate.getReturnCode());
        clIpPool.setDuration(proxyValidate.getDuration());
        clIpPool.setError(HMStringUtils.getFixedLengthStr(proxyValidate.getError(), 70));
    }
}

```



# 3 定时任务添加

## 3.1 添加 pom文件

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

## 3.2 添加配置类

### 3.2.1 QuartzScanJob类

com.heima.common.quartz.QuartzScanJob

```java
@Data
@Log4j2
@DisallowConcurrentExecution
@PersistJobDataAfterExecution
@Transactional
public class QuartzScanJob extends QuartzJobBean {

    @Value("branch-${info.git.branch?:default}")
    String branch;
    @Value("${spring.quartz.group-prefix}")
    String groupPrefix;
    @Autowired
    SchedulerFactoryBean schedulerFactoryBean;
    @Autowired
    DefaultListableBeanFactory defaultListableBeanFactory;
    @Autowired
    private Scheduler scheduler;
    // 描述器后缀
    private static final String DETAIL_SUFFIX = "AutoJobDetail";
    // 触发器后缀
    private static final String TRIGGER_SUFFIX = "AutoTrigger";

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        String temp = (String)jobExecutionContext.getJobDetail().getJobDataMap().get("branch");
        log.info("当前程序环境是[{}]，变量环境是：[{}]",getBranch(),temp);
        if(!branch.equalsIgnoreCase(temp)) {
            Map<String, AbstractJob> abs = defaultListableBeanFactory.getBeansOfType(AbstractJob.class);
            if(abs!=null){
                this.clearGroupJobAndTrigger(abs);
                for (String key : abs.keySet()) {
                    AbstractJob job = abs.get(key);
                    if(job.isAutoOverwrite()) {
                        String detailBeanName = key + DETAIL_SUFFIX;
                        createJobDetail(key, detailBeanName, job);
                        this.createdTrigger((JobDetail) defaultListableBeanFactory.getBean(detailBeanName), key, job);
                    }
                }
            }
            jobExecutionContext.getJobDetail().getJobDataMap().put("branch",getBranch());
        }else{
            log.info("============= skip auto init jobs");
        }
    }

    /**
     * 清理掉当前分组的JOB和触发器信息
     * @param abs
     */
    private void clearGroupJobAndTrigger(Map<String, AbstractJob> abs){
        try {
            Set<JobKey> jobKeys = scheduler.getJobKeys(GroupMatcher.groupStartsWith(getGroupPrefix()));
            for (JobKey jobKey : jobKeys) {
                String key = jobKey.getName().replace(TRIGGER_SUFFIX,"");
                AbstractJob job = abs.get(key);
                boolean isDelete = true;
                if(job!=null){
                    isDelete = job.isAutoOverwrite();
                }
                if(isDelete){
                    scheduler.deleteJob(jobKey);
                    log.info("auto manger clear job [{}]",jobKey);
                }
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    /**
     * 创建一个参数
     * @param beanName
     * @param job
     * @return
     */
    private void createJobDetail(String beanName,String detailBeanName,AbstractJob job){
        BeanDefinitionBuilder definitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(JobDetailFactoryBean.class);
        definitionBuilder.addPropertyValue("jobClass",job.getClass());
        definitionBuilder.addPropertyValue("beanName",beanName);
        definitionBuilder.addPropertyValue("group",groupPrefix+job.group());
        definitionBuilder.addPropertyValue("durability",job.isComplateAfterDelete());
        definitionBuilder.addPropertyValue("description",job.descJob());
        definitionBuilder.addPropertyValue("requestsRecovery",job.isStartAutoRecovery());
        definitionBuilder.addPropertyValue("jobDataAsMap",job.initParam());
        defaultListableBeanFactory.registerBeanDefinition(detailBeanName,definitionBuilder.getBeanDefinition());
        log.info("success register jobdetail : [{}]",detailBeanName);
    }

    /**
     * 注册触发器
     * @param detail
     * @param beanName
     * @param job
     */
    private void createdTrigger(JobDetail detail,String beanName,AbstractJob job){
        String temp[] = job.triggerCron();
        String name = beanName+"Trigger";
        for (int i = 0; i < temp.length; i++) {
            String  triggerName = name+"_"+i;
            BeanDefinitionBuilder definitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(CronTriggerFactoryBean.class);
            definitionBuilder.addPropertyValue("name",triggerName);
            definitionBuilder.addPropertyValue("group",groupPrefix+name);
            definitionBuilder.addPropertyValue("cronExpression",temp[i]);
            definitionBuilder.addPropertyValue("description",job.descTrigger());
            definitionBuilder.addPropertyValue("jobDetail",detail);
            defaultListableBeanFactory.registerBeanDefinition(triggerName,definitionBuilder.getBeanDefinition());
            try {
                scheduler.scheduleJob(detail, (Trigger) defaultListableBeanFactory.getBean(triggerName));
            }catch (Exception e){
                e.printStackTrace();
            }
            log.info("success register trigger : [{}]",triggerName);
        }
    }
}
```

quartz.properties

```properties
spring.quartz.group-prefix=${spring.application.name}_
spring.quartz.scheduler-name=quartz.${profiles.name}.${spring.application.name}
```

###  3.2.2 QuartzConfig类

com.heima.common.quartz.QuartzConfig

```java
/**
 * 1、每个服务一套Quart表，自动增加前缀groupPrefix
 * 2、每组groupPrefix的任务，命名符合自动管理的（AutoJobDetail结尾），将自动清理和增加、更新
 * 3、支持集群内单点初始化（必要squartzScanJobDetail），setOverwriteExistingJobs必须设置成false
 * 4、生产环境自动按照分支号生成和清理JOB注册
 * 5、启动后自动扫描AbstractJob的子类并注册成调度器
 */
@Data
@Configuration
@ConfigurationProperties(prefix = "spring.quartz")
@PropertySource("classpath:quartz.properties")
@Import(QuartzAutoConfiguration.class)
public class QuartzConfig {

    String groupPrefix;
    String schedulerName;

    /**
     * 重要Bean，务删
     * @return
     */
    @Bean
    @Primary
    public QuartzProperties quartzProperties(){
        QuartzProperties quartzProperties = new QuartzProperties();
        quartzProperties.setJobStoreType(JobStoreType.JDBC);
        quartzProperties.setSchedulerName(getSchedulerName());
        // 必须是false
        quartzProperties.setOverwriteExistingJobs(false);
        quartzProperties.getProperties().put("org.quartz.scheduler.instanceId",getSchedulerName()+"-instanceId");
        quartzProperties.getProperties().put("org.quartz.jobStore.class","org.quartz.impl.jdbcjobstore.JobStoreTX");
        // 优化死锁配置
        quartzProperties.getProperties().put("org.quartz.jobStore.txIsolationLevelSerializable","true");
        quartzProperties.getProperties().put("org.quartz.threadPool.threadCount","30");
        quartzProperties.getProperties().put("org.quartz.jobStore.tablePrefix",groupPrefix.toUpperCase()+"QRTZ_");
        quartzProperties.getProperties().put("org.quartz.jobStore.isClustered","true");
        return quartzProperties;
    }

    /**
     * 重要Bean，务删
     * @return
     */
    @Bean
    public JobDetailFactoryBean quartzScanJobDetail(){
        JobDetailFactoryBean jobDetailFactoryBean = new JobDetailFactoryBean();
        jobDetailFactoryBean.setDurability(true);
        jobDetailFactoryBean.setGroup("init-group-"+getGroupPrefix()+"0");
        jobDetailFactoryBean.setRequestsRecovery(true);
        jobDetailFactoryBean.setBeanName("quartzScanJobDetail");
        jobDetailFactoryBean.setJobClass(QuartzScanJob.class);
        jobDetailFactoryBean.setDescription("初始化集群任务");
        return  jobDetailFactoryBean;
    }

    /**
     * 重要Bean，务删
     * 定义一个Trigger
     * @return
     */
    @Bean
    public SimpleTriggerFactoryBean scanJobDetailCronTrigger(@Qualifier("quartzScanJobDetail") JobDetail quartzScanJobDetail){
        SimpleTriggerFactoryBean trigger = new SimpleTriggerFactoryBean();
        trigger.setJobDetail(quartzScanJobDetail);
        trigger.setStartTime(new Date());
        trigger.setRepeatCount(0);
        trigger.setStartDelay(500);
        trigger.setRepeatInterval(1);
        return  trigger;
    }
}
```



### 3.2.3 AbstractJob类

com.heima.common.quartz.AbstractJob

```java
/**
 * 所有任务的自动管理抽象类
 */
public abstract class AbstractJob extends QuartzJobBean {

    /**
     * 执行完成后从数据库中删除
     * @return
     */
    public boolean isComplateAfterDelete(){return true;}

    /**
     * 是否启动自动尝试恢复
     * @return
     */
    public boolean isStartAutoRecovery(){return true;}

    /**
     * JOB名称
     * @return
     */
    public String name(){return this.getClass().getName();}

    /**
     * JOB分组
     * @return
     */
    public String group(){return "default";}

    /**
     * JOB描述
     * @return
     */
    public String descJob(){return "";}

    /**
     * Trigger描述
     * @return
     */
    public String descTrigger(){return "";}

    /**
     * 初始化参数
     * @return
     */
    public Map<String,?> initParam(){return Maps.newHashMap();}

    /**
     * 是否自动覆盖
     */
    public boolean isAutoOverwrite(){return true;}

    /**
     * 返回调度策略表达式,可以多个
     * @return
     */
    public abstract String[] triggerCron();

    /**
     * 如果是@DisallowConcurrentExecution，是否继承上次任务执行的结果
     * 该方法未做实现
     * @return
     */
    @Deprecated
    public boolean isExtendPreviouData(){return false;}

}
```

### 3.2.4 Quartz支持分布式的表创建

Quartz集群是通过数据库表来感知其他节点的存在的,各个节点之间没有直接的通信,只有使用持久化的JobStore才能完成Quartz集群,而且每个节点都必须分享同一个数据库;

与quartz持久化相关的表现在有11张表

![1572261939577](img\1572261939577.png)

创建的表的sql  

```sql
DROP TABLE IF EXISTS QRTZ_FIRED_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_PAUSED_TRIGGER_GRPS;
DROP TABLE IF EXISTS QRTZ_SCHEDULER_STATE;
DROP TABLE IF EXISTS QRTZ_LOCKS;
DROP TABLE IF EXISTS QRTZ_SIMPLE_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_SIMPROP_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_CRON_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_BLOB_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_JOB_DETAILS;
DROP TABLE IF EXISTS QRTZ_CALENDARS;

CREATE TABLE QRTZ_JOB_DETAILS
(
SCHED_NAME VARCHAR(120) NOT NULL,
JOB_NAME  VARCHAR(200) NOT NULL,
JOB_GROUP VARCHAR(200) NOT NULL,
DESCRIPTION VARCHAR(250) NULL,
JOB_CLASS_NAME   VARCHAR(250) NOT NULL,
IS_DURABLE VARCHAR(1) NOT NULL,
IS_NONCONCURRENT VARCHAR(1) NOT NULL,
IS_UPDATE_DATA VARCHAR(1) NOT NULL,
REQUESTS_RECOVERY VARCHAR(1) NOT NULL,
JOB_DATA BLOB NULL,
PRIMARY KEY (SCHED_NAME,JOB_NAME,JOB_GROUP)
);

CREATE TABLE QRTZ_TRIGGERS
(
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(200) NOT NULL,
TRIGGER_GROUP VARCHAR(200) NOT NULL,
JOB_NAME  VARCHAR(200) NOT NULL,
JOB_GROUP VARCHAR(200) NOT NULL,
DESCRIPTION VARCHAR(250) NULL,
NEXT_FIRE_TIME BIGINT(13) NULL,
PREV_FIRE_TIME BIGINT(13) NULL,
PRIORITY INTEGER NULL,
TRIGGER_STATE VARCHAR(16) NOT NULL,
TRIGGER_TYPE VARCHAR(8) NOT NULL,
START_TIME BIGINT(13) NOT NULL,
END_TIME BIGINT(13) NULL,
CALENDAR_NAME VARCHAR(200) NULL,
MISFIRE_INSTR SMALLINT(2) NULL,
JOB_DATA BLOB NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,JOB_NAME,JOB_GROUP)
REFERENCES QRTZ_JOB_DETAILS(SCHED_NAME,JOB_NAME,JOB_GROUP)
);

CREATE TABLE QRTZ_SIMPLE_TRIGGERS
(
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(200) NOT NULL,
TRIGGER_GROUP VARCHAR(200) NOT NULL,
REPEAT_COUNT BIGINT(7) NOT NULL,
REPEAT_INTERVAL BIGINT(12) NOT NULL,
TIMES_TRIGGERED BIGINT(10) NOT NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
);

CREATE TABLE QRTZ_CRON_TRIGGERS
(
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(200) NOT NULL,
TRIGGER_GROUP VARCHAR(200) NOT NULL,
CRON_EXPRESSION VARCHAR(200) NOT NULL,
TIME_ZONE_ID VARCHAR(80),
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
);

CREATE TABLE QRTZ_SIMPROP_TRIGGERS
(
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(200) NOT NULL,
TRIGGER_GROUP VARCHAR(200) NOT NULL,
STR_PROP_1 VARCHAR(512) NULL,
STR_PROP_2 VARCHAR(512) NULL,
STR_PROP_3 VARCHAR(512) NULL,
INT_PROP_1 INT NULL,
INT_PROP_2 INT NULL,
LONG_PROP_1 BIGINT NULL,
LONG_PROP_2 BIGINT NULL,
DEC_PROP_1 NUMERIC(13,4) NULL,
DEC_PROP_2 NUMERIC(13,4) NULL,
BOOL_PROP_1 VARCHAR(1) NULL,
BOOL_PROP_2 VARCHAR(1) NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
);

CREATE TABLE QRTZ_BLOB_TRIGGERS
(
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(200) NOT NULL,
TRIGGER_GROUP VARCHAR(200) NOT NULL,
BLOB_DATA BLOB NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
);

CREATE TABLE QRTZ_CALENDARS
(
SCHED_NAME VARCHAR(120) NOT NULL,
CALENDAR_NAME  VARCHAR(200) NOT NULL,
CALENDAR BLOB NOT NULL,
PRIMARY KEY (SCHED_NAME,CALENDAR_NAME)
);

CREATE TABLE QRTZ_PAUSED_TRIGGER_GRPS
(
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_GROUP  VARCHAR(200) NOT NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_GROUP)
);

CREATE TABLE QRTZ_FIRED_TRIGGERS
(
SCHED_NAME VARCHAR(120) NOT NULL,
ENTRY_ID VARCHAR(95) NOT NULL,
TRIGGER_NAME VARCHAR(200) NOT NULL,
TRIGGER_GROUP VARCHAR(200) NOT NULL,
INSTANCE_NAME VARCHAR(200) NOT NULL,
FIRED_TIME BIGINT(13) NOT NULL,
SCHED_TIME BIGINT(13) NOT NULL,
PRIORITY INTEGER NOT NULL,
STATE VARCHAR(16) NOT NULL,
JOB_NAME VARCHAR(200) NULL,
JOB_GROUP VARCHAR(200) NULL,
IS_NONCONCURRENT VARCHAR(1) NULL,
REQUESTS_RECOVERY VARCHAR(1) NULL,
PRIMARY KEY (SCHED_NAME,ENTRY_ID)
);

CREATE TABLE QRTZ_SCHEDULER_STATE
(
SCHED_NAME VARCHAR(120) NOT NULL,
INSTANCE_NAME VARCHAR(200) NOT NULL,
LAST_CHECKIN_TIME BIGINT(13) NOT NULL,
CHECKIN_INTERVAL BIGINT(13) NOT NULL,
PRIMARY KEY (SCHED_NAME,INSTANCE_NAME)
);

CREATE TABLE QRTZ_LOCKS
(
SCHED_NAME VARCHAR(120) NOT NULL,
LOCK_NAME  VARCHAR(40) NOT NULL,
PRIMARY KEY (SCHED_NAME,LOCK_NAME)
);

commit;
```

在黑马头条项目中根据业务的不同，可以区分不同的表，主要是前缀不同

![1572264025808](img\1572264025808.png)



## 3.3 定时任务类

### 3.3.1 CrawlerForwardQuartz 类

> 正向抓取定时任务类

com.heima.crawler.job.CrawlerForwardQuartz

```java
@Component
@DisallowConcurrentExecution
@Log4j2
/**
 * 正向抓取定时任务
 * 一个小时抓取一次
 */
public class CrawlerForwardQuartz extends AbstractJob {

    @Autowired
    private ProcessingFlowManager processingFlowManager;

    @Override
    public String[] triggerCron() {
        /**
         * 2019/8/9 10:00:00
         * 2019/8/9 12:00:00
         * 2019/8/9 14:00:00
         * 2019/8/9 16:00:00
         * 2019/8/9 18:00:00
         */
        // return new String[]{"0 0 0-23/2 * * ?"};
        return new String[]{"0 0/10 * * * ?"};
    }

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        long cutrrentTime = System.currentTimeMillis();
        log.info("开始正向抓取");
        processingFlowManager.handel();
        log.info("正向抓取结束,耗时：", System.currentTimeMillis() - cutrrentTime);
    }
}

```

### 3.3.2 CrawlerReverseQuartz

> 反向 抓取定时任务类

com.heima.crawler.job.CrawlerReverseQuartz

```java
/**
 * 逆向抓取
 */
@Component
@DisallowConcurrentExecution
@Log4j2
public class CrawlerReverseQuartz extends AbstractJob {

    @Autowired
    private ProcessingFlowManager processingFlowManager;
    @Autowired
    private CrawlerNewsAdditionalService crawlerNewsAdditionalService;

    @Override
    public String[] triggerCron() {
        /**
         * 2019/8/9 11:00:00
         * 2019/8/9 13:00:00
         * 2019/8/9 15:00:00
         * 2019/8/9 17:00:00
         * 2019/8/9 19:00:00
         */
        return new String[]{"0 0 0/1 * * ?"};
    }

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        long cutrrentTime = System.currentTimeMillis();
        log.info("开始反向抓取");
        processingFlowManager.reverseHandel();
        log.info("反向抓取结束,耗时：", System.currentTimeMillis() - cutrrentTime);

    }
}
```

### 3.3.3 ProxyIpUpdateQuartz

> 代理IP更新定时任务

com.heima.crawler.job.ProxyIpUpdateQuartz

```java
/**
 * 代理Ip定时管理类
 */
@Component
@DisallowConcurrentExecution
@Log4j2
/**
 * 代理IP 更新定时任务
 */
public class ProxyIpUpdateQuartz extends AbstractJob {

    @Autowired
    private ProxyIpManager proxyIpManager;

    @Autowired
    private CrawlerProxyProvider crawlerProxyProvider;


    @Override
    public String[] triggerCron() {
        /**
         * 2019/8/9 10:30:00
         * 2019/8/9 11:00:00
         * 2019/8/9 11:30:00
         * 2019/8/9 12:00:00
         * 2019/8/9 12:30:00
         */
        return new String[]{"0 0/30 * * * ?"};
    }

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        long cutrrentTime = System.currentTimeMillis();
        log.info("开始更新代理IP");
        proxyIpManager.updateProxyIp();
        crawlerProxyProvider.updateProxy();
        log.info("更新代理IP完成，耗时：{}",System.currentTimeMillis()-cutrrentTime);
    }
}
```

### 3.3.4 ProxyIpValidateQuartz

> 代理IP验证定时任务

com.heima.crawler.job.ProxyIpValidateQuartz

```java
/**
 * 代理Ip定时管理类
 */
@Component
@DisallowConcurrentExecution
@Log4j2
public class ProxyIpValidateQuartz extends AbstractJob {

    @Override
    public String[] triggerCron() {
        return new String[]{"0 0/30 * * * ?"};
    }

    @Autowired
    private ProxyIpManager proxyIpManager;

    @Autowired
    private CrawlerProxyProvider crawlerProxyProvider;

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        long currentTimeMillis = System.currentTimeMillis();
        log.info("开始检验代理ip");
        proxyIpManager.validateProxyIp();
        crawlerProxyProvider.updateProxy();
        log.info("检验代理ip完成，耗时:{}",System.currentTimeMillis()-currentTimeMillis);
    }
}

```

# 4 爬虫文章自动审核

## 4.1 思路分析

查看资料文件夹中：爬虫文章审核流程.pdf

## 4.2 实体类

创建类：com.heima.model.article.pojos.ApArticleLabel

```java
@Setter
@Getter
public class ApArticleLabel {

    public ApArticleLabel(Integer articleId, Integer labelId) {
        this.articleId = articleId;
        this.labelId = labelId;
    }

    public ApArticleLabel() {
    }

    private Integer id;

    private Integer articleId;

    private Integer labelId;

    private Integer count;
}
```

## 4.3 mapper定义

创建mapper接口：com.heima.model.mappers.app.ApArticleLabelMapper

```java
public interface ApArticleLabelMapper {
    
    int insert(ApArticleLabel record);

    int insertSelective(ApArticleLabel record);

    int updateByPrimaryKeySelective(ApArticleLabel record);

    List<ApArticleLabel> selectList(ApArticleLabel apArticleLabel);
}
```

ApArticleLabelMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.heima.model.mappers.app.ApArticleLabelMapper">
    <resultMap id="BaseResultMap" type="com.heima.model.article.pojos.ApArticleLabel">

        <id column="id" property="id"/>
        <result column="article_id" property="articleId"/>
        <result column="label_id" property="labelId"/>
        <result column="count" property="count"/>

    </resultMap>
    <sql id="Base_Column_List">
        <!--
          WARNING - @mbggenerated
          This element is automatically generated by MyBatis Generator, do not modify.
        -->
        id, article_id, label_id, count
    </sql>
    <sql id="Base_Column_Where">

        <where>
            <if test="id!=null and id!=''">
                and id=#{id}
            </if>
            <if test="articleId!=null and articleId!=''">
                and article_id=#{articleId}
            </if>
            <if test="labelId!=null and labelId!=''">
                and label_id=#{labelId}
            </if>
            <if test="count!=null and count!=''">
                and count=#{count}
            </if>
        </where>
    </sql>

    <select id="selectList" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List"/>
        from ap_article_label
        <include refid="Base_Column_Where"/>
    </select>
    
    <insert id="insert" parameterType="com.heima.model.article.pojos.ApArticleLabel">
        <!--
          WARNING - @mbggenerated
          This element is automatically generated by MyBatis Generator, do not modify.
        -->
        insert into ap_article_label (id, article_id, label_id,
        count)
        values (#{id,jdbcType=INTEGER}, #{articleId,jdbcType=INTEGER}, #{labelId,jdbcType=INTEGER},
        #{count,jdbcType=INTEGER})
    </insert>
    <insert id="insertSelective" parameterType="com.heima.model.article.pojos.ApArticleLabel">
        <!--
          WARNING - @mbggenerated
          This element is automatically generated by MyBatis Generator, do not modify.
        -->
        insert into ap_article_label
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="id != null">
                id,
            </if>
            <if test="articleId != null">
                article_id,
            </if>
            <if test="labelId != null">
                label_id,
            </if>
            <if test="count != null">
                count,
            </if>
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="id != null">
                #{id,jdbcType=INTEGER},
            </if>
            <if test="articleId != null">
                #{articleId,jdbcType=INTEGER},
            </if>
            <if test="labelId != null">
                #{labelId,jdbcType=INTEGER},
            </if>
            <if test="count != null">
                #{count,jdbcType=INTEGER},
            </if>
        </trim>
    </insert>
    <update id="updateByPrimaryKeySelective"
            parameterType="com.heima.model.article.pojos.ApArticleLabel">
        <!--
          WARNING - @mbggenerated
          This element is automatically generated by MyBatis Generator, do not modify.
        -->
        update ap_article_label
        <set>
            <if test="articleId != null">
                article_id = #{articleId,jdbcType=INTEGER},
            </if>
            <if test="labelId != null">
                label_id = #{labelId,jdbcType=INTEGER},
            </if>
            <if test="count != null">
                count = #{count,jdbcType=INTEGER},
            </if>
        </set>
        where id = #{id,jdbcType=INTEGER}
    </update>
</mapper>
```

## 4.5 service代码编写

创建常量

com.heima.common.admin.constans.AdminConstans

```java
public class AdminConstans {
    public static final String ES_INDEX_TAG_ARTICLE="article";
}
```

在admin端创建接口com.heima.admin.service.ReviewCrawlerArticleService

```java
public interface ReviewCrawlerArticleService {
    /**
     * 爬虫端发布文章审核
     */
    public void autoReviewArticleByCrawler(ClNews clNews) throws Exception;

    public void autoReviewArticleByCrawler() throws Exception;

    public void autoReviewArticleByCrawler(Integer clNewsId) throws Exception;
}
```

实现类com.heima.admin.service.impl.ReviewCrawlerArticleServiceImpl

```java
@Service
@Log4j2
@SuppressWarnings("all")
public class ReviewCrawlerArticleServiceImpl implements ReviewCrawlerArticleService {

    @Autowired
    private AliyunTextScanRequest aliyunTextScanRequest;

    @Autowired
    private AliyunImageScanRequest aliyunImageScanRequest;

    @Autowired
    private JestClient jestClient;

    @Autowired
    private ApArticleConfigMapper apArticleConfigMapper;

    @Autowired
    private ApArticleMapper apArticleMapper;

    @Autowired
    private ApArticleContentMapper apArticleContentMapper;

    @Autowired
    private ApAuthorMapper apAuthorMapper;

    @Autowired
    private AdChannelMapper adChannelMapper;

    @Autowired
    private ClNewsMapper clNewsMapper;

    @Autowired
    private ApArticleLabelMapper apArticleLabelMapper;

    @Override
    public void autoReviewArticleByCrawler() throws Exception {
        ClNews param = new ClNews();
        param.setStatus((byte) 1);
        List<ClNews> clNewsList = clNewsMapper.selectList(param);

        if (null != clNewsList && !clNewsList.isEmpty()) {
            log.info("定时任务自动审核检索未审核数量：{}", clNewsList.size());
            for (ClNews clNews : clNewsList) {
                autoReviewArticleByCrawler(clNews);
            }
        } else {
            log.info("定时任务自动审核未检索出数据");
        }
    }

    @Override
    public void autoReviewArticleByCrawler(Integer clNewsId) throws Exception {
        //1.查询待审核的列表
        ClNews param = new ClNews();
        param.setId(clNewsId);
        param.setStatus((byte) 1);
        ClNews clNews = clNewsMapper.selectByIdAndStatus(param);
        if (null != clNews) {
            autoReviewArticleByCrawler(clNews);
        }
    }

    @Override
    public void autoReviewArticleByCrawler(ClNews clNews) throws Exception {
        long currentTime = System.currentTimeMillis();
        log.info("开始自动审核流程");
        if (clNews != null) {
            log.info("开始内容审核流程");
            String content = clNews.getUnCompressContent();
            String title = clNews.getTitle();
            //2.检查文章标题与内容的匹配度
            if (content == null || title == null) {
                updateClNews(clNews, "文章内容或标题为空");
                return;
            }
            //设置固定长度
            title = HMStringUtils.getFixedLengthStr(title, 50);
            double degree = Compute.SimilarDegree(content, title);
            if (degree <= 0) {
                //文章标题与内容匹配
                updateClNews(clNews, "文章标题与内容不匹配");
                return;
            }
            log.info("开始文本审核流程");
            //3.审核文本
            List<String> images = new ArrayList<>();
            StringBuilder sb = new StringBuilder();
            JSONArray jsonArray = JSON.parseArray(content);
            handlerTextAndImages(images, sb, jsonArray);
            //截取之前五张图片
            images = HMStringUtils.getFixedLengthContentList(images, 1000, 2);
            String response = aliyunTextScanRequest.textScanRequest(sb.toString());
            if ("review".equals(response)) {//人工审核
                return;
            }
            if ("block".equals(response)) {//审核失败
                updateClNews(clNews, "文本内容审核失败");
                return;
            }
            //审核文章中的图片信息，阿里接口
            String imagesResponse = aliyunImageScanRequest.imageScanRequest(images);
            if (imagesResponse != null) {
                if ("review".equals(imagesResponse)) {//人工审核
                    return;
                }
                if ("block".equals(imagesResponse)) {//审核失败
                    updateClNews(clNews, "文本内容审核失败");
                    return;
                }
            } else {
                updateClNews(clNews, "图片审核出现问题");
                return;
            }

            //5.审核通过 存入数据   ap_article_config   ap_article   ap_article_content  ap_author
            Integer channelId = clNews.getChannelId();
            String channelName = "";
            if (null != channelId) {
                AdChannel adChannel = adChannelMapper.selectByPrimaryKey(channelId);
                if (adChannel != null) {
                    channelName = adChannel.getName();
                }
            }

            int type = clNews.getType();//布局类型 0 无图 1 单图  2 多图
            log.info("文章作者入库");
            //查询是否已经存在作者信息，如果不存在则插入一条新数据
            ApAuthor apAuthor = saveApAuthor(clNews);

            //文章信息表，存储已发布的文章
            log.info("保存文章数据");
            ApArticle apArticle = saveApArticleByCrawler(images, channelId, channelName, apAuthor.getId(), clNews);
            //保存标签
            saveApArticleLabel(apArticle);
            //APP已发布文章配置表
            ApArticleConfig apArticleConfig = saveApArticleConfigByCrawler(apArticle);
            //APP已发布文章内容表
            saveApArticleContentByCrawler(clNews.getContent(), apArticle);

            log.info("开始创建索引");
            //6.创建索引
            try {
                createEsIndex(apArticle, content, title, channelId);
            } catch (Exception e) {
                e.printStackTrace();
            }
            //修改状态为审核通过待发布
            log.info("更新原始文章状态为待发布");
            updateClNewsSuccess(clNews);
        }
        log.info("审核流程结束，耗时：{}", System.currentTimeMillis() - currentTime);
    }

    private ApAuthor saveApAuthor(ClNews clNews) {
        ApAuthor apAuthor = apAuthorMapper.selectByAuthorName(clNews.getName());
        if (apAuthor == null || apAuthor.getId() == null) {
            apAuthor = new ApAuthor();
            apAuthor.setCreatedTime(clNews.getCreatedTime());
            apAuthor.setName(clNews.getName());
            apAuthor.setType(2);
            apAuthorMapper.insert(apAuthor);
        }
        return apAuthor;
    }


    private void createEsIndex(ApArticle apArticle, String content, String title, Integer channelId) throws IOException {
        EsIndexEntity esIndexEntity = saveEsIndexEntityByCrawler(content, title, channelId, apArticle);
        Index.Builder builder = new Index.Builder(esIndexEntity);
        builder.id(apArticle.getId().toString());
        builder.refresh(true);
        Index index = builder.index(ESIndexConstants.ARTICLE_INDEX).type(ESIndexConstants.DEFAULT_DOC).build();
        JestResult result = jestClient.execute(index);
        if (result != null && !result.isSucceeded()) {
            throw new RuntimeException(result.getErrorMessage() + "插入更新索引失败!");
        }
    }

    /**
     * 创建索引信息 crawler
     *
     * @param content
     * @param title
     * @param channelId
     * @param apArticle
     * @return
     */
    private EsIndexEntity saveEsIndexEntityByCrawler(String content, String title, Integer channelId, ApArticle apArticle) {
        EsIndexEntity esIndexEntity = new EsIndexEntity();
        esIndexEntity.setId(new Long(apArticle.getId()));
        if (null != channelId) {
            esIndexEntity.setChannelId(new Long(channelId));
        }
        esIndexEntity.setContent(content);
        esIndexEntity.setPublishTime(new Date());
        esIndexEntity.setStatus(new Long(1));
        esIndexEntity.setTitle(title);
        esIndexEntity.setTag(AdminConstans.ES_INDEX_TAG_ARTICLE);
        return esIndexEntity;
    }

    /**
     * 保存文章作者信息  crawler
     *
     * @param createdTime
     * @param authorName
     * @return
     */
    private ApAuthor saveApAuthorByCrawler(Date createdTime, String authorName) {
        ApAuthor apAuthor = new ApAuthor();
        apAuthor.setCreatedTime(createdTime);
        apAuthor.setName(authorName);
        apAuthor.setType(2);
        apAuthorMapper.insert(apAuthor);
        return apAuthor;
    }

    /**
     * 保存ApArticleContent  爬虫系统
     *
     * @param content
     * @param apArticle
     * @return
     */
    private void saveApArticleContentByCrawler(String content, ApArticle apArticle) {
        ApArticleContent apArticleContent = new ApArticleContent();
        apArticleContent.setArticleId(apArticle.getId());
        apArticleContent.setContent(content);
        apArticleContentMapper.insert(apArticleContent);
    }

    /**
     * 保存ApArticleConfig  crawler 爬虫系统
     *
     * @param apArticle
     * @return
     */
    private ApArticleConfig saveApArticleConfigByCrawler(ApArticle apArticle) {
        ApArticleConfig apArticleConfig = new ApArticleConfig();
        apArticleConfig.setArticleId(apArticle.getId());
        apArticleConfig.setIsComment(true);
        apArticleConfig.setIsDelete(false);
        apArticleConfig.setIsDown(false);
        apArticleConfig.setIsForward(true);
        apArticleConfigMapper.insert(apArticleConfig);
        return apArticleConfig;
    }

    /**
     * 保存文章信息  爬虫系统
     *
     * @param title
     * @param images
     * @param channelId
     * @param channelName
     * @param createdTime
     * @param publishTime
     * @param authorName
     * @return
     */
    private ApArticle saveApArticleByCrawler(List<String> images, Integer channelId, String channelName, Integer authorId, ClNews clNews) {
        ApArticle apArticle = new ApArticle();
        apArticle.setChannelId(channelId);
        apArticle.setChannelName(channelName);
        apArticle.setAuthorName(clNews.getName());
        apArticle.setCreatedTime(clNews.getCreatedTime());
        apArticle.setOrigin(false);
        StringBuilder sb = new StringBuilder();
        Short layout = 0;
        for (int i = 0; i < images.size() && i < 3; i++) {
            if (i != 0) {
                sb.append(",");
            }
            layout++;
            sb.append(images.get(i));
        }
        apArticle.setImages(sb.toString());
        apArticle.setLabels(clNews.getLabels());
        apArticle.setTitle(clNews.getTitle());
        apArticle.setPublishTime(new Date());
        apArticle.setAuthorId(new Long(authorId));
        apArticle.setLayout(layout);
        long currentTime = System.currentTimeMillis();
        log.info("开始插入ApArticle表");
        apArticleMapper.insert(apArticle);
        log.info("插入ApArticle表完成,耗时：{}", System.currentTimeMillis() - currentTime);
        return apArticle;
    }

    /**
     * 保存标签
     *
     * @param apArticle
     */
    private void saveApArticleLabel(ApArticle apArticle) {
        if (null != apArticle && StringUtils.isNotEmpty(apArticle.getLabels())) {
            String[] labelIdArray = apArticle.getLabels().split(",");
            for (String labelId : labelIdArray) {
                ApArticleLabel tmp = new ApArticleLabel(apArticle.getId(), Integer.parseInt(labelId));
                List<ApArticleLabel> apArticleLabelList = apArticleLabelMapper.selectList(tmp);
                if (null != apArticleLabelList && !apArticleLabelList.isEmpty()) {
                    ApArticleLabel apArticleLabel = apArticleLabelList.get(0);
                    apArticleLabel.setCount(apArticleLabel.getCount() + 1);
                    apArticleLabelMapper.updateByPrimaryKeySelective(apArticleLabel);
                } else {
                    tmp.setCount(1);
                    apArticleLabelMapper.insertSelective(tmp);
                }
            }
        }
    }

    /**
     * 文章审核失败，更新原有库状态及告知原因
     *
     * @param clNews
     * @param reason
     */
    private void updateClNews(ClNews clNews, String reason) {
        clNews.setStatus((byte) 2);
        clNews.setReason(reason);
        clNewsMapper.updateStatus(clNews);
    }

    /**
     * 文章审核成功
     * 因为mq 不能保证实时，可能存在重复读取的问题，所以优先修改状态
     *
     * @param clNews
     * @param reason
     */
    private void updateClNewsSuccess(ClNews clNews) {
        clNews.setStatus((byte) 9);
        clNewsMapper.updateStatus(clNews);
    }

    /**
     * 处理content  找出文本和图片列表
     *
     * @param images
     * @param sb
     * @param jsonArray
     */
    private void handlerTextAndImages(List<String> images, StringBuilder sb, JSONArray jsonArray) {
        for (Object obj : jsonArray) {
            JSONObject jsonObj = (JSONObject) obj;
            String type = (String) jsonObj.get("type");
            if ("image".equals(type)) {
                String value = (String) jsonObj.get("value");
                images.add(value);
            }
            if ("text".equals(type)) {
                sb.append(jsonObj.get("value"));
            }
        }
    }
}
```

## 4.6 消息接收

### 4.6.1 接收消息审核文章

修改：com.heima.admin.kafka.AutoReviewArticleListener

```java
@Component
@Log4j2
public class AutoReviewArticleListener implements KafkaListener<String,String> {

    @Autowired
    private KafkaTopicConfig kafkaTopicConfig;

    @Autowired
    private ObjectMapper mapper;

    @Autowired
    private ReviewMediaArticleService reviewMediaArticleService;
    
    @Autowired
    private ReviewCrawlerArticleService reviewCrawlerArticleService;

    @Override
    public String topic() {
        return kafkaTopicConfig.getSubmitArticleAuth();
    }

    @Override
    public void onMessage(ConsumerRecord<String, String> consumerRecord, Consumer<?, ?> consumer) {
        String value = consumerRecord.value();
        log.info("接收到的消息为：{}"+value);
        try {
            SubmitArticleAuthMessage message = mapper.readValue(value, SubmitArticleAuthMessage.class);
            if(message!=null){
                SubmitArticleAuto.ArticleType type = message.getData().getType();
                if(type==SubmitArticleAuto.ArticleType.WEMEDIA){
                    Integer articleId = message.getData().getArticleId();
                    if(articleId!=null){
                        //审核文章信息
                        reviewMediaArticleService.autoReviewArticleByMedia(articleId);
                    }
                }else if(type==SubmitArticleAuto.ArticleType.CRAWLER){
                    Integer articleId = message.getData().getArticleId();
                    if(articleId!=null){
                        //审核文章信息
                        try {
                            reviewCrawlerArticleService.autoReviewArticleByCrawler(articleId);
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
            log.error("处理自动审核文章错误:[{}],{}",value,e);
            throw new RuntimeException("WS消息处理错误",e);
        }
    }
}
```



## 4.7 定时任务扫描爬虫未审核文章信息

让admin项目项目引入quartz，**必须放置在mysql之后加载**

![1572265706295](img\1572265706295.png)

创建定时任务类：com.heima.admin.job.CrawlerReviewArticleJob

```java
@Component
@DisallowConcurrentExecution
@Log4j2
public class CrawlerReviewArticleJob extends AbstractJob {

    @Autowired
    private ReviewCrawlerArticleService reviewCrawlerArticleService;

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        long currentTime = System.currentTimeMillis();
        log.info("开始执行自动审核定时任务");
        try {
            reviewCrawlerArticleService.autoReviewArticleByCrawler();
        } catch (Exception e) {
            e.printStackTrace();
        }
        log.info("自动审核定时任务执行完成,耗时：{}", System.currentTimeMillis() - currentTime);
    }

    @Override
    public String[] triggerCron() {
        return new String[]{"0 0/5 * * * ?"};
    }

    @Override
    public String descTrigger() {
        return "每天晚上23:30执行";
    }
}
```

## 4.8 测试

修改com.heima.admin.service.test.ReviewArticleTest，添加测试方法

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class ReviewArticleTest {

    @Autowired
    private ReviewCrawlerArticleService reviewCrawlerArticleService;

    @Test
    public void testReviewCraeler() throws Exception {
        reviewCrawlerArticleService.autoReivewArticleByCrawler(35181);
    }
}
```

