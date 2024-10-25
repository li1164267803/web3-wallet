<template>
    <div>
        <van-space>
            <van-button type="primary" @click="createWallet">创建助记词</van-button>
            <!-- <van-button type="primary">导入钱包</van-button> -->
        </van-space>
        <template v-if="showMn">
            <p>{{ mnemonic }}</p>
            <van-button type="primary" @click="confirmSaveMnemonic">我已经保存</van-button>
        </template>
        <van-dialog v-model:show="show" title="请输入密码" show-cancel-button @confirm="confirmPassword">
            <van-cell-group inset>
                <van-field v-model="password" label="密码：" placeholder="请输入用户名" type="password" />
            </van-cell-group>
        </van-dialog>
        <van-dialog v-model:show="showMnDialog" title="请输入助记词" show-cancel-button @confirm="confirmMn">
            <van-cell-group inset>
                <van-field v-model="mnemonic2" label="助记词：" type="textarea" placeholder="请输入助记词" />
            </van-cell-group>
        </van-dialog>
    </div>
</template>
<script setup>
import { ref } from 'vue'
import { showNotify } from 'vant'
import * as bip39 from 'bip39'
import { hdkey } from 'ethereumjs-wallet'
import store2 from 'store2'

import 'vant/es/dialog/style'
import 'vant/es/notify/style'

const show = ref(false)
const password = ref('')
const mnemonic = ref('')
const mnemonic2 = ref('')
const showMn = ref(false)
const showMnDialog = ref(false)

const createWallet = () => {
    show.value = true
}

const confirmPassword = () => {
    if (password.value === '') {
        showNotify({ type: 'danger', message: '请输入密码' })
    } else {
        const walletInfo = store2.get('walletInfo')
        mnemonic.value = walletInfo ? walletInfo[0]['mnemonic'] : bip39.generateMnemonic()

        if (walletInfo) {
            confirmMn()
        } else {
            showMn.value = true
        }
    }
}

const confirmSaveMnemonic = () => {
    showMnDialog.value = true
}

const confirmMn = async () => {
    if (mnemonic2.value === '') return showNotify({ type: 'danger', message: '请输入助记词' })
    showMn.value = false
    const storeWallet = store2.get('walletInfo') || []
    if (mnemonic.value !== mnemonic2.value && storeWallet.length === 0) {
        return
    }
    const seed = await bip39.mnemonicToSeed(mnemonic.value)
    // 将助记词转换为种子
    const hdWallet = hdkey.fromMasterSeed(seed)
    // 从种子生成hd钱包
    const addressIndex = storeWallet.length === 0 ? 0 : storeWallet.length + 1
    // 如果本地存储为空，则地址索引为0，否则为本地存储长度加1
    const keyPair = hdWallet.derivePath(`m/44'/60'/0'/0/${addressIndex}`)
    // 从hd钱包中派生出keyPair
    const wallet = keyPair.getWallet()

    // 获取钱包地址字符串
    const lowerCaseAddress = wallet.getAddressString()
    // 获取校验和地址字符串
    const CheckSumAddress = wallet.getChecksumAddressString()
    // 获取私钥字符串
    const privateKey = wallet.getPrivateKey().toString('hex')
    // 将钱包转换为V3格式
    const keyStore = await wallet.toV3(password.value)
    // 模拟存储，实际开发不能存储到本地中
    const walletObj = {
        id: addressIndex,
        address: lowerCaseAddress,
        privateKey,
        keyStore,
        mnemonic: mnemonic.value,
        balance: 0,
    }
    storeWallet.push(walletObj)
    store2('walletInfo', storeWallet)
    window.location.reload()
}
</script>

<style lang="less">
p {
    user-select: all;
}
</style>
