<template>
  <div>
    <h1>My Circle Progress Bar</h1>
    <p>控制选项</p>
    <!-- <button @click="changeProcessing" :disabled="this.processing">开始/结束</button>
    <button @click="changePause">暂停/继续</button> -->
    <circle-progress-bar
      class="circle-progress-bar"
      :processing="processing"
      :pause="pause"
      :options="options"
      @stateChanged="handleStateChanged"
    ></circle-progress-bar>
  </div>
</template>

<script>
import CircleProgressBar from "@/components/CircleProgressBar";

export default {
  name: "Demo",
  components: {
    CircleProgressBar
  },
  data() {
    return {
      processing: false,
      pause: false,
      options: {
        // 总的步数（将360°等分；刷新的次数）
        totalSteps: 200,
        // 每一步的时间（单位毫秒，20-50合适）
        stepTime: 20,
        // 只设置canvas画布大小,进度条尽量占满，随其大小变化，高度与宽度默认相同
        width: 200,
        // height: 300,
        // 进度条颜色
        barColor: "green",
        // 进度条宽度
        barWidth: 16,
        // 背景颜色，因设置了overflow:hidden，圆形canvas下的其他背景不显示
        backgroundColor: "yellowgreen",
        // 内填充颜色
        innerBackgroundColor: "#ccc",
        // 对timer组件的设置,timer组件绝对定位在父元素中，其位置使用者可通过配置占父元素的长宽百分比来得到，可控性强
        timerPositionRow: 0.45,
        timerPositionCol: 0.45,
        // timer组件的字体设置
        timerOptions: {
          textFontSize: "24px",
          textFontFamily: "Arial"
        }
      }
    };
    // options:{}
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
    handleStateChanged(e) {
      console.log(e);
    }
  }
};
</script>

<style lang="stylus" scoped>
.circle-progress-bar
  width 50%
  height 0
  padding-bottom 50%
</style>
