## 服务消费者接入CSE--Feign和Hystrix的结合

Hystrix是一个帮助解决分布式系统交互时超时处理和容错的类库。

基于微服务构造的分布式系统，被分层解耦为诸多存在调用依赖关系的微服务，一个请求需要调用多个微服务是非常常见。较底层的服务如果出现故障，会连锁传导到依赖他的上层服务，并逐渐扩散到影响整个系统，发生“雪崩”效应。为了解决这个问题，业界提出了断路器模型，当对特定的服务的调用的不可用达（错误或超时）到一个阀值（比如5秒20次）断路器将会被打开，避免连锁故障，fallback方法降级直接返回一个固定值。

Hystrix就是用来解决这类问题的。

这里我们基于Eureka Client创建一个服务消费者服务，演示如何接入CSE。详细文档可参考[Spring Cloud应用接入CSE](https://support.huaweicloud.com/devg-cse/cse_03_0096.html)

### 1.修改pom文件

- 删除spring-cloud-starter-eureka-server依赖，并增加spring-boot-starter-web运行依赖

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
- dependencyManagement中增加cse-dependency，方便管理三方件。可根据需求选择不同CSE版本，当前最新为2.3.12。

```xml
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>${spring-cloud.version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
		<dependency>
			<groupId>com.huawei.paas.cse</groupId>
			<artifactId>cse-dependency</artifactId>
			<version>2.3.12</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```
- 增加CSE服务注册和发现依赖

```xml
<dependency>
	<groupId>org.apache.servicecomb</groupId>
	<artifactId>spring-boot-starter-registry</artifactId>
</dependency>
<dependency>
	<groupId>org.apache.servicecomb</groupId>
	<artifactId>spring-boot-starter-configuration</artifactId>
</dependency>
<dependency>
	<groupId>org.apache.servicecomb</groupId>
	<artifactId>spring-boot-starter-discovery</artifactId>
</dependency>
<dependency>
	<groupId>com.huawei.paas.cse</groupId>
	<artifactId>foundation-auth</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```

### 2.自定义RibbonClient
采用CSE服务实例清单的维护机制，需要替代Ribbon默认的负载均衡策略，可以通过配置文件来自定义RibbonClient。
修改 application.propertie或application.yaml，增加如下配置：

```
service-provider.ribbon.NIWSServerListClassName=org.apache.servicecomb.springboot.starter.discovery.ServiceCombServerList
```
其中：

* service-provider.ribbon.NIWSServerListClassName: RibbonClient的配置规则，<服务名>.ribbon.<类型>
* org.apache.servicecomb.springboot.starter.discovery.ServiceCombServerList: CSE服务实例清单的维护机制

### 3.启用服务注册和发现

在原 ConsumerApplication.java 中增加<html>@ImportResource</html>，自动注入CSE依赖Bean。

```Java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
@ImportResource(locations = "classpath*:META-INF/spring/*.bean.xml")
public class ConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApplication.class, args);
	}
}
```

### 4.修改应用配置
src/main/resources下增加微服务描述文件microservice.yaml，如下配置：

```yaml
cse-config-order: 100
APPLICATION_ID: springcloud-2-cse-sample
service_description:
  name: service-consumer-feign-hystrix
  version: 0.0.1
cse:
  service:
    registry:
      address: https://cse.cn-north-1.myhwclouds.com:443
      instance:
        watch: false
  config:
    client:
      serverUri: https://cse.cn-north-1.myhwclouds.com:443
      refreshMode: 1
      refresh_interval: 15000
  rest:
    address: 0.0.0.0:7095 # 7095端口与src/main/resources/application.yml中server.port保持一致
  credentials:
    accessKey: your access key in CSE
    secretKey: your secret key in CSE
    akskCustomCipher: default
```
其中：

* cse.credentials.accessKey: 用户华为云账户AK
* cse.credentials.secretKey: 用户华为云账户SK
* cse.rest.address: 注册到服务中心的EndPoint，需要与src/main/resources/application.yml中server.port保持一致

### 5.启动应用
直接运行ConsumerApplication的main函数

访问[http://localhost:7071/](http://localhost:7071/)，可以看到Eureka Server自带的UI管理界面上新增一条SERVICE-CONSUMER-FEIGN-HYSTRIX服务实例记录

访问[http://localhost:7095/hello/springcloud](http://localhost:7095/hello/springcloud)，返回：

hello springcloud

停掉service-provider服务，再次调用上面的接口，返回：

fallback springcloud