---
layout: default
---

## Spring Fabric Gateway

Spring Fabric Gateway 是基于Hyperledger官方的[Hyperledger Fabric Gateway SDK for Java](https://github.com/hyperledger/fabric-gateway-java)工具的拓展。主要功能是将Chaincode（链码）的函数调用，通过Spring MVC的方式进行封装，简化使用。

### 一、前提条件

1. Hyperledger Fabric网络1.4及以上。（必须）
2. CouchDB，部分查询需要CouchDB支持。（非必须）
3. 安装通用链码（Common Chaincode），否则部分服务不可用。（非必须）

### 二、使用方法

第一步： 加载`fabric-gateway-spring-boot-starter`

```
<dependency>
	<groupId>io.github.ecsoya</groupId>
	<artifactId>fabric-gateway-spring-boot-starter</artifactId>
	<version>1.0.4</version>
</dependency>
```

第二步： 修改`application.yml`，配置Fabric网络基础信息。

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
```

第三步： 使用

 * 如果未安装通用链码，可以使用`IFabricInfoService`进行Fabric网络基本信息的查询操作，也可以基于`FabricContext`实现数据的读写操作。
 * 如果安装了通用链码，可以直接调用`IFabricService`服务进行数据的CRUD操作。

### 三、配置参数介绍

| 参数          | 说明               | 是否必需 | 默认值 |
|:-------------|:-------------------|:------|:--------|
|spring.fabric.chaincode.identify|链码标识（名称）|是|无|
|spring.fabric.chaincode.name|链码名称，用于Fabric网络基本信息查询|否|无|
|spring.fabric.chaincode.version|链码版本，用于Fabric网络基本信息查询|否|无|
|spring.fabric.channel|通道名称|是|无|
|spring.fabric.organizations|组织名称，用于Fabric网络基本信息查询|否|无|
|spring.fabric.name|Fabric网络名称，用于Fabric网络基本信息查询|否|无|
|spring.fabric.gateway.wallet.memory|基于内存创建Wallet实例|否|是|
|spring.fabric.gateway.wallet.file|如果不是基于内存创建Wallet示例，需要指定Wallet的加载目录|否|无|
|spring.fabric.gateway.wallet.identity|Wallet实例的标识|是|admin|
|spring.fabric.gateway.commit-timeout|Fabric网络提交超时时间，单位：秒，`1.0.4+`。如果出现执行写操作超时，增加此时间设置，一般就会解决。|否|5秒|
|spring.fabric.network.file|Fabric网络配置文件路径|是|无|
|spring.fabric.network.name|Fabric网络名称，用于Fabric网络基本信息查询|是|无|

### 四、通用链码

通用链码是一个集合了CRUD的链码，由于使用了结合`id`和`type`的`CompositeKey`，可以在很大程度上实现基于Id和类型的数据的CRUD操作。

基于性能，此链码使用`go`语言开发。

```
/*
 * CommonContract
 */

package main

/* Imports
 * 4 utility libraries for formatting, handling bytes, reading and writing JSON, and string manipulation
 * 2 specific Hyperledger Fabric specific libraries for Common Contracts
 */
import (
	"bytes"
	"fmt"
	"strconv"
	"time"

	"github.com/hyperledger/fabric/core/chaincode/shim"
	sc "github.com/hyperledger/fabric/protos/peer"
)

// CommonContract Define the Common Contract structure, it stores the structure of tracing objects.
type CommonContract struct {
}

/*
 * The Init method is called when the Common Contract is instantiated by the blockchain network
 * Best practice is to have any Ledger initialization in separate function -- see initLedger()
 */
func (s *CommonContract) Init(APIstub shim.ChaincodeStubInterface) sc.Response {
	function, args := APIstub.GetFunctionAndParameters()
	if function == "init" {
		return s.create(APIstub, args);
	}
	return shim.Success([]byte("Chaincode say 'hi' to you"))
}

/*
 * The Invoke method is called as a result of an application request to run the Common Contract
 * The calling application program has also specified the particular Common contract function to be called, with arguments
 */
func (s *CommonContract) Invoke(APIstub shim.ChaincodeStubInterface) sc.Response {

	// Retrieve the requested Common Contract function and arguments
	function, args := APIstub.GetFunctionAndParameters()
	// Route to the appropriate handler function to interact with the ledger appropriately
	if function == "create" {
		return s.create(APIstub, args)
	} else if function == "get" {
		return s.get(APIstub, args)
	} else if function == "update" {
		return s.update(APIstub, args)
	} else if function == "delete" {
		return s.delete(APIstub, args)
	} else if function == "list" {
		return s.list(APIstub, args)
	} else if function == "query" {
		return s.query(APIstub, args)
	} else if function == "count" {
		return s.count(APIstub, args)
	} else if function == "exists" {
		return s.exists(APIstub, args)
	} else if function == "history" {
		return s.history(APIstub, args)
	}
	return shim.Error("Unsupport function " + function)
}

func (s *CommonContract) history(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {
	if len(args) < 2 {
		return shim.Error("Incorrect number of arguments. Expecting 2")
	}

	class := args[0]
	key := args[1]

	objectType := "type~key"
	compositeKey, err := APIstub.CreateCompositeKey(objectType, []string{class, key})
	if err != nil {
		return shim.Error(err.Error())
	}

	resultsIterator, err := APIstub.GetHistoryForKey(compositeKey)
	if err != nil {
		return shim.Error(err.Error())
	}
	defer resultsIterator.Close()

	// buffer is a JSON array containing historic values for the marble
	var buffer bytes.Buffer
	buffer.WriteString("[")

	bArrayMemberAlreadyWritten := false
	for resultsIterator.HasNext() {
		response, err := resultsIterator.Next()
		if err != nil {
			return shim.Error(err.Error())
		}
		// Add a comma before array members, suppress it for the first array member
		if bArrayMemberAlreadyWritten == true {
			buffer.WriteString(",")
		}
		buffer.WriteString("{\"txId\":")
		buffer.WriteString("\"")
		buffer.WriteString(response.TxId)
		buffer.WriteString("\"")

		buffer.WriteString(", \"value\":")
		// if it was a delete operation on given key, then we need to set the
		//corresponding value null. Else, we will write the response.Value
		//as-is (as the Value itself a JSON marble)
		if response.IsDelete {
			buffer.WriteString("null")
		} else {
			buffer.WriteString(string(response.Value))
		}

		buffer.WriteString(", \"timestamp\":")
		buffer.WriteString("\"")
		buffer.WriteString(time.Unix(response.Timestamp.Seconds, int64(response.Timestamp.Nanos)).String())
		buffer.WriteString("\"")

		buffer.WriteString(", \"isDelete\":")
		buffer.WriteString("\"")
		buffer.WriteString(strconv.FormatBool(response.IsDelete))
		buffer.WriteString("\"")

		buffer.WriteString("}")
		bArrayMemberAlreadyWritten = true
	}
	buffer.WriteString("]")

	fmt.Printf("- history returning:\n%s\n", buffer.String())

	return shim.Success(buffer.Bytes())
}

func (s *CommonContract) query(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {
	if len(args) < 1 {
		return shim.Error("Incorrect number of arguments. At least 1 argument with query string should be set.")
	}
	query := args[0]

	var pageSize int32 = -1
	bookmark := ""
	if len(args) > 1 {
		value, err := strconv.ParseInt(args[1], 10, 32)
		if err == nil {
			pageSize = int32(value)
		}
	}
	if len(args) > 2 {
		bookmark = args[2]
	}

	var resultsIterator shim.StateQueryIteratorInterface
	var err error

	var metadata *sc.QueryResponseMetadata

	if pageSize > -1 {
		resultsIterator, metadata, err = APIstub.GetQueryResultWithPagination(query, pageSize, bookmark);
	} else {
		resultsIterator, err = APIstub.GetQueryResult(query)
	}

	if err != nil {
		return shim.Error(err.Error())
	}
	defer resultsIterator.Close()

	// buffer is a JSON array containing QueryResults
	var buffer bytes.Buffer
	buffer.WriteString("{")
	buffer.WriteString("\"data\":")
	buffer.WriteString("[")

	bArrayMemberAlreadyWritten := false
	for resultsIterator.HasNext() {
		queryResponse, err := resultsIterator.Next()
		if err != nil {
			return shim.Error(err.Error())
		}

		// Add a comma before array members, suppress it for the first array member
		if bArrayMemberAlreadyWritten == true {
			buffer.WriteString(",")
		}

		// Record is a JSON object, so we write as-is
		buffer.WriteString(string(queryResponse.Value))

		bArrayMemberAlreadyWritten = true
	}
	buffer.WriteString("]")

	if metadata != nil {
		buffer.WriteString(",")
		buffer.WriteString("\"meta\":")
		buffer.WriteString("{")
		buffer.WriteString("\"recordsCount\":")
		buffer.WriteString("\"")
		buffer.WriteString(fmt.Sprintf("%v", metadata.FetchedRecordsCount))
		buffer.WriteString("\"")
		buffer.WriteString(",")
		buffer.WriteString("\"bookmark\":")
		buffer.WriteString("\"")
		buffer.WriteString(metadata.Bookmark)
		buffer.WriteString("\"")
		buffer.WriteString("}")
	}

	buffer.WriteString("}")

	fmt.Printf("- query:\n%s\n", buffer.String())

	return shim.Success(buffer.Bytes())
}

func (s *CommonContract) count(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {
	if len(args) < 1 {
		return shim.Error("Incorrect number of arguments. At least 1 argument with query string should be set.")
	}
	query := args[0]

	resultsIterator, err := APIstub.GetQueryResult(query)

	if err != nil {
		return shim.Error(err.Error())
	}
	defer resultsIterator.Close()

	// buffer is a JSON array containing QueryResults
	count := 0

	for resultsIterator.HasNext() {
		_, err := resultsIterator.Next()
		if err != nil {
			return shim.Error(err.Error())
		}

		count = count + 1
	}

	return shim.Success([]byte(strconv.Itoa(count)))
}

func (s *CommonContract) exists(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {
	if len(args) < 1 {
		return shim.Error("Incorrect number of arguments. At least 1 argument with query string should be set.")
	}
	query := args[0]

	resultsIterator, err := APIstub.GetQueryResult(query)

	if err != nil {
		return shim.Error(err.Error())
	}
	defer resultsIterator.Close()

	var exists string
	if resultsIterator.HasNext() {
		exists = "true"
	} else {
		exists = "false"
	}

	return shim.Success([]byte(exists))
}

func (s *CommonContract) delete(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {
	if len(args) < 2 {
		return shim.Error("Incorrect number of arguments. Expecting 2")
	}

	class := args[0]
	key := args[1]

	objectType := "type~key"
	compositeKey, err := APIstub.CreateCompositeKey(objectType, []string{class, key})
	if err != nil {
		return shim.Error(err.Error())
	}
	err = APIstub.DelState(compositeKey)
	if err != nil {
		return shim.Error(err.Error())
	}
	return shim.Success(nil)
}

func (s *CommonContract) list(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {

	startKey := ""
	endKey := ""
	resultsIterator, err := APIstub.GetStateByRange(startKey, endKey)
	if err != nil {
		return shim.Error(err.Error())
	}
	defer resultsIterator.Close()

	// buffer is a JSON array containing QueryResults
	var buffer bytes.Buffer
	buffer.WriteString("[")

	bArrayMemberAlreadyWritten := false
	for resultsIterator.HasNext() {
		queryResponse, err := resultsIterator.Next()
		if err != nil {
			return shim.Error(err.Error())
		}

		// Add a comma before array members, suppress it for the first array member
		if bArrayMemberAlreadyWritten == true {
			buffer.WriteString(",")
		}
		buffer.WriteString("{\"Key\":")
		buffer.WriteString("\"")
		buffer.WriteString(queryResponse.Key)
		buffer.WriteString("\"")

		buffer.WriteString(", \"Record\":")
		// Record is a JSON object, so we write as-is
		buffer.WriteString(string(queryResponse.Value))
		buffer.WriteString("}")

		bArrayMemberAlreadyWritten = true
	}
	buffer.WriteString("]")

	fmt.Printf("- list:\n%s\n", buffer.String())

	return shim.Success(buffer.Bytes())
}

func (s *CommonContract) create(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {
	if len(args) < 3 {
		return shim.Error("Incorrect number of arguments. Expecting 2")
	}

	class := args[0]
	key := args[1]

	objectType := "type~key"
	compositeKey, err := APIstub.CreateCompositeKey(objectType, []string{class, key})
	if err != nil {
		return shim.Error(err.Error())
	}

	valueAsBytes := []byte(args[2])

	err = APIstub.PutState(compositeKey, valueAsBytes)
	if err != nil {
		return shim.Error("Can not create value")
	}

	return shim.Success(valueAsBytes)
}

func (s *CommonContract) update(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {
	if len(args) < 3 {
		return shim.Error("Incorrect number of arguments. Expecting 2")
	}

	class := args[0]
	key := args[1]

	objectType := "type~key"
	compositeKey, err := APIstub.CreateCompositeKey(objectType, []string{class, key})
	if err != nil {
		return shim.Error(err.Error())
	}

	err = APIstub.DelState(compositeKey)
	if err != nil {
		return shim.Error("Can not replace the old value")
	}

	valueAsBytes := []byte(args[2])

	err = APIstub.PutState(compositeKey, valueAsBytes)
	if err != nil {
		return shim.Error("Can not update the value")
	}
	return shim.Success(valueAsBytes)
}

func (s *CommonContract) get(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {
	if len(args) < 2 {
		return shim.Error("Incorrect number of arguments. Expecting 2")
	}

	class := args[0]
	key := args[1]

	objectType := "type~key"
	compositeKey, err := APIstub.CreateCompositeKey(objectType, []string{class, key})
	if err != nil {
		return shim.Error(err.Error())
	}
	valueAsBytes, err1 := APIstub.GetState(compositeKey)
	if err1 != nil {
		return shim.Error(err1.Error())
	}
	fmt.Printf("get: >%s", string(valueAsBytes))

	return shim.Success(valueAsBytes)
}

// The main function is only relevant in unit test mode. Only included here for completeness.
func main() {

	// Create a new Common Contract
	err := shim.Start(new(CommonContract))
	if err != nil {
		fmt.Printf("Error creating new Common Contract: %s", err)
	}
}

```

下载此[链码](https://github.com/ecsoya/spring-fabric-gateway/raw/master/spring-fabric-gateway/src/chaincode/common/chaincode.go)

##### 链码函数及参数说明

| 函数            | 参数 | 说明      |
| :-------------- | :-------------------  | :-------- |
| create | 1. `key` (唯一标识，必需)<br>2. `type`（数据类型，必需）| **创建**。 |
| get | 1. `key` (唯一标识，必需)<br>2. `type`（数据类型，必需）| **读取**。获取单个记录。 |
| update | 1. `key` (唯一标识，必需)<br>2. `type`（数据类型，必需）| **更新**。 |
| delete | 1. `key` (唯一标识，必需)<br>2. `type`（数据类型，必需）| **删除**。 |
| list | 无 | 查询所有的数据，不建议使用，仅供开发时少量数据的测试。 |
| query | 1. `selector` (CouchDB的JSON查询字符串，必需)<br>2. `pageSize` (分页查询页面大小， 非必需)<br>3. `bookmark` (分页查询定位符，非必需) | **查询**。第一个变量为必需，而且必需是使用了CouchDB的Fabric网络才有用。关于CouchDB的selector，请参考官方文档[CouchDB](http://docs.couchdb.org/en/stable/api/database/find.html?highlight=find#post--db-_find)。如果需要实现分页查询，则必需添加后面2个变量。 |
| count | 1. `selector` (CouchDB的JSON查询字符串，必需) | **数量统计**。必需是使用了CouchDB的Fabric网络才有用。关于CouchDB的selector，请参考官方文档[CouchDB](http://docs.couchdb.org/en/stable/api/database/find.html?highlight=find#post--db-_find)|
| exists | 1. `selector` (CouchDB的JSON查询字符串，必需) | **是否存在**。必需是使用了CouchDB的Fabric网络才有用。关于CouchDB的selector，请参考官方文档[CouchDB](http://docs.couchdb.org/en/stable/api/database/find.html?highlight=find#post--db-_find)|
| history | 1. `key` (唯一标识，必需)<br>2. `type`（数据类型，必需）| **查询数据的历史记录**，由于区块链的特殊性，所有的数据的添加、修改、删除都有历史记录，即便是删除，也会在区块链上留下痕迹。此方法会返回某个类型的数据的所有修改记录，包含交易id，修改时间等信息。|

##### 关于CompositeKey

```
// CreateCompositeKey combines the given `attributes` to form a composite
// key. The objectType and attributes are expected to have only valid utf8
// strings and should not contain U+0000 (nil byte) and U+10FFFF
// (biggest and unallocated code point).
// The resulting composite key can be used as the key in PutState().
CreateCompositeKey(objectType string, attributes []string) (string, error)
```

CompositeKey其实很好理解，就是将几个变量拼成一个特定的类型的ID，然后使用，只不过在进行查询的时候也要进行区别对待。

在此通用链码中，便使用了CompositeKey的概念，将数据的Key和Type组合成了唯一的ID，然后进行操作，在很大程度上重用了链码的功能。

### 五、FabricContext 介绍

FabricContext是对[Hyperledger Fabric Gateway SDK for Java](https://github.com/hyperledger/fabric-gateway-java)中的`Gateway`的包装，并提供了通用的读写操作方法。

[Hyperledger Fabric Gateway SDK for Java](https://github.com/hyperledger/fabric-gateway-java)的示例如下：

```
package org.example;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.concurrent.TimeoutException;

import org.hyperledger.fabric.gateway.Contract;
import org.hyperledger.fabric.gateway.ContractException;
import org.hyperledger.fabric.gateway.Gateway;
import org.hyperledger.fabric.gateway.Network;

public final class Sample {
    public static void main(String[] args) throws IOException {

        // Load an existing wallet holding identities used to access the network.
        Path walletDirectory = Paths.get("wallet");
        Wallet wallet = Wallets.newFileSystemWallet(walletDirectory);

        // Path to a common connection profile describing the network.
        Path networkConfigFile = Paths.get("connection.json");

        // Configure the gateway connection used to access the network.
        Gateway.Builder builder = Gateway.createBuilder()
                .identity(wallet, "user1")
                .networkConfig(networkConfigFile);

        // Create a gateway connection
        try (Gateway gateway = builder.connect()) {

            // Obtain a smart contract deployed on the network.
            Network network = gateway.getNetwork("mychannel");
            Contract contract = network.getContract("fabcar");

            // Submit transactions that store state to the ledger.
            byte[] createCarResult = contract.submitTransaction("createCar", "CAR10", "VW", "Polo", "Grey", "Mary");
            System.out.println(new String(createCarResult, StandardCharsets.UTF_8));

            // Evaluate transactions that query state from the ledger.
            byte[] queryAllCarsResult = contract.evaluateTransaction("queryAllCars");
            System.out.println(new String(queryAllCarsResult, StandardCharsets.UTF_8));

        } catch (ContractException | TimeoutException e) {
            e.printStackTrace();
        }
    }
}
```

调用FabricContext的示例如下：

```
	public FabricResponse execute(FabricRequest request) {
		try {
			logger.debug("Fabric execute " + request.function + " ==>");
			FabricResponse response = fabricContext.execute(request);
			if (response.isOk()) {
				logger.debug("Fabric execute " + request.function + " <== OK");
			} else {
				logger.debug("Fabric execute " + request.function + " <== FAILED, " + response.errorMsg);
			}
			return response;
		} catch (Exception e) {
			logger.error("Fabric execute " + request.function + " <==", e);
			return FabricResponse.fail(e.getMessage());
		}
	}
```

##### 关键信息梳理

不论是哪种调用方式，关键的信息有以下几点：

1. 用于初始化Fabric网络的**配置文件**，参考SDK中的`NetworkConfig`及项目[Fabric Network Builder](https://github.com/ecsoya/fabric-network-builder)。
2. 用于加载Wallet的**identify**。
3. 用于创建网络连接的**通道名称**（channel）。
4. 用于操作链码Chaincode的**链码名称**。
5. 用于读写操作的链码Chaincode中的**函数名**及**参数**。

##### 关于Request和Response

FabricRequest只是将函数名和参数做了简单的封装。

```
package io.github.ecsoya.fabric;

public class FabricRequest {

	public String function;
	public String[] arguments;

	public FabricRequest(String function, String... arguments) {
		this.function = function;
		this.arguments = arguments;
	}

	public void checkValidate() throws FabricException {
		if (function == null || function.equals("")) {
			throw new FabricException("The executable function name is empty.");
		}
	}
}
```

FabricResponse也只是对返回结果做了简单的封装。

```
package io.github.ecsoya.fabric;

import org.hyperledger.fabric.sdk.BlockEvent.TransactionEvent;

public class FabricResponse {

	public static final int SUCCESS = 1;
	public static final int FAILURE = -505;

	public final int status;

	public final String errorMsg;

	private String transactionId;

	public FabricResponse(int status, String errorMsg) {
		this.status = status;
		this.errorMsg = errorMsg;
	}

	public boolean isOk() {
		return status == SUCCESS;
	}

	public boolean isOk(boolean all) {
		return isOk();
	}

	public FabricResponse setTransactionId(String transactionId) {
		this.transactionId = transactionId;
		return this;
	}

	public String getTransactionId() {
		return transactionId;
	}

	public static FabricResponse fail(String errorMsg) {
		return new FabricResponse(FAILURE, errorMsg);
	}

	public static FabricResponse ok() {
		return new FabricResponse(SUCCESS, null);
	}

	public static FabricResponse create(TransactionEvent event) {
		if (event == null) {
			return fail("Invalid transaction event");
		}
		FabricResponse res = new FabricResponse(SUCCESS, null);
		res.setTransactionId(event.getTransactionID());
		return res;
	}

}

```

有兴趣的朋友可以了解一下`FabricQueryRequest`、`FabricQueryResponse`、`FabricQuery`和`FabricPagination`的实现。

### 六、IFabricInfoService

此服务提供了Fabric 区块链网络基本信息查询，包含区块链信息，区块信息，交易信息等。

```
package io.github.ecsoya.fabric.service;

import java.util.List;

import io.github.ecsoya.fabric.FabricPagination;
import io.github.ecsoya.fabric.FabricPaginationQuery;
import io.github.ecsoya.fabric.FabricQueryResponse;
import io.github.ecsoya.fabric.bean.FabricBlock;
import io.github.ecsoya.fabric.bean.FabricHistory;
import io.github.ecsoya.fabric.bean.FabricLedger;
import io.github.ecsoya.fabric.bean.FabricTransaction;
import io.github.ecsoya.fabric.bean.FabricTransactionRWSet;

/**
 * Default service to provided fabric blockchain info, such as blocks,
 * transactions and ledger.
 * 
 * @author ecsoya
 *
 */
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

### 七、IFabricService

此服务是基于[通用链码](https://github.com/ecsoya/spring-fabric-gateway/raw/master/spring-fabric-gateway/src/chaincode/common/chaincode.go)和通用API（FabricObject）的CRUD实现。

FabricObject是一个简单的通用Bean，只有`id`、`type`和`values`三个属性。

```
package io.github.ecsoya.fabric.bean;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

import lombok.Data;

/**
 * Common fabric object bean.
 * 
 * Using the CompositeKey with the id and type to identify a specific object.
 * 
 * @author ecsoya
 *
 */
@Data
public class FabricObject implements IFabricObject {

	private String id;

	private String type;

	private List<FabricQueryHistory> queryHistories;

	private Map<String, Object> values;

	@Override
	public String getType() {
		return type;
	}

	public void put(String key, Object value) {
		if (values == null) {
			values = new HashMap<String, Object>();
		}
		values.put(key, value);
	}
}

```

最终，将会以JSON的形式存储到Fabric区块链网络中。

### 八、服务扩展

可参考`IFabricBaseService`、`IFabricBlockService`、`IChaincodeService`等的实现。

### 参考文档

1. [Hyperledger Fabric Gateway SDK for Java](https://github.com/hyperledger/fabric-gateway-java)
2. [Java SDK for Hyperledger Fabric](https://github.com/hyperledger/fabric-sdk-java)
3. [Fabric 网络部署](https://ecsoya.github.io/fabric/pages/network.html)
4. [Fabric Network Config](https://ecsoya.github.io/fabric/pages/network-config.html)

* * *

[首页](http://ecsoya.github.io/fabric)
