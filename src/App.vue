<template>
  <div id="main">
    <v-app>
      <v-container class="app-content" grid-list-md text-xs-center>
        <v-flex x12>
          <h1>Config file generator for Bitblocks.</h1>
          <p>This is the easy way to configure your Bitblocks plugin. Place the generated .json file in your server's root folder.</p>
          <v-form ref="form">
            <v-text-field
              label="RTWire User"
              v-model="user"
              :rules="fieldRule"
              required
            ></v-text-field>
            <v-text-field
              label="RTWire Pass"
              v-model="pass"
              :type="hidePass ? 'password' : 'text'"
              :append-icon-cb="() => (hidePass = !hidePass)"
              :append-icon="hidePass ? 'visibility_off' : 'visibility'"
              :rules="fieldRule"
              required
            ></v-text-field>
            <v-btn
              @click="checkCredentials"
              :disabled="isCheckingCreds"
            >
              {{checkingMsg}}
            </v-btn>
            <div v-if="showError">
              <v-alert color="error" icon="warning" value="false">
                Wrong credentials for the mainnet.
              </v-alert>
            </div>
            <div v-else-if="showSuccess">
              <v-alert color="success" icon="warning" value="false">
                Your RTWire credentials work! Transactions and Bitcoin wallet generation will work!
              </v-alert>
            </div>
            <v-checkbox v-bind:label="'Show Scoreboard with bitcoin balance to players'" v-model="showSB"></v-checkbox>
            <v-checkbox v-bind:label="'Enable Villager Market'" v-model="enableVM"></v-checkbox>
            <v-checkbox v-bind:label="'Enable Land Market'" v-model="enableLM"></v-checkbox>
            <div v-if="enableVM">
              <v-text-field
                label="Title for the Villager Market"
                v-model="vmTitle"
                required
                :rules="fieldRule"
              ></v-text-field>
              <v-text-field
                label="Account ID for villager market sales (number)"
                v-model="villagerSalesID" 
                :type="'number'"
                required
                :rules="fieldRule"

              ></v-text-field>
            </div>
            <div v-if="enableLM">
              <v-text-field
                label="Price of 1 chunk of land in SATOSHIS (number)"
                v-model="chunkPrice" 
                :type="'number'"
                required
                :rules="fieldRule"
                
              ></v-text-field>
              <v-text-field
                label="Account ID for land sales (number)"
                v-model="landSalesID" 
                :type="'number'"
                required
                :rules="fieldRule"
                
              ></v-text-field>
            </div>
              <v-btn
                @click="downloadFile"
                :disabled= "isDownloadDisabled"
              >
                Download json file
              </v-btn>
          </v-form>
            <p>You need to get your credentials checked and have all fields valid to enable the download button.</p>
            <p>This site checks your credentials using the <a href="https://rtwire.com/docs/">RTWire API</a> and it doesn't store any information afterwards. </p>
            <p>When you click download your browser generates your file automatically using your provided input.</p>
            <p>You can see the <a href="https://github.com/jordigoyanes/config-generator">source code</a></p>

          <v-footer class="pa-3">
            <v-spacer></v-spacer>
            <div><p>© {{currentYear}} Bitblocks. Made by <a href="https://github.com/jordigoyanes">Jordi Goyanes</a></p></div>
          </v-footer>
        </v-flex>
      </v-container>
    </v-app>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  data: () => ({
    user: this.user,
    pass: this.pass,
    showSB: true,
    enableLM: true,
    enableVM: true,
    vmTitle: "Market | Priced in bits",
    chunkPrice: 2000,
    landSalesID:"",
    villagerSalesID:"",
    hidePass: true,
    isCheckingCreds: false,
    showError: false,
    showSuccess: false,
    fieldRule: [
        (v) => !!v || 'This field is required!',
      ],
    checkingMsg: "Check credentials (mainnet)"
  }),
  methods: {
    checkCredentials () {
        this.isCheckingCreds = true;
        this.checkingMsg = "Loading..."
        var token = window.btoa(this.user + ':' + this.pass);
       axios.get('https://galvanize-cors-proxy.herokuapp.com/https://api.rtwire.com/v1/mainnet/accounts/',{
         headers:{
           "Authorization": `Basic ` + token
         }
        }).then(response => {
          console.log(response)
         console.log(response.data.success)
          if(response.status == 200){
            this.showSuccess = true;
            this.showError = false;
            this.isCheckingCreds = false;
            this.checkingMsg = "Check credentials (mainnet)"
          }
        }).catch((error) => {
          var rCode = error.response.status;
          console.log(rCode)
          if(rCode != 200) {
            this.showSuccess = false;
            this.showError = true;
            this.isCheckingCreds = false;
            this.checkingMsg = "Check credentials (mainnet)"
          }
      })
      
    },
    downloadFile (){
      var configData = {
        "user": this.user,
        "pass": this.pass,
        "enableLandMarket":this.enableLM,
        "enableVillagerMarket":this.enableVM,
        "showScoreboard": this.showSB,
        "villagerMarketTitle": this.vmTitle,
        "pricePerChunkSAT": parseInt(this.chunkPrice),
        "marketSalesAccountID": parseInt(this.villagerSalesID),
        "landSalesAccountID": parseInt(this.landSalesID)
      }
      var finalData = "text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(configData));
      var a = document.createElement('a');
      a.href = 'data:' + finalData;
      a.download = 'bitblocks-config.json';
      a.click();
    }
  },
  computed:{
    isDownloadDisabled(){
      if(this.showSuccess==true && this.$refs.form.validate()){
        return false;
      }else{
        return true;
      }
    },
    currentYear(){
      return new Date().getFullYear();
    }
  },
  watch: {
    user: function (v) {
      this.showSuccess = false;
    },
    pass: function(v){
      this.showSuccess = false;
    }
  }
}
</script>
<style>
a{
  text-decoration: none;
}
#main{
  margin: 0 auto;
  width: 900px;

}
body{
  background-image: url(/images/mcwallpaper.png);
  background-size: 100%;
  background-attachment: fixed;
  opacity: 0.85;
}
.app-content{
  padding: 30px !important;

}
.pa-3{
  display: block;
}
</style>
