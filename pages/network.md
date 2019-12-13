---
layout: default
---

## Hyperledger Fabric区块链网络的几种部署方案

最近，在做一个区块链溯源的项目，有空学习了一下时下流行的区块链框架Hyperledger Fabric。和旨在打造公有链为主的比特币和以太坊相比，Hyperledger Fabric更多的是实现私有链和联盟链，更适合企业和联盟去打造区块链即服务的BAAS平台。

目前，各大云商（BAT，华为等）都推出了基于Fabric的区块链即服务平台（BAAS）。京东，蚂蚁区块链更是推出了基于Fabric的溯源平台。项目之初，在学习技术的同时，我也参考了大量的云平台的建设方案和使用费用，总体感觉就是，Fabric区块链网络的搭建需要巨大的硬件投入和资金支持，对于小公司和项目探索开发阶段来说，选择云BAAS平台将是笔不小的开销。今天有空，特意将我在开发阶段尝试过的几种Fabric区块链网络的搭建方法做个总结，希望可以帮到大家。

### 一、Build Your First Network

第一种方案，当属官方[fabric-samples](https://github.com/hyperledger/fabric-samples.git)里的`first-network`了，这个Demo里将各种操作都集合在了几个脚本文件中，基本上可以做到拿来即用，是Fabric学习和本地开发最直接的工具。

**具体的操作步骤，请参考如下内容**：

1. 修改`configtx.yaml`，配置组织（Organization），背书方案及排序方案等。
2. 修改`crypto-config.yaml`，配置节点（Peer）等账号相关信息。
3. 运行`byfn.sh generate`，生成证书和配置文件。
4. 运行`byfn.sh up`，创建Fabric区块链网络。

**需要注意的地方**：

1. 由于是通用的demo，通道默认为`mychannel`，域名默认为`example.com`，组织默认为`Org1..*`，如需修改，需要做大量的替换工作。
2. 智能合约（chaincode）的目录也用的是fabric-samples中的目录结构，如果需要在创建网络时安装智能合约，请勿忘记在`docker-compose-cli.yaml`中的cli节点下的`volumns`中修改映射目录。
3. 有时候初始化智能合约时，会出现`找不到_byfn网路`的错误，是因为在`peer-base.yaml`中配置的`CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_byfn`环境变量的问题，`${COMPOSE_PROJECT_NAME}`有时候会读不到，可以考虑硬编码，然后解决。
4. 在最近发布的版本中，加入了自动生成网络连接（`connection-*.yaml`或`connection-*.json`）的脚本，可以生成用来初始化NetworkConfig的配置文件。

### 二、多服务器部署方案

第二种方案，Altoros提供的多机（服务器）部署方案，即
[Ansible Fabric Starter](https://github.com/Altoros/Ansible-Fabric-Starter.git)。此方案使用了Ansible这个轻量的运维工具，实现了基于本地运行，然后在各个服务器上实现自动安装依赖和Fabric网络的功能。

**具体的操作步骤如下**：

1. 修改`host_xxx.yml`文件，xxx为不同的后缀名，具体用哪个根据你自己的业务决定。在这里可以定义域名、节点结每个节点所要部署的服务器的连接信息等。（所有的Fabric网络的节点信息都可以在这个文件中定义）
2. 本地运行`ansible-playbook install-dependencies.yml -i hosts_xxx.yml`，在各个服务器上安装所需要的依赖，如fabric的二进制文件，docker等。
3. 本地运行`ansible-playbook config-network.yml -i hosts_xxx.yml`，在各个服务器上安装Fabric网络。

**需要注意的地方**：

1. 所有的信息都在`host_xxx.yml`文件中配置，包括服务器的连接信息。Ansible会使用SSH工具来连接服务器，最好是配置密钥文件来连接。
2. 区块链的通用配置文件及证书等，会生成到本地`artifacts`目录下，可以自行甄别使用。
3. 可以通过修改`group_vars`目录下的`all.yml`文件，来修改一些网络的基本信息，如fabric的版本，网络的名称等。
4. 在各个服务器中的`root`文件夹中，会生成Fabric网络的配置文件及证书，可以下载到本地以便加载到项目中使用。

### 三、k8s多节点部署方案

第三种方案，使用Kubernetes容器服务，将Fabric网络安装到k8s容器中。参照[Cello Ansible agent](https://github.com/hyperledger/cello.git)中`src/agent/ansible`目录下的实现方案，通过Ansible自动运维工具来实现。

**具体实现方案如下**：

1. 将K8s的配置文件，下载到vars目录下，保存为`kubeconfig`。
2. 创建Fabric网络的配置文件，保存为`network.yml`。具体可参考已有参考的文件。
3. 修改`resource.yml`文件，配置各个节点的资源使用配置（内存，CPU等资源分配）。
4. 运行`ansible-playbook -i hosts -e "mode=apply env=network" setupfabric.yml`，创建Fabric网络。

**需要注意的地方**：

1. 运行ansible脚本时注意，`env`变量即Fabric网络配置文件的名称。
2. `resource.yml`是对k8s容器服务器的资源分配，如果分配过大，会导致某些节点不能创建，所以要特别注意。另外，服务器的配置要求一般为8核16G或更高。
3. 运行`ansible-playbook -i hosts -e "mode=destroy env=network" setupfabric.yml`，即可删除已创建的fabric网络。
4. 在配置Fabric网络时，其中有一项为` storageclass`，此项为服务器所支持的存储类型的名称，如腾讯云为`cbs`，你也可以自己创建及命名，一般是按需收费。

以上这些，就是我在搭建Fabric区块链网络时，所用过的一些方法及总结，有不到之处，希望大家能够指正。

* * *

[首页](http://ecsoya.github.io/fabric)
