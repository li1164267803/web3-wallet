# web3-wallet-app

> 这是一个使用 vue3+web3js 实现的一个钱包功能，只做一些能力表述，部分代码不可直接用于工作中
> 功能有创建助记词、创建账户、产生 keyStore、数字签名、发起转账功能

## node 版本

```
v18.19.0
```

## 安装依赖

```
npm install
```

### 本地运行

```
npm run serve
```

# 概念

钱包
去中心化钱包 前端直接和区款连交互 不经过后端
区块链浏览器
查看所有的账户信息 余额 和 交易记录

账户系统
钱包地址（区块地址）
私钥
助记词
keystore

1.  通过助记词创建账户
2.  通过助记词 + 密码产生 keystore
3.  发起转账

## 项目使用到的 api

#### account

##### 通过助记词分层钱包创建账户

通过助记词分层钱包创建账户

```javascript
import { hdkey } from "ethereumjs-wallet";
import \* as bip39 from "bip39";
let mnemonic = bip39.generateMnemonic();

      //1.将助记词转成seed
      let seed = await bip39.mnemonicToSeed(mnemonic);
      //3.通过hdkey将seed生成HD Wallet
      let hdWallet = hdkey.fromMasterSeed(seed);
      //4.生成钱包中在m/44'/60'/0'/0/i路径的keypair
      let keyPair = hdWallet.derivePath("m/44'/60'/0'/0/0");
      // 获取钱包对象
      let wallet = keyPair.getWallet();
      // 获取钱包地址
      let lowerCaseAddress = wallet.getAddressString();
      // 获取钱包校验地址
      let CheckSumAddress = wallet.getChecksumAddressString();
      // 获取私钥
      let prikey = wallet.getPrivateKey().toString("hex");
      console.log(prikey, data.pass1);
      let keystore = await wallet.toV3(data.pass1);
```

##### 通过助记词导入账户

```javascript
//1.将助记词转成seed
let seed = await bip39.mnemonicToSeed(data.mnemonic)
//3.通过hdkey将seed生成HD Wallet
let hdWallet = hdkey.fromMasterSeed(seed)
//4.生成钱包中在m/44'/60'/0'/0/i路径的keypair
let keyPair = hdWallet.derivePath("m/44'/60'/0'/0/0")
// 获取钱包对象
let wallet = keyPair.getWallet()
// 获取钱包地址
let lowerCaseAddress = wallet.getAddressString()
// 获取钱包校验地址
let CheckSumAddress = wallet.getChecksumAddressString()
// 获取私钥
let prikey = wallet.getPrivateKey().toString('hex')
console.log(prikey, data.pass1)
let keystore = await wallet.toV3(data.pass1)
```

##### 通过 密码 + keystore 获取私钥

```javascript
import ethwallet from 'ethereumjs-wallet'
let pass = prompt('请输入密码')
let wallet
try {
    wallet = await ethwallet.fromV3(keystore, pass)
} catch (error) {
    alert('密码错误')
    return false
}
let key = wallet.getPrivateKey().toString('hex')
```

#### eth

##### 创建 web3 实例对象

```javascript
import Web3 from 'web3'
const geerliWS = 'wss://goerli.infura.io/ws/v3/e4f789009c9245eeaad1d93ce9d059bb'
this.web3 = new Web3(Web3.givenProvider || wsUrl)
```

##### 获取 eth 余额

```javascript
this.web3.eth.getBalance(address).then(balanceWei => {
    const balance = Web3.utils.fromWei(balanceWei, 'ether')
})
```

##### eth 单位之间的转换

```javascript
// wei 转化为eth
Web3.utils.fromWei(balanceWei, 'ether')
// eth 转化为 wei
Web3.utils.toWei(value)
```

##### 创建 eth 转账交易 hash

```javascript
// 创建交易hash
  async createTransationHx(key, fromAddress, toAddress, value) {
    const nonce = await this.web3.eth.getTransactionCount(fromAddress);
    var privateKey = new Buffer(key, "hex");
    // 获取预计转账gas费
    let gasPrice = await this.web3.eth.getGasPrice();
    // 转账金额以wei为单位
    let valueWei = Web3.utils.toWei(value);
    console.log(valueWei);
    // 转账的记录对象
    var rawTx = {
      from: fromAddress,
      nonce: nonce,
      gasPrice: gasPrice,
      to: toAddress,
      value: valueWei,
      data: "0x00", //转Token代币会用到的一个字段
    };
    //需要将交易的数据进行预估gas计算，然后将gas值设置到数据参数中
    let gas = await this.web3.eth.estimateGas(rawTx);
    rawTx.gas = gas;
    // 通过tx实现交易对象的加密操作
    var tx = new Tx(rawTx);
    tx.sign(privateKey);
    var serializedTx = tx.serialize();
    var transationHx = "0x" + serializedTx.toString("hex");
    return transationHx;
  }
```

##### 通过交易 hash 发起转账

```javascript
this.web3.eth
    .sendSignedTransaction(hx)
    .on('transactionHash', txid => {
        console.log('交易成功,请在区块链浏览器查看')
        console.log('交易id', txid)
        console.log(`https://goerli.etherscan.io/tx/${txid}`)
    })
    .on('receipt', ret => {
        cb && cb(ret)
        console.log('receipt', ret)
        const { transactionHash } = ret
        this.createOrderData(transactionHash)
    })
    .on('latestBlockHash', (...arg) => {
        console.log('latestBlockHash', arg)
    })
    .on('error', err => {
        console.log('error:')
        console.log(err)
    })
```
