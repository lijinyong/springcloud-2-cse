## 服务消费者接入CSE

这里我们基于Eureka Client创建一个服务消费者服务，演示如何接入CSE。详细文档可参考[Spring Cloud应用接入CSE](https://support.huaweicloud.com/devg-cse/cse_03_0096.html)

### 1.修改pom文件

- 删除spring-cloud-starter-eureka依赖，并增加spring-boot-starter-web运行依赖

```xml
<!--<dependency>-->
	<!--<groupId>org.springframework.cloud</groupId>-->
	<!--<artifactId>spring-cloud-starter-eureka</artifactId>-->
<!--</dependency>-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

- 增加CSE服务注册和发现依赖。可根据需求选择不同CSE版本，当前最新为2.3.20

```xml
<dependency>
	<groupId>com.huawei.paas.cse</groupId>
	<artifactId>cse-solution-spring-cloud</artifactId>
	<version>2.3.19</version>
</dependency>
```

### 2.自定义RibbonClient
采用CSE服务实例清单的维护机制，需要替代Ribbon默认的负载均衡策略，可以通过配置文件来自定义RibbonClient。
修改 application.properties或application.yaml，增加如下配置：

```
service-provider.ribbon.NIWSServerListClassName=org.apache.servicecomb.springboot.starter.discovery.ServiceCombServerList
```
其中：

* service-provider.ribbon.NIWSServerListClassName: RibbonClient的配置规则，<服务名>.ribbon.<类型>
* org.apache.servicecomb.springboot.starter.discovery.ServiceCombServerList: CSE服务实例清单的维护机制

### 3.修改应用配置
修改 application.properties或application.yaml，从而接入CSE服务中心，增加如下配置：

```yaml
cse.credentials.accessKey=your access key in CSE
cse.credentials.secretKey=your secret key in CSE
cse.credentials.akskCustomCipher=default
cse.credentials.project=cn-north-1
cse.service.registry.address=https://cse.cn-north-1.myhuaweicloud.com
cse.config.client.serverUri=https://cse.cn-north-1.myhuaweicloud.com
```
其中：

* cse.credentials.accessKey: 用户华为云账户AK
* cse.credentials.secretKey: 用户华为云账户SK
* cse.credentials.akskCustomCipher: 加密方式，默认不加密
* cse.credentials.project: 可选华北-北京（cn-north-1）、华南-广州（cn-south-1）、华东-上海二（cn-east-2），默认cn-north-1
* cse.service.registry.address: CSE注册中心地址，默认连接华北-北京一
* cse.config.client.serverUri: CSE配置中心地址，默认连接华北-北京一

**附区域、注册与配置中心地址：**

| 区域(Region)   |   cse.credentials.project   |    cse.service.registry.address / cse.config.client.serverUri |   
| -------------- | --------------------------- | ---------------------------------------  | 
|华北-北京一  | cn-north-1      | https://cse.cn-north-1.myhuaweicloud.com |    
|华南-广州    | cn-south-1      | https://cse.cn-south-1.myhuaweicloud.com |
|华东-上海二  | cn-east-2     | https://cse.cn-east-2.myhuaweicloud.com |

### 4.启动服务
直接运行ConsumerApplication的main函数

访问[http://localhost:7091/consumer/hello/springcloud](http://localhost:7091/consumer/hello/springcloud)，同步方式调用服务/hello接口

访问[http://localhost:7091/consumer/hello-async/springcloud](http://localhost:7091/consumer/hello-async/springcloud)，异步方式调用服务/hello接口
