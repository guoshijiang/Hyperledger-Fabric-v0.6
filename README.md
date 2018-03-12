# Fabric_code_explain_0.6

##  项目研究内容

本项目是一个代码解析项目，主要解析fabric的几大模块以及整个项目的架构，本项目主要研究一下几个问题 
* hypledger fabric的安全机制以及证书体系 
* Hyperledger fabric的共识机制  
* Hyperledger fabric的存储机制 
* hyperledger fabric包含的网络协议
* hyperledger fabric包含依赖包解析（例如gRPC）

## Ubuntu下搭建Hyperledger fabric开发环境(一个节点)

### 一、安装docker

#### 1、docker要求Linux内核版本不低于 3.10
* 检查Linux的内核版本,如果内核版本太低，升级内核
* 查看内核的版本命令 `uname -a` 

#### 2、根据不同的Ubuntu版本安装docker

* 查看Ubuntu版本命令 `lsb_release -a`

#### 3、对于16.04的Ubuntu版本安装

    sudo apt-get install docker-engine

#### 4、启动

    sudo systemctl enable docker
    sudo systemctl start docker

### 二、从docker上拉取镜像

#### 1、检验docker是否安装好

    docker --help

#### 2、从docker上拉镜像

    docker pull hyperledger/fabric-peer:latest
    docker pull hyperledger/fabric-membersrvc:latest

#### 3、校验镜像是否拉取完成

    docker images

### 三、安装docker-compose项目

#### 1、安装pip工具
pip工具会依赖Python，而Ubuntu下默认已经安转好Python2.7
      
    apt-get install python-pip

#### 2、安转docker compose项目

    sudo pip install -U docker-compose

#### 3、校验docker compose是否安装好
    docker-compose -h

### 四、安装并配置nsenter工具
 方法一、
 
     wget https://www.kernel.org/pub/linux/utils/util-linux/v2.29/util-linux-2.29.tar.xz; tar xJvf util-linux-2.29.tar.xz
     cd util-linux-2.29
     ./configure --without-ncurses && make nsenter
     sudo cp nsenter /usr/local/bin

方法二、建议下载 .bashrc_docker，并将内容放到 .bashrc 中
        
    wget -P ~ https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker;
     echo "[ -f ~/.bashrc_docker ] && . ~/.bashrc_docker" >> ~/.bashrc; source ~/.bashrc

### 五、启动节点

#### 1、在root的Home目录下创建Docker-compose.yml并写入一下内容

    membersrvc:
       image: hyperledger/fabric-membersrvc
     ports:
       - "7054:7054"
     command: membersrvc
       vp0:
     image: hyperledger/fabric-peer
    ports:
      - "7050:7050"
      - "7051:7051"
      - "7053:7053"
    environment:
      - CORE_PEER_ADDRESSAUTODETECT=true
      - CORE_VM_ENDPOINT=unix:///var/run/docker.sock
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_ID=vp0
      - CORE_PEER_PKI_ECA_PADDR=membersrvc:7054
      - CORE_PEER_PKI_TCA_PADDR=membersrvc:7054
      - CORE_PEER_PKI_TLSCA_PADDR=membersrvc:7054
      - CORE_SECURITY_ENABLED=true
      - CORE_SECURITY_ENROLLID=test_vp0
      - CORE_SECURITY_ENROLLSECRET=MwYpmSRjupbT
    links:
      - membersrvc
    command: sh -c "sleep 5; peer node start --peer-chaincodedev"

#### 2、通过`docker-compose up`运行已近配置好的节点

### 六、进入容器
#### 1、`docker -ps` 找到要进入的容器的Container ID
#### 2、用`docker-pid`指令获取需要进入容器的PID
    echo PID=(docker-pid  b4378c920828)
#### 3、借助PID进入容器
    sudo nsenter --target 10981 --mount --uts --ipc --net --pid

### 七、测试环境是否搭建好
#### 1、选择源码中的一个例子chaincode机型编译
    cd $GOPATH/src/github.com/hyperledger/fabric/examples/chaincode/Go/chaincode_example02
    go build
    
#### 2、注册和运行chaincode
    CORE_CHAINCODE_ID_NAME=mycc CORE_PEER_ADDRESS=0.0.0.0:7051 ./chaincode_example02

#### 3、另起一个终端，进入容器的方式和“六”一样，以WebAppAdmin的形式登录到节点上
    peer network login WebAppAdmin -p DJY27pEnl16d
#### 4、执行下面的代码，在另外一个终端里面会看到执行结果
安全模式
#### 1. 部署交易
    CORE_SECURITY_ENABLED=true CORE_SECURITY_PRIVACY=true peer chaincode deploy -u WebAppAdmin -n mycc -c '{"Function":"init", "Args": ["a","100", "b", "200"]}'
#### 2.调用交易
    CORE_SECURITY_ENABLED=true CORE_SECURITY_PRIVACY=true peer chaincode invoke -u WebAppAdmin -l golang -n mycc -c '{"Function": "invoke", "Args": ["a", "b", "10"]}'
#### 3.查询交易
    CORE_SECURITY_ENABLED=true CORE_SECURITY_PRIVACY=true peer chaincode query -u WebAppAdmin -l golang -n mycc -c '{"Function": "query", "Args": ["b"]}'

## Ubuntu下搭建Hyperledger fabric开发环境(四个节点)

### 一、安装Docker

#### 1、docker要求Linux内核版本不低于3.10

检查Linux的内核版本,如果内核版本太低，升级内核

查看内核的版本命令`uname-a`

#### 2、根据不同的Ubuntu版本安装docker

查看Ubuntu版本命令`lsb_release-a`

#### 3、对于16.04的Ubuntu版本安装

    sudo apt-get installdocker-engine

#### 4、启动

    sudosystemctl enabledocker
    sudosystemctl startdocke

### 二、安装docker-compose项目

#### 1、安装pip工具

pip工具会依赖Python，而Ubuntu下默认已经安转好Python2.7

    apt-get install python_pip

#### 2、安转dockercompose项目

    sudo pip install -Udocker-compose

#### 3、校验dockercompose是否安装好

    docker-compose -h

### 三、获取镜像

获取镜像的方式有4种

#### 1.通过Dockerfile来定制镜像(虽然推荐使用这种方法，但这里并不介绍；如果要定制镜像，你得去学习docker)

#### 2.使用社区镜像

    docker pull hyperledger/fabric-peer:x86_64-0.6.1-preview \&& 
    docker pull hyperledger/fabric-membersrvc:x86_64-0.6.1-preview \&&
    docker pull yeasy/blockchain-explorer:latest \ && 
    docker tag hyperledger/fabric-peer:x86_64-0.6.1-preview hyperledger/fabric-peer \&& 
    docker tag hyperledger/fabric-peer:x86_64-0.6.1-preview hyperledger/fabric-baseimage \&& 
    docker tag hyperledger/fabric-membersrvc:x86_64-0.6.1-preview hyperledger/fabric-membersrvc

#### 3.使用IBM认证的镜像

    docker pull ibmblockchain/fabric-peer:x86_64-0.6.1-preview \&& 
    docker pull ibmblockchain/fabric-membersrvc:x86_64-0.6.1-preview \&&
    docker pull yeasy/blockchain-explorer:latest \&& 
    docker tag ibmblockchain/fabric-peer:x86_64-0.6.1-preview hyperledger/fabric-peer \&& 
    docker tag ibmblockchain/fabric-peer:x86_64-0.6.1-preview hyperledger/fabric-baseimage \&& 
    docker tag ibmblockchain/fabric-membersrvc:x86_64-0.6.1-preview hyperledger/fabric-membersrvc

#### 4.从github上获取

    docker pull yeasy/hyperledger-fabric-base:0.6-dp \&& 
    docker pull yeasy/hyperledger-fabric-peer:0.6-dp \&& 
    docker pull yeasy/hyperledger-fabric-membersrvc:0.6-dp \&& 
    docker pull yeasy/blockchain-explorer:latest \&& 
    docker tag yeasy/hyperledger-fabric-peer:0.6-dp hyperledger/fabric-peer \&& 
    docker tag yeasy/hyperledger-fabric-base:0.6-dp hyperledger/fabric-baseimage \&& 
    docker tag yeasy/hyperledger-fabric-membersrvc:0.6-dp hyperledger/fabric-membersrvc

### 四.准备配置文件

有两种方法

#### 1.自己写(这里不介绍)

#### 2.从github上获取

    git clone https://github.com/yeasy/docker-compose-files

### 五.启动节点

现在我们假设我们是从github上获取的配置文件

#### 1.进入pbft配置文件所在的目录，你可以看到这些文件


这是pbft目录下的文件，`4-peers-with-explorer.yml`没有成员服务的可在浏览器端访问4个节点配置文件；`4-peers-with-membersrvc.yml`带有成员服务的4个节点配置文件；`4-peers-with-membersrvc-explorer.yml`四个带有成员服务的节点配置文件；`4-peers.yml`其普通的四个节点，没有成员服务，部署交易的时候在非安全模式下部署；`explorer.yml`可浏览器端访问的基础配置文件、`peer.yml`节点基础配置文件、`membersrvc.yml`成员服务基础配置文件，其他文件都得继承他们。同样的在noops目录下也会有这些文件只是他们启动起来使用的共识插件是noops。

