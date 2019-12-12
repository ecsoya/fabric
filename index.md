### Hyperledger Fabric

Hyperledger Fabric 是 Hyperledger 超级账本项目的基石。它是基于许可的区块链，或者更准确地说是一种分布式分类帐技术（DLT），该技术最初由 IBM 公司和 Digital Asset 创建，它更适合企业和联盟去打造区块链即服务的BAAS平台。

### 网络搭建

Fabric网络的搭建有很多方式，许多云商都有自己的Fabric BAAS平台，可以直接选择。也可以自己动手搭建，详细的方法请参考以前的文章：[Hyperledger Fabric区块链网络的几种部署方案](pages/network.html).

### 官方SDK

1. [Java SDK for Hyperledger Fabric](https://github.com/hyperledger/fabric-sdk-java)，官方的Java SDK。
2. [Hyperledger Fabric Gateway SDK for Java](https://github.com/hyperledger/fabric-gateway-java)，官方的Gateway项目，更加简化了Fabric网络的调用。

### Spring Fabric Gateway

这是我基于官方的Gateway项目，结合Spring MVC做出的一套框架，将Chaincode的函数调用，包装成了Spring的服务。

  - 项目地址：[spring-fabric-gateway](https://github.com/ecsoya/spring-fabric-gateway)
  - 包含模块：
   	  -  spring-fabric-gateway，基础模块，包含基于Fabric Gateway的Spring服务。
   	  -  fabric-gateway-spring-boot-starter，包含spring-fabric-gateway的Spring boot starter。
   	  -  fabric-explorer-spring-boot-starter，一个包含精简版Fabric浏览器的spring boot starter，使用很少的代码构建区块链浏览器。
