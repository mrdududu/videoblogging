<template>
  <q-page class="column bg">
    <h1>
      Free app<br />for<br />
      video content creators
    </h1>
    <h2>
      Video camera +<br />
      Script assistant<br />
      in one app
    </h2>
    <q-page-sticky position="bottom" :offset="[0, 18]">
      <q-btn
        rounded
        size="lg"
        icon="camera"
        color="accent"
        label="Start"
        to="~/recorder"
      />
    </q-page-sticky>
  </q-page>
</template>
<script>
import { defineComponent, ref, onMounted } from "vue";
export default defineComponent({
  setup() {},
});
</script>
<style lang="scss" scoped>
.bg {
  background: url(~assets/img/bg-creator.jpg) no-repeat center center fixed;
  background-size: auto 100%;
  background-position: 5% 0%;
  padding: 40px;
}
H1 {
  font-size: 30px;
  color: white;
  font-weight: bold;
  line-height: 40px;
  margin: 0;
  padding: 0;
}
H2 {
  font-size: 24px;
  color: white;
  font-weight: normal;
  line-height: 34px;
  margin: 25px 0 0 0;
  padding: 0;
}
@media screen and (min-width: 600px) {
  .bg {
    background-size: 100% auto;
    background-position: 50% 50%;
  }
  H1 {
    font-size: 48px;
    line-height: 52px;
  }
  H2 {
    font-size: 36px;
    line-height: 40px;
  }
}
</style>