### 六.启动节点

    docker-compose -f 4-peers.yml up ---->非安全模式
    docker-compose -f 4-peers-with-membersrvc.yml up --->安全模式
    docker-compose -f 4-peers-with-explorer.yml up --->通过浏览器的方式
    docker-compose -f 4-peers-with-membersrvc-explorer.yml up --->安全模式下可通过浏览器访问的方式

### 七.另起一个终端，进入其中的一个节点

    docker exec -it spbft_vp1_1 bash
    docker exec -it hyperledger_vp3_1 bash
    docker exec -it pbft_vp0_1 bash

spbft_vp1_1, hyperledger_vp3_1,pbft_vp0_1z这些是你用docker ps命令查看到的容器的名字，这里你可能需要更换成你自己的docker容器的名字

### 八.执行交易

进入容器后，编译chaincode_example02后，执行下面的的代码，在启动窗口中你可以看到共识的debug过程

以非安全模式部署智能合约
        
    peer chaincode deploy -p github.com/hyperledger/fabric/examples/chaincode/Go/chaincode_example02 -c '{"Function":"init", "Args": ["a","100", "b", "200"]}'

调用交易

    peer chaincode invoke -n github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Function": "invoke", "Args": ["a", "b", "10"]}'

### 九.安全模式下调用交易

你需要先登录一下（下面以WebAppAdmin为例），以WebAppAdmin的形式登录到节点上
    
    peer network login WebAppAdmin -p DJY27pEnl16d

安全模式
    
    peer chaincode deploy -u jim -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Function":"init", "Args": ["a","100", "b", "200"]}'



## 链码接口

#### Chaincode接口必须被所有的链上代码实现,fabric运行交易通过调用这些指定的函数

    type Chaincode interface {
	    // 在容器建立连接之后再部署交易期间调用Init函数，准许链上代码初始化内部数据
	    Init(stub ChaincodeStubInterface, function string, args []string) ([]byte, error)

	    // 每次调用交易都会调用Invoke接口. 链上代码可能会改变状态变量
	    Invoke(stub ChaincodeStubInterface, function string, args []string) ([]byte, error)

	    // 查询交易时调用Query接口. 链上代码仅仅读（而不是修改）它的状态变量并其返回结果
	    Query(stub ChaincodeStubInterface, function string, args []string) ([]byte, error)
	    }

#### ChaincodeStubInterface用来部署链上代码apps来进入和修改他们的账本
	type ChaincodeStubInterface interface {
		// Get the arguments to the stub call as a 2D byte array
		// 获取stub调用的参数来作为一个2维字节数组
		GetArgs() [][]byte

		// Get the arguments to the stub call as a string array
		// 获取stub调用的参数来作为一个字符数组
		GetStringArgs() []string

		// 获取交易的ID
		GetTxID() string

		// InvokeChaincode 本地调用指定的链上代码，`Invoke`使用相同的交易，也就说链上代码调用
		// 链上代码不会创建一个新的交易消息
		InvokeChaincode(chaincodeName string, args [][]byte) ([]byte, error)

		// InvokeChaincode 本地调用指定的链上代码，`Query`使用相同的交易，也就说链上代码调用
		// 链上代码不会创建一个新的交易消息
		QueryChaincode(chaincodeName string, args [][]byte) ([]byte, error)

		// GetState通过Key来返回数组的特定值
		GetState(key string) ([]byte, error)

		// PutState向账本中写入特定的键和值
		PutState(key string, value []byte) error

		// DelState从账本中移除指定的键和值
		DelState(key string) error

		// RangeQueryState函数可以通过chaincode调用来查询状态范围内的键。假设startKey和endKey
		// 在词典中，将返回一个迭代器，它可以用来遍历startKey和endKey之间的所有键。
		// 迭代器返回键的顺序是随机的。
		RangeQueryState(startKey, endKey string) (StateRangeQueryIteratorInterface, error)

		// CreateTable创建一张新表，给出表名和列定义
		CreateTable(name string, columnDefinitions []*ColumnDefinition) error

		// GetTable如果表存在，返回指定的一张表，如果表不存在ErrTableNotFound错误
		GetTable(tableName string) (*Table, error)

		// DeleteTable删除表和实体相关的所有行
		DeleteTable(tableName string) error

		// InsertRow 插入一个新行进入指定的表
		// 返回 -
		// 如果行成功插入返回true和no error
		// 如果已经存在给定的行就返回false和no error
		// 如果指定的表名不存在返回false和TableNotFoundError
		// 如果出现一个没有预料到的错误条件返回false和error
		InsertRow(tableName string, row Row) (bool, error)

		// ReplaceRow 在指定的表中更新行.
		// 返回 -
		// 如果行成功更新就返回false和no error
		// 如果给出的行不存在相应的键就返回false和no error
		// 如果指定的表名不存在返回false和TableNotFoundError
		// 如果出现一个没有预料到的错误条件返回false和error
		ReplaceRow(tableName string, row Row) (bool, error)

		// 通过键从指定的表中获取行
		GetRow(tableName string, key []Column) (Row, error)

		// 基于特定的key来返回多行。例如，给出表| A | B | C | D |，A,C和D是键，可以使用[A，C]调用GetRows来返回所有具有A，
		// C和任何D值的行作为它们的键。 GetRows也可以用A调用，返回所有具有A和C和D作为其键值的行。
		GetRows(tableName string, key []Column) (<-chan Row, error)

		//DeleteRow从指定的表中通过key来删除特定的行
		DeleteRow(tableName string, key []Column) error

		// ReadCertAttribute用来从交易证书中读取指定的属性
		// *attributeName* 是这个函数的入参.
		// 例如:
		//  attrValue,error:=stub.ReadCertAttribute("position")
		ReadCertAttribute(attributeName string) ([]byte, error)

		// VerifyAttribute用于验证事务证书是否具有名称为* attribute Name *和value * attributeValue *的属性，
		// attributeName和attributeValue是此函数接收的输入参数
		// 例如:
		//    containsAttr, error := stub.VerifyAttribute("position", "Software Engineer")
		VerifyAttribute(attributeName string, attributeValue []byte) (bool, error)

		// VerifyAttributes与VerifyAttribute相同，但它检查属性列表及其相应的值，而不是单个属性/值对
		// 例如:
		//containsAttrs, error:= stub.VerifyAttributes(&attr.Attribute{"position",  "Software Engineer"}, &attr.Attribute{"company", "ACompany"})
		VerifyAttributes(attrs ...*attr.Attribute) (bool, error)

		// VerifySignature核实交易的签名，如果正确返回true,否则返回false
		VerifySignature(certificate, signature, message []byte) (bool, error)

		// GetCallerCertificate 返回调用者证书
		GetCallerCertificate() ([]byte, error)

		// GetCallerMetadata 返回调用方元数据
		GetCallerMetadata() ([]byte, error)

		// GetBinding 返回交易捆绑
		GetBinding() ([]byte, error)

		// GetPayload 返回交易的payload, payload是一个定义在fabric/protos/chaincode.proto
		// 中的ChaincodeSpecwhich
		GetPayload() ([]byte, error)

		// GetTxTimestamp返回交易创建的时间戳，这个时间戳是peer收到交易的当前时间。
		// 请注意，此时间戳可能与其他对等端peer的时间不同
		GetTxTimestamp() (*timestamp.Timestamp, error)

		// SetEvent保存当交易组成一个块时要发送的事件
		SetEvent(name string, payload []byte) error
		}

#### StateRangeQueryIteratorInterface允许在一个链上代码在状态上迭代一定范围的键值
	
	type StateRangeQueryIteratorInterface interface {

		// HasNext如果查询迭代器范围内包含额外的键和值就返回true
		HasNext() bool

		// Next在迭代器范围内返回下一个键和值
		Next() (string, []byte, error)

		// Close 关闭范围查询迭代器，当从迭代器中读完被释放资源的时候被调用
		Close() error
	}

