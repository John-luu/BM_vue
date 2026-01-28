<template>
  <div>
    <router-view style="margin-bottom: 50px; overflow: auto"></router-view>
    <TabLayout>
      <div
        v-for="(tab, index) in tabs"
        class="tab-item"
        :class="{ active: index === active }"
        @click="go(tab, index)"
      >
        {{ tab.name }}
      </div>
    </TabLayout>
  </div>
</template>

<script>
import TabLayout from "@/components/TabLayout";

export default {
  name: "Main",
  components: { TabLayout },

  data() {
    return {
      active: 0,
      tabs: [
        { name: "论坛", url: "/student/forum" },
        { name: "座位", url: "/student/seat/reservation" },
        { name: "我的", url: "/student/mine" },
      ],
    };
  },

  methods: {
    getCurIndex() {
      const path = this.$route.path;
      if (path.includes("/student/forum")) return 0;
      if (path.includes("/student/seat")) return 1;
      if (path.includes("/student/mine")) return 2;
      return 0;
    },

    go(tab, index) {
      this.active = index;
      if (this.$route.path !== tab.url) {
        this.$router.replace(tab.url);
      }
    },
  },

  created() {
    // 页面首次加载时同步一次
    this.active = this.getCurIndex();
  },

  watch: {
    "$route.path"() {
      // 路由变化时自动同步 tab
      this.active = this.getCurIndex();
    },
  },
};
</script>
<style scoped>
.tab-item {
  flex: 1;
  text-align: center;

  font-size: 14px;
  color: #666;

  padding: 8px 0;
  border-radius: 12px;

  transition: all 0.25s ease;
}
.tab-item.active {
  color: #409eff;
  font-weight: 600;
  background: rgba(64, 158, 255, 0.12);
}
.tab-item.active::before {
  content: "";
  position: absolute;
  top: 6px;
  left: 50%;
  transform: translateX(-50%);
  width: 20px;
  height: 3px;
  background: #409eff;
  border-radius: 2px;
}
</style>
