---
layout: default
---

## Fabric 完整示例

### 一、简介

1. 源码：[https://github.com/ecsoya/fabric-demo](https://github.com/ecsoya/fabric-demo)
2. 功能：本地Fabric网络搭建、`fabric-gateway-spring-boot-starter`及`fabric-explorer-spring-boot-starter`使用。

### 二、Spring Boot 开始

创建一个Spring Boot Maven工程，并导入`fabric-explorer-spring-boot-starter`相关jar包。

```
		<dependency>
			<groupId>io.github.ecsoya</groupId>
			<artifactId>fabric-explorer-spring-boot-starter</artifactId>
			<version>1.0.3</version>
		</dependency>
```

### 三、Fabric 网络搭建

1- 运行`first-network`文件夹中的`byfn.sh`脚本，此方法为官方的[fabric-samples](https://github.com/hyperledger/fabric-samples.git)中的示例，做了少量修改。

```
./byfn.sh up -a -s couchdb
```

2- 具体的修改如下：
为了安装[通用链码](https://github.com/ecsoya/spring-fabric-gateway/raw/master/spring-fabric-gateway/src/chaincode/common/chaincode.go)，几处修改如下。

    a. 由于将链码放到了`src/chaincode`目录下，所以在`scripts/script.sh`中对链码的位置做了相应的修改。

       CC_SRC_PATH="github.com/chaincode/common/"

    b. 同样的，将`docker-composite-cli.yaml`中关于链码的映射路径(83行)做了修改。

        - ./../src/chaincode/:/opt/gopath/src/github.com/chaincode 

    c. 修改了`utils.sh`中执行Chaincode的默认的`init`、`invoke`和`query`方法。

        1. invoke -> create
        2. query -> get

3- 生成Fabric网络配置文件

运行`src/main/test`中的`NetworkGenerator`文件，在`src/main/resource/network`中生成`connection-Org1.yml`及`connection-Org2.yml`文件。

关于Fabric网络配置文件的相关信息，请参考文档[Fabric Network Config](https://ecsoya.github.io/fabric/pages/network-config.html)。

### 四、application.yml配置

```
spring:
   mvc:
      locale: zh_CN
      locale-resolver: fixed
   fabric:
      chaincode:
         identify: mycc
         name: Common Chaincode
         version: 1.0
      channel: mychannel
      peers: 2
      organizations:
      - Org1
      - Org2
      name: Common Fabric Network
      gateway:
         wallet:
            identify: admin
      network:
         file: network/connection-Org1.yml
         name: example-fabric
# Fabric explorer
      explorer:
         title: Fabric Explorer
#         logo: img/logo.png
         copyright: Ecsoya (jin.liu@soyatec.com)
         hyperledger-explorer-url: http://www.hyperleder.org
         path: /explorer
```

注意：
1. `first-network`示例中链码默认名称为`mycc`，通道默认名称为`mychannel`，都没有做修改。
2. `network:file`路径为上一步中生成的Fabric网络配置文件的路径。

### 五、运行

运行DemoApplication，启动Spring Boot服务，然后在浏览器中输入[http://localhost:8080/explorer](http://localhost:8080/explorer)，查看区块链浏览器。

### 参考文档

1. [Hyperledger Fabric Gateway SDK for Java](https://github.com/hyperledger/fabric-gateway-java)
2. [Java SDK for Hyperledger Fabric](https://github.com/hyperledger/fabric-sdk-java)
3. [Fabric 网络部署](https://ecsoya.github.io/fabric/pages/network.html)
4. [Fabric Network Config](https://ecsoya.github.io/fabric/pages/network-config.html)
5. [Spring Fabric Gateway](https://ecsoya.github.io/fabric/pages/gateway.html)
6. [Spring Fabric Explorer](https://ecsoya.github.io/fabric/pages/explorer.html)

* * *

[首页](http://ecsoya.github.io/fabric)
