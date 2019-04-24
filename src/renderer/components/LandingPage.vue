<template>
  <div>
    <el-button v-show="!loggedIn" @click="cardScanned = true">Scan card</el-button>
    <el-button v-show="!loggedIn" @click="loggedIn = true">Log in</el-button>

    <div v-if="!loggedIn">
      <img src="../assets/SBER_logo.png" alt="logo" class="welcome_logo">

      <div v-if="cardScanned" class="start-screen-text">
        Enter PIN
      </div>
      <div v-else class="start-screen-text">
        Scan card
      </div>

      <div style="margin-top: 80px; height: 60px;">
        <transition name="el-fade-in-linear">
          <div v-show="cardScanned" class="input-row">
            <el-row>
              <el-col class="pin-pass-box" :offset="6" :span="2">
                <h1 class="pin-dot" v-if="notEmpty(0)" >•</h1>

              </el-col>
              <el-col class="pin-pass-box" :offset="1" :span="2">
                <h1 class="pin-dot" v-if="notEmpty(1)">•</h1>

              </el-col>
              <el-col class="pin-pass-box" :offset="1" :span="2">
                <h1 class="pin-dot" v-if="notEmpty(2)">•</h1>

              </el-col>
              <el-col class="pin-pass-box" :offset="1" :span="2">
                <h1 class="pin-dot" v-if="notEmpty(3)">•</h1>
              </el-col>
            </el-row>
          </div>
        </transition>

        <el-button v-if="cardScanned" @click="cardScanned = false" class="cancel-button">
          temporary cancel button
        </el-button>

        <div v-show="cardScanned" class="pin-screen-confirm">[A] Confirm</div>
        <div v-show="cardScanned" class="pin-screen-clear">[C] Clear</div>
        <div v-show="cardScanned" class="pin-screen-cancel">[D] Cancel</div>
      </div>
    </div>

    <div v-if="loggedIn">
      <div class="side-menu-bg">
        <img src="../assets/SBER_icon.png" alt="logo" class="logo-icon">

        <div class="button-type-2">[A] Quick Cash</div>
        <div class="button-type-2">[B] Check Balance</div>
        <div class="button-type-2">[C] Withdraw</div>
        <div class="button-type-1">[D] Log out</div>

        <el-button @click="loggedIn = false; cardScanned = false" class="logout-button">
          Log out
        </el-button>
      </div>
      <div class="main-bg">
        <div>
          <transition name="el-fade-in-linear">
            <div v-show="menuCash" class="menuCash">
              <div style="position: absolute;
              background-color: black; width: 1000px; height: 20px;"></div>
            </div>
          </transition>
        </div>
      </div>


      <img src="../assets/SBER_text.png" alt="logo" class="logo_loggedIn">
    </div>
  </div>
</template>

