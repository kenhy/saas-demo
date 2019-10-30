# saas-demo

多租户实现思路：

Spring Boot实现SAAS平台的基本思路
　一、SAAS是什么

　SaaS是Software-as-a-service（软件即服务）它是一种通过Internet提供软件的模式，厂商将应用软件统一部署在自己的服务器

   上，客户可以根据自己实际需求，通过互联网向厂商定购所需的应用软件服务，按定购的服务多少和时间长短向厂商支付费用，

　并通过互联网获得厂商提供的服务。用户不用再购买软件，而改用向提供商租用基于Web的软件，来管理企业经营活动，且无需

　对软件进行维护，服务提供商会全权管理和维护软件。

　二、SAAS模式有哪些角色

　①服务商：服务商主要是管理租户信息，按照不同的平台需求可能还需要统合整个平台的数据，作为大数据的基础。服务商在SAAS

　模式中是提供服务的厂商。

　②租户：租户就是购买/租用服务商提供服务的用户，租户购买服务后可以享受相应的产品服务。现在很多SAAS化的产品都会划分

　系统版本，不同的版本开放不同的功能，还有基于功能收费之类的，不同的租户购买不同版本的系统后享受的服务也不一样。

   三、SAAS模式有哪些特点

　①独立性：每个租户的系统相互独立。

　②平台性：所有租户归平台统一管理。

　③隔离性：每个租户的数据相互隔离。

   在以上三个特性里面，SAAS系统中最重要的一个标志就是数据隔离性，租户间的数据完全独立隔离。

   四、数据隔离有哪些方案

   ①独立数据库

   即一个租户一个数据库，这种方案的用户数据隔离级别最高，安全性最好，但成本较高。

   优点：

   为不同的租户提供独立的数据库，有助于简化数据模型的扩展设计，满足不同租户的独特需求，如果出现故障，恢复数据比较简单。

   缺点：

   增多了数据库的安装数量，随之带来维护成本和购置成本的增加。 如果定价较低，产品走低价路线，这种方案一般对运营商来说是无法承受的。

   ②共享数据库，隔离数据架构

   即多个或所有租户共享数据库，但是每个租户一个Schema。

   优点：

   为安全性要求较高的租户提供了一定程度的逻辑数据隔离，并不是完全隔离，每个数据库可支持更多的租户数量。

   缺点：

   如果出现故障，数据恢复比较困难，因为恢复数据库将牵涉到其他租户的数据 如果需要跨租户统计数据，存在一定困难。

   ③共享数据库，共享数据架构

   即租户共享同一个数据库、同一个Schema，但在表中增加TenantID多租户的数据字段。这是共享程度最高、隔离级别最低的模式。 

   优点：

   三种方案比较，第三种方案的维护和购置成本最低，允许每个数据库支持的租户数量最多。

   缺点：

   隔离级别最低，安全性最低，需要在设计开发时加大对安全的开发量，数据备份和恢复最困难，需要逐表逐条备份和还原。

   如果希望以最少的服务器为最多的租户提供服务，并且租户接受牺牲隔离级别换取降低成本，这种方案最适合。

　五、基于spring boot 、spring-data-jpa实现共享数据库，隔离数据架构的SAAS系统。

　　　在实现系统之前我们需要明白这套实现是共享数据库，隔离数据架构的，在上面三个方案里面的第二种，为什么选择第二种。

　第一种基本上只有对数据的隔离性要求非常高，并且有烧钱买服务器的觉悟才能搞。第三种对数据的隔离性太差，只要在程序实现

　上出现些问题就可能导致数据混乱的问题，并且数据备份还原的代价非常高。所以折中我们选择第二种。

　　　首先在SAAS系统中，一般都是一套系统多个租户，也就是说所有的租户共享同一套系统，但是每个租户看的数据又要不一样。

　确定了数据隔离级别之后，我们就需要明确SAAS系统在实现上的难点：①动态创建数据库；②动态切换数据库；我们都知道传统的

　系统中数据源的信息一般都是写死在系统配置文件的，在启动系统的时候加载配置信息创建数据源，这样的系统是单数据源的。这明显不适用

