<template>
  <div class="container mx-auto flex flex-col gap-4 max-w-xl">
    <h1 class="text-3xl">Mint Santa gang</h1>
    <div>
      <p>Currently Minting: {{ minting }}</p>
    </div>
    <input
      class="p-3 border border-gray-500 rounded-lg"
      type="number"
      v-model="number"
      placeholder="Number of NFTS to mint"
    />
    <input
      class="p-3 border border-gray-500 rounded-lg"
      type="number"
      v-model="times"
      placeholder="Times to mint at a time"
    />
    <button
      class="bg-blue-500 mx-auto px-5 py-2 text-white rounded-lg font-semibold"
      v-if="!account"
      @click="connect"
    >
      Connect
    </button>
    <button
      class="bg-blue-500 mx-auto px-5 py-2 text-white rounded-lg font-semibold"
      @click="mint"
    >
      Mint
    </button>
  </div>
</template>

<script setup>
import abi from "./abi.json";
import { ethers } from "ethers";
import { onMounted, ref } from "vue";

const account = ref(null);
const number = ref(null);
const times = ref(null);
const minting = ref(0);
// const gas = ref(1);

const connect = async () => {
  try {
    const { ethereum } = window;
    if (!ethereum) {
      console.log("no ethereum");
      return;
    }
    if (!(await checkIfConnected())) {
      await requestAccess();
    }
  } catch (error) {
    console.log(error);
  }
};

onMounted(() => {
  connect();
});

const checkIfConnected = async () => {
  const { ethereum } = window;
  const accounts = await ethereum.request({ method: "eth_accounts" });
  if (accounts.length !== 0) {
    account.value = accounts[0];
    return 1;
  } else {
    return 0;
  }
};

const requestAccess = async () => {
  const { ethereum } = window;
  const accounts = await ethereum.request({
    method: "eth_requestAccounts",
  });
  account.value = accounts[0];
};

const getContract = async () => {
  try {
    const { ethereum } = window;
    const provider = new ethers.providers.Web3Provider(ethereum);
    const signer = provider.getSigner();
    const connectedContract = new ethers.Contract(
      "0x20d502029b1dF8cf760404786660Ee89dB6cc56A",
      abi,
      signer
    );
    return connectedContract;
  } catch (error) {
    console.log(error);
    console.log("connected contract not found");
    return null;
  }
};

const mint = async () => {
  try {
    const { ethereum } = window;
    if (ethereum) {
      const connectedContract = await getContract();
      let nftTxn = await connectedContract.mint(number.value);
      await nftTxn.wait();
      minting.value = number.value;
      for (let i = 0; i < times.value; i++) {
        setTimeout(async () => {
          const connectedContract = await getContract();
          let nftTxn = await connectedContract.mint(number.value);
          await nftTxn.wait();
          minting.value = i * number.value;
        }, 60000);
      }
    }
  } catch (error) {
    console.log(error);
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
