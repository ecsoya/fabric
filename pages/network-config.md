---
layout: default
---

## Fabric Network Config

Fabric网络配置文件支持JSON和YMAL两种格式，主要用于加载Fabric网络的Client，Peers和证书等信息，具体的加载类为[Java SDK for Hyperledger Fabric](https://github.com/hyperledger/fabric-sdk-java)中的
`org.hyperledger.fabric.sdk.NetworkConfig`。

### 一、概述

1. **client**: 客户端配置。其中的`organization`属性为必需项，指定到`organizations`配置中的某一个具体的组织。
2. **channels**: 通道配置。所有的通道都配置在这里，每一个通道里都要配置排序节点和peer节点。
3. **organizations**: 组织配置。配置所有的组织的mspId，ca实体及admin用户的证书等。
4. **orderers**: 排序节点配置。url，证书等。
5. **peers**: Peer节点配置。url，grpc属性，证书等。
6. **certificateAuthorities**: ca节点配置。

### 二、注意点

1- 由于所有的证书都是颁发给带域名的url的，而我们通常会使用IP来访问，所以必须要配置`hostnameOverride`属性指定到相关的url。

```
   orderer.example.com:
      url: grpcs://127.0.0.1:7050   //url with ip
      grpcOptions:
         grpc-max-send-message-length: 15
         grpc.keepalive_time_ms: 360000
         grpc.keepalive_timeout_ms: 180000
         hostnameOverride: orderer.example.com // url with domain
```

2- 证书的加载支持`path`和`pem`两种方式。

```
      tlsCACerts:
         pem: |
            -----BEGIN CERTIFICATE-----
            xxx
            -----END CERTIFICATE-----
```

及

```
	  tlsCACerts:
         path: peerOrganizations/Org1.example.com/peers/peer0.Org1.example.com/msp/tlscacerts/tlsca.Org1.example.com-cert.pem
 
```

如果使用`path`的方式，会涉及到运行时加载路径的问题，所以最好是使用`pem`的方式，将证书内容预先加载到配置文件中。

### 三、示例

具体示例：[https://github.com/ecsoya/fabric-network-builder](https://github.com/ecsoya/fabric-network-builder)

### 四、参考

1. [Fabric 网络部署](https://ecsoya.github.io/fabric/pages/network.html)
2. [Java SDK for Hyperledger Fabric](https://github.com/hyperledger/fabric-sdk-java)。

* * * 

[首页](http://ecsoya.github.io/fabric)
