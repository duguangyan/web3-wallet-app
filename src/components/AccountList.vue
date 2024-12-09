<script setup>
import { ref, defineProps, computed } from 'vue'
import Web3 from "web3";
import ethwallet, { hdkey}  from 'ethereumjs-wallet'
const props = defineProps(['walletInfo'])
const web3 = new Web3(Web3.givenProvider || 'https://linea-sepolia.infura.io/v3/638355f30f124b0daf4131933304abd0');
console.log(props.walletInfo)

let show = ref(false)

let password = ref("")
let keystore = ref('')
let fromaddress = ref('')

const walletInfoFilter = computed(()=>{
    props.walletInfo.filter(async (item) => {
        const originalAddress = item.address
        item.splitAddress = item.address.slice(0, 6) + "..." + item.address.slice(-6)
        // 根据地址获取余额
        let result = await web3.eth.getBalance(originalAddress)
        item.balance  = web3.utils.fromWei(result, 'ether')
    })
    return props.walletInfo
})

// 转账
const send = async (keystore,pass) => {

    let walletobj;

    try {
        walletobj = await ethwallet.fromV3(keystore, pass)
    } catch (error) {
        alert("密码错误")
        return
    }
    let key = walletobj.getPrivateKey().toString('hex')

    var privateKey = new Buffer(key, 'hex')

    // 1.构建转账参数
    // 获取转账次数
    const nonce = await web3.eth.getTransactionCount(fromaddress.value)
    console.log('nonce',nonce)
    // 获取预计转账gas费用
    const gasPrice = await web3.eth.getGasPrice()
    console.log('gasPrice',gasPrice)

    // 转账金额： 以wei作为单位
    const value = web3.utils.toWei('0.0001', 'ether')
    console.log('value',value)
    // 构建转账对象
    const rawTx = {
        from: fromaddress.value,
        to:"0xCA685B7805CCF8d8335DC26adA43D24eC85a4Fbf",
        nonce,
        gasPrice,
        value,
        data: "0x00",
    }

    console.log("rawTx", rawTx)

    // 2.生成serializedTx
    // 转为私钥
    const pKey = Buffer.from(privateKey.value, 'hex')

    console.log("pKey", pKey)
    // gas 估算
    const gas = await web3.eth.estimateGas(rawTx)
    console.log("gas", gas)
    rawTx.gas = gas
    console.log('rawTx',rawTx)
    // 签名
    const tx = new Tx(rawTx)
    tx.sign(pKey)
    const serializedTx = '0x'+ tx.serialize().toString('hex')
    console.log("serializedTx", serializedTx)

    // 3.发送交易
    const trans = web3.eth.sendSignedTransaction(serializedTx)
    console.log("trans", trans)
    trans.on('transactionHash',(txid) => {
        console.log("交易txid", txid)
        console.log(`https://sepolia.lineascan.build/tx/${txid}`)
    })

    trans.on("receipt",(res=>{
        console.log("第一个节点确认", res)
    }))

    trans.on("confirmation",((confirmationNumber, receipt)=>{
        console.log("第N个节点确认", receipt,confirmationNumber)
    }))

    trans.on("error",(err=>{
        console.log("错误", err)
    }))
}


const getPassword = (keyStore,address) => {
    console.log("keyStore",keyStore)
    keystore.value = keyStore
    fromaddress.value = address
    show.value = true
}

const confirmPassword = async() => {
    
    console.log("keystore",keystore.value)
    console.log("password",password.value)
    password.value = ''
    send(keystore.value,password.value)
}

</script>
<template>
   <div>
       <van-cell :title="wi.splitAddress" icon="user-o"  v-for="wi in walletInfoFilter" :key="wi.address">
            <template #right-icon>
                <van-button size="mini" @click="getPassword(wi.keyStore,wi.address)">{{wi.balance}}</van-button>
            </template>
       </van-cell>

       <van-dialog v-model:show="show" title="请输入密码" show-cancel-button @confirm="confirmPassword">
            <van-cell-group>
                <van-field label="密码:" type="password" v-model="password" placeholder="请输入" />
            </van-cell-group>
        </van-dialog>

   </div>
</template>
<style lang="less">
p{
    user-select: all;
}
</style>
  