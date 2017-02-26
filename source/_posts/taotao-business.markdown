---
layout:     post
title:      "淘淘电商网"
subtitle:   "淘淘电商网项目详细说明"
date:       2016-10-14
author:     "XuZheng"
header-img: "/img/content-bg1.jpg"
tags:
    - Java
    - 后端
    - Redis
    - Nginx
    - Solr
---

> 一款类综合性的B2C平台，类似于京东商城、天猫商城。

### 项目简介
淘淘网上商城是一个综合性的B2C平台，类似京东商城、天猫商城。会员可以在商城浏览商品、下订单，以及参加各种动。管理员、运营可以在平台后台管理系统中管理商品、订单、会员等。客服可以在后台管理系统中处理用户的询问以及投诉。
### 开发环境
  - IntelliJ IDEA
  - mysql
  - Tomcat
  - Nginx
  - Centos

### 使用框架
  **Spring＋SptingMVC＋Mybatis**
### 技术架构
  - 采用分布式架构设计

      ![](http://olpuebn54.bkt.clouddn.com/jiagou.png)
    - 分布式架构：把系统按照模块拆分成多个子系统。
    - 优点：
      1. 把模块拆分，使用接口通信，降低模块之间的耦合度。
      2. 把项目拆分成若干个子项目，不同的团队负责不同的子项目。
      3. 增加功能时只需要再增加一个子项目，调用其他系统的接口就可以。
      4. 可以灵活的进行分布式部署。
    - 缺点：系统之间交互需要使用远程通信，接口开发增加工作量。

### 依赖管理
  - 使用Maven来管理项目的依赖和生命周期

    - pom文件
          <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        	<modelVersion>4.0.0</modelVersion>
        	<groupId>com.taotao</groupId>
        	<artifactId>taotao-parent</artifactId>
        	<version>0.0.1-SNAPSHOT</version>
        	<packaging>pom</packaging>
        	<!-- 集中定义依赖版本号 -->
        	<properties>
        		<junit.version>4.12</junit.version>
        		<spring.version>4.1.3.RELEASE</spring.version>
        		<mybatis.version>3.2.8</mybatis.version>
        		<mybatis.spring.version>1.2.2</mybatis.spring.version>
        		<mybatis.paginator.version>1.2.15</mybatis.paginator.version>
        		<mysql.version>5.1.32</mysql.version>
        		<slf4j.version>1.6.4</slf4j.version>
        		<jackson.version>2.4.2</jackson.version>
        		<druid.version>1.0.9</druid.version>
        		<httpclient.version>4.3.5</httpclient.version>
        		<jstl.version>1.2</jstl.version>
        		<servlet-api.version>2.5</servlet-api.version>
        		<jsp-api.version>2.0</jsp-api.version>
        		<joda-time.version>2.5</joda-time.version>
        		<commons-lang3.version>3.3.2</commons-lang3.version>
        		<commons-io.version>1.3.2</commons-io.version>
        		<commons-net.version>3.3</commons-net.version>
        		<pagehelper.version>3.4.2-fix</pagehelper.version>
        		<jsqlparser.version>0.9.1</jsqlparser.version>
        		<commons-fileupload.version>1.3.1</commons-fileupload.version>
        		<jedis.version>2.7.2</jedis.version>
        		<solrj.version>4.10.3</solrj.version>
        	</properties>
            <dependencies>
            	<!-- 依赖具体定义 -->
              ...
            </dependencies>
            <build>
          		<finalName>${project.artifactId}</finalName>
          		<plugins>
          			<!-- 资源文件拷贝插件 -->
          			<plugin>
          				<groupId>org.apache.maven.plugins</groupId>
          				<artifactId>maven-resources-plugin</artifactId>
          				<version>2.7</version>
          				<configuration>
          					<encoding>UTF-8</encoding>
          				</configuration>
          			</plugin>
          			<!-- java编译插件 -->
          			<plugin>
          				<groupId>org.apache.maven.plugins</groupId>
          				<artifactId>maven-compiler-plugin</artifactId>
          				<version>3.2</version>
          				<configuration>
          					<source>1.7</source>
          					<target>1.7</target>
          					<encoding>UTF-8</encoding>
          				</configuration>
          			</plugin>
          		</plugins>
          		<pluginManagement>
          			<plugins>
          				<!-- 配置Tomcat插件 -->
          				<plugin>
          					<groupId>org.apache.tomcat.maven</groupId>
          					<artifactId>tomcat7-maven-plugin</artifactId>
          					<version>2.2</version>
          				</plugin>
          			</plugins>
          		</pluginManagement>
          	</build>
          </project>

### Redis集群

  - redis-cluster架构图

    ![](http://olpuebn54.bkt.clouddn.com/redis_jaigou.png)

    - (1)所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽.

    - (2)节点的fail是通过集群中超过半数的节点检测失效时才生效.

    - (3)客户端与redis节点直连,不需要中间proxy层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可

    - (4)redis-cluster把所有的物理节点映射到[0-16383]slot上,cluster 负责维护node<->slot<->value

      > Redis 集群中内置了 16384 个哈希槽，当需要在 Redis 集群中放置一个 key-value 时，redis 先对 key 使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，redis 会根据节点数量大致均等的将哈希槽映射到不同的节点

  - Spring与redis-cluster整合

      - 配置applicationContext.xml
              <!-- 连接池配置 -->
          	<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
          		<!-- 最大连接数 -->
          		<property name="maxTotal" value="30" />
          		<!-- 最大空闲连接数 -->
          		<property name="maxIdle" value="10" />
          		<!-- 每次释放连接的最大数目 -->
          		<property name="numTestsPerEvictionRun" value="1024" />
          		<!-- 释放连接的扫描间隔（毫秒） -->
          		<property name="timeBetweenEvictionRunsMillis" value="30000" />
          		<!-- 连接最小空闲时间 -->
          		<property name="minEvictableIdleTimeMillis" value="1800000" />
          		<!-- 连接空闲多久后释放, 当空闲时间>该值 且 空闲连接>最大空闲连接数 时直接释放 -->
          		<property name="softMinEvictableIdleTimeMillis" value="10000" />
          		<!-- 获取连接时的最大等待毫秒数,小于零:阻塞不确定的时间,默认-1 -->
          		<property name="maxWaitMillis" value="1500" />
          		<!-- 在获取连接的时候检查有效性, 默认false -->
          		<property name="testOnBorrow" value="true" />
          		<!-- 在空闲时检查有效性, 默认false -->
          		<property name="testWhileIdle" value="true" />
          		<!-- 连接耗尽时是否阻塞, false报异常,ture阻塞直到超时, 默认true -->
          		<property name="blockWhenExhausted" value="false" />
          	</bean>
          	<!-- redis集群 -->
          	<bean id="jedisCluster" class="redis.clients.jedis.JedisCluster">
          		<constructor-arg index="0">
          			<set>
          				<bean class="redis.clients.jedis.HostAndPort">
          					<constructor-arg index="0" value="192.168.101.3"></constructor-arg>
          					<constructor-arg index="1" value="7001"></constructor-arg>
          				</bean>
          				<bean class="redis.clients.jedis.HostAndPort">
          					<constructor-arg index="0" value="192.168.101.3"></constructor-arg>
          					<constructor-arg index="1" value="7002"></constructor-arg>
          				</bean>
          				<bean class="redis.clients.jedis.HostAndPort">
          					<constructor-arg index="0" value="192.168.101.3"></constructor-arg>
          					<constructor-arg index="1" value="7003"></constructor-arg>
          				</bean>
          				<bean class="redis.clients.jedis.HostAndPort">
          					<constructor-arg index="0" value="192.168.101.3"></constructor-arg>
          					<constructor-arg index="1" value="7004"></constructor-arg>
          				</bean>
          				<bean class="redis.clients.jedis.HostAndPort">
          					<constructor-arg index="0" value="192.168.101.3"></constructor-arg>
          					<constructor-arg index="1" value="7005"></constructor-arg>
          				</bean>
          				<bean class="redis.clients.jedis.HostAndPort">
          					<constructor-arg index="0" value="192.168.101.3"></constructor-arg>
          					<constructor-arg index="1" value="7006"></constructor-arg>
          				</bean>
          			</set>
          		</constructor-arg>
          		<constructor-arg index="1" ref="jedisPoolConfig"></constructor-arg>
          	</bean>

      - 测试
            private ApplicationContext applicationContext;
        	@Before
        	public void init() {
        		applicationContext = new ClassPathXmlApplicationContext(
        				"classpath:applicationContext.xml");
        	}

        	//redis集群
        	@Test
        	public void testJedisCluster() {
        	JedisCluster jedisCluster = (JedisCluster) applicationContext
        					.getBean("jedisCluster");

        			jedisCluster.set("name", "zhangsan");
        			String value = jedisCluster.get("name");
        			System.out.println(value);
        	}

      - 缓存同步

        当数据库中的内容信息发生改变后，例如首页大广告为的广告内容发生变化后，如何实现redis中的数据同步更新呢？可以在taotao-rest工程中发布一个服务，就是专门同步数据用的，其实只需要把缓存中的数据清空即可。当管理后台更新了内容信息后，需要调用此服务。

### 问题
  - 电商网站的商品搜索

    - 解决方案：搭建Solr服务器来实现站内搜索

    - 高级解决方案：使用SolrCloud（看书时看到）
      SolrCloud需要Solr基于Zookeeper部署，Zookeeper是一个集群管理软件，由于SolrCloud需要由多台服务器组成，由zookeeper来进行协调管理。

       - 图例说明

        ![](http://olpuebn54.bkt.clouddn.com/solrCloud.png)


  - 高并发访问

    - 解决方案：采用Tomcat集群＋Nginx作为负载均衡服务器
