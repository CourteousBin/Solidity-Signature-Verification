<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <button @click="onConnect">Connect Wallet</button><br />
    <p>Address: {{ Account }}</p>
    <button @click="onSign">Sign with metamask</button>
  </div>
</template>

<script>
import { ethers } from "ethers";

export default {
  name: "HelloWorld",
  props: {},
  data: function () {
    return {
      msg: "VerifySignature",
      Account: "Not connect",
    };
  },
  methods: {
    async onConnect() {
      if (typeof window.ethereum !== "undefined") {
        console.log("MetaMask is installed!");
      }

      let ethereum = window.ethereum;

      try {
        await ethereum.request({
          method: "wallet_switchEthereumChain",
          params: [{ chainId: "0x4" }],
        });
      } catch (switchError) {
        // This error code indicates that the chain has not been added to MetaMask.
        if (switchError.code === 4902) {
          try {
            await ethereum.request({
              method: "wallet_addEthereumChain",
              params: [
                {
                  chainId: "0x4",
                  chainName: "Rinkeby",
                  rpcUrls: ["https://rinkeby.infura.io/v3/"] /* ... */,
                },
              ],
            });
          } catch (addError) {
            // handle "add" error
            console.log(addError);
          }
        }
        // handle other "switch" errors
        console.log(switchError);
      }

      this.getAccount(ethereum);
    },
    // 授权获取账户
    async getAccount(ethereum) {
      const accounts = await ethereum.request({
        method: "eth_requestAccounts",
      });
      const myAccount = accounts[0];
      this.Account = myAccount;
    },
    // 签名
    async onSign() {
      // 1. 构建 provider
      let ethereum = window.ethereum;
      const provider = new ethers.providers.Web3Provider(ethereum);
      let signer = await provider.getSigner();

      // 2. 签名内容进行 solidityKeccak256格式 Hash
      let message = ethers.utils.solidityKeccak256(["string"], ["HelloWorld"]);

      // 3.转成UTF8 bytes
      let arrayifyMessage = ethers.utils.arrayify(message);

      // 4.使用私钥进行消息签名
      let flatSignature = await signer.signMessage(arrayifyMessage);
      console.log(flatSignature);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