<script>
  import axios from 'axios';
  const SerialPort = require('serialport');
  const Readline = require('@serialport/parser-readline')

  const port = new SerialPort('COM10', { baudRate: 9600 });
  const parser = port.pipe(new Readline({ delimiter: '\r\n' }));

  export default {
    name: 'landing-page',

    data() {
      return {
        cardScanned: false,
        inputCount: 0,
        pinAttempt: 0,
        loggedIn: false,
        0: '',
        1: '',
        2: '',
        3: '',
        menuCash: false,
        menuBalance: false,
        menuWithdraw: false,
      }
    },

    methods: {
      notEmpty(number) {
        return this[number] !== '';
      },

      removeInput() {
        if (this.inputCount === 0) {
          return;
        }
        this[this.inputCount] = '';
        this.inputCount -= 1;
      },

      resetPinAttempt() {
        this.pinAttempt = 0;
      },

      login() {
        this.loggedIn = true;
      },

      testlogin() {
        const accountNumber = '';
      },

      login2() {
        const tempAccountNumber = 'RU02SBERaccount3';
        const loginRoute = 'http://localhost:8080/login'

        axios.post(loginRoute, {
          iban: tempAccountNumber,
          pin: '3333',
        }, { headers: {  } })
        .then((response) => {
          console.log(response)
        })
        .catch((response) => {
          console.log(response)
        })
      },

      quickCash() {
        const tempAccountNumber = 'RU02SBERaccount3';
        const loginRoute = 'http://localhost:8080/withdraw'

        axios.post(loginRoute, {
          iban: tempAccountNumber,
          amount: 10,
        }, { headers: {  } })
        .then((response) => {
          console.log(response)
        })
        .catch((response) => {
          console.log(response)
        })
      },

      getBalance() {
        const tempAccountNumber = 'RU02SBERaccount3';
        const loginRoute = 'http://localhost:8080/getbalance'

        axios.post(loginRoute, {
          iban: tempAccountNumber,
          pin: '3333'
        }, { headers: {  } })
        .then((response) => {
          console.log(response)
        })
        .catch((response) => {
          console.log(response)
        })
      },

      withdraw() {
        const tempAccountNumber = 'RU02SBERaccount3';
        const loginRoute = 'http://localhost:8080/withdraw'

        axios.post(loginRoute, {
          iban: tempAccountNumber,
          amount: 2,
        }, { headers: {  } })
        .then((response) => {
          console.log(response)
        })
        .catch((response) => {
          console.log(response)
        })
      }
    },

    mounted() {
      parser.on("data", (data) => {
        console.log(data);
        if(data === ' 60 6C 0C A3') {
          console.log(data);
          this.cardScanned = true;
        }

        if(data.length === 1 && !this.loggedIn) {
          console.log(data);
          
          if(!isNaN(data)) {
            if(this.inputCount < 4) {
              this[this.inputCount] = data;
              this.inputCount++;
            }
          }
          if(data === 'A') {
            this.login();
          }
          if(data === 'B') {
            console.log(data);
          }
          if(data === 'C') {
            this.inputCount--;
            this[this.inputCount] = '';
          }
          if(data === 'D') {
            if(this.loggedIn == false && this.cardScanned == true) {
              this.cardScanned = false;
            }
            if(this.loggedIn == true) {
              this.loggedIn = false;
              this.cardScanned = false;
            }
          }
        }

        if(data.length === 1 && this.loggedIn) {
          if(data === '#') {
            this.login2();
          }
          if(data === 'A') {
            this.menuCash = true;
            this.menuBalance = false;
            this.menuWithdraw = false;
            this.quickCash();
          }
          if(data === 'B') {
            this.menuCash = false;
            this.menuBalance = true;
            this.menuWithdraw = false;
            this.getBalance();
          }
          if(data === 'C') {
            this.menuCash = false;
            this.menuBalance = false;
            this.menuWithdraw = true;
            this.withdraw();
          }
          if(data === 'D') {
            if(this.loggedIn == false && this.cardScanned == true) {
              this.cardScanned = false;

              this.menuCash = false;
              this.menuBalance = false;
              this.menuWithdraw = false;
            }
            if(this.loggedIn == true) {
              this.loggedIn = false;
              this.cardScanned = false;

              this.menuCash = false;
              this.menuBalance = false;
              this.menuWithdraw = false;
            }
          }
        }
      });
    }
  }
</script>

<style>
  .welcome_logo {
    display: block;
    margin-left: auto;
    margin-right: auto;
    width: 20%;
  }

  .start-screen-text {
    font-size: 40px;
    margin-top: 7%;
    text-align: center;
  }

  .pin-screen-confirm {
    font-size: 24px;
    position: absolute;
    right: 10px;
    bottom: 90px;
  }

  .pin-screen-clear {
    font-size: 24px;
    position: absolute;
    right: 36px;
    bottom: 50px;
  }

  .pin-screen-cancel {
    font-size: 24px;
    position: absolute;
    right: 18px;
    bottom: 10px;
  }

  .pin-dot{
    text-align: center;
    padding-top: 0px;
    margin-top: 44px;
    font-size: 70px;
    margin-bottom: 0;
  }

  /* .input-row{
    margin-left: auto;
    margin-right: auto;
  } */

  .pin-pass-box{
    height: 108px;
    -webkit-box-shadow: 0px 0px 5px 0px rgba(0,0,0,0.75);
    -moz-box-shadow: 0px 0px 5px 0px rgba(0,0,0,0.75);
    box-shadow: 0px 0px 5px 0px rgba(0,0,0,0.75);
    border-radius: 5px;
  }

  .border{
    border: 1px solid #000;
  }

  .logo_loggedIn {
    position: absolute;
    right: 20px;
    bottom: 20px;
    width: 10%;
  }

  .side-menu-bg {
    float: left;
    background: #2f2f2f;
    width: 280px;
    height: 100vh;
  }

  .logo-icon {
    width: 50%;
    margin-top: 30px;
    margin-left: 65px;
    margin-bottom: 18px;
  }

  .button-type-1 {
    margin: 30px;
    padding: 20px;
    background: lightgrey;
    box-shadow: 4px 5px 9px  black;

    font-size: 20px;
    color: black;
    text-shadow: 1px 1px grey;
  }

  .button-type-2 {
    margin: 30px;
    padding: 20px;
    background: #1f9d59;
    box-shadow: 4px 5px 9px  black;

    font-size: 20px;
    color: white;
    text-shadow: 2px 1px #000;
  }

  .main_bg {
    width: 1000;
    height: 100vh;
    float: right;
  }

  .menuCash {

  }

  .menuBalance {

  }

  .menuWithdraw {

  }

  .cancel-button {
    position: absolute;
    right: 200px;
    bottom: 10px;
  }

  .logout-button {
    position: absolute;
    left: 10px;
    bottom: 10px;
  }
  
  .top-pad{
    margin-top: 100px;
  }
</style>
