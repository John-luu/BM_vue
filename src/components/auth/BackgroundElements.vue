<template>
  <div class="background-elements">
    <div class="circle circle-1"></div>
    <div class="circle circle-2"></div>
    <div class="circle circle-3"></div>
    <div
      class="floating-element element-1"
      :style="{ backgroundColor: elementColor }"
    ></div>
    <div
      class="floating-element element-2"
      :style="{ backgroundColor: elementColor }"
    ></div>
    <div
      class="floating-element element-3"
      :style="{ backgroundColor: elementColor }"
    ></div>
  </div>
</template>

<script>
export default {
  name: "BackgroundElements",
  props: {
    // 可以自定义背景元素的颜色
    circleColor: {
      type: String,
      default: "rgba(255, 255, 255, 0.1)",
    },
    elementColor: {
      type: String,
      default: "rgba(255, 255, 255, 0.15)",
    },
    // 可以自定义动画速度
    animationDuration: {
      type: Number,
      default: 6,
    },
  },
};
</script>

<style scoped>
.background-elements {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  z-index: 0;
  pointer-events: none; /* 防止干扰点击事件 */
}

.circle {
  position: absolute;
  border-radius: 50%;
  background: v-bind(circleColor);
  animation: float v-bind('animationDuration + "s"') ease-in-out infinite;
}

.circle-1 {
  width: 300px;
  height: 300px;
  top: 10%;
  left: 5%;
  animation-delay: 0s;
}

.circle-2 {
  width: 200px;
  height: 200px;
  bottom: 15%;
  right: 5%;
  animation-delay: 2s;
}

.circle-3 {
  width: 150px;
  height: 150px;
  top: 60%;
  left: 15%;
  animation-delay: 4s;
}

.floating-element {
  position: absolute;
  width: 40px;
  height: 60px;
  background: v-bind(elementColor);
  border-radius: 4px;
  animation: float 8s ease-in-out infinite;
}

.floating-element:after {
  content: "";
  position: absolute;
  width: 100%;
  height: 100%;
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 4px;
  top: 5px;
  left: 5px;
}

.element-1 {
  top: 20%;
  right: 15%;
  animation-delay: 1s;
}

.element-2 {
  top: 70%;
  left: 15%;
  animation-delay: 3s;
}

.element-3 {
  bottom: 20%;
  right: 25%;
  animation-delay: 5s;
}

@keyframes float {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-20px);
  }
}

/* 响应式调整 */
@media (max-width: 768px) {
  .circle-1 {
    width: 200px;
    height: 200px;
  }

  .circle-2 {
    width: 150px;
    height: 150px;
  }

  .circle-3 {
    width: 100px;
    height: 100px;
  }

  .floating-element {
    width: 30px;
    height: 45px;
  }
}
</style>
