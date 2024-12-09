
<script setup>
import { showNotify } from 'vant/es'
import { ref } from 'vue'
import * as bip39 from 'bip39'
import { hdkey } from 'ethereumjs-wallet'
import store2 from 'store2'
let show = ref(false)
let showMn = ref(false)
let showMnDialog = ref(false)
let password = ref("")
let mnemonic = ref("")
let mnemonic2 = ref("")
// 创建钱包
const createWallet = () =>{

    console.log(100)
    show.value = true
}

const confirmPassword = () => {
    console.log(password.value)
    if(!password.value) {
        showNotify({type:'danger',message:'密码不能为空'})
    } else {
        const walletInfo = store2.get('walletInfo') || null
        if(walletInfo) {
            mnemonic.value = walletInfo[0].mnemonic
            confirmMn()
        }else{
            mnemonic.value = bip39.generateMnemonic()
            showMn.value = true
        }
       
    }
}

// 保存助记词
const confirmSaveMnemonic = () => {
    showMnDialog.value = true
}

// 确认助记词
const confirmMn = async () => {
    console.log(mnemonic2.value)
    showMn.value = false
    const storeWallet = store2.get('walletInfo') || []
    if(mnemonic2.value != mnemonic.value && storeWallet.length === 0) {
        return
    }
    console.log('助记词正确')
    const seed = bip39.mnemonicToSeedSync(mnemonic.value)
    // 钱包种子
    const hdWallet = hdkey.fromMasterSeed(seed)
    // 地址索引        
    const addressIndex = storeWallet.length === 0 ? 0 : storeWallet.length + 1
    // 密钥对
    const keyPair = hdWallet.derivePath(`m/44'/60'/0'/0/${addressIndex}`)
    // 获取钱包
    const wallet = keyPair.getWallet()
    // 获取地址
    const lowerCaseAddress = wallet.getAddressString()

    const checkSumAddress = wallet.getChecksumAddressString()
    // 获取私钥
    const privateKey  = wallet.getPrivateKeyString().toString("hex")
    // 获取公钥
    const keyStore = await wallet.toV3(password.value)
    console.log(lowerCaseAddress, privateKey)

    const walletObj = {
        id: addressIndex,
        address: lowerCaseAddress,
        privateKey,
        keyStore,
        mnemonic:mnemonic.value,
        balance:0
    }
    console.log("walletInfo", walletObj)
    storeWallet.push(walletObj)
    store2('walletInfo',storeWallet)
}

</script>
<template>
   <div>
        <van-space>
            <van-button type="primary" @click="createWallet">创建钱包</van-button>
            <van-button type="primary">导入钱包</van-button>
        </van-space>
        
        <van-dialog v-model:show="show" title="请输入密码" show-cancel-button @confirm="confirmPassword">
            <van-cell-group>
                <van-field label="密码:" type="password" v-model="password" placeholder="请输入" />
            </van-cell-group>
        </van-dialog>

        <div v-if="showMn">
            <p>{{ mnemonic }}</p>
            <van-button v-if="mnemonic" type="primary" size="small" @click="confirmSaveMnemonic">我已经保存</van-button>
        </div>

        <van-dialog v-model:show="showMnDialog" title="请输入助记词" show-cancel-button @confirm="confirmMn">
            <van-cell-group>
                <van-field label="助记词:" v-model="mnemonic2" placeholder="请输入" />
            </van-cell-group>
        </van-dialog>
   </div>
</template>
<style lang="less">
p{
    user-select: all;
}
</style>
  