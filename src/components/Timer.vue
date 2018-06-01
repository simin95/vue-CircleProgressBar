<template>
  <div>
    <!-- <button @click="start">倒计时开始</button> -->
    <span>{{minutes}}</span>:<span>{{seconds}}</span>
  </div>
</template>

<script>
export default {
  name: "Timer",
  props: {
    totalSecond: Number,
    processing: Boolean,
    pause: Boolean
  },
  data() {
    return {
      currentSecond: 0
    };
  },
  watch: {
    processing: function(val, oldVal) {
      console.log(val);
      console.log(oldVal);
      if (val === true) {
        this.start();
      }
    }
  },
  computed: {
    minutes() {
      if (this.currentSecond / 60 < 10 && this.currentSecond / 60 >= 0) {
        return "0" + Math.floor(this.currentSecond / 60);
      } else if (this.currentSecond / 60 < 60 && this.currentSecond / 60 >= 10) {
        return Math.floor(this.currentSecond / 60);
      } else {
        return "00";
      }
    },
    seconds() {
      if (this.currentSecond / 60 < 10 && this.currentSecond / 60 >= 0) {
        return "0" + this.currentSecond % 60;
      } else if (this.currentSecond / 60 < 60 && this.currentSecond / 60 >= 10) {
        return this.currentSecond % 60;
      } else {
        return "00";
      }
    }
  },
  methods: {
    start() {
      this.currentSecond = this.totalSecond;
      let timer = setInterval(() => {
        this.currentSecond--;
        if (this.currentSecond === 0) clearInterval(timer);
      }, 1000);
    }
  }
};
</script>

<style lang="stylus" scoped>
</style>
