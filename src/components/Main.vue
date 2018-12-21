<template>
  <div class="hello">
    <h1>Cryptonile Wallet</h1>
    <div class="Wrapper" v-if="!auth">
      <v-form class="form" v-model="valid">
        <v-text-field
          v-model="password"
          :append-icon="show ? 'visibility_off' : 'visibility'"
          :rules="passwordRules"
          :type="show ? 'text' : 'password'"
          name="Password"
          label="Password"
          class="input-group--focused"
          style="margin-bottom: 10px;"
          @click:append="show = !show"
          box
        ></v-text-field>
        <v-btn class="button" :disabled="!valid" v-on:click="generate()">Generate Wallet</v-btn>
      </v-form>
      <center class="seperator">
        or
      </center>
      <upload-btn color="grey lighten-3" title="Import Wallet" flat accept=".json" :fileChangedCallback="fileChanged">Import Wallet</upload-btn>
    </div>
    <div class="Wrapper" v-else>
    <h2>Address: {{ address }}</h2>
    <h3>Balance: {{ balance }}</h3>
    <div>
      <v-btn :loading="loading" :disabled="loading" v-on:click="saveKeystore">Download Keystore File</v-btn>
      <v-btn v-on:click="auth=false">Logout</v-btn>
    </div>
    <center style="margin-top: 40px;">
      <v-text-field
        v-model="recipient"
        name="Recipient"
        label="Recipient"
        class="recipient"
        @click:append="show = !show"
        box
      ></v-text-field>
      <v-text-field
        v-model="amount"
        name="Amount"
        label="Amount"
        class="amount"
        @click:append="show = !show"
        box
      ></v-text-field>
      <v-btn class="sendBtn" v-on:click="sendCrypto()">Send (NILE)</v-btn>
    </center>
    </div>
  </div>
</template>

<script>
import kameleonClient from 'kameleon-client'
import FileSaver from 'file-saver'
import UploadButton from 'vuetify-upload-button'

export default {
  components: {
      'upload-btn': UploadButton
  },
  data () {
    return {
      client: new kameleonClient('http://138.197.183.212:26657'),
      auth: false,
      wallet: {},
      address: '',
      balance: '',
      password: '',
      recipient: '',
      amount: '',
      loader: null,
      loading: false,
      valid: false,
      show: false,
      passwordRules: [
        value => !!value || 'Password is required for generating a Wallet',
        value => value.length >= 8 || 'Password must be more than 8 characters'
      ]
    }
  },
  methods: {
    generate() {
      this.wallet = this.client.generateWallet()
      this.address = this.wallet.getAddressString()
      var self = this;
      this.getBalance().then(function(balance){
        self.balance = balance
        self.auth = true
      }).catch(alert)
    },
    saveKeystore() {
      this.loader = 'loading'
      console.log(this.wallet, this.password)
      var keystore = JSON.stringify(this.wallet.toV3(this.password))
      console.log(keystore)

      var blob = new Blob([keystore], {type: "text/plain;charset=utf-8"})
      FileSaver.saveAs(blob, "Keystore.json")
    },
    fileChanged(file) {
      var self = this
      var walletData
      var reader = new FileReader()
      reader.readAsText(file)

      reader.onload = function () {
        var walletPass = prompt('Enter your wallet password')
        self.authenticate(reader.result, walletPass)
      }
    },
    authenticate(walletData, walletPassword) {
      // console.log(walletData, walletPassword)
      this.wallet = this.client.decrypt(JSON.parse(walletData), walletPassword)
      this.address = this.wallet.getAddressString()
      this.pollBalance()
    },
    getBalance () {
      console.log('ooh')
      var self = this;
      return new Promise(function(resolve, reject){
          self.client.query('getBalance', self.address.slice(2)).then(resolve).catch(reject)
      })
    },
    pollBalance () {
      var self = this;
      setInterval(() => {
        this.getBalance().then(function(balance){
          self.balance = balance
          self.auth = true
        }).catch(alert)
      },5000)
    },
    sendCrypto () {
      var self = this
      this.client.send(this.wallet, {}, this.recipient, this.amount)
      .then(this.getBalance().then(function(balance){
          self.balance = balance
          self.auth = true
        }).catch(alert))
    }
  },
  computed: {
    role: {
      get: function(){
        console.log(Client)
      },
      set: function(){
        Vue.set(Client,Client)
      }
    }
  },
  watch: {
    loader () {
      const l = this.loader
      this[l] = !this[l]

      setTimeout(() => (this[l] = false), 3000)

      this.loader = null
    }
  }
}
</script>

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
.Wrapper {
  padding-top: 70px;
  max-width: 1100px;
  margin-right: auto;
  margin-left: auto;
}
.form {
  max-width: 400px;
  margin-left: auto;
  margin-right: auto;
}
.button {
  width: 400px;
  margin-top: 10px;
  margin-left: 0px;
  margin-right: 0px;
  height: 45px;
}
.seperator {
  margin-top: 20px;
  margin-bottom: 10px;
}
#labeluploadFile {
  height: 45px;
  width: 400px;
  color: black;
}
.recipient {
  margin-right: 30px;
  margin-left: 100px;
  width: 550px;
  float: left
  
}
.amount {
  margin-right: 30px;
  max-width: 200px;
  float: left;
}
.sendBtn {
  height: 56px;
  margin-top: 0px;
  box-shadow: none !important;
  margin-left: -50px;
}
</style>
