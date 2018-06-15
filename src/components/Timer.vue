<template>
  <div class="timer">
    <!-- <button @click="start">倒计时开始</button> -->
    <p class="timer-wrapper" :style="fontStyle"><span class="number">{{minutes}}</span><span>:</span><span class="number">{{seconds}}</span></p>
  </div>
</template>

<script>
export default {
  name: 'Timer',
  props: {
    currentMicrosecond: Number,
    processing: Boolean,
    pause: Boolean,
    options: {
      type: Object,
      default() {
        return {
          textFontSize: '48px',
          textFontFamily: 'Arial',
        };
      },
    },
  },
  data() {
    return {
      // currentSecond: 0
    };
  },
  watch: {
    processing: function(val, oldVal) {
      if (val === true) {
        this.start();
      }
    },
    pause: function(val, oldVal) {
      console.log('pause change to:' + val);
    },
  },
  computed: {
    currentSecond: {
      // 读取此值时执行的操作，如 vm.currentSecond
      get: function() {
        if (!this.processing && !this.pause) {
          return 0;
        } else {
          return Math.ceil(this.currentMicrosecond / 1000);
        }
      },
      // 设置此值时执行的操作，如 vm.currentSecond = 100
      set: function() {},
    },
    minutes() {
      if (this.currentSecond / 60 < 10 && this.currentSecond / 60 >= 0) {
        return '0' + Math.floor(this.currentSecond / 60);
      } else if (
        this.currentSecond / 60 < 60 &&
        this.currentSecond / 60 >= 10
      ) {
        return Math.floor(this.currentSecond / 60);
      } else {
        return '00';
      }
    },
    seconds() {
      if (this.currentSecond % 60 < 10 && this.currentSecond % 60 >= 0) {
        return '0' + this.currentSecond % 60;
      } else if (
        this.currentSecond % 60 < 60 &&
        this.currentSecond % 60 >= 10
      ) {
        return this.currentSecond % 60;
      } else {
        return '00';
      }
    },
    fontStyle() {
      return {
        fontSize: this.options.textFontSize,
        fontFamily: this.options.textFontFamily,
      };
    },
  },
  methods: {
    start() {
      let timer = setInterval(() => {
        this.currentSecond--;
        if (this.currentSecond === 0) clearInterval(timer);
      }, 1000);
    },
  },
};
</script>

<style lang="stylus" scoped>
.timer {
  display: inline-block;

  .timer-wrapper {
    margin: 0;
  }

  .number {
    vertical-align: middle;
  }
}
</style>
