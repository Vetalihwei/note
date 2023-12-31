## 1 分布式系统中相关概念

<details>
<summary> </summary>

### 1.1 大型互联网项目架构目标
- 高性能--提供快速访问体验
- 高可用--网站服务一直可以正常访问
- 可伸缩--通过硬件增加、减少，提高、降低处理能力
- 高可扩展--系统间耦合低，方便的通过新增/移除方式，增加/减少新的功能/模块
- 安全性--提供网站安全访问和数据加密，安全存储等策略
- 敏捷性：随需应变，快速响应
**衡量网站的性能指标**：
- **响应时间**：指执行一个请求从开始到最后收到响应数据所花费的总体时间
- **并发数**：指系统同时能处理的请求数量
  - **并发连接数**：指的是客户端向服务器发起请求，并建立了TCP连接，每秒钟服务器连接的总TCP数量
  - **请求数**：QPS，每秒请求数
  - **并发用户数**：单位时间用户数
- **吞吐量**：指单位时间内系统能处理的请求数量
  - **QPS**：Query Per Second 每秒查询数
  - **TPS**：Transactions Per Second 每秒事务数
  - 一个事务是指一个客户机向服务器发送请然后服务器做出反应的过程。客户机在发送请求时开始计时，收到服务器响应后结束计时，以此来计算使用的时间和完成的事务个数
  - 一个页面的一次访问，只会形成一个TPS，但一次页面请求，可能产生多次对服务器的请求，就会有多个QPS

---

### 1.2 集群与分布式
- 集群：很多人，做同一件事
  - 一个业务模块，部署在多台服务器上
- 分布式：很多人，做不同一件事，但共同为一件大事服务
  - 一个大的业务系统，拆分为小的业务模块，分别部署在不同的机器上

**集群分布式特性**
- 高性能
- 高可用
- 高伸缩
- 高可扩展


</details>

---

## 2 Dubbo概念

<details>
<summary> </summary>

### 2.1 概念
- Dubbo是高性能、轻量级的JavaRPC框架
- 致力于提供高性能和透明化的RPC远程服务调用方案，以及SOA服务治理方案

### 2.2 架构
![](/img/Dubbo/Structrue.png)



</details>

---

## 3 dubbo入门

<details>
<summary> </summary>

### 3.1 注册中心安装-zookeeper

**安装**
```
#将zookeeper安装包放至/opt/zooKeeper
#解压
tar -zxvf apache-zookeeper-3.6.4-bin.tar.gz /opt/zookeeper/
```
**配置启动**
- 创建数据存放处
```
mkdir /opt/zooKeeper/zkdata
```
- 修改配置文件
```
#copy配置文件
cp ./conf/zoo_sample.conf ./conf/zoo.conf
#编辑，将DataDir值改为zkdata路径
vim ./conf/zoo.conf

```
- 启动服务
```
./bin/zkServer.sh start
```

### 3.2 利用springboot3.1.4搭建基本程序
![](/img/Dubbo/Structrue.png)
**步骤**
- 创建服务提供者Provider模块
- 创建服务消费者Consumer模块
- 在Provider模块编写UserServiceImpl提供服务
- 在Consumer中的UserController远程调用UserServiceImpl提供的服务
- 分别启动两个服务，测试

**dubbo/zookeeper依赖**
consumer/provider均需要加
```xml
<!--Dubbo起步依赖-->
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo</artifactId>
    <version>3.2.2</version>
</dependency>
<dependency>
    <artifactId>zookeeper</artifactId>
    <groupId>org.apache.zookeeper</groupId>
    <version>3.8.1</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-x-discovery</artifactId>
    <version>4.2.0</version>
</dependency>
```
#### 3.3 Provider
**Service改造**
- 将Service注解更换为dubbo的@DubboService注解  
将这个类提供的方法对外发布。将访问的地址ip，端口，路径注册到注册中心  
- 启动类添加@EnableDubbo
- 修改配置文件
```yml
dubbo:
  #配置模块名称
  application:
    name: user-service-provider
  #配置注册地址，端口默认2181
  registry:
    address: zookeeper://192.168.52.129:2181
  protocol:
    name:  dubbo
    port:  20880
```
#### 3.4 Consumer
**Controller改造**
- @Autowired改为@DubboReference，远程注入
  - 从zookeeper注册中心获取对象的访问url
  - 进行远程调用RPC
  - 将结果封装为一个代理对象，给变量赋值
