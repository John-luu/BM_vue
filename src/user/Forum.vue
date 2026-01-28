<template>
  <div>
    <van-swipe
      class="my-swipe"
      vertical
      :autoplay="4000"
      :duration="800"
      :loop="true"
      :show-indicators="false"
    >
      <van-swipe-item
        v-for="item in announceRows"
        :key="item.id"
        @click="openAnnounce(item)"
      >
        <div class="announce-card">
          <!-- NEW 角标 -->
          <span v-if="isNewAnnounce(item)" class="new-badge">NEW</span>

          <div class="an-title">{{ item.title }}</div>
          <div class="an-content">{{ item.content }}</div>
        </div>
      </van-swipe-item>
    </van-swipe>

    <el-divider content-position="right"
      ><van-tag
        @click="$router.push('/PushArticle')"
        plain
        type="primary"
        size="medium"
        >发帖</van-tag
      ></el-divider
    >
    <div class="forum-item" v-for="item in rows" @click="forumInfo(item)">
      <div class="title">{{ item.title }}</div>
      <div class="content">{{ item.content }}</div>
      <div class="time">{{ $formatTime(item.datetime) }}</div>
    </div>
    <!-- ✅ 公告弹窗 -->
    <van-popup
      v-model="showAnnounce"
      round
      position="bottom"
      :style="{ height: '60%' }"
      closeable
      @close="onAnnounceClose"
    >
      <div class="announce-popup" v-if="currentAnnounce">
        <div class="new-tip" v-if="isNewAnnounce(currentAnnounce)">
          🔔 这是一个新的公告
        </div>

        <div class="popup-title">
          {{ currentAnnounce.title }}
        </div>
        <div class="popup-content">
          {{ currentAnnounce.content }}
        </div>
      </div>
    </van-popup>
  </div>
</template>

<script>
import request from "@/req";

export default {
  name: "Forum",
  data() {
    return {
      rows: null,
      announceRows: null,
      showAnnounce: false,
      currentAnnounce: null,
    };
  },
  methods: {
    forumInfo(item) {
      localStorage.setItem("forum", JSON.stringify(item));
      this.$router.push("/ForumInfo");
    },
    // ✅ 打开公告弹窗
    openAnnounce(item) {
      this.currentAnnounce = item;
      this.showAnnounce = true;
    },
    isNewAnnounce(item) {
      if (!item || !item.datetime) return false;

      const now = Date.now();
      const oneDay = 24 * 60 * 60 * 1000;

      return now - item.datetime < oneDay;
    },
    // ✅ 关闭公告
    onAnnounceClose() {
      this.showAnnounce = false;

      if (this.currentAnnounce) {
        localStorage.setItem("lastAnnounceId", this.currentAnnounce.id);
      }
    },
  },
  created() {
    request.get("/forum/getArticle").then((res) => {
      this.rows = res.rows;
    });

    request.get("/admin/getAnnounce").then((res) => {
      this.announceRows = res.rows;

      // ✅ 自动弹出最新公告（只弹一次）
      if (this.announceRows && this.announceRows.length > 0) {
        const latest = this.announceRows[0]; // 最新公告
        const lastId = localStorage.getItem("lastAnnounceId");

        if (String(latest.id) !== lastId) {
          this.currentAnnounce = latest;
          this.showAnnounce = true;
        }
      }
    });
  },
};
</script>

<style scoped>
.my-swipe {
  height: 140px;
  background: linear-gradient(135deg, #5aa9ff, #6fd6ff);
  border-radius: 12px;
  margin: 12px;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.15);
}
.new-badge {
  position: absolute;
  top: 10px;
  right: 12px;
  background: linear-gradient(135deg, #ff5f6d, #ff9966);
  color: #fff;
  font-size: 11px;
  padding: 2px 6px;
  border-radius: 8px;
  font-weight: 600;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
}

.announce-card {
  height: 100%;
  padding: 16px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.an-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 6px;
  color: #fff;
}

.an-content {
  font-size: 14px;
  line-height: 1.5;
  color: rgba(255, 255, 255, 0.9);
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}
.forum-item {
  background: #fff;
  margin: 12px;
  padding: 12px;
  border-radius: 10px;

  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.forum-item + .forum-item {
  margin-top: 10px;
}

.title {
  font-size: 15px;
  font-weight: 600;
  margin-bottom: 6px;
  border-left: 4px solid #409eff;
  padding-left: 8px;
}

.content {
  font-size: 14px;
  color: #555;
  line-height: 1.5;
}

.time {
  font-size: 12px;
  color: #999;
  text-align: right;
  margin-top: 6px;
}
/* ===== 公告弹窗样式（重点） ===== */
.announce-popup {
  padding: 20px;
  height: 100%;
  overflow-y: auto;
}

.popup-title {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 12px;
  color: #333;
}

.popup-content {
  font-size: 15px;
  line-height: 1.7;
  color: #555;
  white-space: pre-wrap;
}
.new-tip {
  background: #f0f9ff;
  color: #409eff;
  padding: 6px 10px;
  border-radius: 6px;
  font-size: 13px;
  margin-bottom: 12px;
  text-align: center;
}
</style>
