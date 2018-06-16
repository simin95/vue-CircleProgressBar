# vue-CircleProgressBar

![示例1](https://github.com/ChenSimin0103/vue-CircleProgressBar/tree/master/static/gif/001.gif) ![示例2](https://github.com/ChenSimin0103/vue-CircleProgressBar/tree/master/static/gif/002.gif) ![示例3](https://github.com/ChenSimin0103/vue-CircleProgressBar/tree/master/static/gif/003.gif) ![示例4](https://github.com/ChenSimin0103/vue-CircleProgressBar/tree/master/static/gif/004.gif) ![示例5](https://github.com/ChenSimin0103/vue-CircleProgressBar/tree/master/static/gif/005.gif) ![示例6](https://github.com/ChenSimin0103/vue-CircleProgressBar/tree/master/static/gif/006.gif)

> 更改记录

2018.06.01：完成主要功能实现

2018.06.15：修复问题，添加组件在不同屏幕上的适配方法，README

# 这是什么组件？

### 一个用 canvas 绘制的圆形进度条，考虑使用场景，提供了较多的配置项，canvas 的画布大小与容器等同方便适配，还内置了一个可选的倒计时小组件

### 最初是开发给葛莉+的箭牌卫浴插件

# 好在哪？

1.  在进度条组件内部定义了开始、结束，暂停。继续这些状态，使用时以 props 传入状态控制位（processing,pause）进入该组件来控制组件状态，父组件只需修改状态控制位的数据，就像开关一样组件在相应的状态变化时传出事件，父组件监听这些事件并添加处理逻辑

2.  子组件大小可通过 css 设置，组件内部元素自适应布局位置，在容器大小变化时能保持显示一致，且方便适配

3.  以倒计时作为进度标的，倒计时时间由配置的 步数\*每步时间相乘得到，背景颜色，线宽，文字样式，显示位置可配置

注：刷新帧率为 canvas 重绘的时间间隔，一般设置为 20~50ms，太小影响性能，太大变化不连贯

# 如何使用？

复制 CircleProgressBar 和 Timer 组件到自己的目录下

如在 demo.vue 中：

```html
    <circle-progress-bar
      class="circle-progress-bar"           // 组件的类名，大小需在css中设置
      :processing="processing"              // 控制位：开始/停止
      :pause="pause"                        // 控制位：暂停/继续
      :options="options"                    // 参数设置
      @stateChanged="handleStateChanged"    // 事件监听
    ></circle-progress-bar>
```

```javascript
  import CircleProgressBar from 'src/components/CircleProgressBar';
  ...
  ...
  components: {
    CircleProgressBar,
  },
  data() {
    return {
      // 控制位：开始/停止
      processing: false,
      // 控制位：暂停/继续
      pause: false,
      // 参数设置
      options: {
        // 总的步数（将360°等分；刷新的次数）
        totalSteps: 800,
        // 每一步的时间（单位毫秒，20-50合适）
        stepTime: 20,
        // 进度条颜色
        lineColor: 'orange',
        // 进度条宽度
        lineWidth: 8,
        // 内填充颜色
        innerBackgroundColor: '#eee',
        // 对timer组件的设置,timer组件绝对定位在父元素中，其位置使用者可通过配置占父元素的长宽百分比来得到，可控性强
        // 是否显示Timer组件
        showTimer: true,
        // 距离左边的百分比距离
        timerPositionRow: 0.18,
        // 距离上方的百分比距离
        timerPositionCol: 0.32,
        // timer组件的字体设置
        timerOptions: {
          textFontSize: '42px',
          textFontFamily: 'Arial',
        },
      },
    };
  },
  methods: {
    handleStateChanged(e) {
      console.log('事件监听：：');
      switch (e[0]) {
        case 'start':
          this.processing = true;
          // 处理开始时的逻辑
          break;
        case 'done':
          this.processing = false;
          // 处理结束时的逻辑
          break;
        case 'pause':
          this.pasue = true;
          // 处理暂停时的逻辑
          break;
        case 'continue':
          this.pause = false;
          // 处理继续时的逻辑
          break;
        default:
          // 无相应事件名匹配时
          break;
      }
    },
  },
```

# 其他

> 欠缺：

单纯作为 UI 显示，组件内部只做绘制不带判断逻辑和倒计时的使用方式，即拿到当前的进度百分比然后绘制出来，可能写成一个更简单的组件更好。

再添加一个可选的百分比进度的子组件，能有更多的使用场景

> bug:

1.  开始-暂停-停止-开始-暂停 会停不下来
