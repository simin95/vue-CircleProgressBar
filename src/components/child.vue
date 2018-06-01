<template>
  <div>
    <p class="text-normal">this is child</p>
    <p class="text-change">this is child</p>
  </div>
</template>

<script>
export default {
  name: "child",
  data() {
    return {
      msg: "child show"
    };
  },
  created() {
    console.log("child created");
  },
  mounted() {
    console.log("mounted!");
  },
  // keep-alive包裹的组件挂载与停用时的生命周期钩子
  activated() {
    console.log("activated!");
  },
  deactivated() {
    console.log("deactivated!");
  }
};
</script>

<style lang="stylus">
.text-normal
  font-size 12px
.text-change
  font-size 2px
</style>
