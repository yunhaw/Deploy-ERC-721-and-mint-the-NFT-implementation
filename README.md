# 發行 ERC-721 與鑄造 NFT 之實作

###### tags: `Notes`
###### Title: Deploy the ERC-721 smart contract and mint the non-fungible token 
###### Author: Raheem
###### Date: 2022.01.27

---
# Outline
> * HackMD
> * GitHub
> * Referece
1. Environment and tools
2. Introduction
3. Implementation
4. Conclusions

## HackMD
> * https://hackmd.io/@Raheem/HJXMl8eCK

## GitHub 
> * https://github.com/yunhaw/Deploy-ERC-721-and-mint-the-NFT-implementation
> * 下載後請將資料夾名稱改為：my-nft
> * [ .env ] 檔案內的 key 請自行填入
> * 智能合約與腳本內的地址與網址請自行更改
> * 因上傳檔案數限制，[ node_modules ] 資料夾為空 
> > * 請自行透過 npm 安裝相關套件

## Referece
> * Source：https://ethereum.org/en/developers/tutorials/how-to-write-and-deploy-an-nft/
> * Related work： https://hackmd.io/@Raheem/Byc6pMCYK

## 1. Environment and tools
> * OS：macOS Monterey (MacBook M1)
> * VM：paralles Ubuntu 20.04.2 ARM64
> * Editor：Visual Studio code
> * Smart contract language：Solidity ^0.8.0
> * Wallet：Metamask
> * Blockchain testnet：Ethererum Ropsten
> * Blockchain developer platform：Alchemy
> * NFT metadata：Pinata 
> * Development environment：Hardhat
> * API, library and module：Alchemy Web3 / Ether.js / OpenZepplin

## 2. Introduction

* ERC-721
    * Reference：
        * ERC-721 官方介紹文件
https://ethereum.org/en/developers/docs/standards/tokens/erc-721/
    * 又稱為 非同質化代幣（Non-fungible token, NFT） 
    * 是智能合約的其一項目，可以依照 ERC-721 智能合約標準規範發行與鑄造貨幣
    * 圖像、影片、音樂、文件等數位檔案皆可發行並鑄造成 NFT
    * 每個 NFT 具備唯一性（Token ID），且可以交易流通
    * 因為是智能合約，儲存於區塊鏈，該智能合約內容無法再變更且公開透明（可以在區塊鏈瀏覽器上查詢）
    * NFT 的數位內容則儲存於 [ 外部伺服器 ]，像是 IPFS，使數位內容不可竄改且永久保存

* NFT 圈流行術語與環境圖
    * Reference： 
        * 幣修學分
https://www.facebook.com/114033260281793/posts/477966090555173/?d=n
    * 以下內容來自 [ 幣修學分 ]
    
    >【 白名單 White List 】
    > > 可以搶先購買 or 鑄造 NFT、不用跟一般人搶的資格，簡稱 WL
    
    >【 Discord 】
    > > 一個通訊軟體，近幾個月有許多 GameFi、NFT 項目在上面組建自己的社群，簡稱 DC
    
    > 【 肝 】
    > > 在 Discord 裡聊天升等以拿到白名單資格的行為
    
    > 【 OpeaSea 】
    > > 目前最多人使用、交易量最大的 NFT 交易平台
    
    > 【 一級市場 】
    > > 初次用貨幣換取 NFT 的場域，通常會在項目方的官網發生（直接給 ETH Mint 出 NFT）
    
    > 【 二級市場 】
    > > 將從一級市場得到的 NFT 再次上架出售的場域，如 OpenSea
    
    > 【 Mint 】
    > > 一般人從項目的官網上鑄造出 NFT 的過程
    
    > 【 空投 】
    > > 項目方常見的空投是項目方直接 Mint NFT 到某人的錢包
    
    > 【 地板價 】
    > > 目前該 NFT 在二級市場的最低價格
    
    > 【 科學家 】
    > > 能寫出厲害機器人來幫自己達成目的的工程師，例如在項目公售時以最快速度掃貨
    
    > 【 Gas 】
    > > 凡是在鏈上做任何操作（如 Mint NFT、打幣）時都須支付給礦工的手續費，Gas 會隨當下鏈上活動的熱度浮動
    
    > 【 Gas War 】
    > > 當有熱門項目在公售時，Gas 會飆升至平均水準的好幾倍，這時支付高昂的 Gas 才會更快完成交易 aka 搶到 NFT，就像一場戰爭
    
    > 【 奶 】
    > > 吹捧、站台、說好話的意思
    
    > 【 Looks rare 】
    > > 比喻 NFT 看起來很稀有