#### chaincode_example02解析
	
	package main
	import (
		"errors"
		"fmt"
		"strconv"

		"github.com/hyperledger/fabric/core/chaincode/shim"
	)

	// SimpleChaincode 样例链上代码实现
	type SimpleChaincode struct {
	}

	func (t *SimpleChaincode) Init(stub shim.ChaincodeStubInterface, function string, args []string) ([]byte, error) {
		var A, B string    // 字符实体
		var Aval, Bval int // 资产持股
		var err error

		if len(args) != 4 {
			return nil, errors.New("Incorrect number of arguments. Expecting 4")
		}

		// 初始化链上代码
		A = args[0]
		Aval, err = strconv.Atoi(args[1])
		if err != nil {
			return nil, errors.New("Expecting integer value for asset holding")
		}
		B = args[2]
		Bval, err = strconv.Atoi(args[3])
		if err != nil {
			return nil, errors.New("Expecting integer value for asset holding")
		}
		fmt.Printf("Aval = %d, Bval = %d\n", Aval, Bval)

		// 写状态到账本
		err = stub.PutState(A, []byte(strconv.Itoa(Aval)))
		if err != nil {
			return nil, err
		}

		err = stub.PutState(B, []byte(strconv.Itoa(Bval)))
		if err != nil {
			return nil, err
		}

		return nil, nil
	}

	// 支持从A到B支付X股
	func (t *SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface, function string, args []string) ([]byte, error) {
		if function == "delete" {
			// Deletes an entity from its state
			// 从他的状态中删除entity
			return t.delete(stub, args)
		}

		var A, B string    // 字符实体
		var Aval, Bval int // 持股资产
		var X int          // 交易值
		var err error

		if len(args) != 3 {
			return nil, errors.New("Incorrect number of arguments. Expecting 3")
		}

		A = args[0]
		B = args[1]

		// 从账本中获取状态
		// TODO: will be nice to have a GetAllState call to ledger
		Avalbytes, err := stub.GetState(A)
		if err != nil {
			return nil, errors.New("Failed to get state")
		}
		if Avalbytes == nil {
			return nil, errors.New("Entity not found")
		}
		Aval, _ = strconv.Atoi(string(Avalbytes))

		Bvalbytes, err := stub.GetState(B)
		if err != nil {
			return nil, errors.New("Failed to get state")
		}
		if Bvalbytes == nil {
			return nil, errors.New("Entity not found")
		}
		Bval, _ = strconv.Atoi(string(Bvalbytes))

		// 执行execution
		X, err = strconv.Atoi(args[2])
		if err != nil {
			return nil, errors.New("Invalid transaction amount, expecting a integer value")
		}
		Aval = Aval - X
		Bval = Bval + X
		fmt.Printf("Aval = %d, Bval = %d\n", Aval, Bval)

		// 写状态到账本
		err = stub.PutState(A, []byte(strconv.Itoa(Aval)))
		if err != nil {
			return nil, err
		}

		err = stub.PutState(B, []byte(strconv.Itoa(Bval)))
		if err != nil {
			return nil, err
		}

		return nil, nil
	}

	// 从账本中删除实体
	func (t *SimpleChaincode) delete(stub shim.ChaincodeStubInterface, args []string) ([]byte, error) {
		if len(args) != 1 {
			return nil, errors.New("Incorrect number of arguments. Expecting 1")
		}

		A := args[0]

		// 从账本的状态中删除密钥
		err := stub.DelState(A)
		if err != nil {
			return nil, errors.New("Failed to delete state")
		}

		return nil, nil
	}

	// Query callback representing the query of a chaincode
	// Query 查询链上代码
	func (t *SimpleChaincode) Query(stub shim.ChaincodeStubInterface, function string, args []string) ([]byte, error) {
		if function != "query" {
			return nil, errors.New("Invalid query function name. Expecting \"query\"")
		}
		var A string // 字符实体
		var err error

		if len(args) != 1 {
			return nil, errors.New("Incorrect number of arguments. Expecting name of the person to query")
		}

		A = args[0]

		// 从账本中获取状态
		Avalbytes, err := stub.GetState(A)
		if err != nil {
			jsonResp := "{\"Error\":\"Failed to get state for " + A + "\"}"
			return nil, errors.New(jsonResp)
		}

		if Avalbytes == nil {
			jsonResp := "{\"Error\":\"Nil amount for " + A + "\"}"
			return nil, errors.New(jsonResp)
		}

		jsonResp := "{\"Name\":\"" + A + "\",\"Amount\":\"" + string(Avalbytes) + "\"}"
		fmt.Printf("Query Response:%s\n", jsonResp)
		return Avalbytes, nil
	}

	func main() {
		// ChainCode 调用 err := shim.Start(new(SimpleChaincode))
		// 接入到ChainCodeSupportServer
		err := shim.Start(new(SimpleChaincode))
		if err != nil {
			fmt.Printf("Error starting Simple chaincode: %s", err)
		}
	}
### peer启动过程的源码分析
	func serve(args []string) error {
		// Parameter overrides must be processed before any paramaters are
		// cached. Failures to cache cause the server to terminate immediately.
		//在其他参数被缓存起来之前参数覆盖必须处理，失败缓存导致服务立即结束
		if chaincodeDevMode {
			logger.Info("Running in chaincode development mode")
			logger.Info("Set consensus to NOOPS and user starts chaincode")
			logger.Info("Disable loading validity system chaincode")

			viper.Set("peer.validator.enabled", "true")
			viper.Set("peer.validator.consensus", "noops")
			viper.Set("chaincode.mode", chaincode.DevModeUserRunsChaincode)

		}
		// 调用CacheConfiguration() 函数设置缓存数据，缓存数据包括该peer的LocalAdress、
		// PeerEndpoint（是VP or NVP）
		if err := peer.CacheConfiguration(); err != nil {
			return err
		}

		peerEndpoint, err := peer.GetPeerEndpoint()
		if err != nil {
			err = fmt.Errorf("Failed to get Peer Endpoint: %s", err)
			return err
		}

		// 启动grpc服务，监听7051端口
		// 设置服务器地址，创建服务器实例，后续代码会使用lis
		listenAddr := viper.GetString("peer.listenAddress")

		if "" == listenAddr {
			logger.Debug("Listen address not specified, using peer endpoint address")
			listenAddr = peerEndpoint.Address
		}

		lis, err := net.Listen("tcp", listenAddr)
		if err != nil {
			grpclog.Fatalf("Failed to listen: %v", err)
		}
		//创建EventHub服务器，通过调用createEventHubServer方法实现，该服务也是grpc,只有VP
		//才能开启
		ehubLis, ehubGrpcServer, err := createEventHubServer()
		if err != nil {
			grpclog.Fatalf("Failed to create ehub server: %v", err)
		}

		logger.Infof("Security enabled status: %t", core.SecurityEnabled())
		if viper.GetBool("security.privacy") {
			if core.SecurityEnabled() {
				logger.Infof("Privacy enabled status: true")
			} else {
				panic(errors.New("Privacy cannot be enabled as requested because security is disabled"))
			}
		} else {
			logger.Infof("Privacy enabled status: false")
		}
		//启动rockdb数据库
		db.Start()

		var opts []grpc.ServerOption
		if comm.TLSEnabled() {
			creds, err := credentials.NewServerTLSFromFile(viper.GetString("peer.tls.cert.file"),
				viper.GetString("peer.tls.key.file"))

			if err != nil {
				grpclog.Fatalf("Failed to generate credentials %v", err)
			}
			opts = []grpc.ServerOption{grpc.Creds(creds)}
		}

		//创建一个grpc服务
		grpcServer := grpc.NewServer(opts...)

		//注册Chaincode支持服务器
		secHelper, err := getSecHelper()
		if err != nil {
			return err
		}

		secHelperFunc := func() crypto.Peer {
			return secHelper
		}

		registerChaincodeSupport(chaincode.DefaultChain, grpcServer, secHelper)

		var peerServer *peer.Impl

		// 创建peer服务器，主意区分VP和NVP节点
		if peer.ValidatorEnabled() {
			logger.Debug("Running as validating peer - making genesis block if needed")
			makeGenesisError := genesis.MakeGenesis()
			if makeGenesisError != nil {
				return makeGenesisError
			}
			logger.Debugf("Running as validating peer - installing consensus %s",
				viper.GetString("peer.validator.consensus"))

			// 网络初始化的过程中执行以下内容，在创建节点Engine过程中该节点作为客户端的身份
			// 连接到其他Peer
			peerServer, err = peer.NewPeerWithEngine(secHelperFunc, helper.GetEngine)
		} else {
			logger.Debug("Running as non-validating peer")
			peerServer, err = peer.NewPeerWithHandler(secHelperFunc, peer.NewPeerHandler)
		}

		if err != nil {
			logger.Fatalf("Failed creating new peer with handler %v", err)

			return err
		}

		// 注册peer服务
		pb.RegisterPeerServer(grpcServer, peerServer)

		// 注册管理服务器
		pb.RegisterAdminServer(grpcServer, core.NewAdminServer())

		// 注册Devops服务器
		// 在Peer节点初始化的时候 创建DevopsServer
		serverDevops := core.NewDevopsServer(peerServer)
		pb.RegisterDevopsServer(grpcServer, serverDevops)

		// 注册ServerOpenchain服务器
		serverOpenchain, err := rest.NewOpenchainServerWithPeerInfo(peerServer)
		if err != nil {
			err = fmt.Errorf("Error creating OpenchainServer: %s", err)
			return err
		}

		pb.RegisterOpenchainServer(grpcServer, serverOpenchain)

		// 如果配置了的话创建和注册REST服务
		if viper.GetBool("rest.enabled") {
			go rest.StartOpenchainRESTServer(serverOpenchain, serverDevops)
		}

		logger.Infof("Starting peer with ID=%s, network ID=%s, address=%s, rootnodes=%v, validator=%v",
			peerEndpoint.ID, viper.GetString("peer.networkId"), peerEndpoint.Address,
			viper.GetString("peer.discovery.rootnode"), peer.ValidatorEnabled())

		// 启动GRPC服务器. 如果是必须的话在一个goroutine中完成这样我们能够部署genesis
		serve := make(chan error)

		sigs := make(chan os.Signal, 1)
		signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)
		go func() {
			sig := <-sigs
			fmt.Println()
			fmt.Println(sig)
			serve <- nil
		}()

		go func() {
			var grpcErr error
			if grpcErr = grpcServer.Serve(lis); grpcErr != nil {
				grpcErr = fmt.Errorf("grpc server exited with error: %s", grpcErr)
			} else {
				logger.Info("grpc server exited")
			}
			serve <- grpcErr
		}()

		if err := writePid(viper.GetString("peer.fileSystemPath")+"/peer.pid", os.Getpid()); err != nil {
			return err
		}

		// 启动eventHub服务
		if ehubGrpcServer != nil && ehubLis != nil {
			go ehubGrpcServer.Serve(ehubLis)
		}

		if viper.GetBool("peer.profile.enabled") {
			go func() {
				profileListenAddress := viper.GetString("peer.profile.listenAddress")
				logger.Infof("Starting profiling server with listenAddress = %s", profileListenAddress)
				if profileErr := http.ListenAndServe(profileListenAddress, nil); profileErr != nil {
					logger.Errorf("Error starting profiler: %s", profileErr)
				}
			}()
		}

		// Block until grpc server exits 产生块直到grpc服务退出
		return <-serve
	}

