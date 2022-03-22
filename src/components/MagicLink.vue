<template>
  <div class="hello">
    <div class="container">
      <h1>Vue / MagicLink</h1>
      <hr />
      <div v-if="!checking">
        <div v-if="!isLoggedIn">
          <p>Please sign up or login</p>
          <input
            type="email"
            name="email"
            v-model="email"
            placeholder="Enter your email"
          />
          <button v-on:click="loginWithEmail">Send</button><br />
        </div>
        <div v-if="isLoggedIn">
          <b>User address is:</b><br />
          {{ userAddress }}<br /><br />
          <b>Balance is:</b><br />
          {{ userBalance }} ETH<br /><br />
          <b>Network is:</b><br />
          {{ network.name }}
          <br /><br />
          <div v-if="isSending">Sending transaction, please wait..</div>
          <button
            v-if="!tx && !isSending"
            v-bind:disabled="userBalance <= 0.1"
            v-on:click="mint"
          >
            MINT</button
          ><br /><br />
          <div v-if="tx">
            <b>Tx submitted at:</b><br />
            <a :href="'https://rinkeby.etherscan.io/tx/' + tx.hash">{{ tx.hash }}</a><br /><br />
            <div v-if="receipt"><b>Receipt is:</b><br />{{ receipt }}</div>
          </div>
          <hr />
          <br /><br />
          <button v-on:click="logout">Logout</button>
        </div>
      </div>
      <div v-if="checking">Checking Magic status..</div>
    </div>
  </div>
</template>

<script>
import { Magic } from "magic-sdk";
import { ethers } from "ethers";

const magic = new Magic("pk_live_2B9B8268A7D5D33F", {
  network: "rinkeby",
});

export default {
  name: "MagicLink",
  data() {
    return {
      email: "",
      userMetadata: {},
      userAddress: "",
      userBalance: 0,
      checking: true,
      isLoggedIn: false,
      isSending: false,
      network: {},
      contract_address: "0x9b3ae7419809ce1a690c8c5fc64d51949365dd75",
      ABI: [
        {
          inputs: [],
          name: "buyNFT",
          outputs: [],
          stateMutability: "payable",
          type: "function",
          payable: true,
        },
      ],
      tx: "",
      receipt: "",
    };
  },
  mounted() {
    const app = this;
    app.render();
  },
  methods: {
    async loginWithEmail() {
      const app = this;
      if (app.email.length > 0) {
        await magic.auth.loginWithMagicLink({ email: app.email });
        app.render();
      }
    },
    async render() {
      const app = this;
      const isLoggedIn = await magic.user.isLoggedIn();
      console.log("User is logged?", isLoggedIn);
      if (isLoggedIn) {
        app.isLoggedIn = true;
        const provider = new ethers.providers.Web3Provider(magic.rpcProvider);
        console.log("User is logged in..");
        app.userMetadata = await magic.user.getMetadata();
        const signer = provider.getSigner();
        app.network = await provider.getNetwork();
        console.log("Network is: ", app.network);
        app.userAddress = await signer.getAddress();
        console.log("Address is " + app.userAddress);
        app.userBalance = ethers.utils.formatEther(
          await provider.getBalance(app.userAddress)
        );
        console.log("User balance is " + app.userBalance);
        app.checking = false;
      } else {
        app.checking = false;
      }
    },
    async mint() {
      const app = this;
      const isLoggedIn = await magic.user.isLoggedIn();
      console.log("User is logged?", isLoggedIn);
      if (isLoggedIn) {
        app.isSending = true;
        const provider = new ethers.providers.Web3Provider(magic.rpcProvider);
        console.log("User is logged in..");
        try {
          app.userMetadata = await magic.user.getMetadata();
          const signer = provider.getSigner();
          let contract = new ethers.Contract(
            app.contract_address,
            app.ABI,
            signer
          );
          app.tx = await contract.buyNFT({
            value: ethers.utils.parseEther("0.1"),
          });
          console.log("Submitted:", app.tx);
          app.receipt = await app.tx.wait();
          console.log("Completed:", app.receipt);
          app.isSending = false;
        } catch (e) {
          console.log(e);
          app.isSending = false;
        }
      }
    },
    async logout() {
      const app = this;
      await magic.user.logout();
      app.isLoggedIn = false;
    },
  },
};
</script>