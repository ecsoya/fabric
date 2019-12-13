---
layout: default
---

## Spring Fabric Explorer

Spring Fabric Explore 是基于 [Spring Fabric Gateway](https://ecsoya.github.io/fabric/pages/gateway.html)开发的简化版的Fabric区块链浏览器。

### 预览

![首页](https://ecsoya.github.io/fabric/img/explorer-1.png)
![区块信息](https://ecsoya.github.io/fabric/img/explorer-2.png)
![交易信息](https://ecsoya.github.io/fabric/img/explorer-3.png)
![历史记录](https://ecsoya.github.io/fabric/img/explorer-4.png)

### 实现方法

第一步：加载`fabric-explorer-spring-boot-starter`：

```
<dependency>
	<groupId>io.github.ecsoya</groupId>
	<artifactId>fabric-explorer-spring-boot-starter</artifactId>
	<version>1.0.2</version>
</dependency>
```

第二步：配置`application.yml`

```
# Fabric Network Configure      
spring:         
   fabric:
      chaincode: 
         identify: your chaincode id
         name: Common chaincode
         version: 1.0
      channel: your channel name
      organizations:
      - org1
      - org2
      name: your fabric network name
      gateway:
         wallet:
            identify: admin
      network:
         file: network/connection.yml //your fabric network config file.
         name: fabric network name
# Fabric explorer
      explorer: 
         title: Fabric Explorer
         logo: img/logo.png
         copyright: Ecsoya (jin.liu@soyatec.com)
         hyperledger-explorer-url: http://www.hyperleder.org
```

* * * 

[首页](http://ecsoya.github.io/fabric)
