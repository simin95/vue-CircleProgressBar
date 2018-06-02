<template>
  <div class="canvas-wrapper" :style="wrapperStyle">
    <!-- <p>在这里绘制canvas</p> -->
    <timer class="timer" :style="timerStyle" :currentMicrosecond="currentMicrosecond" :processing="processing" :pause="pause"></timer>
    <button @click="changeProcessing" style="position: absolute;top: 80px;left: 60px" :disabled="this.processing">开始</button>
    <button @click="changePause" style="position: absolute;top: 110px;left: 60px">暂停/继续</button>
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
      drawing: 0,
      processing: false,
      pause: false,
      options: {
        // 只设置canvas画布大小,进度条尽量占满，随其大小变化
        width: 300,
        height: 300,
        barColor: "green",
        barWidth: 16,
        backgroundColor: "yellowgreen",
        innerBackgroundColor: "#ccc",
        textFontSize: "48px",
        textFontFamily: "Arial",
        // 对timer组件的设置,timer组件绝对定位在父元素中，其位置使用者可通过配置占父元素的长宽百分比来得到，可控性强
        timerPositionRow: 0.4,
        timerPositionCol: 0.45
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
      this.options.width / 2 + 1,
      0,
      Math.PI * 2
    );
    ctx.fillStyle = "#ccc";
    ctx.fill();
  },
  watch: {
    processing: function(val, oldVal) {
      // 向外派发事件
      // if (val === true) {
      //   console.log(this);
      //   this.$emit("stateChanged", ["start"]);
      // } else {
      //   this.$emit("stateChanged", ["done"]);
      // }
      // 执行开始进度条逻辑
      if (val === true) {
        console.log("执行开始进度条逻辑");
        this.progress();
      }
    },
    pause: function(val, oldVal) {
      // 向外派发事件
      // if (val === true) {
      //   this.$emit("stateChanged", ["pause"]);
      // } else {
      //   this.$emit("stateChanged", ["continue"]);
      // }
      if (val === true) {
        this.handlePause();
      } else if (val === false && this.processing === true) {
        this.progress();
      }
    }
  },
  computed: {
    currentMicrosecond() {
      return this.totalSteps * this.stepTime;
    },
    stepLength() {
      return 2 * Math.PI / this.totalSteps;
    },
    timerStyle() {
      return {
        position: "absolute",
        left:
          (this.options.width * this.options.timerPositionRow).toString() +
          "px",
        top:
          (this.options.height * this.options.timerPositionCol).toString() +
          "px"
        // top: "48px",
        // left: "96px"
      };
    },
    wrapperStyle() {
      return {
        width: this.options.width,
        height: this.options.height
      };
    }
  },
  methods: {
    changeProcessing() {
      this.processing = true;
    },
    changePause() {
      if (this.processing) {
        this.pause = !this.pause;
      }
    },
    handlePause() {
      console.log("暂停！");
      clearInterval(this.drawing);
    },
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
        this.options.width / 2 + 1,
        0,
        Math.PI * 2
      );
      ctx.fillStyle = "#ccc";
      ctx.fill();
      // 定时绘制进度条
      this.drawing = setInterval(() => {
        this.endPosition += this.stepLength;
        this.draw(ctx);
        // 停止的判断及处理
        if (this.endPosition >= 2 * Math.PI) {
          clearInterval(this.drawing);
          console.log("----------");
          // console.log(drawing);
          this.processing = false;
          this.startPosition = 0;
          this.endPosition = 0;
        }
        // 暂停的判断
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
  display: inline-block;
  border-radius: 50%;
  overflow: hidden;

  .timer {
    position: absolute;
  }

  .my-canvas {
    // width: 320px;
    // height: 320px;
    border: 1px solid yellowgreen;
    background: yellowgreen;
    vertical-align: bottom;
  }
}
</style>
