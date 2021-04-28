# 简介

Hyperledger Fabric 是 Hyperledger 超级账本项目的基石。它是基于许可的区块链，或者更准确地说是一种分布式分类帐技术（DLT），该技术最初由 IBM 公司和 Digital Asset 创建，它更适合企业和联盟去打造区块链即服务的BAAS平台。

Spring Fabric Gateway 是我基于官方的Gateway项目，结合Spring MVC做出的一套框架，这将大大的简化Fabric区块链项目的开发。

[在线演示](http://139.155.177.74:8081/)

### 一、网络部署

Fabric网络的搭建有很多方式，许多云商都有自己的Fabric BAAS平台，可以直接选择。也可以自己动手搭建，详细的方法请参考[Fabric 网络部署](pages/network.html)。

### 二、官方SDK

1. [Java SDK for Hyperledger Fabric](https://github.com/hyperledger/fabric-sdk-java)，官方的Java SDK。
2. [Hyperledger Fabric Gateway SDK for Java](https://github.com/hyperledger/fabric-gateway-java)，官方的Gateway项目，更加简化了Fabric网络的调用。

### 三、Spring Fabric Gateway

这是我基于官方的Gateway项目，结合Spring MVC做出的一套框架，将Chaincode的函数调用，包装成了Spring的服务。

1. 项目地址：[spring-fabric-gateway](https://github.com/ecsoya/spring-fabric-gateway)
2. 详细文档：[Spring Fabric Gateway](pages/gateway.html)
3. 最新版本：[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.github.ecsoya/spring-fabric-gateway/badge.svg?style=plastic)](https://maven-badges.herokuapp.com/maven-central/io.github.ecsoya/spring-fabric-gateway)
4. Maven地址：

```
<dependency>
	<groupId>io.github.ecsoya</groupId>
	<artifactId>fabric-gateway-spring-boot-starter</artifactId>
	<version>${latest_version}</version>
</dependency>
```

### 四、Spring Fabric Explorer

一个精简版的Fabric区块链浏览器。

1. 项目地址：[spring-fabric-gateway](https://github.com/ecsoya/spring-fabric-gateway)
2. 详细文档：[Spring Fabric Explorer](pages/explorer.html)
3. Maven地址：

```
<dependency>
	<groupId>io.github.ecsoya</groupId>
	<artifactId>fabric-explorer-spring-boot-starter</artifactId>
	<version>${latest_version}</version>
</dependency>
```

### 五、Fabric Network Config

以上的项目，包含官方的[Hyperledger Fabric Gateway SDK for Java](https://github.com/hyperledger/fabric-gateway-java)，都离不开 Fabric Network Config 文件的支持。

所谓的配置文件，就是将所有的组织、Peer和其相关的证书，全部配置到一个JSON文件或YAML文件中，方便在项目中读取。

详细文档：[Fabric Network Config](https://ecsoya.github.io/fabric/pages/network-config.html)

### 六、完整示例

1. [文档](https://ecsoya.github.io/fabric/pages/demo.html)
2. [源码](https://github.com/ecsoya/fabric-demo)
3. [在线演示](http://139.155.177.74:8081/)
* * *

[更多文章](http://ecsoya.github.io)
