<template>
    <div>
        <van-cell :title="item.address" icon="user-o" v-for="item in walletInfoAddressFilter">
            <!-- 使用 right-icon 插槽来自定义右侧图标 -->
            <template #right-icon>
                <van-button size="small" class="btn" @click="getPassword(item.keyStore, item.originalAddress)">{{ item.balance }}</van-button>
            </template>
        </van-cell>
    </div>
    <van-dialog v-model:show="show" title="请输入密码" show-cancel-button @confirm="confirmPassword">
        <van-cell-group inset>
            <van-field v-model="password" label="密码：" placeholder="请输入用户名" type="password" />
            <van-field v-model="toaddress" label="转入账户：" placeholder="请输入转入账户名" />
            <van-field v-model="number" label="转出金额：" placeholder="请输入转出金额：" />
        </van-cell-group>
    </van-dialog>
</template>

<script setup>
import { ref, defineProps, computed } from 'vue'
import Web3 from 'web3'
import ethwallet from 'ethereumjs-wallet'
import Tx from 'ethereumjs-tx'

const props = defineProps(['wallet-info'])
const web3 = new Web3(Web3.givenProvider || 'wss://goerli.infura.io/ws/v3/cb7e63cf28244e4499b4b6fb6162e746')

const walletInfoAddressFilter = computed(() => {
    props.walletInfo.forEach(async (item, index) => {
        const originalAddress = item.address
        const head = item.address.slice(0, 8)
        const tail = item.address.slice(item.address.length - 8)
        item.address = head + '...' + tail
        item.originalAddress = originalAddress

        // 根据地址获取余额
        const result = await web3.eth.getBalance(originalAddress)
        item.balance = web3.utils.fromWei(result, 'ether')
        item.balance = Number(item.balance).toFixed(6)
    })
    return props.walletInfo
})

const show = ref(false)
const password = ref('')
const keystore = ref('')
const fromaddress = ref('')
const toaddress = ref('')
const number = ref('')

const send = async (keystore, pass, fromaddress) => {
    let walletobj
    try {
        walletobj = await ethwallet.fromV3(keystore, pass)
    } catch (error) {
        alert('密码错误')
        return false
    }
    let key = walletobj.getPrivateKey().toString('hex')
    var privateKey = Buffer(key, 'hex')

    // 获取账户交易次数
    let nonce = await web3.eth.getTransactionCount(fromaddress.slice(2))
    // 获取预计转账gas费
    let gasPrice = await web3.eth.getGasPrice()
    // 转账金额以wei为单位

    let balance = Web3.utils.toWei(number.value)

    var rawTx = {
        from: fromaddress,
        nonce: nonce,
        gasPrice: gasPrice,
        to: toaddress.value,
        value: balance,
        data: '0x00', //转Token代币会用到的一个字段
    }

    //需要将交易的数据进行预估gas计算，然后将gas值设置到数据参数中
    let gas = await web3.eth.estimateGas(rawTx)
    rawTx.gas = gas
    // 通过 ethereumjs-tx 实现私钥加密Ï
    var tx = new Tx(rawTx)
    tx.sign(privateKey)
    var serializedTx = tx.serialize()

    web3.eth
        .sendSignedTransaction('0x' + serializedTx.toString('hex'))
        .on('transactionHash', txid => {
            console.log('交易成功,请在区块链浏览器查看')
            console.log('交易id', txid)
            console.log(`https://goerli.etherscan.io/tx/${txid}`)
        })
        // .on('receipt', (ret)=>{console.log('receipt')})
        // .on('confirmation', (ret)=>{console.log('confirmation')})
        .on('error', err => {
            console.log('error:' + err)
        })
}

const getPassword = (keyStore, address) => {
    show.value = true
    keystore.value = keyStore
    fromaddress.value = address
}

const confirmPassword = () => {
    send(keystore.value, password.value, fromaddress.value)
    password.value = ''
}
</script>

<style lang="less" scoped>
.btn {
    width: 100px;
}
</style>
