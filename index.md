# 简介

Hyperledger Fabric 是 Hyperledger 超级账本项目的基石。它是基于许可的区块链，或者更准确地说是一种分布式分类帐技术（DLT），该技术最初由 IBM 公司和 Digital Asset 创建，它更适合企业和联盟去打造区块链即服务的BAAS平台。

### 一、网络搭建

Fabric网络的搭建有很多方式，许多云商都有自己的Fabric BAAS平台，可以直接选择。也可以自己动手搭建，详细的方法请参考以前的文章：[Hyperledger Fabric区块链网络的几种部署方案](pages/network.html).

### 二、官方SDK

1. [Java SDK for Hyperledger Fabric](https://github.com/hyperledger/fabric-sdk-java)，官方的Java SDK。
2. [Hyperledger Fabric Gateway SDK for Java](https://github.com/hyperledger/fabric-gateway-java)，官方的Gateway项目，更加简化了Fabric网络的调用。

### 三、Spring Fabric Gateway

这是我基于官方的Gateway项目，结合Spring MVC做出的一套框架，将Chaincode的函数调用，包装成了Spring的服务。

1. 项目地址：[spring-fabric-gateway](https://github.com/ecsoya/spring-fabric-gateway)
2. 包含模块：
     *  `spring-fabric-gateway`，基础模块，包含基于Fabric Gateway的Spring服务。
     *  `fabric-gateway-spring-boot-starter`，包含spring-fabric-gateway的spring boot starter，点击[链接](pages/gateway.html)获取更多帮助。
     *  `fabric-explorer-spring-boot-starter`，一个包含精简版Fabric浏览器的spring boot starter，使用很少的代码构建区块链浏览器，点击[链接](pages/explorer.html)获取更多帮助。
3. Maven地址

```
<dependency>
	<groupId>io.github.ecsoya</groupId>
	<artifactId>fabric-gateway-spring-boot-starter</artifactId>
	<version>1.0.2</version>
</dependency>
```

### 四、Fabric Network Builder

以上的项目，包含官方的`fabric-gateway-java`，都离不开 Fabric Network 配置文件的支持。

所谓的配置文件，就是将所有的组织、Peer和其相关的证书，全部配置到一个JSON文件或YAML文件中，方便在项目中读取。

参考项目[fabric-network-builder](https://github.com/ecsoya/fabric-network-builder)，即可轻松的生成Fabric网络配置的YAML文件。

* * *

[更多](http://ecsoya.github.io)