* NFT 環境圖
    * 圖片來源：幣修學分
![](https://i.imgur.com/yQA47Lj.jpg)

* 實作 NFT 的檔案結構與元件圖
    * 將在 3. Implementation 描述每個檔案的生成方式與用途
> * my-nft
> > * Contract 
> > > * MyNFT.sol
> * Scripts
> > > * deploy.js
> > > * mint-nft.js
> * node_modules
> > > * Hardhat
> > > * OpenZenppelin lib
> > > * Ether.js
> > > * Alchemy Web3 
> * .env
> * nft-metadata.json
> * package.json
> * hardhat.config.js

> 外部環境
> > * Ropsten
> > * Pinata
> > * Alchemy
> > * Metamask

* 檔案結構與相關元件示意圖

![](https://i.imgur.com/XK58eHj.png)

* 實作流程總共分為三個階段，分別為：
    * Step 1 為 ERC-721 的 [ 發行（Deploy） ] 流程
    * Step 2 為 NFT 的 [ 鑄造（Mint） ] 流程
    * Step 3 為將鑄造後的 NFT 加入錢包（metamask）中預覽

## 3. Implementation
### Step 1-1： 註冊 Alchemy
> * Alchemy 透過瀏覽器操作，不需要額外下載安裝軟體
> * Alchemy 可以查看鏈上資訊與發行的 NFT 項目之相關訊息

1. 註冊 Alchemy 帳號
    * https://auth.alchemyapi.io/?redirectUrl=https%3A%2F%2Fdashboard.alchemyapi.io%2Fsignup
1. NETWORK 選擇 Ropsten
1. 依照註冊流程即可創建 NFT 項目專案

![](https://i.imgur.com/LMuc8pj.jpg)

![](https://i.imgur.com/5gZ9vDj.jpg)

![](https://i.imgur.com/5RjYd4v.jpg)

![](https://i.imgur.com/xtAWjsG.jpg)


### Step 1-2： 開啟 Metamask 
> * Metamask 選擇 Ropsten 測試網路
> * Metamask 官方網站： https://metamask.io
> * 部署智能合約需要 gas fee
> * 領取測試幣筆記： https://hackmd.io/@Raheem/r1dDDdq6F

* 並將地址加入 Alchemy

![](https://i.imgur.com/C9HGXHJ.png)

![](https://i.imgur.com/E77WY6i.png)

![](https://i.imgur.com/ZNqHJ35.png)

### Step 1-3： 建立檔案
> * 於本機建立 NFT 相關檔案
> * 以下指令皆透過 terminal 執行
> * 注意 npm 版本

* 安裝 npm
```shell=
$ sudo apt install npm 
```
![](https://i.imgur.com/VN5W12V.jpg)

1. 首先創建一個 [ 資料夾 ]，路徑不拘，名稱為： my-nft
1. 初始化
1. 持續按 enter 直到在檔案夾內出現 [ package.json ]
2. 於 my-nft 資料夾底下輸入以下指令

```shell=
$ npm init
```

![](https://i.imgur.com/iHCVMYr.jpg)

![](https://i.imgur.com/cX2uxjS.jpg)

* package.json 檔案內容如下：
    * 隨著執行後續步驟，該內容會有差異

```json=
package name: (my-nft)
version: (1.0.0)
description: My first NFT!
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
About to write to /Users/thesuperb1/Desktop/my-nft/package.json:

{
  "name": "my-nft",
  "version": "1.0.0",
  "description": "My first NFT!",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

### Step 1-4： 安裝 Hardhat
> * Hardhat 是提供 [ 智能合約 ] 的編譯、部署、測試、偵錯等的開發環境工具
> * 類似 Remix IDE
> * Hardhat 官方網站： https://hardhat.org

* 透過以下指令安裝 hardhat

```shell=
$ npm install --save-dev hardhat
```

![](https://i.imgur.com/S4m4aiI.jpg)

* 開啟 Hardhat 服務

```shell=
$ npx hardhat
```

![](https://i.imgur.com/IA39OpP.jpg)

* 建立 [ hardhat.config.js ]
    * 用方向鍵選擇選項 Create an empty hardhat.config.js

![](https://i.imgur.com/GVStuRZ.jpg)

### Step 1-5： 撰寫智能合約
> * 於 [ my-nft ] 資料夾內建立兩個資料夾，名稱為 [ contracts ] 與 [ scripts ]
> * [ contracts ] 資料夾存放智能合約
> * [ scripts ] 資料夾存放部署合約腳本
> * VScode 官方網站： https://code.visualstudio.com
> * OpenZeppelin ERC-721 文件： https://docs.openzeppelin.com/contracts/3.x/erc721

* 建立 contracts 與 scripts 資料夾如下圖，其餘檔案為後續步驟，先不必理會
![](https://i.imgur.com/r2OZKd0.jpg)

* 透過以下指令於 my-nft 資料夾底下安裝 openzepplelin lib

```shell=
$ npm install @openzeppelin/contracts
```

* 於 [ contracts ] 資料夾內創建 [ MyNFT.sol ] 智能合約檔案
* MyNFT.sol 內容如下：

```solidity=
//Contract based on [https://docs.openzeppelin.com/contracts/3.x/erc721](https://docs.openzeppelin.com/contracts/3.x/erc721)
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";

contract MyNFT is ERC721URIStorage, Ownable {
    using Counters for Counters.Counter;
    Counters.Counter private _tokenIds;

    constructor() public ERC721("MyNFT", "NFT") {}

    function mintNFT(address recipient, string memory tokenURI)
        public onlyOwner
        returns (uint256)
    {
        _tokenIds.increment();

        uint256 newItemId = _tokenIds.current();
        _mint(recipient, newItemId);
        _setTokenURI(newItemId, tokenURI);

        return newItemId;
    }
}
```

![](https://i.imgur.com/B5DfO2f.jpg)

### Step 1-6： 連結 Metamask 與 Alchemy
> * 需要到 Metamask 導出私鑰（private key）
> * 需要到 Alchemy 提取 NFT 專案的 API key

* 在 my-nft 資料夾底下輸入以下指令：

```shell=
$ npm install dotenv --save
```

![](https://i.imgur.com/GpQJZy2.jpg)

1. 在 my-nft 資料夾內建立 [ .env ] 檔案
1. 於 [ .env ] 檔案內輸入以下
    * your-api-key 填入 Alchemy 的 API key
    * your-metamask-private-key 填入 metamask 導出的私鑰

```json=
API_URL="https://eth-ropsten.alchemyapi.io/v2/your-api-key"
PRIVATE_KEY="your-metamask-private-key"
```

* Metamask 提取私鑰

![](https://i.imgur.com/o0LQ1dU.png)

![](https://i.imgur.com/QwVK4OS.png)

* Alchemy 提取 API key

![](https://i.imgur.com/kJVv4Fz.jpg)

### Step 1-7： 安裝 Ether.js 與更新 hardhat.config.js
> * Ether.js 是協助與 Ethererum 互動的 lib
> * Ether.js 詳細敘述可看 Source
> * hardhat.config.js 為 Step 1-4 透過 hardhat 服務所建立
> * hardhat.config.js 在 my-nft 資料夾內

1. 於 my-nft 資料夾底下安裝 ether.js 
1. 輸入以下指令

```shell=
$ npm install --save-dev @nomiclabs/hardhat-ethers ethers@^5.0.0
```

![](https://i.imgur.com/m9AO2N3.jpg)

* 更改 hardhat.config.js 內容如下
    * hardhat.config.js 在 my-nft 資料夾內

```javascript=
/**
* @type import('hardhat/config').HardhatUserConfig
*/
require('dotenv').config();
require("@nomiclabs/hardhat-ethers");
const { API_URL, PRIVATE_KEY } = process.env;
module.exports = {
   solidity: "0.8.0",
   defaultNetwork: "ropsten",
   networks: {
      hardhat: {},
      ropsten: {
         url: API_URL,
         accounts: [`0x${PRIVATE_KEY}`]
      }
   },
}
```

![](https://i.imgur.com/bAS6D7d.jpg)

### Step 1-8： 編譯與部署智能合約
> * 透過 hardhat 進行智能合約的編譯與部署

* 對 my-nft 檔案夾進行編譯

```shell=
$ npx hardhat compile
```

![](https://i.imgur.com/ACV096I.jpg)

1. 撰寫智能合約部署腳本
2. 於 [ scripts ] 資料夾內建立 [ deploy.js ] 檔案
3. [ deploy.js ] 內容如下

```javascript=
async function main() {
  const MyNFT = await ethers.getContractFactory("MyNFT")

  // Start deployment, returning a promise that resolves to a contract object
  const myNFT = await MyNFT.deploy()
  await myNFT.deployed()
  console.log("Contract deployed to address:", myNFT.address)
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error)
    process.exit(1)
  })
```

![](https://i.imgur.com/77IgIzb.jpg)

1. 部署智能合約
2. 透過 hardhat 執行 [ deploy.js ] 進行智能合約的部署
3. 於 my-nft 資料夾底下，指令如下

```shell=
$ npx hardhat --network ropsten run scripts/deploy.js
```

![](https://i.imgur.com/Fi0z3Sk.jpg)


### Step 1-9： 查看智能合約部署結果
> * Ropsten 測試鏈瀏覽器：
> * 可以透過區塊鏈瀏覽器查看智能合約是否部署成功（寫入區塊被清算且上鏈）
> * 此次實驗的交易紀錄： https://ropsten.etherscan.io/address/0x77604a0282e5ad8c775430b58663822651218d83

* Etherscan 

![](https://i.imgur.com/cqo8Mba.png)

![](https://i.imgur.com/ft2BE6w.png)

* Alchemy

![](https://i.imgur.com/YW8tiT3.jpg)

![](https://i.imgur.com/wNy76TK.jpg)


### Step 2-1： 上傳 NFT 的內容檔案
> * 需要先註冊 Pinata 帳號
> * 準備欲發行的測試圖像
> * 準備 nft-metadata.json，為 NFT 數位內容的描述檔
> * Pinata 官方網站： https://app.pinata.cloud
> * Pinata 為協助上傳 NFT 數位內容至 IPFS 的服務平台

* 註冊 Pinata帳號

![](https://i.imgur.com/brF7GY1.png)

* 準備欲鑄造成 NFT 的檔案
    * 此次實驗使用圖像作為發行內容，如下圖

![](https://i.imgur.com/dEZpu5q.png)

* 於 Pinata 網站上傳檔案

![](https://i.imgur.com/Wxreo21.png)

![](https://i.imgur.com/uhg0knw.png)

![](https://i.imgur.com/Q0Besho.png)

* 撰寫 nft-metadata.json
> * 描述檔之內容皆可隨意更改
> * 程式碼第 13 行，網址最後的 CID 要改成該 NFT metadata 的 CID
> * CID 於 Pinata 提取 

1. 於 my-nft 資料夾內建立 [ nft-metadata ] 檔案
1. nft-metadata.json 內容如下

```json=
{
  "attributes": [
    {
      "trait_type": "Breed",
      "value": "Maltipoo"
    },
    {
      "trait_type": "Eye color",
      "value": "Mocha"
    }
  ],
  "description": "The world's most adorable and sensitive pup.",
  "image": "https://gateway.pinata.cloud/ipfs/QmWmvTJmJU3pozR9ZHFmQC2DNDwi2XJtf3QGyYiiagFSWb",
  "name": "Ramses"
}
```

* 將 nft-metadata.json 上傳 Pinata

![](https://i.imgur.com/sg2Zhrs.png)

* 數位內容 與 nft-metadata.json 檔皆上傳 Pinata
> * 確保該 NFT 之內容不可再變更且永久保存

![](https://i.imgur.com/Ep7yGLC.png)

### Step 2-2： 建立 NFT 鑄造腳本

1. 首先安裝 Alchemy Web3 套件
1. 於 my-nft 資料夾底下輸入以下指令

```shell=
$ npm install @alch/alchemy-web3
```

![](https://i.imgur.com/MYc32az.jpg)

1. 對 my-nft 資料夾中的 [ .env ] 檔案進行修改
1. [ .env ] 加入 metamask的 地址（public key）
2. 第 9 行為 Step 1 部署上鏈的智能合約地址
3. 第 50 行 mintNFT 最後的 CID 為 Pinata 上 nft-metadata.json 之 CID

```shell=
$ PUBLIC_KEY = "your-public-account-address"
```

![](https://i.imgur.com/n2bGKbm.jpg)

1. 於 [ scripts ] 資料夾內建立 [ mint-nft.js ] 腳本
1. 執行 [ mint-nft.js ] 可進行鑄造 NFT

```javascript=
require("dotenv").config()
const API_URL = process.env.API_URL;
const PUBLIC_KEY = process.env.PUBLIC_KEY;
const PRIVATE_KEY = process.env.PRIVATE_KEY;
const { createAlchemyWeb3 } = require("@alch/alchemy-web3")
const web3 = createAlchemyWeb3(API_URL)

const contract = require("../artifacts/contracts/MyNFT.sol/MyNFT.json")
const contractAddress = "0x77604a0282e5ad8c775430b58663822651218d83"
const nftContract = new web3.eth.Contract(contract.abi, contractAddress)

async function mintNFT(tokenURI) {
  const nonce = await web3.eth.getTransactionCount(PUBLIC_KEY, 'latest'); //get latest nonce

  //the transaction
  const tx = {
    from: PUBLIC_KEY,
    to: contractAddress,
    nonce: nonce,
    gas: 500000,
    data: nftContract.methods.mintNFT(PUBLIC_KEY, tokenURI).encodeABI(),
  }

  const signPromise = web3.eth.accounts.signTransaction(tx, PRIVATE_KEY)
  signPromise
    .then((signedTx) => {
      web3.eth.sendSignedTransaction(
        signedTx.rawTransaction,
        function (err, hash) {
          if (!err) {
            console.log(
              "The hash of your transaction is: ",
              hash,
              "\nCheck Alchemy's Mempool to view the status of your transaction!"
            )
          } else {
            console.log(
              "Something went wrong when submitting your transaction:",
              err
            )
          }
        }
      )
    })
    .catch((err) => {
      console.log(" Promise failed:", err)
    })
}

mintNFT(
    "https://gateway.pinata.cloud/ipfs/QmcZhDW178zvrNFxsCCkgcPd3T4Kg1496G9uFALMLvEsM7"
)
```

![](https://i.imgur.com/op2xnnZ.jpg)

### Step 2-3 ： 執行 mint-nft.js 進行該 NFT 項目的鑄造

* 於 my-nft 資料夾底下執行以下指令：

```shell=
$ node scripts/mint-nft.js
```

![](https://i.imgur.com/WHFz36E.jpg)

### Step 2-4 ：於 Etherscan 上查看鑄造結果
> * 交易紀錄： https://ropsten.etherscan.io/address/0x77604a0282e5ad8c775430b58663822651218d83

![](https://i.imgur.com/seZFaZR.png)

![](https://i.imgur.com/rPKP2K3.png)

### Step 3-1 ： 將數位收藏品（NFT）加入錢包（metamask）
> * 目前 Metamask 僅提供手機版 APP 預覽收藏品（NFT）
> * Metamask 瀏覽器插件目前尚不支援預覽收藏品（NFT）
> * 未來會更新

1. 開啟手機版 metamask 
1. 選擇 Ropsten Test Network
1. 點選 [ 收藏品 ]
1. 點選 [ 添加收藏品 ]

![](https://i.imgur.com/n1TKlqS.png)

1. 輸入該 NFT 的智能合約 [ 地址 ]
1. 輸入該 NFT 鑄造生成的 [ Token ID ]
2. 點選 [ 添加 ]

![](https://i.imgur.com/kfg0HVv.png)

![](https://i.imgur.com/YC1EEEw.png)

* 查看 NFT

![](https://i.imgur.com/q9Fr3rB.png)

![](https://i.imgur.com/Gii1Keb.png)

## 4. Conclusions

* 本次實驗主要針對發行（deploy） ERC-721 智能合約與鑄造（mint）NFT
    * 此次實驗是發行並鑄造圖檔，下次試試看發行音樂檔案
        * 發行並鑄造 NFT 的方法還有許多，可以試試其他方式，嘗試不同的開發工具與環境
    * 下次將研究 NFT marketplace platform 二級市場
    * 深入研究 ERC-721 智能合約的相關進階應用
        * 像是當作區塊鏈遊戲的入場券
        * 設定版稅、嵌入版權資訊等
 
* 本次實作過程遇到的困難
    * 使用 Mackbook M1 進行開發，透過 parallels 虛擬機的 Ubuntu 當作開發環境，因為 ARM64 的架構，npm 安裝套件時會出現許多非預期的問題
        * 下次換個開發環境應該會較順利
