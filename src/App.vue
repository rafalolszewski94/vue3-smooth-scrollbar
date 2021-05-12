<template>
  <div class="container">
    <SmoothScrollbar>
      <div class="box"></div>
      <div class="box"></div>
      <div class="box"></div>
    </SmoothScrollbar>
  </div>
</template>

<script>
import SmoothScrollbar from "@/SmoothScrollbar"
export default {
  name: "App",
  components: { SmoothScrollbar },
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
}

*,
*:before,
*:after {
  box-sizing: border-box;
}

body {
  height: 100%;
  width: 100%;
  overflow: hidden;
}

.container {
  height: 100vh;
  width: 100%;
  overflow: auto;
}

.box {
  width: 100%;
  height: 800px;
}
.box:nth-child(1) {
  background-color: #8fffe4;
}
.box:nth-child(2) {
  background-color: #e0ceff;
}
.box:nth-child(3) {
  background-color: #ffdd9c;
}
</style>
