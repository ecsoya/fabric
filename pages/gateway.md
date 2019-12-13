---
layout: default
---

## Spring Fabric Gateway

Spring Fabric Gateway 是基于Hyperledger官方的[fabric-gateway-java](https://github.com/hyperledger/fabric-gateway-java)工具的拓展。主要功能是将Chaincode（链码）的函数调用，通过Spring MVC的方式进行封装，简化使用。

### 前提条件

1. Hyperledger Fabric网络1.4及以上。（必须）
2. CouchDB，部分查询需要CouchDB支持。（非必须）
3. 安装通用链码（Common Chaincode），否则部分服务不可用。（非必须）

### 使用方法

第一步： 加载`fabric-gateway-spring-boot-starter`

```
<dependency>
	<groupId>io.github.ecsoya</groupId>
	<artifactId>fabric-gateway-spring-boot-starter</artifactId>
	<version>1.0.2</version>
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

### 配置参数介绍

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
|spring.fabric.network.file|Fabric网络配置文件路径|是|无|
|spring.fabric.network.name|Fabric网络名称，用于Fabric网络基本信息查询|是|无|

### 通用链码

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

| 函数&nbsp;             | 参数&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | 说明&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;       |
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

### FabricContext 介绍

### IFabricInfoService介绍

### IFabricService介绍



* * *

[返回](http://ecsoya.github.io/fabric)
