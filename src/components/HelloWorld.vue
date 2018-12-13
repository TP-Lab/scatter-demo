<template>
  <div class="hello">
    <p>
      <button @click="queryTPT">Query without Scatter</button>
    </p>
    <hr>
    <div v-if="scatter">
      <p v-if="!currentAccount">
        <button @click="login">Login</button>
      </p>

      <div v-else>
        {{currentAccount}}
        <button @click="logout">Logout</button>
        <hr>
        <p>
          <button @click="send">Demo 1: Transfer EOS to TokenPocket</button>
        </p>
        <p>
          <button @click="vote">Demo 2: Vote for us (itokenpocket)</button>
        </p>
        <p>
          <button @click="airgrabContract">Demo 3: Airgrab POOR</button>
        </p>
        <p>
          <button @click="getMyBalance">Demo 4: Get My EOS Balance</button>
        </p>

        <p>
          <input type="text" placeholder="data to be sign" v-model="dataTobeSign">
          <label for="is-hash">
            <input id="is-hash" type="checkbox" v-model="isHash">
            isHash
          </label>
          <input type="text" placeholder="what for" v-model="whatfor">
        </p>
        <p>
          <button @click="getArbitrarySignature">Demo 5: getArbitrarySignature</button>
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import Eos from "eosjs";
import ScatterJS from "scatterjs-core";
import ScatterEOS from "scatterjs-plugin-eosjs";

ScatterJS.plugins(new ScatterEOS());

const network = {
  blockchain: "eos",
  host: "mainnet.eoscanada.com",
  port: 443,
  protocol: "https",
  chainId: "aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906"
};

// Put eosClient in data will case a weird problem in scatter-desktop.
let eosClient = null;

const requiredFields = { accounts: [network] };

export default {
  name: "HelloWorld",
  data() {
    return {
      currentAccount: null,
      currentPermission: null,
      scatter: null,
      readOnlyEos: null,
      pubKey: null,
      dataTobeSign: "",
      whatfor: "tp-demo",
      isHash: false
    };
  },
  created() {
    let chainId = network.chainId;
    let httpEndpoint =
      network.protocol + "://" + network.host + ":" + network.port;

    this.readOnlyEos = Eos({
      chainId,
      httpEndpoint
    });

    ScatterJS.scatter.connect("scatter-demo").then(connected => {
      // User does not have Scatter Desktop, Mobile or Classic installed.
      if (!connected) return false;

      this.scatter = ScatterJS.scatter;

      window.scatter = null;
      window.ScatterJS = null;
    });
  },
  methods: {
    queryTPT() {
      this.readOnlyEos
        .getTableRows(true, "eosiotptoken", "eosiotptoken", "global")
        .then(data => {
          alert("TPT Total Staked: " + data.rows[0].total_delegate_quantity);
        });
    },
    login() {
      this.scatter
        .getIdentity(requiredFields)
        .then(() => {
          const account = this.scatter.identity.accounts.find(
            x => x.blockchain === "eos"
          );

          let scatter = this.scatter;

          this.pubKey = this.scatter.identity.publicKey;

          this.currentAccount = account.name;
          this.currentPermission = account.authority;
          eosClient = this.scatter.eos(network, Eos);
        })
        .catch(err => {
          alert(JSON.stringify(err));
        });
    },
    logout() {
      this.scatter.forgetIdentity();
      this.currentAccount = null;
      this.currentPermission = null;
    },
    send() {
      eosClient
        .transfer(
          this.currentAccount,
          "itokenpocket",
          "0.0001 EOS",
          "from scatter-demo"
        )
        .then(data => {
          alert("succeed");
          console.log(data);
        })
        .catch(err => {
          alert(JSON.stringify(err));
        });
    },
    vote() {
      eosClient
        .transaction({
          actions: [
            {
              account: "eosio",
              name: "voteproducer",
              authorization: [
                {
                  actor: this.currentAccount,
                  permission: this.currentPermission
                }
              ],
              data: {
                voter: this.currentAccount,
                proxy: "",
                producers: ["itokenpocket"]
              }
            }
          ]
        })
        .then(data => {
          alert("succeed");
          console.log(data);
        })
        .catch(err => {
          alert(JSON.stringify(err));
        });
    },
    airgrabContract() {
      eosClient.contract("poormantoken").then(contract => {
        contract
          .signup(this.currentAccount, "0.0000 POOR")
          .then(data => {
            alert("succeed");
            console.log(data);
          })
          .catch(err => {
            alert(JSON.stringify(err));
          });
      });
    },
    getMyBalance() {
      eosClient
        .getCurrencyBalance("eosio.token", this.currentAccount, "EOS")
        .then(data => {
          alert(this.currentAccount + ": " + data[0]);
        });
    },
    getArbitrarySignature() {
      this.scatter
        .getArbitrarySignature(
          this.pubKey,
          this.dataTobeSign || "tokenpocket",
          this.whatfor,
          this.isHash
        )
        .then(data => {
          alert(data);
        })
        .catch(err => {
          alert("error:" + err);
        });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
button {
  padding: 6px 12px;
}

hr {
  width: 90%;
  border: none;
  border-top: 1px solid #f1f1f1;
}
</style>
