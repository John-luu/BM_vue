<template>
  <div class="tabLayout" ref="tabLayout" @click="verifyChild($event)">
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: "LeftLayout",
  props: {
    tabs: null,
  },
  data() {
    return {
      current: 1,
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
      if (this.$route.path !== this.tabs[index].url) {
        this.$router.replace(this.tabs[index].url);
      }
      this.$emit("changeTab", index);
      // let tabLayout = this.$refs.tabLayout;
      // let winWidth = window.innerWidth;
      // if (node) {
      //   let liLeft = node.offsetLeft,
      //       liWidth = node.offsetWidth,
      //       liCenter = (winWidth - liWidth) / 2,
      //       liTarget = liLeft - liCenter;
      //
      //   tabLayout.scrollTo({left: liTarget, top: 0, behavior: 'smooth'});
      //   // tabLayout.scrollLeft = liTarget;
      //
      // }
    },
  },
  created() {
    for (let i = 0; i < this.tabs.length; i++) {
      if (this.$route.path === this.tabs[i].url) {
        this.$emit("changeTab", i);
      }
    }
  },
};
</script>
<style scoped>
.tabLayout {
  position: fixed;
  top: 0;
  left: 0;
  width: 15vw;
  height: 100vh;
  background: #111827;
  padding-top: 80px;
  box-sizing: border-box;
}

/* 单个菜单项 */
.tabLayout > * {
  height: 48px;
  margin: 8px 16px;
  border-radius: 10px;

  display: flex;
  align-items: center;
  justify-content: center;

  color: #cbd5e1;
  font-size: 16px;
  font-weight: 500;

  cursor: pointer;
  transition: all 0.25s ease;

  position: relative; /* ✅ 必须加 */
}

/* hover 状态 */
.tabLayout > *:hover {
  background: rgba(255, 255, 255, 0.05);
}

/* 当前选中（你 v-for 里已经在控制） */
.tabLayout > .active {
  background: rgba(255, 255, 255, 0.08);
  color: #ffffff;
  font-weight: 600;
}
.tabLayout > .active::before {
  content: "";
  position: absolute;
  left: -8px; /* 微微露出 */
  top: 20%;
  width: 3px;
  height: 60%;
  background: #94a3b8; /* 灰蓝，不抢戏 */
  border-radius: 4px;
}
.tabLayout > * {
  letter-spacing: 1px;
}
</style>
