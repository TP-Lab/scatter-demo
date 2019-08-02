<template>
  <div class="hello">
    <div>
      <button @click="login">Login</button>
      {{currentAccount}}
      <button @click="logout">Logout</button>
      <hr>
      <p>
        <button @click="send">Demo 1: Transfer EOS to TokenPocket</button>
      </p>
      <p>
        <button @click="vote">Demo 2: Vote for us (itokenpocket)</button>
      </p>
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

let eosClient = null;
const requiredFields = { accounts: [network] };

export default {
  name: "HelloWorld",
  data() {
    return {
      currentAccount: null,
      currentPermission: null,
      scatter: null,
      pubKey: null
    };
  },
  created() {
    ScatterJS.scatter.connect("scatter-demo").then(connected => {
      // User does not have Scatter Desktop, Mobile or Classic installed.
      if (!connected) return false;

      this.scatter = ScatterJS.scatter;
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
          this.signatureForDemo5 = data;
        })
        .catch(err => {
          alert("error:" + err);
        });
    },
    authenticate() {
      this.scatter.authenticate().then(data => {
        alert(data);
        this.signatureForDemo6 = data;
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