　SAAS系统，SAAS系统是有多少个租户就需要多少个数据源的，并且会根据租户的信息动态的切换数据源。

　技术准备：spring boot ， spring-data-jpa ， redis，消息队列， mysql，maven等。

　工具准备：IDEA，PostMan

　项目结构：这里准备了两套系统，平台管理端和租户端，这两套系统是独立存在的可以单独运行。

　

　在demo里面，管理端（saas-admin）创建的是一个独立的spring boot项目，这里只是实现了租户的注册，及通过消息队列通知租户端创建数据库。

　首先在saas-admin系统的POM.xml里面添加依赖

复制代码
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j</artifactId>
            <version>1.3.8.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.31</version>
        </dependency>

        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.6</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
复制代码
 配置spring boot的全局配置文件application.properties，需要注意spring.jpa.properties.hibernate.hbm2ddl.auto=update属性，首次启动需要先创建saas_admin数据库，不需要建表。

复制代码
server.port= 8080
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8

# Database
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/saas_admin?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&failOverReadOnly=false
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.hbm2ddl.auto=update

# Session
spring.session.store-type=none

# Redis
spring.redis.database=0
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=
spring.redis.pool.max-active=8
spring.redis.pool.max-wait=-1
spring.redis.pool.max-idle=8
spring.redis.pool.min-idle=0
spring.redis.timeout=1200
复制代码
准备租户实体类，这里使用了spring-data-jpa技术，类似hibernate的用法，使用过hibernate的应该很容易看懂。

复制代码
@Entity
@Table(name = "tenant")
public class Tenant implements Serializable {

    @Id
    @Column(name = "id",length = 32)
    private String id;

    @Column(name = "account",length = 30)
    private String account;

    @Column(name = "token",length = 32)
    private String token;

    @Column(name = "url",length = 125)
    private String url;

    @Column(name = "data_base",length = 30)
    private String database;

    @Column(name = "username",length = 30)
    private String username;

    @Column(name = "password",length = 32)
    private String password;

    @Column(name = "domain_name",length = 64)
    private String domainName;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getDatabase() {
        return database;
    }

    public void setDatabase(String database) {
        this.database = database;
    }

    public String getDomainName() {
        return domainName;
    }

    public void setDomainName(String domainName) {
        this.domainName = domainName;
    }

    public String getAccount() {
        return account;
    }

    public void setAccount(String account) {
        this.account = account;
    }

    public String getToken() {
        return token;
    }

    public void setToken(String token) {
        this.token = token;
    }
复制代码
 接下来直接看注册租户的的实现，其实就是保存租户信息，然后使用redis的消息队列通知租户端创建租户的数据库，redis消息队列的实现代码会放到github上面。

管理端就这样了，在实际的系统的中租户一般也是注册信息到管理端，并且注册信息的时候可以选择使用版本，并且如果系统需要收费的话，也是在支付费用之后才会发送创建

数据库的消息。

下面主要看下租户端，动态创建数据库和切换数据库都是发生在租户端的。

租户端也是一个独立的spring boot项目，可以独立运行部署，使用的技术完全和管理端一样，POM配置完全相同。

复制代码
server.port= 9090
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8

# Database
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/saas_tenant?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&failOverReadOnly=false
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

# Hibernate
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.hbm2ddl.auto=update
spring.jpa.properties.hibernate.multiTenancy=SCHEMA
spring.jpa.properties.hibernate.tenant_identifier_resolver=com.michael.saas.tenant.config.MultiTenantIdentifierResolver
spring.jpa.properties.hibernate.multi_tenant_connection_provider=com.michael.saas.tenant.config.MultiTenantConnectionProviderImpl

# Session
spring.session.store-type=none

# Redis
spring.redis.database=0
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=finance123
spring.redis.pool.max-active=8
spring.redis.pool.max-wait=-1
spring.redis.pool.max-idle=8
spring.redis.pool.min-idle=0
spring.redis.timeout=1200
复制代码
全局配置文件application.properties有几个需要注意的地方。

①spring.jpa.properties.hibernate.multiTenancy=SCHEMA；

这个是hibernate的多租户模式的支持，我们这里配置SCHEMA，表示独立数据库；

②spring.jpa.properties.hibernate.tenant_identifier_resolver；

租户ID解析器，简单来说就是这个设置指定的类负责每次执行sql语句的时候获取租户ID；

③spring.jpa.properties.hibernate.multi_tenant_connection_provider；

这个设置指定的类负责按照租户ID来提供相应的数据源；

其中②和③是需要自己实现的；

租户ID解析器：

复制代码
public class MultiTenantConnectionProviderImpl extends AbstractDataSourceBasedMultiTenantConnectionProviderImpl {

