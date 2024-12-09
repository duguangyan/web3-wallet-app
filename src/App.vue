<template>
  <Button />
  <AccountList v-if="walletInfo" :walletInfo="walletInfo"/>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import Button from './components/Button.vue';
import AccountList from './components/AccountList.vue';
import store2 from 'store2';
const walletInfo = ref(null);
onMounted(() => {
  console.log(store2.get('walletInfo'));
  walletInfo.value = store2.get('walletInfo') || [];
});
</script>

<style lang="less">
body{
  padding: 10px;
}
</style>
