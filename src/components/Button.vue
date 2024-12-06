
<script setup>
import { ref } from 'vue'
let show = ref(false)
let password = ref("")
// 创建钱包
const createWallet = () =>{

    console.log(100)
    show.value = true
}

const confirm = () => {
    console.log(password.value)
}
</script>
<template>
   <div>
        <van-space>
            <van-button type="primary" @click="createWallet">创建钱包</van-button>
            <van-button type="primary">导入钱包</van-button>
        </van-space>

        <van-dialog v-model:show="show" title="请输入密码" show-cancel-button @confirm="confirm">
            <van-cell-group>
                <van-field label="密码:" type="password" v-model="password" placeholder="请输入" />
            </van-cell-group>
        </van-dialog>
   </div>
</template>
<style lang="less">

</style>
  