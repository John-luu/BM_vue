<template>
  <div class="tabLayout" ref="tabLayout" @click="verifyChild($event)">
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: "TabLayout",
  data() {
    return {
      current: 1,
      lastItemIndex: null,
    };
  },
  methods: {
    verifyChild(e) {
      let tabLayout = this.$refs.tabLayout;
      for (let i = 0; i < tabLayout.children.length; i++) {
        let child = tabLayout.children[i];
        if (child.contains(e.target)) {
          this.toggleTab(i, child);
        }
      }
    },
    toggleTab(index, node) {
      this.lastItemIndex = this.current;
      this.$emit("changeTab", index);
      let tabLayout = this.$refs.tabLayout;
      let winWidth = window.innerWidth;
      if (node) {
        let liLeft = node.offsetLeft,
          liWidth = node.offsetWidth,
          liCenter = (winWidth - liWidth) / 2,
          liTarget = liLeft - liCenter;

        tabLayout.scrollTo({ left: liTarget, top: 0, behavior: "smooth" });
        // tabLayout.scrollLeft = liTarget;
      }
    },
  },
};
</script>
<style scoped>
.tabLayout {
  position: fixed;
  bottom: 10px;
  left: 12px;
  right: 12px;

  height: 56px;
  background: #fff;

  display: flex;
  align-items: center;
  justify-content: space-around;

  border-radius: 16px;

  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.12);
  z-index: 99999;

  overflow: hidden; /* 关键 */
}

.tabLayout::-webkit-scrollbar {
  display: none;
}

.tabLayout > * {
  height: 44px;
  line-height: 44px;
  display: inline-block;
  flex: 1;
  text-align: center;
  justify-content: space-evenly;
}
</style>
