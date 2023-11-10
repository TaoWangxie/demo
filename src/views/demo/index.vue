<template>
  <div class="container">666666666</div>
</template>

<script setup lang='ts'>
import { ref, onMounted, nextTick } from "vue";

onMounted(() => {
  nextTick(() => {
    // #ifdef H5
    setTimeout(() => {
      historyPush();
      addEvent();
    }, 400);
    // #endif
  });
});

const addEvent = () => {
  // #ifdef H5
  window.addEventListener("popstate", handlePopup, false);
  // #endif
};

const removeEvent = () => {
  // #ifdef H5
  window.removeEventListener("popstate", handlePopup, false);
  // #endif
};

const historyPush = () => {
  // #ifdef H5
  window.history.pushState(history.state, document.title, location.href);
  // #endif
};

const handlePopup = () => {
  alert(88888888);
};
</script>

<style lang="scss" scoped>
.container {
  height: 100vh;
}
</style>
