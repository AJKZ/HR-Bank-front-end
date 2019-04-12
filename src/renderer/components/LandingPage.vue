<template>
  <div>
    <el-button @click="cardScanned = true">Replicate card scan</el-button>
    <el-button @click="loggedIn = true">Replicate log in</el-button>

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
          <div v-show="cardScanned">
            <el-row class="input-row">
              <el-col class="pin-pass-box" :offset="3" :span="2">
                <h1 class="pin-dot" v-if="notEmpty(0)">•</h1>

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

        <div class="menu-button">[A] </div>
        <div class="menu-button">[B] </div>
        <div class="menu-button">[C] </div>
        <div class="menu-button">[D] </div>

        <el-button @click="loggedIn = false; cardScanned = false" class="logout-button">
          temporary logout button
        </el-button>
      </div>



      <img src="../assets/SBER_text.png" alt="logo" class="logo_loggedIn">
    </div>
  </div>
</template>

<script>
  const SerialPort = require('serialport');
  const Readline = require('@serialport/parser-readline')

  const port = new SerialPort('COM10', { baudRate: 9600 });


  export default {
    name: 'landing-page',

    data() {
      return {
        cardScanned: true,
        inputCount: 0,
        pinAttempt: 0,
        loggedIn: false,
        0: '',
        1: '',
        2: '',
        3: ''
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
      }
    },

    mounted() {
      const parser = new Readline();
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
    margin-top: 0px;
    font-size: 130px;
    margin-bottom: 0;
  }

  .input-row{
  }

  .pin-pass-box{
    height: 120px;
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

  .menu-button {
    margin: 30px;
    padding: 20px;
    background: #1f9d59;
    box-shadow: 4px 5px 9px  black;

    font-size: 20px;
    color: white;
    text-shadow: 1px 1px #000;
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
