<template>
  <div id="app">
    <ChangeChairsTouchDevice v-show="isTouchDevice"></ChangeChairsTouchDevice>
    <ChangeChairsPC v-show="!isTouchDevice"></ChangeChairsPC>
  </div>
</template>

<script>
import ChangeChairsTouchDevice from './components/ChangeChairsTouchDeviceComp.vue';
import ChangeChairsPC from './components/ChangeChairsPCComp.vue'

export default {
  name: 'App',
  components: {
    ChangeChairsTouchDevice,
    ChangeChairsPC
  },
  data() {
    return {
      isTouchDevice: '',
    }
  },
  mounted() {
    let touch_event = window.ontouchstart;
    let touch_points = navigator.maxTouchPoints;
    console.log(touch_event)
    console.log(touch_points)
    if(touch_event !== undefined && 0 < touch_points) {
      this.isTouchDevice = true;
    } else {
      this.isTouchDevice = false;
    }
    console.log("isTouchDevice: ",this.isTouchDevice);
  }
}
</script>


