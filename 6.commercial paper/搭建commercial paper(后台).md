#### 一、commercial paper简介

此商业票据（commercial paper）系统包含2个组织：magnetocorp、digibank，它们之间通过票据网络（papernet）进行商业票据的发行（issue）、购买（buy）和赎回（redeem）。

当启动fabric网络后，该网络作为票据网络papernet，是两个组织实现交易的平台。首先，magnetocorp公司一个名叫isabella的员工发行（issue）了一张商业票据，然后digibank公司的员工balaji以某个价格A购买（buy）这张票据，并持有一段时间，再将这张票据以另一个价格B卖给magnetocorp公司，即该公司进行赎回（redeem）操作，这样balaji通过买卖票据赚取了B-A的利润。

业务流程如下：

![img](https://i.loli.net/2020/08/19/owUTpEsF1nSlKBd.png)



#### 二、目录

**（1）前期准备**

1.1安装node.js（npm）

1.2设置docker

1.3安装工具

**（2）启动fabric网络**

**（3）启动项目监控容器**

**（4）启动magnetocorp组织的cli**

**（5）安装和实例化合约**

5.1合约介绍

5.2合约安装

5.3合约实例化

**（6）发布票据（magnetocorp）**

6.1安装依赖

6.2导入钱包

6.3发布商业票据

**（7）启动digibank组织的cli**

**（8）购买和赎回票据（digibank）**

8.1安装依赖

8.2导入钱包

8.3购买票据

8.4赎回票据

**（9）关闭网络及其他服务**



#### 三、配置说明：

|       名称       |                  版本                  |        备注        |
| :--------------: | :------------------------------------: | :----------------: |
|      ubuntu      |                18.04LTS                |                    |
|      fabric      |      <font color=red>1.4.2</font>      |                    |
| commercial paper |      <font color=red>1.4.2</font>      | fabric-samples自带 |
|  node.js（npm）  | <font color=red>8.11.4（6.9.0）</font> |     nvm 0.35.3     |
|      docker      |                19.03.12                |   安装docker-ce    |
|  docker-compose  |                 1.14.0                 |                    |
|      golang      |                 1.14.6                 |                    |
|       make       |                  4.1                   |                    |
|       curl       |                 7.58.0                 |                    |
|       wget       |                 1.19.4                 |                    |
|       g++        |                 7.5.0                  |                    |

<font color=red>本项目ubuntu克隆自另一个ubuntu 18.04LTS，已安装fabric1.4.2，自带fabric-samples，内含commercial paper源码</font>

**==（1）前期准备==**

**1、安装node.js（npm）**

```
nvm install 8.11.4
node -v && npm -v       //node为8.11.4,npm为5.6.0，npm需升级到6.9.0
npm config set registry https://registry.npm.taobao.org
sudo apt install libssl1.0-dev nodejs-dev node-gyp npm
npm -g install npm@6.9.0   //升级npm到6.9.0
```

安装成功：

<img src="https://i.loli.net/2020/08/19/IUJOvix2VednM4N.png" alt="image-20200817175032184" style="zoom:50%;" />

**2、设置docker**

```
sudo groupadd docker    //创建docker用户组
sudo usermod -aG docker ${USER}     //将当前用户添加进docker用户组
sudo systemctl restart docker     //重启docker
newgrp docker       //更新docker用户组列表
sudo apt update
docker login
```

**3、安装工具**

```
sudo apt install make g++ wget curl
```



==**（2）启动fabric网络**==

在/home/wangxin/go/fabric-samples/basic-network/目录，使用start.sh脚本启动网络：

```
./start.sh
```

启动成功：

![image-20200817180556584](https://i.loli.net/2020/08/17/2RwQKkr6aPnASLe.png)

查看启动的网络组件：

![image-20200817180645871](https://i.loli.net/2020/08/17/xFEOW1Gky3C2Yq7.png)



**==（3）启动项目监控容器==**

此处使用logspout来监控docker日志输出，logspout支持将不同docker日志集中到一处显示

使用monitordocker.sh脚本启动logspout服务，该脚本结构如下：

![image-20200817230651677](https://i.loli.net/2020/08/17/324SJEMlH8Pm7d5.png)

执行monitordocker.sh脚本，启动logspout监控docker日志：

```
cd /home/wangxin/go/fabric-samples/commercial-paper/organization/magnetocorp/configuration/cli/
docker network ls     //查看网络列表，找到启动的网络名称为net_basic
./monitordocker.sh net_basic
```

启动成功：

![image-20200817231206776](https://i.loli.net/2020/08/17/Q3ETGC2A96dO1Up.png)



**==（4）启动MagnetoCorp组织的cli==**

新建终端2，启动cli容器，支持模拟MagnetoCorp组织进行合约做操

```
cd /home/wangxin/go/fabric-samples/commercial-paper/organization/magnetocorp/configuration/cli/
docker-compose -f docker-compose.yml up -d cliMagnetoCorp
```

启动成功：

![image-20200817231616266](https://i.loli.net/2020/08/17/vYdz7hWH3SEMnUF.png)

查看启动状态：

![image-20200818001616150](https://i.loli.net/2020/08/18/QeX3BnjcpztlWML.png)



**==（5）安装和实例化合约==**

**1、合约介绍**

此项目智能合约（链码）位于/home/wangxin/go/fabric-samples/commercial-paper/organization/magnetocorp/contract/lib/下的papercontract.js文件，代码如下：

```
/*
SPDX-License-Identifier: Apache-2.0
*/

'use strict';

// Fabric smart contract classes
const { Contract, Context } = require('fabric-contract-api');

// PaperNet specifc classes
const CommercialPaper = require('./paper.js');
const PaperList = require('./paperlist.js');

/**
 * A custom context provides easy access to list of all commercial papers
 */
class CommercialPaperContext extends Context {

    constructor() {
        super();
        // All papers are held in a list of papers
        this.paperList = new PaperList(this);
    }

}

/**
 * Define commercial paper smart contract by extending Fabric Contract class
 *
 */
class CommercialPaperContract extends Contract {

    constructor() {
        // Unique name when multiple contracts per chaincode file
        super('org.papernet.commercialpaper');
    }

    /**
     * Define a custom context for commercial paper
    */
    createContext() {
        return new CommercialPaperContext();
    }

    /**
     * Instantiate to perform any setup of the ledger that might be required.
     * @param {Context} ctx the transaction context
     */
    async instantiate(ctx) {
        // No implementation required with this example
        // It could be where data migration is performed, if necessary
        console.log('Instantiate the contract');
    }

    /**
     * Issue commercial paper
     *
     * @param {Context} ctx the transaction context
     * @param {String} issuer commercial paper issuer
     * @param {Integer} paperNumber paper number for this issuer
     * @param {String} issueDateTime paper issue date
     * @param {String} maturityDateTime paper maturity date
     * @param {Integer} faceValue face value of paper
    */
    async issue(ctx, issuer, paperNumber, issueDateTime, maturityDateTime, faceValue) {

        // create an instance of the paper
        let paper = CommercialPaper.createInstance(issuer, paperNumber, issueDateTime, maturityDateTime, faceValue);

        // Smart contract, rather than paper, moves paper into ISSUED state
        paper.setIssued();

        // Newly issued paper is owned by the issuer
        paper.setOwner(issuer);

        // Add the paper to the list of all similar commercial papers in the ledger world state
        await ctx.paperList.addPaper(paper);

        // Must return a serialized paper to caller of smart contract
        return paper.toBuffer();
    }

    /**
     * Buy commercial paper
     *
     * @param {Context} ctx the transaction context
     * @param {String} issuer commercial paper issuer
     * @param {Integer} paperNumber paper number for this issuer
     * @param {String} currentOwner current owner of paper
     * @param {String} newOwner new owner of paper
     * @param {Integer} price price paid for this paper
     * @param {String} purchaseDateTime time paper was purchased (i.e. traded)
    */
    async buy(ctx, issuer, paperNumber, currentOwner, newOwner, price, purchaseDateTime) {

        // Retrieve the current paper using key fields provided
        let paperKey = CommercialPaper.makeKey([issuer, paperNumber]);
        let paper = await ctx.paperList.getPaper(paperKey);

        // Validate current owner
        if (paper.getOwner() !== currentOwner) {
            throw new Error('Paper ' + issuer + paperNumber + ' is not owned by ' + currentOwner);
        }

        // First buy moves state from ISSUED to TRADING
        if (paper.isIssued()) {
            paper.setTrading();
        }

        // Check paper is not already REDEEMED
        if (paper.isTrading()) {
            paper.setOwner(newOwner);
        } else {
            throw new Error('Paper ' + issuer + paperNumber + ' is not trading. Current state = ' +paper.getCurrentState());
        }

        // Update the paper
        await ctx.paperList.updatePaper(paper);
        return paper.toBuffer();
    }

    /**
     * Redeem commercial paper
     *
     * @param {Context} ctx the transaction context
     * @param {String} issuer commercial paper issuer
     * @param {Integer} paperNumber paper number for this issuer
     * @param {String} redeemingOwner redeeming owner of paper
     * @param {String} redeemDateTime time paper was redeemed
    */
    async redeem(ctx, issuer, paperNumber, redeemingOwner, redeemDateTime) {

        let paperKey = CommercialPaper.makeKey([issuer, paperNumber]);

        let paper = await ctx.paperList.getPaper(paperKey);

        // Check paper is not REDEEMED
        if (paper.isRedeemed()) {
            throw new Error('Paper ' + issuer + paperNumber + ' already redeemed');
        }

        // Verify that the redeemer owns the commercial paper before redeeming it
        if (paper.getOwner() === redeemingOwner) {
            paper.setOwner(paper.getIssuer());
            paper.setRedeemed();
        } else {
            throw new Error('Redeeming owner does not own paper' + issuer + paperNumber);
        }

        await ctx.paperList.updatePaper(paper);
        return paper.toBuffer();
    }

}

module.exports = CommercialPaperContract;

```

该智能合约特点：

1.从fabric 1.4开始新增了`fabric-contract-api` NPM包，包含`Contract, Context`两个新概念以及一些高级api接口

2.以前实现合约需要先实现init invoke（即定义路由实现），在集成`fabric-contract-api`后只要定义合约方法即可，内部自动实现路由

**2、合约安装**

安装合约作用是模拟cliMagnetoCorp组织来操作合约，目前此功能仅限于当前启动的basic-network的单组织网络，taget使用peer0，管理员也是org1的admin

```
cd /home/wangxin/go/fabric-samples/commercial-paper/organization/magnetocorp/contract/lib/
docker exec cliMagnetoCorp peer chaincode install -n papercontract -v 0 -p /opt/gopath/src/github.com/contract -l node
```

安装成功：

![image-20200817232534388](https://i.loli.net/2020/08/17/IRcEWwimsQOxYru.png)

**3、合约实例化**

```
docker exec cliMagnetoCorp peer chaincode instantiate -n papercontract -v 0 -l node -c '{"Args":["org.papernet.commercialpaper:instantiate"]}' -C mychannel -P "AND ('Org1MSP.member')"
```

实例化成功：

![image-20200817233138536](https://i.loli.net/2020/08/17/KnmJeaCu2i6U7wb.png)



**==（6）magnetocorp组织发布票据==**

在/home/wangxin/go/fabric-samples/commercial-paper/organization/magnetocorp/application/目录，文件结构如下：

![image-20200817234309032](https://i.loli.net/2020/08/17/d6zMwqgnxJyBa4o.png)

magnetocorp组织发布票据逻辑如下：

![img](https://hyperledger-fabric.readthedocs.io/en/release-1.4/_images/commercial_paper.diagram.8.png)

**1、安装依赖**

```
cd /home/wangxin/go/fabric-samples/commercial-paper/organization/magnetocorp/application/
npm install
```

安装成功：

![image-20200817234538100](https://i.loli.net/2020/08/17/v4o3bzperlBGcPk.png)

**2、导入钱包**

在magnetocorp/application目录，将basic-network里面org1的user1账户信息导入到wallet中

```
node addToWallet.js
```

导入成功：

![image-20200817234836173](https://i.loli.net/2020/08/17/wCdiAEjQehbnuyI.png)

查看导入钱包的账号信息：

```
cd /home/wangxin/go/fabric-samples/commercial-paper/organization/magnetocorp/identity/user/isabella/wallet$
tree
```

![image-20200817235207906](https://i.loli.net/2020/08/17/51NWAd9yquzsUtF.png)

**3、发布商业票据**

此处使用issue.js脚本创建并发布商品票据（commercial paper）

```
node issue.js
```

发布成功：

![image-20200817235512916](https://i.loli.net/2020/08/19/IcWF7YvwR3BMaDr.png)



**==（7）启动DigiBank组织的cli==**

magnetocorp组织发布票据， DigiBank组织购买票据，首先需要启动一个属于DigiBank组织的cli

```
cd /home/wangxin/go/fabric-samples/commercial-paper/organization/digibank/configuration/cli/
docker-compose -f docker-compose.yml up -d cliDigiBank
```

启动成功：

![image-20200818000145371](https://i.loli.net/2020/08/18/RpFDxSlX7jIeLc6.png)

查看启动状态：

![image-20200818001505701](https://i.loli.net/2020/08/18/bUBs5vZcoeIMHwT.png)



**==（8）DigiBank组织购买和赎回票据==**

在/home/wangxin/go/fabric-samples/commercial-paper/organization/digibank/application/目录，文件结构如下：

![image-20200818000412863](https://i.loli.net/2020/08/18/bLxgwBWthM6HrXN.png)

digibank组织购买、赎回票据逻辑如下：

![img](https://i.loli.net/2020/08/18/FoYhty6XOzuZLQI.png)

**1、安装依赖**

```
cd /home/wangxin/go/fabric-samples/commercial-paper/organization/digibank/application/
npm install
```

安装成功：

![image-20200818000726279](https://i.loli.net/2020/08/18/q2QdFNlEuS85J3o.png)

**2、导入钱包**

在digibank/application目录，将basic-network里面org1的user1账户信息导入到wallet中

```
node addToWallet.js
```

导入成功：

![image-20200818000803883](https://i.loli.net/2020/08/18/X3T7FdIMcphNbVL.png)

**3、购买票据**

```
node buy.js
```

购买成功：

![image-20200818000854421](https://i.loli.net/2020/08/18/j3b5xFKZMLA6EIh.png)

**4、赎回票据**

```
node redeem.js
```

赎回成功：

![image-20200818000941133](https://i.loli.net/2020/08/18/CTuIbNarW7qzfkO.png)



**==（9）关闭网络及其他服务==**

```
//关闭网络
cd /home/wangxin/go/fabric-samples/basic-network/
./stop.sh

//关闭magnetocorp的cli容器
cd /home/wangxin/go/fabric-samples/commercial-paper/organization/magnetocorp/configuration/cli/
docker-compose -f docker-compose.yml down

//关闭digibank的cli容器
cd /home/wangxin/go/fabric-samples/commercial-paper/organization/digibank/configuration/cli/
docker-compose -f docker-compose.yml down

//关闭logspout日志监听
docker stop logspout

docker ps
```