由代码我们可以看出Peer启动过程中注册了一下这些服务:
* EventHub服务器
* grpc服务
* Chaincode支持服务器
* peer服务器
* Devops服务器
* ServerOpenchain服务器
* REST服务(只有在配置了的情况下才会去创建)

### PBFT源码解析
#### 一、PBFT的原理概述

1.算法公式:

replicaCount  int 变量定义在pbftCore结构体中
N (N在代码中对应replicaCount整型变量)是所有replicas的集合，每一个replica用一个整数来表示，如{ 0, 1, 2, 3,...N - 1 }


N-1 = 3f -----> f = N- 1/3


f 是最大可容忍的出错节点，也就是说准许错为1/3

#### 2.图解PBFT执行过程

图片1： 
    ![图片1](https://github.com/guoshijiang/Hyperledger-Fabric-v0.6/blob/master/images/qq.jpg "图片1")

假设系统要求每次产生区块的时间间隔为𝑡(实际上自己可以配置)，则在一切正常的情况下，算法按照以下流程执行： 
1.任意节点向全网广播交易数据，并附上发送者的签名 
2.所有备份节点均独立监听全网的交易数据，并记录在内存 
3.主节点在经过时间𝑡后,发送〈𝑃𝑒𝑟𝑝𝑎𝑟𝑒𝑅𝑒𝑞𝑢𝑒𝑠𝑡,ℎ,𝑣,𝑝,𝑏𝑙𝑜𝑐𝑘,〈𝑏𝑙𝑜𝑐𝑘〉𝜎𝑝〉 
4.备份节点𝑖在收到提案后，发送〈𝑃𝑒𝑟𝑝𝑎𝑟𝑒𝑅𝑒𝑠𝑝𝑜𝑛𝑠𝑒,ℎ,𝑣,𝑖,〈𝑏𝑙𝑜𝑐𝑘〉𝜎𝑖〉 
5.任意节点在收到至少𝑛−𝑓个〈𝑏𝑙𝑜𝑐𝑘〉𝜎𝑖后，共识达成并发布完整的区块 
6.任意节点在收到完整区块后，将包含的交易从内存中删除，并开始下一轮共识

该算法要求参与共识的节点中，至少有𝑛−𝑓个节点具有相同的初始状态：即对于所有的节点𝑖，具有相同的区块高度ℎ和视图编号𝑣。而这个要求很容易达成：通过区块同步来达到ℎ的一致性，通过视图更换来达到𝑣的一致性。节点在监听全网交易以及在收到提案后，需要对交易进行合法性验证。如果发现非法交易，则不能将其写入内存池；如果非法交易包含在提案中，则放弃本次共识并立即开始视图更换。
交易的验证流程如下： 

1. 交易的数据格式是否符合系统规则，如果不符合则判定为非法； 
2. 交易在区块链中是否已经存在，如果存在则判定为非法； 
3. 交易的所有合约脚本是否都正确执行，如果没有则判定为非法； 
4. 交易中有没有多重支付行为，如果有则判定为非法； 
5. 如果以上判定都不符合，则为合法交易；

当节点𝑖在经过2𝑣+1.𝑡的时间间隔后仍未达成共识，或接收到包含非法交易的提案后，开始进入视图更换流程： 
1.令𝑘 = 1，𝑣𝑘 = 𝑣 + 𝑘； 
2.节点𝑖发出视图更换请求〈𝐶ℎ𝑎𝑛𝑔𝑒𝑉𝑖𝑒𝑤,ℎ,𝑣,𝑖,𝑣𝑘〉； 
3.任意节点收到至少𝑛 − 𝑓个来自不同𝑖的相同𝑣𝑘后，视图更换达成，令𝑣 = 𝑣𝑘并开始共识； 
4.如果在经过2𝑣𝑘+1.𝑡的时间间隔后，视图更换仍未达成，则𝑘递增并回到第2步； 
随着𝑘的增加，超时的等待时间也会呈指数级增加，可以避免频繁的视图更换操作，并使各节点尽快对𝑣达成一致。 
而在视图更换达成之前，原来的视图𝑣依然有效，由此避免了因偶然性的网络延迟超时而导致不必要的视图更换。

#### 客户端
Client---->REQUEST--->replicas

1. REQUEST携带 operation, timestamp，给每一个REQUEST加上时间戳，这样后来的REQUEST会有高于前面的时间戳

2. replicas会接收请求，如果他们验证了条请求，就会将它写入到自己的log中。在共识算法保证下每个replica完成对该请求的执行后直接将回复返回给client：

3. REPLY携带当前的view序号和时间戳，还有replica节点的编号，会返回执行结果

4. 共识算法中有一个weak certificate，在这里，client也会等待weak certificate：即有f+1个replicas回复，并且它们的回复拥有相同的 t 和 r，由于至多有f个faulty replicas，所以确保了回复是合法的。我们叫这个weak certificate为 reply certificate。

5. 处于active状态每一个replica会与每一个的client共享一份秘钥。

#### pre-prepare阶段

1. 主节点收到来自Client的一条请求并分配了一个编号给这个请求，
2. 主节点会广播一条PRE-PREPARE信息给备份节点，
3. 这个PRE-PREPARE信息包含该请求的编号、所在的view和自身的一个digest。
4. 直到该信息送达到每一个备份节点，接下来就看收到信息的备份节点们同不同意
5. 主节点分配给该请求的这个编号n，即是否accept这条PRE-PREPARE信息，
6. 如果一个备份节点accept了这条PRE-PREPARE，它就会进入下面的prepare阶段。

#### prepare阶段
1. 备份节点进入prepared阶段后，广播一条PREPARE信息给主节点和其它的备份节点，直到PREPARE信息都抵达那三个节点。同时，该备份节点也会分别收到来自其它备份节点的PREPARE信息。
2. 该备份节点将综合这些PREPARE信息做出自己对编号n的最终裁决。当这个备份节点开始综合比较来自其它两个备份节点的PREPARE信息和自身的PREPARE信息时，如果该备份节点发现其它两个节点都同意主节点分配的编号，又看了一下自己，自己也同意主节点的分配，a quorum certificate with the PRE-PREPARE and 2 f matching PREPARE messages for sequence number n, view v, and request m，如果一个replica达到了英文所说的条件，比如就是上面的斜体字描述的一种情况，那么我们就说该请求在这个replica上的状态是prepared，该replica就拥有了一个证书叫prepared certificate。那我们是不是就可以说至此排序工作已经完成，全网节点都达成了一个一致的请求序列呢，每一个replica开始照着这个序列执行吧。这是有漏洞的，设想一下，在t1时刻只有replica 1把请求m（编号为n）带到了prepared状态，其他两个备份节点replica 2， replica 3还没有来得及收集完来自其它节点的PREPARE信息进行判断，那么这时发生了view change进入到了一个新的view中，replica 1还认为给m分配的编号n已经得到了一个quorum同意，可以继续納入序列中，或者可以执行了，但对于replica 2来说，它来到了新的view中，它失去了对请求m的判断，甚至在上个view中它还有收集全其他节点发出的PREPARE信息，所以对于replica 2来说，给请求m分配的编号n将不作数，同理replica 3也是。那么replica 1一个人认为作数不足以让全网都认同，所以再新的view中，请求m的编号n将作废，需要重新发起提案。所以就有了下面的commit阶段。
### 注意：
该备份节点会将自己收到的PRE-PREPARE和发送的PREPARE信息记录到自己的log中。
该备份节点发出PREPARE信息表示该节点同意主节点在view v中将编号n分配给请求m，不发即表示不同意。
如果一个replica对请求m发出了PRE-PREPARE和PREPARE信息，那么我们就说该请求m在这个replica节点上处于pre-prepared状态。

#### Commit阶段
紧接着prepare阶段，当一个replica节点发现有一个quorum同意编号分配时，它就会广播一条COMMIT信息给其它所有节点告诉他们它有一个prepared certificate了。与此同时它也会陆续收到来自其它节点的COMMIT信息，如果它收到了2f+1条COMMIT（包括自身的一条，这些来自不同节点的COMMIT携带相同的编号n和view v），我们就说该节点拥有了一个叫committed certificate的证书，请求在这个节点上达到了committed状态。此时只通过这一个节点，我们就能断定该请求已经在一个quorum中到达了prepared状态，寄一个quorum的节点们都同意了编号n的分配。当请求m到达commited状态后，该请求就会被该节点执行。

#### View Changes

当主节点挂掉后就触发了view change协议。我们需要确保在新的view中如何来延续上一个view最终的状态，比如给这时来的新请求的编号，还有如何处理上一个view还没来得及完全处理好的请求。

#### 由此观之核心代码执行的过程如下

图片2： 
    ![图片2](https://github.com/guoshijiang/Hyperledger-Fabric-v0.6/blob/master/images/ff.png "图片2")

### 共识算法代码解析

#### 1.代码目录结构

图片3： 
    ![图片3](http://img.blog.csdn.net/20161221170800572?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamlhbmdfeGlueGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center "图片3")


它初始化一个consenter和一个helper，并互相把一个句柄赋值给了对方。这样做的目的，就是为了可以让外部调用内部，内部可以调用外部


	func GetEngine(coord peer.MessageHandlerCoordinator) (peer.Engine, error) {

	    var err error

	    engineOnce.Do(func() {

		engine = new(EngineImpl)

		engine.helper = NewHelper(coord)

		engine.consenter = controller.NewConsenter(engine.helper)

		engine.helper.setConsenter(engine.consenter)

		engine.peerEndpoint, err = coord.GetPeerEndpoint()

		engine.consensusFan = util.NewMessageFan()

		go func() {

		    logger.Debug("Starting up message thread for consenter")

		    // The channel never closes, so this should never break

		    for msg := range engine.consensusFan.GetOutChannel() {

			engine.consenter.RecvMsg(msg.Msg, msg.Sender)

		    }

		}()

	    })

	    return engine, err

	}

调用controller获取一个plugin，当选择是pbft算法时，它会调用pbft.go 里的GetPlugin(c consensus.Stack)方法，在pbft.go里面把所有的外部参数读进算法内部

	func NewConsenter(stack consensus.Stack) consensus.Consenter {

	    plugin := strings.ToLower(viper.GetString("peer.validator.consensus.plugin"))

	    if plugin == "pbft" {

		logger.Infof("Creating consensus plugin %s", plugin)

		return pbft.GetPlugin(stack)

	    }

	    logger.Info("Creating default consensus plugin (noops)")

	    return noops.GetNoops(stack)

	}

controller目录下是共识插件选择模块的函数
---->HyperLedger提供了两种算法PBFT和noops
---->默认单节点情况下使用noops即相当于没有共识算法

	func NewConsenter(stack consensus.Stack) consensus.Consenter {

	    plugin := strings.ToLower(viper.GetString("peer.validator.consensus.plugin"))

	    if plugin == "pbft" {

		logger.Infof("Creating consensus plugin %s", plugin)

		return pbft.GetPlugin(stack)

	    }

	    logger.Info("Creating default consensus plugin (noops)")

	    return noops.GetNoops(stack)

	}

函数中可以看出目前Hyperledger Fabric只支持PBFT和NOOPS
executor和helper是两个相互依赖的模块
---->主要提供了共识算法和外部衔接的一块代码。主要负责事件处理的转接
helper
--->这里面主要包含了对外部接口的一个调用，比如执行处理transaction，stateupdate，持久化一些对象等
noops 
--->noops相当于没有共识算法
pbft
---> HyperLedger的默认共识算法
util 
--->交互需要的工具包，最主要的一个实现的功能就是它的消息机制。


	func GetPlugin(c consensus.Stack) consensus.Consenter {

	    if pluginInstance == nil {

		pluginInstance = New(c)

	    }

	    return pluginInstance

	}


	func New(stack consensus.Stack) consensus.Consenter {

	    handle, _, _ := stack.GetNetworkHandles()

	    id, _ := getValidatorID(handle)

	    switch strings.ToLower(config.GetString("general.mode")) {

	    case "batch":

		return newObcBatch(id, config, stack)

	    default:

		panic(fmt.Errorf("Invalid PBFT mode: %s", config.GetString("general.mode")))

	    }

	}

在newobcbatch时，会初始化得到一个pbftcore的一个实例，这个是算法的核心模块。并此时会启动一个batchTimer（这个batchTimer是一个计时器，当batchTimer timeout后会触发一个sendbatch操作，这个只有primary节点才会去做）。当然此时会创建一个事件处理机制，这个事件处理机制是各个模块沟通的一个bridge。

	func newObcBatch(id uint64, config *viper.Viper, stack consensus.Stack) *obcBatch {

	    var err error

	    op := &obcBatch{

		obcGeneric: obcGeneric{stack: stack},

	    }

	    op.persistForward.persistor = stack

	    logger.Debugf("Replica %d obtaining startup information", id)

	    op.manager = events.NewManagerImpl() // TODO, this is hacky, eventually rip it out

	    op.manager.SetReceiver(op)

	    etf := events.NewTimerFactoryImpl(op.manager)

	    op.pbft = newPbftCore(id, config, op, etf)

	    op.manager.Start()

	    blockchainInfoBlob := stack.GetBlockchainInfoBlob()

	    op.externalEventReceiver.manager = op.manager

	    op.broadcaster = newBroadcaster(id, op.pbft.N, op.pbft.f, op.pbft.broadcastTimeout, stack)

	    op.manager.Queue() <- workEvent(func() {

		op.pbft.stateTransfer(&stateUpdateTarget{

		    checkpointMessage: checkpointMessage{

			seqNo: op.pbft.lastExec,

			id:    blockchainInfoBlob,

		    },

		})

	    })

	    op.batchSize = config.GetInt("general.batchsize")

	    op.batchStore = nil

	    op.batchTimeout, err = time.ParseDuration(config.GetString("general.timeout.batch"))

	    if err != nil {

		panic(fmt.Errorf("Cannot parse batch timeout: %s", err))

	    }

	    logger.Infof("PBFT Batch size = %d", op.batchSize)

	    logger.Infof("PBFT Batch timeout = %v", op.batchTimeout)

	    if op.batchTimeout >= op.pbft.requestTimeout {

		op.pbft.requestTimeout = 3 * op.batchTimeout / 2

		logger.Warningf("Configured request timeout must be greater than batch timeout, setting to %v", op.pbft.requestTimeout)

	    }

	    if op.pbft.requestTimeout >= op.pbft.nullRequestTimeout && op.pbft.nullRequestTimeout != 0 {

		op.pbft.nullRequestTimeout = 3 * op.pbft.requestTimeout / 2

		logger.Warningf("Configured null request timeout must be greater than request timeout, setting to %v", op.pbft.nullRequestTimeout)

	    }

	    op.incomingChan = make(chan *batchMessage)

	    op.batchTimer = etf.CreateTimer()

	    op.reqStore = newRequestStore()

	    op.deduplicator = newDeduplicator()

	    op.idleChan = make(chan struct{})

	    close(op.idleChan) // TODO remove eventually

	    return op

	}



	func newPbftCore(id uint64, config *viper.Viper, consumer innerStack, etf events.TimerFactory) *pbftCore {

	    var err error

	    instance := &pbftCore{}

	    instance.id = id

	    instance.consumer = consumer

	    //==============================================================================

	    // 4. 在初始化pbftcore时，在把所用配置读进的同时，创建了三个timer

	    //==============================================================================



	    //==========================================================================

	    //newViewTimer对应于viewChangeTimerEvent{}，当这个timer在一定时间没有close时，

	    //就会触发一个viewchange事件

	    //==========================================================================



	    instance.newViewTimer = etf.CreateTimer()

	    //==========================================================================

	    //vcResendTimer对应viewChangeResendTimerEvent，发出viewchange过时时会

	    //触发一个将viewchange从新发送

	    //==========================================================================



	    instance.vcResendTimer = etf.CreateTimer()

	    //==========================================================================

	    //nullRequestTimer对应nullRequestEvent，如果主节点长期没有发送preprepare消息，

	    //也就是分配了seq的reqBatch。它timeout就认为主节点挂掉了然后发送viewchange消息

	    //==========================================================================

	    instance.nullRequestTimer = etf.CreateTimer()

	    instance.N = config.GetInt("general.N") //网络中验证器的最大数量赋值

	    //==================================================================================//

	    /*N是所有replicas的集合，每一个replica用一个整数来表示，依次为

	    { 0, …, |N - 1 }

	    简单起见，我们定义

	    |N = 3f + 1

	    f 是最大可容忍的faulty节点

	    另外我们将一个view中的primary节点定义为replica p，

	    p = v mod |N

	    v 是view的编号，从0开始一直连续下去，这样可以理解为从replica 0 到 replica |N-1 依次当primary节点，当每一次view change发生时。

	    */

	    //==================================================================================//

	    instance.f = config.GetInt("general.f") //默认的最大容错数量赋值

	    if instance.f*3+1 > instance.N {        //默认的最大容错数量大于网络中验证器的最大数量

		panic(fmt.Sprintf("need at least %d enough replicas to tolerate %d byzantine faults, but only %d replicas configured", instance.f*3+1, instance.f, instance.N))

	    }

	    instance.K = uint64(config.GetInt("general.K")) //检查点时间段赋值

	    //计算日志的大小值赋值,日志倍增器

	    instance.logMultiplier = uint64(config.GetInt("general.logmultiplier"))

	    if instance.logMultiplier < 2 {

		panic("Log multiplier must be greater than or equal to 2")

	    }

	    //日志大小计算

	    instance.L = instance.logMultiplier * instance.K // 日志大小

	    //自动视图改变的时间段

	    instance.viewChangePeriod = uint64(config.GetInt("general.viewchangeperiod"))

	    //这个节点是否故意充当拜占庭;testnet用于调试

	    instance.byzantine = config.GetBool("general.byzantine")

	    //请求过程超时

	    instance.requestTimeout, err = time.ParseDuration(config.GetString("general.timeout.request"))

	    if err != nil {

		panic(fmt.Errorf("Cannot parse request timeout: %s", err))

	    }

	    //重发视图改变之前超时

	    instance.vcResendTimeout, err = time.ParseDuration(config.GetString("general.timeout.resendviewchange"))

	    if err != nil {

		panic(fmt.Errorf("Cannot parse request timeout: %s", err))

	    }

	    //新的视图超时

	    instance.newViewTimeout, err = time.ParseDuration(config.GetString("general.timeout.viewchange"))

	    if err != nil {

		panic(fmt.Errorf("Cannot parse new view timeout: %s", err))

	    }

	    //超时持续

	    instance.nullRequestTimeout, err = time.ParseDuration(config.GetString("general.timeout.nullrequest"))

	    if err != nil {

		instance.nullRequestTimeout = 0

	    }

	    //广播过程超时

	    instance.broadcastTimeout, err = time.ParseDuration(config.GetString("general.timeout.broadcast"))

	    if err != nil {

		panic(fmt.Errorf("Cannot parse new broadcast timeout: %s", err))

	    }

	    //查看view发生

	    instance.activeView = true

	    //replicas的数量; PBFT `|R|`

	    instance.replicaCount = instance.N

	    logger.Infof("PBFT type = %T", instance.consumer)

	    logger.Infof("PBFT Max number of validating peers (N) = %v", instance.N)

	    logger.Infof("PBFT Max number of failing peers (f) = %v", instance.f)

	    logger.Infof("PBFT byzantine flag = %v", instance.byzantine)

	    logger.Infof("PBFT request timeout = %v", instance.requestTimeout)

	    logger.Infof("PBFT view change timeout = %v", instance.newViewTimeout)

	    logger.Infof("PBFT Checkpoint period (K) = %v", instance.K)

	    logger.Infof("PBFT broadcast timeout = %v", instance.broadcastTimeout)

	    logger.Infof("PBFT Log multiplier = %v", instance.logMultiplier)

	    logger.Infof("PBFT log size (L) = %v", instance.L)

	    //超时持续

	    if instance.nullRequestTimeout > 0 {

		logger.Infof("PBFT null requests timeout = %v", instance.nullRequestTimeout)

	    } else {

		logger.Infof("PBFT null requests disabled")

	    }

	    //在自动视图改变的时间段

	    if instance.viewChangePeriod > 0 {

		logger.Infof("PBFT view change period = %v", instance.viewChangePeriod)

	    } else {

		logger.Infof("PBFT automatic view change disabled")

	    }

	    // init the logs

	    //跟踪法定证书请求

	    instance.certStore = make(map[msgID]*msgCert)

	    //跟踪请求批次

	    instance.reqBatchStore = make(map[string]*RequestBatch)

	    //跟踪检查点设置

	    instance.checkpointStore = make(map[Checkpoint]bool)

	    //检查点状态; 映射lastExec到全局hash

	    instance.chkpts = make(map[uint64]string)

	    //跟踪视view change消息

	    instance.viewChangeStore = make(map[vcidx]*ViewChange)

	    instance.pset = make(map[uint64]*ViewChange_PQ)

	    instance.qset = make(map[qidx]*ViewChange_PQ)

	    //跟踪我们接收后者发送的最后一个新视图

	    instance.newViewStore = make(map[uint64]*NewView)

	    // initialize state transfer

	    //观察每一个replica最高薄弱点序列数

	    instance.hChkpts = make(map[uint64]uint64)

	    //检查点状态; 映射lastExec到全局hash

	    instance.chkpts[0] = "XXX GENESIS"

	    // 在我们使用视图改变期间最后超时

	    instance.lastNewViewTimeout = instance.newViewTimeout

	    //跟踪我们是否正在等待请求批处理执行

	    instance.outstandingReqBatches = make(map[string]*RequestBatch)

	    //对于所有已经分配我们可能错过的在视图改变期间的非检查点的请求批次

	    instance.missingReqBatches = make(map[string]bool)

	    //将变量的值恢复到初始状态

	    instance.restoreState()

	    // 执行视图改变的下一个序号

	    instance.viewChangeSeqNo = ^uint64(0) // infinity

	    //更新视图改变序列号

	    instance.updateViewChangeSeqNo()

	    return instance

	}	

图片5： 
    ![图片5](https://github.com/guoshijiang/Hyperledger-Fabric-v0.6/blob/master/images/ee.png "图片5")

### 配置四个节点共识
##### 1、在docker-compose.yml文件中配置好节点
	docker-compose up启动节点
##### 2、通过“六”的方式依次进入各个节点，修改core.yaml和config.yaml文件
	core.yaml文件
 	peer.validator.consensus的值设置为pbft
	peer.id 值设置为节点名，例如vp0
	config.yaml文件中
  	general.mode的值设置为batch
	viewchanged的值设置成你想要的值
	general.N 的值设置为为节点个数N
 	general.batchoice设置为每个批的数量
	默认算法PBFT改为true
##### 3、也可以直接执行下面命令
 	CORE_PEER_VALIDATOR_CONSENSUS_PLUGIN=pbft 
	CORE_PBFT_GENERAL_MODE=batch
配置节点共识需要定制dockers镜像,也可以直接docker commit提交镜像，还可以在启动文件里面配置

### Hyperledger Fabric继peer启动之后的源码解析

下面是Hyperledger中相关监听的服务端口,默认包括：以下监听的服务端口对应hyperledger0.6版本，与其他版本无关。

7050: REST 服务端口

7051：peer gRPC 服务监听端口

7052：peer CLI 端口

7053：peer 事件服务端口

7054：eCAP7055：eCAA

7056：tCAP

7057：tCAA

7058：tlsCAP

7059：tlsCAA



CacheConfiguration计算和缓存经常使用的常量且计算常量做为包变量，按照惯例前面的全局变量。已经被嵌入在这里为了保留原始的抽象状态

	func CacheConfiguration() (err error) {

	    // getLocalAddress 返回正在操作的本地peer的address:port，受到env:peer.addressAutoDetect的影响

	    getLocalAddress := func() (peerAddress string, err error) {

		if viper.GetBool("peer.addressAutoDetect") {

		    // 需要从peer.address设置中获取端口号，并将其添加到已经确定的主机ip后

		    _, port, err := net.SplitHostPort(viper.GetString("peer.address"))

		    if err != nil {

			err = fmt.Errorf("Error auto detecting Peer's address: %s", err)

			return "", err

		    }

		    peerAddress = net.JoinHostPort(GetLocalIP(), port)

		    peerLogger.Infof("Auto detected peer address: %s", peerAddress)

		} else {

		    peerAddress = viper.GetString("peer.address")

		}

		return

	    }

	    // getPeerEndpoint 对于这个Peer实例来说，返回PeerEndpoint，受到env:peer.addressAutoDetect的影响

	    getPeerEndpoint := func() (*pb.PeerEndpoint, error) {

		var peerAddress string

		var peerType pb.PeerEndpoint_Type

		peerAddress, err := getLocalAddress()

		if err != nil {

		    return nil, err

		}

		if viper.GetBool("peer.validator.enabled") {

		    peerType = pb.PeerEndpoint_VALIDATOR

		} else {

		    peerType = pb.PeerEndpoint_NON_VALIDATOR

		}

		return &pb.PeerEndpoint{ID: &pb.PeerID{Name: viper.GetString("peer.id")}, Address: peerAddress, Type: peerType}, nil

	    }

	    localAddress, localAddressError = getLocalAddress()

	    peerEndpoint, peerEndpointError = getPeerEndpoint()

	    syncStateSnapshotChannelSize = viper.GetInt("peer.sync.state.snapshot.channelSize")

	    syncStateDeltasChannelSize = viper.GetInt("peer.sync.state.deltas.channelSize")

	    syncBlocksChannelSize = viper.GetInt("peer.sync.blocks.channelSize")

	    validatorEnabled = viper.GetBool("peer.validator.enabled")

	    securityEnabled = viper.GetBool("security.enabled")

	    configurationCached = true

	    if localAddressError != nil {

		return localAddressError

	    } else if peerEndpointError != nil {

		return peerEndpointError

	    }

	    return

	}

cacheConfiguration如果检查失败打一个错误日志

	func cacheConfiguration() {

	    if err := CacheConfiguration(); err != nil {

		peerLogger.Errorf("Execution continues after CacheConfiguration() failure : %s", err)

	    }

	}

GetPeerEndpoint 从缓存配置中返回peerEndpoint

	func GetPeerEndpoint() (*pb.PeerEndpoint, error) {

	    if !configurationCached {

		cacheConfiguration()

	    }

	    return peerEndpoint, peerEndpointError

	}

cacheConfiguration如果检查失败打一个错误日志

	func cacheConfiguration() {

	    if err := CacheConfiguration(); err != nil {

		peerLogger.Errorf("Execution continues after CacheConfiguration() failure : %s", err)

	    }
	    
	    
接下来，我们要分析的是registering BLOCK，registering CHAINCODE，registering REJECTION和registering REGISTER的整个过程。在这个部分主要处理的事件流，这里一共有四种事件，分别是BLOCK，CHAINCODE，REJECTION和RIGISTER。
	
	func createEventHubServer() (net.Listener, *grpc.Server, error) {

	    var lis net.Listener

	    var grpcServer *grpc.Server

	    var err error

	    // ValidatorEnabled返回peer.validator.enabled是否可用

	    if peer.ValidatorEnabled() {

		lis, err = net.Listen("tcp", viper.GetString("peer.validator.events.address"))

		if err != nil {

		    return nil, nil, fmt.Errorf("failed to listen: %v", err)

		}

		//TODO - do we need different SSL material for events ?

		var opts []grpc.ServerOption

		// TLSEnabled返回peer.tls.enabled配置好的值的缓存值

		if comm.TLSEnabled() {

		    //NewServerTLSFromFile是gRPC库的函数，主要目的是为获取tls的证书和密钥

		    creds, err := credentials.NewServerTLSFromFile(

			viper.GetString("peer.tls.cert.file"),

			viper.GetString("peer.tls.key.file"))

		    if err != nil {

			return nil, nil, fmt.Errorf("Failed to generate credentials %v", err)

		    }

		    opts = []grpc.ServerOption{grpc.Creds(creds)}

		}

		grpcServer = grpc.NewServer(opts...)

		ehServer := producer.NewEventsServer(

		    uint(viper.GetInt("peer.validator.events.buffersize")),

		    viper.GetInt("peer.validator.events.timeout"))

		//注册事件服务

		pb.RegisterEventsServer(grpcServer, ehServer)

	    }

	    return lis, grpcServer, err

	}

ValidatorEnabled返回peer.validator.enabled是否可用

	func ValidatorEnabled() bool {

	    if !configurationCached {

		cacheConfiguration()

	    }

	    return validatorEnabled

	}

cacheConfiguration如果检查失败打一个错误日志

	func cacheConfiguration() {

	    if err := CacheConfiguration(); err != nil {

		peerLogger.Errorf("Execution continues after CacheConfiguration() failure : %s", err)

	    }

	}

TLSEnabled返回peer.tls.enabled配置好的值的缓存值

	func TLSEnabled() bool {

	    if !configurationCached {

		cacheConfiguration()

	    }

	    return tlsEnabled

	}

cacheConfiguration如果检查失败打错误日志.

	func cacheConfiguration() {

	    if err := CacheConfiguration(); err != nil {

		commLogger.Errorf("Execution continues after CacheConfiguration() failure : %s", err)

	    }

	}
	
NewEventsServer 创建并且返回一个事件服务器

	func NewEventsServer(bufferSize uint, timeout int) *EventsServer {

	    if globalEventsServer != nil {

		panic("Cannot create multiple event hub servers")

	    }

	    globalEventsServer = new(EventsServer)

	    //初始化并且开启事件

	    initializeEvents(bufferSize, timeout)

	    //initializeCCEventProcessor(bufferSize, timeout)

	    return globalEventsServer

	}

初始化并且开启事件

	func initializeEvents(bufferSize uint, tout int) {

	    if gEventProcessor != nil {

		panic("should not be called twice")

	    }

	    gEventProcessor = &eventProcessor{eventConsumers: make(map[pb.EventType]handlerList), eventChannel: make(chan *pb.Event, bufferSize), timeout: tout}

	    addInternalEventTypes()

	    //启动事件进程器

	    go gEventProcessor.start()

	}

	func addInternalEventTypes() {

	    AddEventType(pb.EventType_BLOCK)

	    AddEventType(pb.EventType_CHAINCODE)

	    AddEventType(pb.EventType_REJECTION)

	    AddEventType(pb.EventType_REGISTER)

	}

AddEventType 添加支持的事件类型

	func AddEventType(eventType pb.EventType) error {

	    gEventProcessor.Lock()

	    producerLogger.Debugf("registering %s", pb.EventType_name[int32(eventType)])

	    if _, ok := gEventProcessor.eventConsumers[eventType]; ok {

		gEventProcessor.Unlock()

		return fmt.Errorf("event type exists %s", pb.EventType_name[int32(eventType)])

	    }

	    switch eventType {

	    case pb.EventType_BLOCK:

		gEventProcessor.eventConsumers[eventType] = &genericHandlerList{handlers: make(map[*handler]bool)}

	    case pb.EventType_CHAINCODE:

		gEventProcessor.eventConsumers[eventType] = &chaincodeHandlerList{handlers: make(map[string]map[string]map[*handler]bool)}

	    case pb.EventType_REJECTION:

		gEventProcessor.eventConsumers[eventType] = &genericHandlerList{handlers: make(map[*handler]bool)}

	    }

	    gEventProcessor.Unlock()

	    return nil

	}

	func (ep *eventProcessor) start() {

	    producerLogger.Info("event processor started")

	    for {

		//等待事件

		e := <-ep.eventChannel

		var hl handlerList

		eType := getMessageType(e)

		ep.Lock()

		if hl, _ = ep.eventConsumers[eType]; hl == nil {

		    producerLogger.Errorf("Event of type %s does not exist", eType)

		    ep.Unlock()

		    continue

		}

		//lock the handler map lock

		ep.Unlock()

		hl.foreach(e, func(h *handler) {

		    if e.Event != nil {

			h.SendMessage(e)

		    }

		})

	    }

	}

 获取事件消息类型
 
	 func getMessageType(e *pb.Event) pb.EventType {

	    switch e.Event.(type) {

	    case *pb.Event_Register:

		return pb.EventType_REGISTER

	    case *pb.Event_Block:

		return pb.EventType_BLOCK

	    case *pb.Event_ChaincodeEvent:

		return pb.EventType_CHAINCODE

	    case *pb.Event_Rejection:

		return pb.EventType_REJECTION

	    default:

		return -1

	    }

	}

SendMessage 通过流发送一条消息给远程的peer

	func (d *handler) SendMessage(msg *pb.Event) error {

	    err := d.ChatStream.Send(msg)

	    if err != nil {

		return fmt.Errorf("Error Sending message through ChatStream: %s", err)

	    }

	    return nil

	}

	func RegisterEventsServer(s *grpc.Server, srv EventsServer) {

	    s.RegisterService(&_Events_serviceDesc, srv)

	}

一些重要的变量

	type eventProcessor struct {

	    sync.RWMutex

	    eventConsumers map[pb.EventType]handlerList

	    //we could generalize this with mutiple channels each with its own size

	    // 产生多个大小限定的channels

	    eventChannel chan *pb.Event

	    //milliseconds timeout for producer to send an event.

	    //毫秒级别的超时触发器发送一个事件

	    //if < 0, if buffer full, unblocks immediately and not send

	    //如果小于0，如果缓冲区满的，立即解锁并且不发送事件

	    //if 0, if buffer full, will block and guarantee the event will be sent out

	    // 如是0，如果缓冲区满的，将上锁并保证事件不会被发出去

	    //if > 0, if buffer full, blocks till timeout

	    //如果是0，如果缓冲区是满的，上锁直到超时

	    timeout int

	}

通过initializeEvents函数来创建全局eventProcessor单列模式 Openchain producers，Openchain仅仅通过一个重入的静态方法生产并发送事件

	var gEventProcessor *eventProcessor


	type EventType int32


	const (

	    EventType_REGISTER  EventType = 0

	    EventType_BLOCK     EventType = 1

	    EventType_CHAINCODE EventType = 2

	    EventType_REJECTION EventType = 3

	)

	var EventType_name = map[int32]string{

	    0: "REGISTER",

	    1: "BLOCK",

	    2: "CHAINCODE",

	    3: "REJECTION",

	}

	var EventType_value = map[string]int32{

	    "REGISTER":  0,

	    "BLOCK":     1,

	    "CHAINCODE": 2,

	    "REJECTION": 3,

	}


	// Event is used by

	//  - consumers (adapters) to send Register

	//  - producer to advertise supported types and events

	type Event struct {

	    // Types that are valid to be assigned to Event:

	    //    *Event_Register

	    //    *Event_Block

	    //    *Event_ChaincodeEvent

	    //    *Event_Rejection

	    //    *Event_Unregister

	    Event isEvent_Event `protobuf_oneof:"Event"`

	} 

我们都知道，Fabric用的数据库是rocksdb, 下面是启动rockdb数据库的整个流程

	db.Start()
	
启动数据库, 初始化openchainDB实例并打开数据库.注意该方法不能保证正确行为的并发调用

	func Start() {
	    openchainDB.open()
	}

Open打开已经存在于hyperledger中的数据库

	func (openchainDB *OpenchainDB) open() {

	    dbPath := getDBPath()

	    missing, err := dirMissingOrEmpty(dbPath)

	    if err != nil {

		panic(fmt.Sprintf("Error while trying to open DB: %s", err))

	    }

	    dbLogger.Debugf("Is db path [%s] empty [%t]", dbPath, missing)

	    if missing {

		err = os.MkdirAll(path.Dir(dbPath), 0755)

		if err != nil {

		    panic(fmt.Sprintf("Error making directory path [%s]: %s", dbPath, err))

		}

	    }

	    opts := gorocksdb.NewDefaultOptions()

	    defer opts.Destroy()

	    maxLogFileSize := viper.GetInt("peer.db.maxLogFileSize")

	    if maxLogFileSize > 0 {

		dbLogger.Infof("Setting rocksdb maxLogFileSize to %d", maxLogFileSize)

		opts.SetMaxLogFileSize(maxLogFileSize)

	    }

	    keepLogFileNum := viper.GetInt("peer.db.keepLogFileNum")

	    if keepLogFileNum > 0 {

		dbLogger.Infof("Setting rocksdb keepLogFileNum to %d", keepLogFileNum)

		opts.SetKeepLogFileNum(keepLogFileNum)

	    }

	    logLevelStr := viper.GetString("peer.db.loglevel")

	    logLevel, ok := rocksDBLogLevelMap[logLevelStr]

	    if ok {

		dbLogger.Infof("Setting rocks db InfoLogLevel to %d", logLevel)

		opts.SetInfoLogLevel(logLevel)

	    }

	    opts.SetCreateIfMissing(missing)

	    opts.SetCreateIfMissingColumnFamilies(true)

	    cfNames := []string{"default"}

	    cfNames = append(cfNames, columnfamilies...)

	    var cfOpts []*gorocksdb.Options

	    for range cfNames {

		cfOpts = append(cfOpts, opts)

	    }

	    db, cfHandlers, err := gorocksdb.OpenDbColumnFamilies(opts, dbPath, cfNames, cfOpts)

	    if err != nil {

		panic(fmt.Sprintf("Error opening DB: %s", err))

	    }

	    openchainDB.DB = db

	    openchainDB.BlockchainCF = cfHandlers[1]

	    openchainDB.StateCF = cfHandlers[2]

	    openchainDB.StateDeltaCF = cfHandlers[3]

	    openchainDB.IndexesCF = cfHandlers[4]

	    openchainDB.PersistCF = cfHandlers[5]

	}

NewDefaultOptions 创建一个默认的Options.

	func NewDefaultOptions() *Options {

	    return NewNativeOptions(C.rocksdb_options_create())

	}
Options表示当以open形式打开一个数据库时所有可选的options

	type Options struct {

	    c *C.rocksdb_options_t

	    // 保持引用GC.

	    env  *Env

	    bbto *BlockBasedTableOptions

	    // 在Destroy的时候我们要保证能够释放他们的内存

	    ccmp *C.rocksdb_comparator_t

	    cmo  *C.rocksdb_mergeoperator_t

	    cst  *C.rocksdb_slicetransform_t

	    ccf  *C.rocksdb_compactionfilter_t

	}

NewDefaultOptions 创建一个默认的Options.

	func NewDefaultOptions() *Options {

	    return NewNativeOptions(C.rocksdb_options_create())

	}

NewNativeOptions 创建一个Options对象.

	func NewNativeOptions(c *C.rocksdb_options_t) *Options {

	    return &Options{c: c}

	}

NewNativeOptions 创建一个Options对象.

	func NewNativeOptions(c *C.rocksdb_options_t) *Options {

	    return &Options{c: c}

	}

Options表示当以open形式打开一个数据库时所有可选的options

	type Options struct {

	    c *C.rocksdb_options_t

	    // 保持引用GC.

	    env  *Env

	    bbto *BlockBasedTableOptions

	    // 在Destroy的时候我们要保证能够释放他们的内存

	    ccmp *C.rocksdb_comparator_t

	    cmo  *C.rocksdb_mergeoperator_t

	    cst  *C.rocksdb_slicetransform_t

	    ccf  *C.rocksdb_compactionfilter_t

	}

SetMaxLogFileSize 设置信息日志文件的最大尺寸， 如果日志文件大于max_log_file_size常量，新的日志将会被创建。如果max_log_file_size等于0，所有的日志都被写到一个log日志，默认大小:0。

	func (opts *Options) SetMaxLogFileSize(value int) {

	    C.rocksdb_options_set_max_log_file_size(opts.c, C.size_t(value))

	}

SetKeepLogFileNum 设置保存的最大日志信息文件，默认大小: 1000

	func (opts *Options) SetKeepLogFileNum(value int) {

	    C.rocksdb_options_set_keep_log_file_num(opts.c, C.size_t(value))

	}

SetInfoLogLevel 设置日志信息的级别,默认级别: InfoInfoLogLevel

	func (opts *Options) SetInfoLogLevel(value InfoLogLevel) {

	    C.rocksdb_options_set_info_log_level(opts.c, C.int(value))

	}

InfoLogLevel描述日志级别.

	type InfoLogLevel uint

日志级别.

	const (

	    DebugInfoLogLevel = InfoLogLevel(0)

	    InfoInfoLogLevel  = InfoLogLevel(1)

	    WarnInfoLogLevel  = InfoLogLevel(2)

	    ErrorInfoLogLevel = InfoLogLevel(3)

	    FatalInfoLogLevel = InfoLogLevel(4)

	)

SetCreateIfMissing 如果数据库丢失了指定数据库是不是应该被创建。 默认值: false

	func (opts *Options) SetCreateIfMissing(value bool) {

	    C.rocksdb_options_set_create_if_missing(opts.c, boolToChar(value))

	}

SetCreateIfMissingColumnFamilies 如果指定的列丢失啦是不是应该被创建

	func (opts *Options) SetCreateIfMissingColumnFamilies(value bool) {

	    C.rocksdb_options_set_create_missing_column_families(opts.c, boolToChar(value))

	}

打开列家族

    db, cfHandlers, err := gorocksdb.OpenDbColumnFamilies(opts, dbPath, cfNames, cfOpts)
   
#### 加密模块代码解析
这个方法只能被调用一次并且缓存结果, 注意这个加密本质上属于加密包并且在哪儿都可以调用

	func getSecHelper() (crypto.Peer, error) {
		var secHelper crypto.Peer
		var err error
		once.Do(func() {
			if core.SecurityEnabled() {
				enrollID := viper.GetString("security.enrollID")
				enrollSecret := viper.GetString("security.enrollSecret")
				if peer.ValidatorEnabled() {
					logger.Debugf("Registering validator with enroll ID: %s", enrollID)
					if err = crypto.RegisterValidator(enrollID, nil, enrollID, enrollSecret); nil != err {
						return
					}
					logger.Debugf("Initializing validator with enroll ID: %s", enrollID)
					secHelper, err = crypto.InitValidator(enrollID, nil)
					if nil != err {
						return
					}
				} else {
					logger.Debugf("Registering non-validator with enroll ID: %s", enrollID)
					if err = crypto.RegisterPeer(enrollID, nil, enrollID, enrollSecret); nil != err {
						return
					}
					logger.Debugf("Initializing non-validator with enroll ID: %s", enrollID)
					secHelper, err = crypto.InitPeer(enrollID, nil)
					if nil != err {
						return
					}
				}
			}
		})
		return secHelper, err
	}
	
通过这个函数，会进入到加密包中，在加密包中客户端会有不同证书的对应证书池，客户端把获得的证放到证书池里面，当它需要用的时候，它会去证书池里面获取。证书存储在mysql数据库中。
里面使用到的加密方式有很多，例如:对称加密， 非对称加密， RSA等等，同时也使用到了很多分组加密模式。
