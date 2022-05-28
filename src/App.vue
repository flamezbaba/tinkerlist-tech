<template>
  <div class="wrapper">
    <div class="content">
      <TodayComp />
    </div>

  </div>
</template>


<script>
import TodayComp from "./components/TodayComp.vue";

export default {
  components: { TodayComp },
  setup() {},
};
</script>


<style lang="scss">
@import "@/assets/base.css";

.wrapper {
  display: flex;
  width: 100%;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  margin: 0;
  padding: 0;

  &::before {
    content: "";
    background-image: url("@/assets/bg.jpg");
    background-size: cover;
    filter: blur(5px);
    position: absolute;
    width: 100%;
    height: 100%;
  }

  .content {
    width: 80vw;
    min-height: 300px;
    border-radius: 10px;
    z-index: 1000;
    background: linear-gradient(to top, #181818 10%, #2b2b2b 90%);
    padding: 40px 40px;
    margin-top: 60px;
    margin-bottom: 60px;
  }

}
</style>
