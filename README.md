# dynamic-datasource-starter
springboot 动态切换数据的基本思想与实现方法

基本原理: 通过切面根据不同的条件在执行数据库操作前切换数据源

在理想情况下(一个项目应该只有一个业务库)，一个项目有一个可写数据源和多个只读数据源，为了减少数据库的压力，使用轮询的方式选择只读数据库；
考虑到在一个Service中同时会有读和写的操作，所以使用AOP切面通过dao层的方法名切换只读数据源

在使用中，应该避免在同一个Service方法中写入后立即查询，如果一定需要，应当在Service方法上添加@Transactional注解以保证主数据源

##### DynamicRoutingDataSource
    
    该类继承自AbstractRoutingDataSource类，在访问数据库的时会调用该类的 determineCurrentLookupKey() 方法获取数据库的实例的key

##### DataSourceConfigurer
    
    数据源配置类，在该类中生成多个数据源实例并且将其注入到ApplicationContext中
      - 将数据源的 key 放到数据源上下文的 key 集合中，用于切换时判断数据源是否有效
      - 将 Slave数据源的key放在集合中，用于轮循
      
##### DataSourceConfigurer   
  
     DynamicDataSourceAspect，动态数据源切换的切面，切dao层，通过dao的名称来判断使用哪个数据源
     关于切面的Order可以不设，因为 @Transactional是最低的，取决于其他切面的设置，并且在 org.springframework.core.annotation.AnnotationAwareOrderComparator 会重新排序


---

注意
 
    (1) 有@Trasactional注解的方法无法切换数据源   
    
    (2) Consider marking one of the beans as @Primary, updating the consumer to accept multiple beans, or using @Qualifier to identify the bean that should be consumed
        该异常在错误信息中已经说的很清楚了，是因为有多个 DataSource 的实例，所以无法确定该引用那个实例
        
        为数据源的某个Bean添加@Primary注解，该Bean应当是通过DataSourceBuilder.create().build() 得到的 Bean，
        而不是通过 new AbstractRoutingDataSource 的子类实现的 Bean，
        在本项目中可以是 master() 或 slave() 得到的 DataSource，不能是 dynamicDataSource() 得到的 DataSource
        
    (3) @ConfigurationProperties的功能，可以将同类的配置信息自动封装成实体类
        
        @Bean
        @ConfigurationProperties(prefix = "connection")
        public ConnectionSettings connectionSettings(){
           return new ConnectionSettings();
        }
       
