<template>
  <div class="canvas-wrapper">
    <p>在这里绘制canvas</p>
    <timer class="timer" :class="this.timerStyle" :totalSecond="totalSecond" :processing="processing" :pause="pause"></timer>
    <button @click="progress" :disabled="this.processing">开始</button>
    <canvas id="myCanvas" class="my-canvas"></canvas>
  </div>
</template>

<script>
import Timer from "./Timer";
export default {
  name: "HelloWorld",
  components: {
    Timer
  },
  data() {
    return {
      // 通过设置每一步的时间（单位毫秒，刷新一帧的时间）和总的步数，来控制显示进度；总的时间花费由两数相乘计算得到，并传给时间显示组件来以秒为单位倒计时
      // 可能存在的问题：需要计算得到某些整数值时使用了Math.floor()方法，可能导致计算结果不准，误差应该不会超过0.5s
      totalSteps: 200,
      stepTime: 20,
      startPosition: 0,
      endPosition: 0,
      processing: false,
      pause: false,
      options: {
        // 只设置canvas画布大小,进度条尽量占满，随其大小变化
        width: 160,
        height: 320,
        barColor: "green",
        barWidth: 60,
        backgroundColor: "yellowgreen",
        innerBackgroundColor: "#ccc",
        textFontSize: "24px",
        textFontFamily: "Arial",
        // 对timer组件的设置,timer组件绝对定位在父元素中，其位置使用者可通过配置占父元素的长宽百分比来得到，可控性强
        timerPositionRow: "30%",
        timerPositionCol: "30%"
      }
    };
  },
  mounted() {
    // this.progress();
    // 组件挂载时先画出背景
    let c = document.getElementById("myCanvas");
    c.setAttribute("width", this.options.width);
    c.setAttribute("height", this.options.height);
    let ctx = c.getContext("2d");

    // 用于解决retina屏有毛边锯齿的问题
    const width = c.width;
    const height = c.height;
    if (window.devicePixelRatio) {
      c.style.width = width + "px";
      c.style.height = height + "px";
      c.height = height * window.devicePixelRatio;
      c.width = width * window.devicePixelRatio;
      ctx.scale(window.devicePixelRatio, window.devicePixelRatio);
    }

    // 初始化：圆内的背景填充及进度条轨道
    ctx.beginPath();
    ctx.arc(
      this.options.width / 2,
      this.options.width / 2,
      this.options.width / 2,
      0,
      Math.PI * 2
    );
    ctx.fillStyle = "#ccc";
    ctx.fill();
  },
  watch: {
    processing: (val, oldVal) => {
      if (val === true) {
        console.log(this);
        this.$emit("stateChanged", ["start"]);
      } else {
        this.$emit("stateChanged", ["done"]);
      }
    },
    pause: (val, oldVal) => {
      if (val === true) {
        this.$emit("stateChanged", ["pause"]);
      } else {
        this.$emit("stateChanged", ["continue"]);
      }
    }
  },
  computed: {
    totalSecond() {
      return Math.ceil(this.totalSteps * this.stepTime / 1000);
    },
    stepLength() {
      return 2 * Math.PI / this.totalSteps;
    },
    timerStyle() {
      return {
        width: "30%",
        height: "30%"
      };
    }
  },
  methods: {
    progress() {
      let c = document.getElementById("myCanvas");
      let ctx = c.getContext("2d");
      console.log("开始绘制");
      // 准备工作：清屏，绘制背景
      ctx.clearRect(0, 0, this.options.width, this.options.height);
      ctx.beginPath();
      ctx.arc(
        this.options.width / 2,
        this.options.width / 2,
        this.options.width / 2,
        0,
        Math.PI * 2
      );
      ctx.fillStyle = "#ccc";
      ctx.fill();
      // 打开进度条运行标志位，让计时器同步开始工作
      this.processing = true;
      var drawing = setInterval(() => {
        this.endPosition += this.stepLength;
        this.draw(ctx);
        if (this.endPosition >= 2 * Math.PI) {
          clearInterval(drawing);
          console.log("----------");
          console.log(drawing);
          this.processing = false;
          this.startPosition = 0;
          this.endPosition = 0;
        }
      }, this.stepTime);
    },
    draw(ctx) {
      // ctx.clearRect(0, 0, 320, 320);
      ctx.beginPath();
      ctx.arc(
        this.options.width / 2,
        this.options.width / 2,
        this.options.width / 2 - this.options.barWidth / 2,
        this.startPosition - Math.PI / 2,
        this.endPosition - Math.PI / 2
      );
      ctx.strokeStyle = this.options.barColor;
      ctx.lineWidth = this.options.barWidth;
      ctx.stroke();
      // ctx.font = "24px Arial";
      // ctx.fillText(`${this.allSteps - this.steps}`, 80, 80);
      // console.log("重绘!");
      // console.log(this.startPosition);
      // console.log(this.endPosition);
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" scoped>
.canvas-wrapper {
  position: relative;

  .timer {
    position: absolute;
  }

  .my-canvas {
    // width: 320px;
    // height: 320px;
    border: 1px solid yellowgreen;
    background: yellowgreen;
  }
}
</style>
