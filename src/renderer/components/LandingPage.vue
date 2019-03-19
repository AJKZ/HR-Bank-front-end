<template>
  <div id="wrapper">
    <div class="md-layout md-gutter">
      <div v-for="index in inputCount" :key="index" class="md-layout-item border box">
        <div class="pw-dot">•</div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'landing-page',
    data: function() {
      return {
        inputCount: 0,
        pin_dot: false
      }
    },
    mounted() {
      window.addEventListener("keypress", function(e) {
        console.log(String.fromCharCode(e.keyCode));

        if(this.inputCount == 4) {
          return;
        }
        this.inputCount += 1;

      }.bind(this)); 
    }
  }
</script>

<style>
  #wrapper {
    background:
      radial-gradient(
        ellipse at top left,
        rgba(255, 255, 255, 1) 40%,
        rgba(229, 229, 229, .9) 100%
      );
    height: 100vh;
    padding: 10px 10px;
    width: 100vw;
  }

  .md-layout {
    margin-top: 200px;
    border: 1px solid #000;
  }

  .border {
    border: 2px solid #000;
  }

  .box {
    height: 150px;
  }

  .pw-dot {
    padding-top: 60px;
    text-align: center;
    font-size: 80px;
  }
</style>
