<template>
  <div>
    <div v-if="!CARD_LOGGED_IN">
      <img src="../assets/SBER_logo.png" alt="logo" class="welcome_logo">
      
      <div v-if="!CARD_SCANNED && !CARD_WARN" class="start-screen-text">
        Scan card
      </div>
      <div v-else-if="CARD_SCANNED && !CARD_WARN" class="start-screen-text">
        Enter PIN
      </div>
      <div v-else-if="CARD_SCANNED && CARD_WARN" class="start-screen-text">
        Invalid PIN - Re-enter
      </div>
      <div v-else class="start-screen-text">
        Scan card
      </div>

      <div style="margin-top: 80px; height: 60px;">
        <transition name="el-fade-in-linear">
          <div v-show="CARD_SCANNED" class="input-row">
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

        <div v-show="CARD_SCANNED" class="pin-screen-confirm">[#] Confirm</div>
        <div v-show="CARD_SCANNED" class="pin-screen-clear">[C] Clear</div>
        <div v-show="CARD_SCANNED" class="pin-screen-cancel">[D] Cancel</div>
      </div>
    </div>

    <div v-if="CARD_LOGGED_IN">
      <div class="side-menu-bg">
        <img src="../assets/SBER_icon.png" alt="logo" class="logo-icon">

        <div class="button-type-2">[A] Quick Cash</div>
        <div class="button-type-2">[B] Check Balance</div>
        <div class="button-type-2">[C] Withdraw</div>
        <div class="button-type-1">[D] Log out</div>
      </div>

      <div class="main-bg">
        <div v-show="MENU_CASH" v-if="!WITHDRAW_BALANCE_WARN" class="menu-quickcash">
          <p class="display-text">One moment... </p>
        </div>

        <div v-show="MENU_BALANCE" class="menu-balance">
          <div class="menu-balance-text">
            Balance 
            <p class="menu-balance-amount">
              ₽ {{ accountBalance }}
            </p>
          </div>
        </div>

        <div v-show="MENU_WITHDRAW" v-if="!WITHDRAW_BALANCE_WARN" class="menu-withdraw">
          <p class="display-text">Choose amount </p>
          <div class="withdraw-option-1">
            [1] ₽5000
          </div>
          <div class="withdraw-option-2">
            [2] ₽9000
          </div>
          <div class="withdraw-option-3">
            [3] ₽12000
          </div>
          <div class="withdraw-custom">
            [#] Custom
          </div>
        </div>

        <div v-show="WITHDRAW_BALANCE_WARN">
          <p class="display-text">
            Your balance is too low.
          </p>
        </div>

        <div v-show="WITHDRAW_BILL_OPTION" class="bill-option">
          <div v-if="WITHDRAW_BILLS==5000">
            <div class="button-type-3 withdraw-pos1">
              [1] 5x ₽1000
            </div>
            <div class="button-type-3 withdraw-pos2">
              [2] 1x ₽1000, 2x ₽2000 
            </div>
            <div class="button-type-3 withdraw-pos3">
              [3] 3x ₽1000, 1x ₽2000
            </div>
            <div class="button-type-3 withdraw-pos4">
              [4] 1x ₽5000
            </div>
          </div>
          <div v-else-if="WITHDRAW_BILLS==9000">
            <div class="button-type-3 withdraw-pos1">
              [1] 3x ₽1000, 3x ₽2000
            </div>
            <div class="button-type-3 withdraw-pos2">
              [2] 1x ₽1000, 4x ₽2000
            </div>
            <div class="button-type-3 withdraw-pos3">
              [3] 1x ₽5000, 2x ₽2000
            </div>
            <div class="button-type-3 withdraw-pos3">
              [4] 1x ₽5000, 4x ₽1000
            </div>
          </div>
          <div v-else-if="WITHDRAW_BILLS==12000">
            <div class="button-type-3 withdraw-pos1">
              [1] 2x ₽5000, 1x ₽2000
            </div>
            <div class="button-type-3 withdraw-pos2">
              [2] 5x ₽2000, 2x ₽1000
            </div>
            <div class="button-type-3 withdraw-pos3">
              [3] 1x ₽5000, 2x ₽2000, 3x ₽1000
            </div>
            <div class="button-type-3 withdraw-pos4">
              [4] 1x ₽5000, 3x ₽2000, 1x ₽1000
            </div>
          </div>
        </div>

        <div v-if="WITHDRAW_RECEIPT_PROMPT && !WITHDRAW_BILL_OPTION">
          <p class="display-text">Would you like a receipt? </p>
          <div class="receipt-yes button-type-2">
            [A] Yes
          </div>
          <div class="receipt-no button-type-2">
            [B] No
          </div>
        </div>
      </div>

      <img src="../assets/SBER_text.png" alt="logo" class="logo_CARD_LOGGED_IN">
    </div>
  </div>
</template>

<script>
  /**LIBRARIES */
  import axios from 'axios';
  import { setTimeout } from 'timers';
  const SerialPort = require('serialport');
  const Readline = require('@serialport/parser-readline');

  /**MODULE INSTANTIATIONS */
  // ATM port
  const port = new SerialPort('COM10', { baudRate: 9600 });
  const parser = port.pipe(new Readline({ delimiter: '\r\n' }));

  // receipt printer port
  //const rPort = new SerialPort('COM7', { baudRate: 9600 });

  // dispenser port
  //  const dPort = new SerialPort('COM8', { baudRate: 9600 });

  /**GLOBAL VARIABLES */
  const WITHDRAW_TIMEOUT = 2500; // in millis
  const SCREEN_TIMEOUT = 500;

  export default {
    name: 'landing-page',

    data() {
      return {
        /**GUI CONTROL - CARD */
        CARD_SCANNED: false,
        CARD_LOGGED_IN: false,
        CARD_WARN: false,

        /**GUI CONTROL - MENU */
        MENU_CASH: false,
        MENU_BALANCE: false,
        MENU_WITHDRAW: false,

        /**GUI CONTROL - WITHDRAWAL */
        WITHDRAW_BALANCE_WARN: false,
        WITHDRAW_RECEIPT_PROMPT: false,
        WITHDRAW_BILL_OPTION: false,
        WITHDRAW_BILLS: null,

        /**DATA */
        currentCard: null,
        inputCount: 0,
        0: '',
        1: '',
        2: '',
        3: '',
        accountBalance: null,
        dispenserData: null,
      }
    },

    methods: {
      /**Used to display dot on PIN boxes */
      notEmpty(number) {
        return this[number] !== '';
      },

      /**Clear input in PIN boxes */
      clearPinInput() {
        if (this.inputCount === 0) {
          return;
        }
        this[this.inputCount] = '';
        this.inputCount -= 1;
      },

      /**Reset all PIN input */
      resetPIN() {
        this.inputCount = 0;
        this[0] = '';
        this[1] = '';
        this[2] = '';
        this[3] = '';
      },

      /**Log in */
      login() {
        const loginRoute = 'http://145.24.222.43:8080/login'

        // post iban and pin to route to log in
        axios.post(loginRoute, {
          iban: this.currentCard,
          pin: `${this[0]}${this[1]}${this[2]}${this[3]}`,
        }, { headers: {  } })
        .then((response) => {
          console.log(response);
          
          // go to menu screen after log in successful
          this.CARD_LOGGED_IN = true;
        })
        .catch((response) => {
          console.log(response);
          
          this.CARD_WARN = true;
          this.resetPIN();
        })
      },

      /**Get balance */
      getBalance() {
        const loginRoute = 'http://145.24.222.43:8080/getbalance'

        // post iban and pin to route to get balance
        axios.post(loginRoute, {
          iban: this.currentCard,
          pin: `${this[0]}${this[1]}${this[2]}${this[3]}`,
        }, { headers: {  } })
        // on success
        .then((response) => {
          console.log(response);

          // assign response data to account balance data to display in HTML
          this.accountBalance = response.data;
        })
        // on error
        .catch((response) => {
          console.log(response);
        })
      },

      /**QUICK OPTION */
      quickcash() {
        const loginRoute = 'http://145.24.222.43:8080/withdraw'

        // post iban, pin, and amount to route
        axios.post(loginRoute, {
          iban: this.currentCard,
          pin: `${this[0]}${this[1]}${this[2]}${this[3]}`,
          amount: 4000,
        }, { headers: {  } })
        // on success
        .then((response) => {
          console.log(response);

          // log out after quick option
          setTimeout(() => {
              this.logout();
          }, WITHDRAW_TIMEOUT);
        })
        // on error
        .catch((response) => {
          console.log(response);

          // show low balance warning
          this.WITHDRAW_BALANCE_WARN = true;
          // go to main menu
          setTimeout(() => {
              this.resetData();
          }, WITHDRAW_TIMEOUT);
        });
      },

      /**Withdraw */
      withdraw(amount) {
        const loginRoute = 'http://145.24.222.43:8080/withdraw'

        // post iban, pin, and amount to route
        axios.post(loginRoute, {
          iban: this.currentCard,
          pin: `${this[0]}${this[1]}${this[2]}${this[3]}`,
          amount: amount,
        }, { headers: {  } })
        // on success, logout
        .then((response) => {
          console.log(response)
          setTimeout(() => {
              this.logout();
          }, WITHDRAW_TIMEOUT);
        })
        // on error
        .catch((response) => {
          console.log(response);

          // show low balance warning
          this.WITHDRAW_BALANCE_WARN = true;
          // go to main menu
          setTimeout(() => {
              this.resetData();
          }, WITHDRAW_TIMEOUT);
        });
      },

      /**Reset to blank menu */
      resetData() {
        this.MENU_CASH = false;
        this.MENU_BALANCE = false;
        this.MENU_WITHDRAW = false;

        this.WITHDRAW_RECEIPT_PROMPT = false;
        this.WITHDRAW_BALANCE_WARN = false;
        this.WITHDRAW_BILL_OPTION = false;
        this.WITHDRAW_BILLS = null;
      },

      /**Reset everything and log out */
      logout() {
        console.log("one")
        // if on PIN screen
        if(this.CARD_LOGGED_IN == false && this.CARD_SCANNED == true) {
          console.log("two")
          this.resetData();
          this.CARD_SCANNED = false;
        }

        // if logged in
        if(this.CARD_LOGGED_IN == true) {
          console.log("three")
          this.resetData();
          this.CARD_SCANNED = false;
          this.CARD_LOGGED_IN = false;
        }
        console.log('four')
        // clear PIN and IBAN
        this.resetPIN();
        this.currentCard = null;
      },

      /**Delay for menu */
      delaySwitch1() {
        this.WITHDRAW_BILL_OPTION = true;
      },
      delaySwitch2() {
        this.WITHDRAW_RECEIPT_PROMPT = true;
      },
    },

    mounted() {
      parser.on("data", (data) => {
        // keypad only sends one character, 
        // so use data length to filter card reading vs keypad input
        if(data.length > 2) {
          console.log(data);

          this.currentCard = data;
          this.CARD_SCANNED = true;
        }

        // PIN screen - handle keypad input
        if(this.CARD_SCANNED && !this.CARD_LOGGED_IN && 
            !this.MENU_WITHDRAW && !this.MENU_CASH && !this.MENU_BALANCE &&
            !this.WITHDRAW_BALANCE_WARN && !this.WITHDRAW_BILL_OPTION && !this.WITHDRAW_RECEIPT_PROMPT) {
          console.log("keypad " + data);
          
          // checking for only numbers for the actual PIN
          if(!isNaN(data)) {
            if(this.inputCount < 4) {
              this[this.inputCount] = data;
              this.inputCount++;
            }
          }
          // other buttons
          if(data === '#') {
            this.login();
          }
          if(data === 'C') {
            this.clearPinInput();
          }
          if(data === 'D') {
            this.logout();
          }
        }

        // menu - navigation
        if(this.CARD_LOGGED_IN && 
            !this.WITHDRAW_BALANCE_WARN && !this.WITHDRAW_BILL_OPTION && !this.WITHDRAW_RECEIPT_PROMPT) {
          if(data === 'A') {
            this.MENU_CASH = true;
            this.MENU_BALANCE = false;
            this.MENU_WITHDRAW = false;
            this.quickcash();
          }
          if(data === 'B') {
            this.MENU_CASH = false;
            this.MENU_BALANCE = true;
            this.MENU_WITHDRAW = false;
            this.getBalance();
          }
          if(data === 'C') {
            this.MENU_CASH = false;
            this.MENU_BALANCE = false;
            this.MENU_WITHDRAW = true;
          }
          if(data === 'D') {
            this.logout();
          }
        }

        // withdrawal screen
        if(this.CARD_LOGGED_IN && 
            this.MENU_WITHDRAW && !this.MENU_CASH && !this.MENU_BALANCE && 
            !this.WITHDRAW_BALANCE_WARN && !this.WITHDRAW_BILL_OPTION && !this.WITHDRAW_RECEIPT_PROMPT) {
          if(data === '1') {
            //this.withdraw(5000);
            this.MENU_WITHDRAW = false;
            this.WITHDRAW_BILLS = 5000;

            setTimeout(() => {
              this.delaySwitch1();
            }, SCREEN_TIMEOUT);
          }
          if(data === '2') {
            //this.withdraw(10000);
            this.MENU_WITHDRAW = false;
            this.WITHDRAW_BILLS = 9000;

            setTimeout(() => {
              this.delaySwitch1();
          }, SCREEN_TIMEOUT);
          }
          if(data === '3') {
            //this.withdraw(15000);
            this.MENU_WITHDRAW = false;
            this.WITHDRAW_BILLS = 12000;
            
            setTimeout(() => {
              this.delaySwitch1();
            }, SCREEN_TIMEOUT);
          }
          if(data === '#') {
            console.log('custom');
          }
        }

        // prompt bill options
        // format bills <1000, 2000, 5000> 
        if(this.CARD_LOGGED_IN && 
            !this.MENU_WITHDRAW && !this.MENU_CASH && !this.MENU_BALANCE &&
            !this.WITHDRAW_BALANCE_WARN && this.WITHDRAW_BILL_OPTION && !this.WITHDRAW_RECEIPT_PROMPT) {
          if(this.WITHDRAW_BILLS == 5000) {
            if(data === '1') {
              // 5x ₽1000
              this.dispenserData = "<5,0,0>"
              this.WITHDRAW_BILL_OPTION = false;

              setTimeout(() => {
                this.delaySwitch2();
              }, SCREEN_TIMEOUT);
            }
            if(data === '2') {
              // 1x ₽1000, 2x ₽2000 
              this.dispenserData = "<1,2,0>"
              this.WITHDRAW_BILL_OPTION = false;
              
              setTimeout(() => {
                this.delaySwitch2();
              }, SCREEN_TIMEOUT);
            }
            if(data === '3') {
              // 3x ₽1000, 1x ₽2000
              this.dispenserData = "<3,1,0>"
              this.WITHDRAW_BILL_OPTION = false;
              
              setTimeout(() => {
                this.delaySwitch2();
              }, SCREEN_TIMEOUT);
            }
            if(data === '4') {
              // 1x ₽5000
              this.dispenserData = "<0,0,1>"
              this.WITHDRAW_BILL_OPTION = false;
              
              setTimeout(() => {
                this.delaySwitch2();
              }, SCREEN_TIMEOUT);
            }
          }
          if(this.WITHDRAW_BILLS == 9000) {
            if(data === '1') {
              // 3x ₽1000, 3x ₽2000
              this.dispenserData = "<3,3,0>"
              this.WITHDRAW_BILL_OPTION = false;
              
              setTimeout(() => {
                this.delaySwitch2();
              }, SCREEN_TIMEOUT);
            }
            if(data === '2') {
              // 1x ₽1000, 4x ₽2000
              this.dispenserData = "<1,4,0>"
              this.WITHDRAW_BILL_OPTION = false;
              
              setTimeout(() => {
                this.delaySwitch2();
              }, SCREEN_TIMEOUT);
            }
            if(data === '3') {
              // 1x ₽5000, 2x ₽2000
              this.dispenserData = "<0,2,1>"
              this.WITHDRAW_BILL_OPTION = false;
              
              setTimeout(() => {
                this.delaySwitch2();
              }, SCREEN_TIMEOUT);
            }
            if(data === '4') {
              // 1x ₽5000, 4x ₽1000
              this.dispenserData = "<4,0,1>"
              this.WITHDRAW_BILL_OPTION = false;
              
              setTimeout(() => {
                this.delaySwitch2();
              }, SCREEN_TIMEOUT);
            }
          }
          if(this.WITHDRAW_BILLS == 12000) {
            if(data === '1') {
              // 2x ₽5000, 1x ₽2000
              this.dispenserData = "<0,1,2>"
              this.WITHDRAW_BILL_OPTION = false;
              
              setTimeout(() => {
                this.delaySwitch2();
              }, SCREEN_TIMEOUT);
            }
            if(data === '2') {
              // 5x ₽2000, 2x ₽1000
              this.dispenserData = "<2,5,0>"
              this.WITHDRAW_BILL_OPTION = false;

              setTimeout(() => {
                this.delaySwitch2();
              }, SCREEN_TIMEOUT);
            }
            if(data === '3') {
              // 1x ₽5000, 2x ₽2000, 3x ₽1000
              this.dispenserData = "<3,2,1>"
              this.WITHDRAW_BILL_OPTION = false;
              
              setTimeout(() => {
                this.delaySwitch2();
              }, SCREEN_TIMEOUT);
            }
            if(data === '4') {
              // 1x ₽5000, 3x ₽2000, 1x ₽1000
              this.dispenserData = "<1,3,1>"
              this.WITHDRAW_BILL_OPTION = false;
              
              setTimeout(() => {
                this.delaySwitch2();
              }, SCREEN_TIMEOUT);
            }
          }
        }

        // ask for receipt after withdrawal
        if(this.CARD_LOGGED_IN && 
            !this.MENU_WITHDRAW && !this.MENU_CASH && !this.MENU_BALANCE &&
            !this.WITHDRAW_BALANCE_WARN && !this.WITHDRAW_BILL_OPTION && this.WITHDRAW_RECEIPT_PROMPT) {
          if(data === 'A') {
            this.withdraw(this.WITHDRAW_BILLS);
          }
          if(data === 'B') {
            this.withdraw(this.WITHDRAW_BILLS);
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
    padding-top: 20px;
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

  .logo_CARD_LOGGED_IN {
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

  .button-type-1, .button-type-2 {
    margin: 30px;
    padding: 20px;
    font-size: 20px;
  }

  .button-type-1 {
    background: lightgrey;
    box-shadow: 4px 5px 9px  black;
    color: black;
    text-shadow: 1px 1px grey;
  }

  .button-type-2 {
    background: #1f9d59;
    box-shadow: 4px 5px 9px  black;
    color: white;
    text-shadow: 2px 1px #000;
  }

  .button-type-3 {
    background: #1f9d59;
    box-shadow: 4px 5px 9px  black;
    color: white;
    text-shadow: 2px 1px #000;
    position: absolute; 
    font-size: 30px;
    margin: 20px;
    padding: 20px;
  }

  .main_bg {
    width: 1000;
    height: 100vh;
    float: right;
  }

  .menu-quickcash, .menu-balance, .menu-withdraw {
    width: 100%;
    height: 100vh;
  }

  .display-text {
    font-size: 50px;
    position: absolute;
    top: 80px;
    left: 400px;
  }

  .menu-balance-text {
    width: 100%;
    font-size: 60px;
    padding-top: 200px;
    padding-left: 450px;
  }

  .menu-balance-amount {
    width: 100%;
    font-size: 70px;
    padding-top: 100px;
  }

  .withdraw-option-1, .withdraw-option-2, 
  .withdraw-option-3, .withdraw-custom {
    background: #1f9d59;
    box-shadow: 4px 5px 9px  black;

    font-size: 20px;
    color: white;
    text-shadow: 2px 1px #000; 

    position: absolute;
    width: 160px;
    height: 54px;
    padding: 16px;
  }

  .withdraw-option-1 {
    top: 240px;
    left: 400px;
  }

  .withdraw-option-2 {
    top: 240px;
    left: 680px;
  }

  .withdraw-option-3 {
    top: 360px;
    left: 400px;
  }

  .withdraw-custom {
    top: 360px;
    left: 680px;
  }

  .withdraw-pos1 {
    left: 360px;
    top: 220px;
  }

  .withdraw-pos2 {
    left: 360px;
    top: 320px;
  }

  .withdraw-pos3 {
    left: 360px;
    top: 420px;
  }

  .withdraw-pos4 {
    left: 360px;
    top: 520px;
  }

  .receipt-yes, .receipt-no {
    position: absolute;
    width: 120px;
    height: 54px;
    padding: 16px;
    font-size: 50px;
  }

  .receipt-yes {
    left: 420px;
    top: 300px;
  }

  .receipt-no {
    left: 720px;
    top: 300px;
  }
</style>
