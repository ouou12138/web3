<template>
  <div
    class="w-full min-h-screen bg-blue-900 flex flex-col justify-center items-center"
  >
    <h1 class="text-8xl font-bold text-white text-shadow text-center">
      Hello, Web3
    </h1>
    <h2 
      class="text-6xl text-center mt-24 text-yellow-300 font-bold"
      v-show='account'
    >
      🖐{{ count }}
    </h2>
    <h3
      class="text-3xl text-center mt-8 text-white"
      v-if='account'
    >
      Logined in as <strong>{{account.substring(0,4)}}...{{account.substring(account.length-4)}}</strong>
    </h3>
    <button
      class="rounded-full py-6 px-12 text-3xl mt-16 text-white bg-purple-700 hover:scale-105 hover:bg-purple-600 transition"
      @click="addCount"
      v-show='account'
    >
      Say Hi
    </button>
    <button
      class='rounded-full py-6 px-12 text-3xl mt-16 text-white bg-purple-700 hover:scale-105 hover:bg-purple-600 transition'
      v-show='!account'
      @click='connectWallet'
    >
      Connect To MetaMask
    </button>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref, onBeforeMount } from "vue";
import { ethers } from 'ethers';
import CounterABI from '@/utils/Counter.json'

const ContractAddress = '0x5FbDB2315678afecb367f032d93F642f64180aa3'
const ContractABI = CounterABI.abi

let count = ref<number>(0);
const account = ref<string>('')

const connectContract = async (ContractAddress,ContractABI)=>{
  const { ethereum } = window
  if(!ethereum){
    throw 'please install ethereum'
  }
  const provider = new ethers.providers.Web3Provider(ethereum)
  const signer = provider.getSigner()
  const Contract = new ethers.Contract(ContractAddress,ContractABI,signer)
  return Contract
}

const addCount = async () => {
  try {
    const CounterContract = await connectContract(ContractAddress,ContractABI)
    let tx = await CounterContract.add()
    await tx.wait()
    console.log(`transaction ok`);
    
    await getCounts()
  } catch (err) {
    console.log(err);
  }
}

const getCounts = async () => {
  try {
    const CounterContract = await connectContract(ContractAddress,ContractABI)
    let counts = await CounterContract.getCounts()
    count.value = counts.toNumber()
    console.log(count.value);
    
  } catch (error) {
    
  }
}

const checkedConnectWallet = async()=>{
  try {
    if(!window.ethereum){
      alert('please install metamask')
      return
    }
    console.log(`metamask is available`);
    
    let addrs = await ethereum.request({method:'eth_accounts'})
    if(addrs.length !== 0){
      account.value = addrs[0]
      console.log(`found account with address`,account.value);
    }else{
      console.log(`no authorized account found`);
    }
  } catch (err) {
    console.log(err);
  }
}

const connectWallet = async()=>{
  try {
    if(!window.ethereum){
      alert('please install metamask')
      return
    }
    let addrs = await ethereum.request({method:'eth_requestAccounts'})
    if(addrs.length !== 0){
      account.value = addrs[0]
    }
  } catch (err) {
    console.log(err);
  }
}

onBeforeMount(() => {
  checkedConnectWallet()
  getCounts()
})
</script>

<style lang="scss" scoped></style>
