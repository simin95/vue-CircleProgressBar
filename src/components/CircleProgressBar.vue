<template>
    <div class="canvas-wrapper" ref="canvasWrapper">
      <timer class="timer" v-if="this.options.showTimer" :style="timerStyle" :currentMicrosecond="currentMicrosecond" :processing="processing" :pause="pause" :options="options.timerOptions"></timer>
      <!-- <p :style="timerStyle">宽：{{options.width}}</p> -->
      <!-- <button @click="changeProcessing" style="position: absolute;top: 80px;left: 60px" :disabled="this.processing">开始</button>
      <button @click="changePause" style="position: absolute;top: 110px;left: 60px">暂停/继续</button> -->
      <canvas id="myCanvas" class="my-canvas"></canvas>
    </div>
</template>

<script>
import Timer from './Timer';
export default {
  name: 'CircleProgressBar',
  props: {
    processing: {
      type: Boolean,
      // default: false
    },
    pause: {
      type: Boolean,
      default: false,
    },
    options: {
      type: Object,
      default() {},
    },
  },
  components: {
    Timer,
  },
  data() {
    return {
      // 通过设置每一步的时间（单位毫秒，刷新一帧的时间）和总的步数，来控制显示进度；总的时间花费由两数相乘计算得到，并传给时间显示组件来以秒为单位倒计时
      // 可能存在的问题：需要计算得到某些整数值时使用了Math.floor()方法，可能导致计算结果不准，误差应该不会超过0.5s
      // 总的步数（将360°等分；刷新的次数）
      // totalSteps: 20000,
      // 每一步的时间（单位毫秒，20-50合适）
      // stepTime: 20,
      // 当前的步数
      currentStep: 0,
      // 开始的位置
      startPosition: 0,
      // 终止的位置
      endPosition: 0,
      // 定时器标志，用于关闭定时器操作
      drawing: 0,
      // isMounted用于判断挂载是否完成
      isMounted: false,

      // 开始(true)/结束(false)标志
      // processing: false,
      // 暂停(true)/继续(false)标志
      // pause: false

      // options: {
      //   // 只设置canvas画布大小,进度条尽量占满，随其大小变化，高度与宽度默认相同
      // width: 200
      //   // height: 300,
      //   // 进度条颜色
      //   barColor: "green",
      //   // 进度条宽度
      //   lineWidth: 2,
      //   // 背景颜色，因设置了overflow:hidden，圆形canvas下的其他背景不显示
      //   backgroundColor: "yellowgreen",
      //   // 内填充颜色
      //   innerBackgroundColor: "#ccc",
      //   // 对timer组件的设置,timer组件绝对定位在父元素中，其位置使用者可通过配置占父元素的长宽百分比来得到，可控性强
      //   timerPositionRow: 0.1,
      //   timerPositionCol: 0.1,
      //   // timer组件的字体设置
      //   timerOptions: {
      //     textFontSize: "40px",
      //     textFontFamily: "Arial"
      //   }
      // }
    };
  },
  mounted() {
    // setInterval(() => {
    //   console.log(this.$refs.canvasWrapper.offsetWidth);
    // }, 500);
    // this.progress();
    // 组件挂载时先画出背景

    this.$nextTick(() => {
      let c = document.getElementById('myCanvas');
      console.log('canvas开始绘制！！！');
      console.log(this.width);
      c.setAttribute('width', this.width);
      c.setAttribute('height', this.width);

      let ctx = c.getContext('2d');

      // 用于解决retina屏有毛边锯齿的问题，缺点：越精细越消耗性能
      let width = c.width;
      let height = c.height;
      if (window.devicePixelRatio) {
        c.style.width = width + 'px';
        c.style.height = height + 'px';
        c.height = height * window.devicePixelRatio;
        c.width = width * window.devicePixelRatio;
        ctx.scale(window.devicePixelRatio, window.devicePixelRatio);
      }

      // 初始化：圆内的背景填充及进度条轨道
      ctx.beginPath();
      ctx.arc(
        this.width / 2 - 1,
        this.width / 2 - 1,
        this.width / 2 - 1,
        0,
        Math.PI * 2,
      );
      ctx.fillStyle = this.options.innerBackgroundColor;
      ctx.fill();
    });

    this.isMounted = true;
  },
  watch: {
    processing: function(val, oldVal) {
      // 向外派发事件
      if (val === true) {
        console.log(this);
        this.$emit('stateChanged', ['start']);
      } else {
        this.$emit('stateChanged', ['done']);
      }
      // 执行开始进度条逻辑
      if (val === true) {
        console.log('执行开始进度条逻辑');
        this.clearRect();
        this.progress();
      } else {
        console.log('执行结束进度条逻辑');
        this.clearRect();
        clearInterval(this.drawing);
        this.drawing = 0;
        // this.processing = false;
        this.startPosition = 0;
        this.endPosition = 0;
        this.currentStep = 0;
      }
    },
    pause: function(val, oldVal) {
      // 向外派发事件
      if (val === true) {
        this.$emit('stateChanged', ['pause']);
      } else {
        this.$emit('stateChanged', ['continue']);
      }
      if (val === true) {
        this.handlePause();
      } else if (val === false && this.processing === true) {
        this.progress();
      }
    },
  },
  computed: {
    width() {
      // if(this.width.indexOf('px') > -1){
      //   let widthPx = Number(this.width.split('px')[0]);
      //   return widthPx
      //   // return this.width.split('px')[0]
      // } else if (this.width.indexOf('rem') > -1){
      //   let widthRem = Number(this.width.split('rem')[0]);
      //   // let htmlFontSize =
      //   // return Math.round()
      // }

      if (this.isMounted) {
        return this.$refs.canvasWrapper.offsetWidth;
      }
    },
    height() {
      return this.width;
    },
    currentMicrosecond() {
      return (
        (this.options.totalSteps - this.currentStep) * this.options.stepTime
      );
    },
    stepLength() {
      return 2 * Math.PI / this.options.totalSteps;
    },
    timerStyle() {
      return {
        position: 'absolute',
        left: (this.width * this.options.timerPositionRow).toString() + 'px',
        top: (this.width * this.options.timerPositionCol).toString() + 'px',
      };
    },
    wrapperStyle() {
      return {
        width: this.width + 'px',
        height: this.width + 'px',
      };
    },
  },
  methods: {
    // changeProcessing() {
    //   this.processing = true;
    // },
    // changePause() {
    //   if (this.processing) {
    //     this.pause = !this.pause;
    //   }
    // },

    // canvas重绘函数: 清屏及重绘背景，在新绘制的进度条内容不能覆盖原来的内容时需要执行此操作，这里仅在开始绘制时调用
    clearRect() {
      console.log('清屏及重绘背景');
      let c = document.getElementById('myCanvas');
      let ctx = c.getContext('2d');
      ctx.clearRect(0, 0, this.width, this.height);
      ctx.beginPath();
      ctx.arc(
        this.width / 2 - 1,
        this.width / 2 - 1,
        this.width / 2 - 1,
        0,
        Math.PI * 2,
      );
      ctx.fillStyle = this.options.innerBackgroundColor;
      ctx.fill();
    },
    handlePause() {
      console.log('暂停！');
      clearInterval(this.drawing);
      this.drawing = 0;
    },
    progress() {
      let c = document.getElementById('myCanvas');
      let ctx = c.getContext('2d');
      console.log('开始绘制');
      // 准备工作：清屏，绘制背景
      // 这里需要解决：每次继续都清屏重绘导致的闪白问题，即不需要对继续操作做重绘处理
      // if (this.processing === true && this.pause === false) {
      //   ctx.clearRect(0, 0, this.width, this.height);
      //   ctx.beginPath();
      //   ctx.arc(
      //     this.width / 2 - 1,
      //     this.width / 2 - 1,
      //     this.width / 2 - 1,
      //     0,
      //     Math.PI * 2,
      //   );
      //   ctx.fillStyle = '#ccc';
      //   ctx.fill();
      // }

      // 定时绘制进度条
      this.drawing = setInterval(() => {
        this.endPosition += this.stepLength;
        this.currentStep++;
        this.draw(ctx);
        // 停止的判断及处理
        if (this.endPosition >= 2 * Math.PI) {
          clearInterval(this.drawing);
          this.drawing = 0;
          console.log('----------');
          // console.log(drawing);
          this.processing = false;
          this.startPosition = 0;
          this.endPosition = 0;
          this.currentStep = 0;
        }
        // 暂停的判断
      }, this.options.stepTime);
    },
    draw(ctx) {
      // ctx.clearRect(0, 0, 320, 320);
      ctx.beginPath();
      ctx.arc(
        this.width / 2 - 1,
        this.width / 2 - 1,
        this.width / 2 - this.options.lineWidth / 2 - 1,
        this.startPosition - Math.PI / 2,
        this.endPosition - Math.PI / 2,
      );
      ctx.strokeStyle = this.options.lineColor;
      ctx.lineWidth = this.options.lineWidth;
      ctx.stroke();
    },
  },
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
    // border: 1px solid yellowgreen;
    // background: yellowgreen;
    vertical-align: bottom;
  }
}
</style>