    // 在没有提供tenantId的情况下返回默认数据源
    @Override
    protected DataSource selectAnyDataSource() {
        return TenantDataSourceProvider.getTenantDataSource("Default");
    }

    // 提供了tenantId的话就根据ID来返回数据源
    @Override
    protected DataSource selectDataSource(String tenantIdentifier) {
        return TenantDataSourceProvider.getTenantDataSource(tenantIdentifier);
    }
}
复制代码
切换数据源的操作是通过spring的aop机制实现的，可以看到切换数据源的操作发生在业务层。通过租户的ID获取存储在本地线程中相应数据源完成业务操作。

复制代码
@Aspect
@Order(-1)
@Component
public class TenantAspect {

    @Pointcut("execution(public * com.michael.saas.tenant.service..*.*(..))")
    public void switchTenant(){}

    @Before("switchTenant()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
        System.out.println(SpObserver.getTenantId());
        TenantDataSourceProvider.getTenantDataSource(SpObserver.getTenantId());
    }

    @AfterReturning(returning = "object", pointcut = "switchTenant()")
    public void doAfterReturning(Object object) throws Throwable {

    }

}
复制代码
过滤器获取存放session中的租户ID存放本地线程

复制代码
　　public class BaseFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) servletRequest;
        Object obj = req.getSession().getAttribute("TENANTID");
        if (null != obj){
            SpObserver.putTenantId(obj.toString());
        }
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {

    }
}
复制代码
值得一提，租户ID是登陆时存放到session中的，也代表着切换数据源的前提是先登陆。

复制代码
　　@PostMapping(value = "/login")
    public boolean login(@RequestParam("username")String username, @RequestParam("password")String password, HttpServletRequest request ){
        try {
            Tenant tenant = tenantService.findByAccountAndToken(username, password);
            if (null != tenant){
                request.getSession().setAttribute("TENANTID",tenant.getId());
            }else {
                return false;
            }
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }
复制代码
既然是独立数据库，就避免不了生成独立数据库的问题，我这边是通过jdbc创建新的数据库，不一定是好的实现，但能完成基本要求，创建新数据库的操作在租户注册账号时完成，这里使用了redis消息队列去异步生成数据库。

复制代码
 　　@PostMapping(value = "/register")
    public boolean register(Tenant tenant){
        try {
            tenant.setUrl("jdbc:mysql://127.0.0.1:3306/");
            tenant.setDatabase("saas_tenant_" + tenant.getAccount());
            tenant.setUsername("root");
            tenant.setPassword("root");
            tenant = tenantService.save(tenant);
            queueService.send("register",tenant);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }
复制代码
复制代码
@Component
public class Receiver {
    private static final Logger logger = Logger.getLogger(Receiver.class);

    private CountDownLatch latch;

    @Autowired
    public Receiver(CountDownLatch latch) {
        this.latch = latch;
    }

    @Autowired
    private TenantService tenantService;

    /**
     * 消息处理
     * @param message
     */
    public void objectMessage(String message) {
        QueueTemplate queueTemplate = JSON.parseObject(message,QueueTemplate.class);
        if ("register".equals(queueTemplate.getMethod())){
            Tenant tenant = JSON.parseObject(JSON.toJSONString(queueTemplate.getObject()),Tenant.class);
            try {
                DBUtils.createDataBase(tenant.getUrl(),tenant.getDatabase(),tenant.getUsername(),tenant.getPassword());
                TenantDataSourceProvider.addDataSource(tenant);
                tenantService.save(tenant);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        latch.countDown();
    }


}
复制代码
SaaS模式基本实现就介绍到这里，github地址奉上https://github.com/gm-xiao/saas-demo.git
