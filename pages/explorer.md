---
layout: default
---

## Spring Fabric Explorer

Spring Fabric Explore 是基于 [Spring Fabric Gateway](https://ecsoya.github.io/fabric/pages/gateway.html)开发的简化版的Fabric区块链浏览器。

### 一、预览

![首页](https://ecsoya.github.io/fabric/img/explorer-1.png)

![区块信息](https://ecsoya.github.io/fabric/img/explorer-2.png)

![交易信息](https://ecsoya.github.io/fabric/img/explorer-3.png)

![历史记录](https://ecsoya.github.io/fabric/img/explorer-4.png)

### 二、实现方法

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

### 三、参数配置

| 参数          | 说明               | 是否必需 | 默认值 |
|:-------------|:-------------------|:------|:--------|
|spring.fabric.explorer.title|大标题|否|Spring Fabric Explorer|
|spring.fabric.explorer.logo|图标|否|![图标](https://ecsoya.github.io/fabric/img/camel.png)|
|spring.fabric.explorer.copyright|版权信息文章|否|Ecsoya (jin.liu@soyatec.com)|
|spring.fabric.explorer.hyperledger-explorer-url|标准的Hyperledger Explorer地址|否|无|

### 四、支持语言

现在支持英语和中文两种，使用了SpringBoot默认的语言处理方式。

如需切换语言，请在`application.yml`中配置：

```
spring:
   mvc:
      locale: zh_CN
      locale-resolver: fixed
```

### 五、参考

1. [Fabric 网络搭建](https://ecsoya.github.io/fabric/pages/network.html)
2. [Spring Fabric Gateway](https://ecsoya.github.io/fabric/pages/gateway.html)
3. [Fabric Network Config](https://ecsoya.github.io/fabric/pages/network-config.html)
4. [Hyperledger Explorer](https://www.hyperledger.org/projects/explorer)

* * * 

[首页](http://ecsoya.github.io/fabric)
