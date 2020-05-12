---
layout: default
---

## 问题汇总

### 一、如何安装Hyperledger Fabric区块链网络？
	
这个问题，其实应该不是问题，如果确实有问题，请详细阅读 [Fabric 网络部署](https://ecsoya.github.io/fabric/pages/network.html) 一节。
	
### 二、如何使用通用链码？

个人建议先将[Spring Fabric Gateway](https://ecsoya.github.io/fabric/pages/gateway.html)一节再看一遍。

* 所谓“通用链码”，其实就是封装了简单的CRUD操作的链码，以便实现数据的持久化。
* 通用链码，就一个go文件，点击[此链接](https://github.com/ecsoya/spring-fabric-gateway/raw/master/spring-fabric-gateway/src/chaincode/common/chaincode.go)可以下载。
	
	
### 三、如何获取Fabric网络的基本信息，如区块高度、交易数量等？

个人建议，还是先将[Spring Fabric Gateway](https://ecsoya.github.io/fabric/pages/gateway.html)一节再看一遍。

然后在源码中找到`FabricLedger`这个类，你会发现，其实你想要的东西都在这里面，什么区块高度、当前交易Hash、通道、组织等等。

怎么才能取到`FabricLedger`这个对象呢？

因为我们使用的Spring框架，在`SpringFabricGatewayAutoConfigure`这个配置中，我们已经为你注册了一个`IFabricInfoService`的服务，你可以直接拿来用。

其实，不止`FabricLedger`，我们还封装了`FabricBlock`，`FabricTransaction`等一系列实用的类，方便在实际的应用场景中使用。

不信你看：

```java
public interface IFabricInfoService {

	/**
	 * Query Fabric Info.
	 */
	FabricQueryResponse<FabricLedger> queryFabricLedger();

	/**
	 * Query fabric block by using block number.
	 */
	FabricQueryResponse<FabricBlock> queryBlockByNumber(long blockNumber);

	/**
	 * Query fabric block by using transaction id.
	 */
	FabricQueryResponse<FabricBlock> queryBlockByTransactionID(String txId);

	/**
	 * Query fabric block by using block hash.
	 */
	FabricQueryResponse<FabricBlock> queryBlockByHash(byte[] blockHash);

	/**
	 * Paging query fabric blocks.
	 * 
	 */
	FabricPagination<FabricBlock> queryBlocks(FabricPaginationQuery<FabricBlock> query);

	/**
	 * Query all transactions of a block number.
	 */
	FabricQueryResponse<List<FabricTransaction>> queryTransactions(long blockNumber);

	/**
	 * Query transaction reads and writes of a transaction id.
	 */
	FabricQueryResponse<FabricTransactionRWSet> queryTransactionRWSet(String txId);

	/**
	 * Query history of object with given key and type.
	 */
	FabricQueryResponse<List<FabricHistory>> queryHistory(String type, String key);

	/**
	 * Query transaction with id.
	 */
	FabricQueryResponse<FabricTransaction> queryTransaction(String txid);
}

```

再次强调一下：

除了`区块高度`、`交易Hash`这些值以外，如`通道`，`链码名`，`节点名`这些信息本身从Fabric网络是取不到的，为了使用方便，我们把这些都配置在了配置文件中。


### 四、如何使用通用链码的Service服务？

正在撰写中……

### 五、如何使用通用链码的查询功能？

正在撰写中……

### 六、如何使用通用链码的分页查询？

正在撰写中……

### 七、使用通用链码，如何自定义自己的数据类型？

正在撰写中……

### 参考文档

1. [Hyperledger Fabric Gateway SDK for Java](https://github.com/hyperledger/fabric-gateway-java)
2. [Java SDK for Hyperledger Fabric](https://github.com/hyperledger/fabric-sdk-java)
3. [Fabric 网络部署](https://ecsoya.github.io/fabric/pages/network.html)
4. [Fabric Network Config](https://ecsoya.github.io/fabric/pages/network-config.html)
5. [Spring Fabric Gateway](https://ecsoya.github.io/fabric/pages/gateway.html)
6. [Spring Fabric Explorer](https://ecsoya.github.io/fabric/pages/explorer.html)

* * *

[首页](http://ecsoya.github.io/fabric)
