<template>
  <div class="hello">
    <div v-if="scatter">
      <p v-if="!currentAccount">
        <button @click="login">Login</button>
      </p>
      
      <p v-else>
        {{currentAccount}}
        <button @click="logout">Logout</button>
        <br>
        <br>
          <button @click="send">send EOS to TokenPocket</button>
      </p>
      
    </div>
    
  </div>
</template>

<script>

import ScatterJS from "scatter-js/dist/scatter.esm";
import Eos from 'eosjs';

const network = {
    blockchain: "eos",
    host: "mainnet.eoscanada.com",
    port: 443,
    protocol: "https",
    chainId: "aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906"
};

export default {
  name: 'HelloWorld',
  data () {
    return {
      currentAccount: '',
      scatter: null,
      eosClient: null
    }
  },
  created() {
    ScatterJS.scatter.connect("scatter-demo").then(connected => {
        // User does not have Scatter Desktop, Mobile or Classic installed.
        if(!connected) return false;

        this.scatter = ScatterJS.scatter;

        const requiredFields = { accounts:[network] };

        window.scatter = null;
    });
  },
  methods: {
    login() {
      const requiredFields = { accounts:[network] };

      this.scatter.getIdentity(requiredFields).then(() => {
        const account = this.scatter.identity.accounts.find(x => x.blockchain === 'eos');
        this.currentAccount = account.name;
        const eosOptions = { expireInSeconds:60 };

        this.eosClient = this.scatter.eos(network, Eos, eosOptions);
      }).catch(error => {
          alert(JSON.stringify(error));
      });
    },
    logout() {
      this.scatter.forgetIdentity();
      this.currentAccount = null;
    },
    send() {
      this.eosClient.transfer(this.currentAccount, 'itokenpocket', '0.0001 EOS', 'from scatter-demo').then(data => {
        alert(JSON.stringify(data));
      }).catch(error => {
        alert(JSON.stringify(error));
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
