---
layout: default
---

## Fabric 完整示例

### 2.0.x更新

1. 此demo已更新到Hyperledger Fabric v2.x版本。
2. 使用[[test-network]](https://hyperledger-fabric.readthedocs.io/en/latest/test_network.html)搭建测试网络。

第一步、 搭建基础网络

`./network.sh up createChannel -c mychannel -s couchdb`

第二步、部署通用链码（go版本） 

`./network.sh deployCC -ccn mycc -ccv 1.0 -ccp ../src/chaincode/common -ccl go`

或、部署通用链码（Java版本）

`./network.sh deployCC -ccn commoncc -ccv 1.0 -ccp ../src/chaincode-java -ccl java`

[Java链码地址](https://github.com/ecsoya/spring-fabric-gateway/tree/master/spring-fabric-gateway-chaincode)

*在部署Java链码时，需要注意：1、如果是Maven开发的，需要将链码打包成jar包，具体的参见源码。2、将链码Jar包拷贝到链码安装目录下的/build/install/链码名称目录下，然后执行上述命令。具体操作请参考demo源码*

### 一、简介

1. 源码：[https://github.com/ecsoya/fabric-demo](https://github.com/ecsoya/fabric-demo)
2. 功能：本地Fabric网络搭建、`fabric-gateway-spring-boot-starter`及`fabric-explorer-spring-boot-starter`使用。

### 二、Spring Boot 开始

创建一个Spring Boot Maven工程，并导入`fabric-explorer-spring-boot-starter`相关jar包。

[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.github.ecsoya/spring-fabric-gateway/badge.svg?style=plastic)](https://maven-badges.herokuapp.com/maven-central/io.github.ecsoya/spring-fabric-gateway)

```xml
<dependency>
	<groupId>io.github.ecsoya</groupId>
	<artifactId>fabric-explorer-spring-boot-starter</artifactId>
	<version>${latest_version}</version>
</dependency>
```

### 三、Fabric 网络搭建 v1.4.x

1- 运行`first-network`文件夹中的`byfn.sh`脚本，此方法为官方的[fabric-samples](https://github.com/hyperledger/fabric-samples.git)中的示例，做了少量修改。

```sh
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

```yml
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

### 六、通用链码CRUD

由于此示例中安装了[通用链码](https://github.com/ecsoya/spring-fabric-gateway/raw/master/spring-fabric-gateway/src/chaincode/common/chaincode.go)，所以能够使用[IFabricService](https://ecsoya.github.io/fabric/pages/gateway.html)来创建基于Fabric区块链的CRUD服务。

1- **FabricDemoController** 

实现基本的CRUD

```java
package io.github.ecsoya.demo.controller;

import javax.servlet.http.HttpServletRequest;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.ModelAndView;

import io.github.ecsoya.fabric.FabricPagination;
import io.github.ecsoya.fabric.FabricPaginationQuery;
import io.github.ecsoya.fabric.FabricResponse;
import io.github.ecsoya.fabric.bean.FabricObject;
import io.github.ecsoya.fabric.service.IFabricService;

@RestController
public class FabricDemoController {

	@Autowired
	private IFabricService fabricService;

	@RequestMapping("/")
	public ModelAndView index(HttpServletRequest request) {
		ModelAndView modelAndView = new ModelAndView("index");
		modelAndView.addObject("baseURL", baseUrl(request));
		return modelAndView;
	}

	private String baseUrl(HttpServletRequest request) {
		String scheme = request.getScheme();
		String serverName = request.getServerName();
		int port = request.getServerPort();
		String path = request.getContextPath();
		return scheme + "://" + serverName + ":" + port + path;
	}

	@RequestMapping("/list")
	public FabricPagination<FabricObject> list(FabricPaginationQuery<FabricObject> query) {
		return fabricService.pagination(query);
	}

	@RequestMapping("/add")
	public FabricResponse add(FabricObject object) {
		return fabricService.create(object);
	}

	@RequestMapping("/update")
	public FabricResponse update(FabricObject object) {
		return fabricService.update(object);
	}

	@RequestMapping("/remove")
	public FabricResponse remove(FabricObject object) {
		return fabricService.delete(object.getId(), object.getType());
	}
}
```

2- 关于**删除**

虽然说区块链是不可篡改的账本，但不是说区块链上的数据就不可删除。其实，Fabric区块链的链码，是支持**删除**操作的，删除之后便不能再被查询。但是，这里的**删除**，并不是物理删除，只是给数据打上了一个**is_delete**的标记，因为所有的操作，都会有对应的交易ID，所以数据即便删除了，仍然在区块链的账本上留下记录。

3- 分页查询

Fabric中的分页查询，是通过*bookmark*标记来实现的。

在链码的API中，提供了如下的查询方法：

```go
APIstub.GetQueryResultWithPagination(query, pageSize, bookmark);
```

**bookmark**是当前查询之后，返回的一个标记，下一次的查询会从这个标记开始。如果是第一次（第一页）查询，可以使用空字符串（""）代替。

在前段实现时，如果要实现自由切换页码，可以将**页码**和**bookmark**缓存到map中进行调用。

以[DataTables](https://datatables.net/download)为例：

```js
//将页码和bookmark缓存
let bookmarkMap = new Map();

let dataTable = $('#dataTable')
	.DataTable(
		{
			"ajax" : function(data, callback) {
			let pageSize = data.length; //每页大小
			let currentPage = data.start / data.length; //当前页
			$.ajax({
				url : baseURL + "/list",
				data : {
					bookmark : bookmarkMap[currentPage], //当前页的bookmark值
					pageSize : pageSize,
					currentPage : currentPage,
					type : $('#type').val()
					}
				}).then(function success(res) {
					bookmarkMap[currentPage + 1] = res.bookmark; //将查询结果中的bookmark值，作为下一页的bookmark
					callback(res); // 调用DataTable的callback函数初始化表。
				}, function fail(data, status) {

				});
			},
			……
```

### 参考文档

1. [Hyperledger Fabric Gateway SDK for Java](https://github.com/hyperledger/fabric-gateway-java)
2. [Java SDK for Hyperledger Fabric](https://github.com/hyperledger/fabric-sdk-java)
3. [Fabric 网络部署](https://ecsoya.github.io/fabric/pages/network.html)
4. [Fabric Network Config](https://ecsoya.github.io/fabric/pages/network-config.html)
5. [Spring Fabric Gateway](https://ecsoya.github.io/fabric/pages/gateway.html)
6. [Spring Fabric Explorer](https://ecsoya.github.io/fabric/pages/explorer.html)

* * *

[首页](http://ecsoya.github.io/fabric)
