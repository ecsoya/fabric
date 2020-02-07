[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.github.ecsoya/spring-fabric-gateway/badge.svg?style=plastic)](https://maven-badges.herokuapp.com/maven-central/io.github.ecsoya/spring-fabric-gateway)

# 版本更新

### 1.0.6

1. Wallet配置文件更新：`spring.fabric.gateway.wallet.identify 改为：spring.fabric.gateway.wallet.identity `
2. 新增了扩展接口和服务：
	- 增加了`io.github.ecsoya.fabric.annotation`包，里面是Bean扩展相关的标记。
	- 更新了服务接口，增加了自定义链码函数的功能。
3. 规范了接口命名：
	- IFabricService -> IFabricObjectService （默认FabricObject的服务接口）
	- IChaincodeContextService2 -> IFabricService （Fabric服务次接口）
	- IChaincodeContextService1 -> IFabricCommonService （Fabric服务主接口）
