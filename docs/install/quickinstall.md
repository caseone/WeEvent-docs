## 快速安装

快速安装是为了方便用户搭建开发和测试环境，在单台机器上快速部署`WeEvent`服务。提供`Docker`镜像、Bash脚本两种安装方式。

如果是第一次安装`WeEvent`，参见这里的[系统要求](./environment.html) 。以下安装过程以`Centos 7.2`为例。

### Docker镜像安装

  ```bash
  $ docker pull weevent/weevent:1.6.0; docker run -d -p 8080:8080 -p 7001:7001 weevent/weevent:1.6.0 /root/run.sh
  ```

  `WeEvent`的镜像里包括了`FISCO-BCOS`网络，`WeEvent`服务的各个子模块以及各种依赖。


### Bash安装

需要的一些基础工具`yum install wget tree tar netstat`。

- 获取安装包

  从`github`下载安装包[weevent-1.6.0.tar.gz](https://github.com/WeBankBlockchain/WeEvent/releases/download/v1.6.0/weevent-1.6.0.tar.gz)，并且解压到`/tmp/` 。

  ```shell
  $ cd /tmp/
  $ wget https://github.com/WeBankBlockchain/WeEvent/releases/download/v1.6.0/weevent-1.6.0.tar.gz
  $ tar -zxf weevent-1.6.0.tar.gz
  ```

解压后目录结构如下：
  
  ```shell
  $ cd weevent-1.6.0/
  $ tree -L 2
  .
  ├── bin
  │   ├── start-all.sh
  │   └── stop-all.sh
  ├── config.properties
  ├── fisco.yml
  ├── install-all.sh
  ├── modules
  │   ├── broker
  │   ├── gateway
  │   ├── governance
  │   ├── lib
  │   ├── processor
  │   └── zookeeper
  ```
  
- 修改配置config.properties

  默认配置文件`./config.properties`如下：

  ```properties
  #java jdk environment
  JAVA_HOME=/usr/local/jdk1.8.0_191
  
  # Required module
  # support 2.x
  fisco-bcos.version=2.0
  # The path of FISCO-BCOS 2.x that contain certificate file ca.crt/sdk.crt/sdk.key
  fisco-bcos.node_path=~/fisco/nodes/127.0.0.1/sdk
  
  # Required module
  gateway.port=8080
  zookeeper.default=true
  zookeeper.connect-string=127.0.0.1:2181
  
  # Required module
  broker.port=7000
  
  # Optional module
  governance.enable=false
  governance.port=7009
  #support both h2 and mysql, default h2
  database.type=h2
  #mysql.ip=127.0.0.1
  #mysql.port=3306
  #mysql.user=xxx
  #mysql.password=yyy
  
  # Optional module
  processor.enable=false
  processor.port=7008
  ```
  
  配置说明 :
  
  - JDK变量`JAVA_HOME`
    
    因为区块链使用的加密算法很多`OpenJDK`版本没有提供。所以这里特别让用户设置符合要求的`JDK`。
    
  - 区块链FISCO-BCOS
  
    - `fisco-bcos.version`
  
      支持`2.0`及其以上版本。
  
    - `fisco-bcos.node_path`
  
      区块链节点的访问证书、私钥存放目录，`FISCO-BCOS 2.x`一般目录为`~/fisco/nodes/127.0.0.1/sdk`。
  
      如果`WeEvent`服务和区块链节点不在同一台机器上，需要把上述目录下文件拷贝到`WeEvent`所在机器的当前目录如 `./conf`目录下，并设置`fisco-bcos.node_path=./conf` 。 **注意**：`fisco-bcos.node_path`目录结构需与节点证书目录`~/fisco/nodes/127.0.0.1/sdk`结构一致，同时不要包含其他文件， 安装时会将`fisco-bcos.node_path`目录下所有内容拷贝到每个模块子目录下。
      
      以下为标准版和国密版依赖的证书文件及目录结构：
      
      ```bash
      $ tree  fisco/nodes/127.0.0.1/sdk/
      fisco/nodes/127.0.0.1/sdk/
      ├── ca.crt
      ├── sdk.crt
      └── sdk.key
      
      $ tree  fisco_gm/nodes/127.0.0.1/sdk/
      fisco_gm/nodes/127.0.0.1/sdk/
      └── gm
          ├── gmca.crt
          ├── gmensdk.crt
          ├── gmensdk.key
          ├── gmsdk.crt
          └── gmsdk.key
      ```
  
  - Gateway
  
    - 监听端口`gateway.port`
  
    - `zookeeper`配置
  
      `zookeeper.default=true`，值为`true`表示使用`zookeeper.connect-string`里配置的端口，在本机安装`zookeeper`服务使用。
  
      `zookeeper.default=false`，值为`false`表示使用`zookeeper.connect-string`的配置去连接用户已有的`zookeeper`服务。
  
  - Broker监听端口`broker.port`
  
  - Governance模块配置
  
    - `governance.enable`是否安装`Governance`模块，默认为`false`不安装
    - 监听端口`governance.port`
    - 默认使用内置的`H2`数据库，也支持`Mysql`配置`mysql.*`
  
  - Processor模块配置
  
    - `proceessor.enable`是否安装`Proceessor`模块，默认为`false`不安装
    - 监听端口`processor.port`
  
- 修改配置fisco.yml

  配置说明：该配置字段`fisco bcos sdk config`部分与FiscoBcos SDK配置一致，详细配置说明参考 [FiscoBcos SDK配置说明](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/sdk/java_sdk/configuration.html) ，安装时关注字段为节点配置。

  ```yaml
  network:
    peers:
      - "127.0.0.1:20200"
      - "127.0.0.1:20201"
  ```


```eval_rst
.. note::
  - 多数情况下以上配置只需修改`config.properties`中证书路径`fisco-bcos.node_path`和`fisco.yml`中`network.peers`连接的节点地址即可运行。其他字段可按照自己实际需求进行修改配置。
```



- 一键安装

  以安装到目录`/usr/local/weevent/`为例。

  ```shell
  $ ./install-all.sh -p /usr/local/weevent/
  ```

  正常安装后，输出有如下关键字:

  ```
  7000 port is okay
  8080 port is okay
  2181 port is okay
  param ok
  install module zookeeper
  install module gateway 
  install gateway success 
  install module broker 
  install broker success 
  ```

  目标安装路径`/usr/local/weevent/`的结构如下:

  ```shell
  $ cd /usr/local/weevent/
  $ tree -L 1
  .
  |-- broker
  |-- lib
  |-- gateway
  |-- start-all.sh
  |-- stop-all.sh
  |-- zookeeper
  ```

- 启停服务
  - 启动服务

    在服务安装目录下`/usr/local/weevent`，通过`start-all.sh`命令启动所有服务 ，正常启动如下：

    ```shell
    $ ./start-all.sh
    ZooKeeper JMX enabled by default
    Using config: /usr/local/weevent/zookeeper/apache-zookeeper-3.6.1-bin/bin/../conf/zoo.cfg
    Starting zookeeper ... STARTED
    start weevent-broker success (PID=3642)
    add the crontab job success
    start weevent-gateway success (PID=3643)
    add the crontab job success
    ```
    
    如果安装了governance服务，默认访问链接为：http://127.0.0.1:8080/weevent-governance/# ，默认用户名为：admin，默认密码为：123456

  - 停止所有服务的命令`./stop-all.sh`。

- 卸载服务

  所有服务停止后，直接删除目录即可。
  


快速安装作为一种简易安装方式，默认使用内置的`H2`数据库。并且所有子服务都是单实例的，生产环境中建议多实例部署。各子模块详细部署参见[Broker模块部署](./module/broker.html)和[Governance模块部署](./module/governance.html)。