- 启动类添加@EnableDubbo
- 修改配置文件，注意端口修改，防止冲突
```yml
dubbo:
  #配置模块名称
  application:
    name: user-web-consumer
  #配置注册地址，端口默认2181
  registry:
    address: zookeeper://192.168.52.129:2181
  protocol:
    name:  dubbo
    port:  20880
server:
  port: 8081 #########

```
#### 3.5 Interface
因为需要将provider和consumer拆分成独立模块，故两者不能有依赖，则需一个端口模块来存放公共端口

</details>

---

## 4 dubbo-admin

<details>
<summary> </summary>

- dubbo-admin是图形化的服务管理页面
- 从注册中心中获取到所有的提供者/消费者进行配置管理
- 路由规则、动态配置、服务降级、访问控制、权重调整、负载均衡等管理功能
> dubbo-admin是一个前后端分离的项目，前端使用vue，后端使用springboot。安装dubbo-admin实际上就是部署该项目  

[下载地址](https://github.com/apache/dubbo-admin)

**修改配置**
dubbo-admin-develop\dubbo-admin-develop\dubbo-admin-server\src\main\resources下application文件
```
admin.registry.address=zookeeper://127.0.0.1:2181 //修改为自己注册中心ip
admin.config-center=zookeeper://127.0.0.1:2181
admin.metadata-report.address=zookeeper://127.0.0.1:2181
```
**部署**
dubbo-admin-develop目录下打开powershell，执行`mvn clean package`




</details>

---

## 5 高级特性

<details>
<summary> </summary>

### 5.1 地址缓存
> 服务中心挂了，服务是否可以正常访问？可以。

- dubbo服务消费者再第一次调用时，会将服务提供方地址缓存到本地，以后再调用则不会访问注册中心
- 当服务提供者地址发生变化时，注册中心会通知服务消费者

### 5.2 超时与重试

- 服务消费者在调用服务提供者时发生了阻塞、等待的情形，这个时候，服务消费者会一直等待下去
- 在某个峰值时，大量的请求都在同时请求服务消费者，会造成线程的大量堆积，势必会造成雪崩
- dubbo利用超时机制来解决，设置一个超时时间，在这个时间段内，无法完成服务访问，则自动端口连接
- 使用timeout属性配置超时时间，默认值1000，单位ms
  ```java
  @DubboService(timeout = 3000,retries = 2)//retries，重试次数
  @DubboReference(timeout = 1000)//若两方都配，消费放消费，建议配在服务提供方
  ```

### 5.3 多版本
- 灰度发布：当出现新功能时，会让一部分用户先使用新功能，用户反馈没问题时，再将所有用户迁移到新功能
- dubbo中使用version属性来设置和调用同一个接口的不同版本
  ```java
  @DubboService(version = "v1.0")
  @DubboReference(version = "v1.0")//远程注入
  ```

### 5.4 负载均衡
- dubbo负载均衡策略:
  - Random：按权重设置随机概率
    ```java
    @DubboService(weight = x)
    ```
  - RoundRobin：按权重轮询
  - LeastActive：按最少活跃调用数，相同活跃数的随机
  - ConsistenHash：一致性Hash，相同参数的请求总是发到同一提供者

**实现**
```java
@DubboReference(loadbalance = "type")
```

### 5.5 集群容错
- 集群容错模式：
  - Failover Cluster：失败重试。当出现失败，重试其他服务器，默认重试两次，使用retries配置，一般用于读操作
  - Failfast Cluster：快速失败，只发起一次调用，失败立即报错，一般用于写操作
  - Failsafe Cluster：失败安全，出现异常时，直接忽略，返回一个空结果
  - Failback Cluster：失败自动恢复，后台记录失败请求，定时重发
  - Forking Clust：并行调用多个服务器，只要成功一个就返回
  - Broadcast Clust：广播调用所有提供者，逐个调用，任意一台报错则报错

### 5.6 服务降级
> 当服务压力过大时，考虑关闭部分服务释放资源，保证核心业务服务运行

- 服务降级方式：
  - `mock=force:return null` 表示消费方对该服务的方法调用都直接返回null值，不发起远程调用，用来屏蔽不重要服务不可用时对调用方的影响
  - `mock=fail:return null` 表示消费方对该服务的方法调用在失败后，再返回null值，不抛异常。用来容忍不重要服务不稳定时对调用方的影响
  - 

</details>

---

## 

<details>
<summary> </summary>




</details>

