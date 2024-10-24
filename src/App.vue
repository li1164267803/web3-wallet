<script setup>
import { ref, onMounted } from 'vue'
import ButtonVue from './components/Button.vue'
import AccountList from './components/AccountList.vue'
import store2 from 'store2'

const walletInfo = ref([])

onMounted(() => {
    walletInfo.value = store2.get('walletInfo') || []
})
</script>

<template>
    <ButtonVue></ButtonVue>
    <AccountList :wallet-info="walletInfo"></AccountList>
</template>

<style lang="less">
body {
    padding: 10px;
}
</style>

leaf patient twelve mercy base chunk attend bomb scorpion retreat wage ethics
