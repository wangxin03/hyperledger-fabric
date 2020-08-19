#### blockchain explorer简介：

区块链浏览器（blockchain explorer）提供区块查询功能，是浏览区块链信息的主要窗口，每个区块记录的信息都可以通过区块链浏览器进行查看。每条区块链都有自己的区块链浏览器，不支持跨链查询。

不同的区块链浏览器支持查询的区块内容不同，通常支持查询区块中的交易信息、区块高度、hash值、区块发布时间和矿工信息等，部分区块链浏览器还支持查询全网算力、算力难度等。

常用的区块链浏览器比如比特币浏览器http://www.btc.com/、以太坊浏览器https://etherchain.org/、https://chainflyer.bitflyer.jp等。

由于区块链包括公链、私链和联盟链，hyperledger explorer是基于hyperledger fabric网络（联盟链）的一种区块链浏览器，支持查询fabric区块链网络中所有区块的交易信息、链码信息等。

Hyperledger Explorer is a simple, powerful, easy-to-use, well maintained, open source utility to <font color=red>browse activity on the underlying blockchain network. </font>Users have the ability to configure and build Hyperledger Explorer on MacOS and Ubuntu.



#### 配置说明：

|         名称         |                  版本                  |     备注      |
| :------------------: | :------------------------------------: | :-----------: |
|        ubuntu        |                18.04LTS                |               |
|        fabric        |      <font color=red>1.4.2</font>      |               |
| hyperledger explorer |    <font color=red>1.0.0-rc2</font>    |               |
|    node.js（npm）    | <font color=red>8.11.4（6.9.0）</font> |  nvm 0.35.3   |
|        docker        |                19.03.12                | 安装docker-ce |
|    docker-compose    |                 1.14.0                 |               |
|          jq          |                 1.5.1                  |               |
|      postgresql      |     <font color=red>9.5.21</font>      |               |
|        golang        |                 1.14.6                 |               |

<font color=red>注：务必保持fabric、hyperledger explorer、node.js（npm）、postgresql版本符合教程要求，否则很容易无法使用浏览器</font>

**==（1）前期准备==**

**1、安装node.js（npm）**

```
nvm install 8.11.4
node -v && npm -v       //node为8.11.4,npm为5.6.0，npm需升级到6.9.0
sudo apt install nodejs-legacy
sudo npm config set registry https://registry.npm.taobao.org
sudo apt install libssl1.0-dev nodejs-dev node-gyp npm
npm -g install npm@6.9.0   //升级npm到6.9.0
```

安装成功：

<img src="https://i.loli.net/2020/08/12/Wfbgu4TVp1KJ3Qe.png" alt="image-20200812001716211" style="zoom:50%;" />

**2、安装jq**

```
sudo apt-get update
sudo apt-get install jq
jq --version
```

**3、安装postgresql数据库**

```
sudo apt-get update
sudo apt install postgresql-9.5    //安装postgresql v9.5，安装成功后会自动添加一个名为postgres的ubuntu操作系统用户，密码是随机的，并且会自动生成一个名字为postgres的数据库，用户名也为postgres，密码也是随机的。因此需修改操作系统用户postgres密码和数据库postgres密码
psql --version
sudo -u postgres psql    //以用户postgres身份打开客户端工具psql
ALTER USER postgres WITH PASSWORD '123456';    //修改postgres数据库用户的密码为123456
\q       //退出客户端工具psql

// 修改配置实现远程访问
cd /etc/postgresql/9.5/main
sudo vim postgresql.conf     //修改listen_addresses = 'localhost' 改为 listen_addresses = '*'，原文件此行被注销

// 设置所有用户可连接
sudo vim pg_hba.conf   //在最后一行插入host all all 0.0.0.0/0 md5，注意按上面格式对齐

// 重启服务
/etc/init.d/postgresql restart
```



**==（2）拉取blockchain-explorer项目==**

```
cd /home/wangxin/go
git clone https://github.com/hyperledger/blockchain-explorer.git    //此处我使用的本地上传项目v1.0.0-rc2压缩包，下载链接：https://pan.baidu.com/s/1j0mf6-ZHFidfLR0c9GVR9w（提取码：yfp2）
sudo apt install unrar
unrar x blockchain-explorer.rar
sudo chown -R wangxin:wangxin ./blockchain-explorer
sudo chmod -R 777 blockchain-explorer
```



**==（3）创建数据库==**

```
cd /home/wangxin/go/blockchain-explorer/app/persistence/fabric/postgreSQL/db/
sudo ./createdb.sh         //创建数据库
sudo -u postgres psql     //连接psql
\l     //查看数据库列表
```

创建成功：

<img src="https://i.loli.net/2020/08/12/TioV7c1x3KpSAha.png" alt="image-20200812005308612" style="zoom: 40%;" />

数据库列表：

<img src="https://i.loli.net/2020/08/12/7GQHncuRiMFbBIz.png" alt="image-20200812005604134" style="zoom:40%;" />



**==（4）启动fabric网络==**

```
cd /home/wangxin/go/fabric-samples/first-network/     //新建终端2
./byfn.sh up
```

网络启动成功：

<img src="https://i.loli.net/2020/08/12/wXaUZJL4EgedFKV.png" alt="image-20200812005941632" style="zoom:30%;" />



**==（5）修改浏览器配置文件==**

```
cd /home/wangxin/go/blockchain-explorer/app/platform/fabric/connection-profile/    //新建终端3
vim first-network.json    //修改如下3个路径为全路径
```

<img src="https://i.loli.net/2020/08/12/YWnxr4Nj1iMeg5U.png" alt="image-20200812010715086" style="zoom:50%;" />



**==（6）构建blockchain explorer==**

```
cd /home/wangxin/go/blockchain-explorer/
./main.sh install    //安装依赖和编译
./main.sh test
```

安装依赖和编译成功：

<img src="https://i.loli.net/2020/08/12/nY5cGuAE8NMIReQ.png" alt="image-20200812011339244" style="zoom:40%;" />

测试成功：

<img src="https://i.loli.net/2020/08/12/8NReFaEoD2fVtbK.png" alt="image-20200812011528092" style="zoom:50%;" />



**==（7）启动blockchain explorer==**

```
sudo ./start.sh
```

启动成功：

<img src="https://i.loli.net/2020/08/12/YCNvyOIVmHjsFBo.png" alt="image-20200812011759324" style="zoom:50%;" />



**==（8）前端展示==**

使用chrome打开http://192.168.0.119:8080/，如下所示：

<img src="https://i.loli.net/2020/08/12/imP4AJNHFx21kja.png" alt="image-20200812011910822" style="zoom:30%;" />

登录账号、密码在/home/wangxin/go/blockchain-explorer/app/platform/fabric/connection-profile/目录下的first-network已设置如下：

<img src="https://i.loli.net/2020/08/12/HAPjIz6gO9TCSY4.png" alt="image-20200812012116069" style="zoom:50%;" />

登录成功，默认进入首页：

<img src="https://i.loli.net/2020/08/12/OcSmi7AFN6QkCEg.png" alt="image-20200812012234986" style="zoom:50%;" />



**==（9）关闭blockchain explorer==**

```
./stop.sh         //在blockchain-explorer目录
./byfn.sh down    //在fabric-samples/first-network目录
```

