<template>
  <div class="student-detail-page">
    <!-- 顶部导航栏 -->
    <navbar title="学生详细信息"></navbar>

    <!-- 背景装饰 -->
    <div class="background-decoration">
      <div class="bg-circle bg-circle-1"></div>
      <div class="bg-circle bg-circle-2"></div>
      <div class="bg-circle bg-circle-3"></div>
    </div>

    <!-- 主要内容区域 -->
    <div class="content-container">
      <!-- 用户头像卡片 -->
      <div class="user-profile-card">
        <div class="avatar-container">
          <div class="avatar-circle">
            <span class="avatar-text">{{ getAvatarText(item.username) }}</span>
          </div>
          <div class="user-badge" :class="getScoreBadgeClass(item.uScore)">
            <span class="badge-text">{{ getScoreLevel(item.uScore) }}</span>
          </div>
        </div>

        <div class="user-info-header">
          <h1 class="user-name">{{ item.username }}</h1>
          <div class="user-id">
            <i class="el-icon-user-solid"></i>
            ID: {{ item.number }}
          </div>
        </div>
      </div>

      <!-- 详细信息卡片 -->
      <el-card class="detail-card" shadow="always">
        <div class="card-content">
          <!-- 基本信息区块 -->
          <div class="info-section">
            <h2 class="section-title">
              <i class="el-icon-s-custom"></i>
              基本信息
            </h2>

            <div class="info-grid">
              <div class="info-item">
                <div class="info-label">
                  <i class="el-icon-user"></i>
                  <span>学工号</span>
                </div>
                <div class="info-value code-text">
                  {{ item.number }}
                </div>
              </div>

              <div class="info-item">
                <div class="info-label">
                  <i class="el-icon-star-on"></i>
                  <span>信用等级</span>
                </div>
                <div class="info-value score-level">
                  {{ getScoreLevel(item.uScore) }}
                  <span class="score-badge" :class="getScoreColor(item.uScore)">
                    {{ item.uScore }}分
                  </span>
                </div>
              </div>
            </div>
          </div>

          <!-- 信用分详细区块 -->
          <div class="score-section">
            <h2 class="section-title">
              <i class="el-icon-data-analysis"></i>
              信用分详情
            </h2>

            <div class="score-display">
              <div class="score-circle-container">
                <div class="score-circle">
                  <div class="score-number">{{ item.uScore }}</div>
                  <div class="score-label">信用分</div>
                </div>
                <div class="score-progress">
                  <el-progress
                    :percentage="getScorePercentage(item.uScore)"
                    :color="getScoreProgressColor(item.uScore)"
                    :show-text="false"
                    stroke-width="8"
                  ></el-progress>
                  <div class="progress-labels">
                    <span>0</span>
                    <span>100</span>
                  </div>
                </div>
              </div>

              <div class="score-description">
                <div class="description-item" v-if="item.uScore >= 90">
                  <i class="el-icon-circle-check" style="color: #67c23a"></i>
                  <span>优秀信用等级，享受预约优先权</span>
                </div>
                <div class="description-item" v-else-if="item.uScore >= 70">
                  <i class="el-icon-warning-outline" style="color: #e6a23c"></i>
                  <span>良好信用等级，请继续保持</span>
                </div>
                <div class="description-item" v-else>
                  <i class="el-icon-circle-close" style="color: #f56c6c"></i>
                  <span>信用等级较低，请注意使用规范</span>
                </div>
              </div>
            </div>
          </div>

          <!-- 状态区块 -->
          <div class="status-section" v-if="item.state !== undefined">
            <h2 class="section-title">
              <i class="el-icon-office-building"></i>
              当前状态
            </h2>
            <div class="status-display">
              <div class="status-badge" :class="getStatusClass(item.state)">
                <span class="status-icon"></span>
                <span class="status-text">{{ getStatusText(item.state) }}</span>
              </div>
              <div class="status-time" v-if="item.startTime">
                <i class="el-icon-time"></i>
                开始时间：{{ formatTime(item.startTime) }}
              </div>
            </div>
          </div>
        </div>
      </el-card>

      <!-- 操作按钮 -->
      <div class="action-buttons">
        <el-button
          class="back-button"
          type="primary"
          plain
          @click="$router.back()"
        >
          <i class="el-icon-back"></i>
          返回列表
        </el-button>
      </div>
    </div>
  </div>
</template>

<script>
import Navbar from "@/components/navbar";

export default {
  name: "StudentMess",
  components: { Navbar },
  data() {
    return {
      item: null,
    };
  },
  methods: {
    getAvatarText(username) {
      if (!username || username.length === 0) return "?";
      return username.substring(0, 1).toUpperCase();
    },
    getScoreLevel(score) {
      if (score >= 90) return "优秀";
      if (score >= 80) return "良好";
      if (score >= 70) return "中等";
      if (score >= 60) return "及格";
      return "较差";
    },
    getScoreBadgeClass(score) {
      if (score >= 90) return "badge-excellent";
      if (score >= 80) return "badge-good";
      if (score >= 70) return "badge-medium";
      if (score >= 60) return "badge-pass";
      return "badge-poor";
    },
    getScoreColor(score) {
      if (score >= 90) return "score-excellent";
      if (score >= 80) return "score-good";
      if (score >= 70) return "score-medium";
      if (score >= 60) return "score-pass";
      return "score-poor";
    },
    getScorePercentage(score) {
      return Math.min(score, 100);
    },
    getScoreProgressColor(score) {
      if (score >= 90) return "#67C23A";
      if (score >= 80) return "#409EFF";
      if (score >= 70) return "#E6A23C";
      if (score >= 60) return "#FF9E52";
      return "#F56C6C";
    },
    getStatusText(state) {
      const statusMap = {
        0: "待签到",
        1: "使用中",
        2: "预约未签到",
        3: "暂离",
        4: "暂离超时未回",
        5: "使用完成",
      };
      return statusMap[state] || "未知状态";
    },
    getStatusClass(state) {
      const classMap = {
        0: "status-pending",
        1: "status-active",
        2: "status-no-checkin",
        3: "status-temp-leave",
        4: "status-timeout",
        5: "status-completed",
      };
      return classMap[state] || "status-default";
    },
    formatTime(time) {
      if (!time) return "";
      const date = new Date(time);
      return `${date.getFullYear()}-${(date.getMonth() + 1)
        .toString()
        .padStart(2, "0")}-${date.getDate().toString().padStart(2, "0")} ${date
        .getHours()
        .toString()
        .padStart(2, "0")}:${date.getMinutes().toString().padStart(2, "0")}`;
    },
  },
  created() {
    this.item = JSON.parse(localStorage.getItem("student"));
    console.log(this.item);
  },
};
</script>

<style scoped>
.student-detail-page {
  min-height: 100vh;
  background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%);
  position: relative;
  overflow-x: hidden;
}

/* 背景装饰 */
.background-decoration {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
  z-index: 0;
}

.bg-circle {
  position: absolute;
  border-radius: 50%;
  background: linear-gradient(
    135deg,
    rgba(64, 158, 255, 0.05),
    rgba(103, 194, 58, 0.05)
  );
  animation: float 6s ease-in-out infinite;
}

.bg-circle-1 {
  width: 300px;
  height: 300px;
  top: -150px;
  right: -150px;
  animation-delay: 0s;
}

.bg-circle-2 {
  width: 200px;
  height: 200px;
  bottom: 50px;
  left: -100px;
  animation-delay: 2s;
}

.bg-circle-3 {
  width: 150px;
  height: 150px;
  top: 40%;
  right: 20%;
  animation-delay: 4s;
}

@keyframes float {
  0%,
  100% {
    transform: translateY(0) rotate(0deg);
  }
  50% {
    transform: translateY(-20px) rotate(180deg);
  }
}

.content-container {
  position: relative;
  z-index: 1;
  padding: 20px;
  max-width: 800px;
  margin: 0 auto;
}

/* 用户头像卡片 */
.user-profile-card {
  background: white;
  border-radius: 24px;
  padding: 30px;
  margin-bottom: 24px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.08);
  display: flex;
  align-items: center;
  gap: 24px;
  animation: slideInUp 0.6s ease-out;
}

.avatar-container {
  position: relative;
}

.avatar-circle {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  background: linear-gradient(135deg, #409eff, #46a3ff);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8px 32px rgba(64, 158, 255, 0.3);
  position: relative;
}

.avatar-circle::after {
  content: "";
  position: absolute;
  top: -4px;
  left: -4px;
  right: -4px;
  bottom: -4px;
  border-radius: 50%;
  background: linear-gradient(135deg, #409eff, #67c23a);
  opacity: 0.3;
  z-index: -1;
}

.avatar-text {
  font-size: 36px;
  font-weight: bold;
  color: white;
  text-transform: uppercase;
}

.user-badge {
  position: absolute;
  bottom: -8px;
  right: -8px;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 600;
  color: white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.badge-excellent {
  background: linear-gradient(135deg, #67c23a, #85ce61);
}
.badge-good {
  background: linear-gradient(135deg, #409eff, #66b1ff);
}
.badge-medium {
  background: linear-gradient(135deg, #e6a23c, #ebb563);
}
.badge-pass {
  background: linear-gradient(135deg, #ff9e52, #ffb980);
}
.badge-poor {
  background: linear-gradient(135deg, #f56c6c, #f78989);
}

.badge-text {
  letter-spacing: 0.5px;
}

.user-info-header {
  flex: 1;
}

.user-name {
  font-size: 28px;
  font-weight: 700;
  color: #1a1a1a;
  margin: 0 0 8px 0;
  background: linear-gradient(135deg, #1a1a1a, #409eff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.user-id {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #718096;
  font-size: 16px;
}

.user-id i {
  color: #409eff;
}

/* 详细信息卡片 */
.detail-card {
  border-radius: 24px !important;
  border: none;
  overflow: hidden;
  animation: slideInUp 0.8s ease-out 0.2s both;
}

.card-content {
  padding: 30px;
}

/* 区块样式 */
.info-section,
.score-section,
.status-section {
  margin-bottom: 32px;
  padding-bottom: 24px;
  border-bottom: 1px solid #f0f2f5;
}

.info-section:last-child,
.score-section:last-child,
.status-section:last-child {
  border-bottom: none;
  margin-bottom: 0;
  padding-bottom: 0;
}

.section-title {
  display: flex;
  align-items: center;
  gap: 12px;
  color: #2d3748;
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 20px;
}

.section-title i {
  font-size: 20px;
}

/* 信息网格 */
.info-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

.info-item {
  padding: 20px;
  background: #f8fafc;
  border-radius: 16px;
  border: 1px solid #edf2f7;
  transition: all 0.3s ease;
}

.info-item:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.08);
  border-color: #409eff;
}

.info-label {
  display: flex;
  align-items: center;
  gap: 10px;
  color: #718096;
  font-size: 14px;
  margin-bottom: 12px;
}

.info-label i {
  font-size: 18px;
  color: #409eff;
}

.info-value {
  font-size: 18px;
  font-weight: 600;
  color: #1a1a1a;
}

.code-text {
  font-family: "Courier New", monospace;
  letter-spacing: 1px;
  color: #409eff;
  font-size: 20px;
}

.score-level {
  display: flex;
  align-items: center;
  gap: 12px;
}

.score-badge {
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 14px;
  font-weight: 500;
  color: white;
}

.score-excellent {
  background: linear-gradient(135deg, #67c23a, #85ce61);
}
.score-good {
  background: linear-gradient(135deg, #409eff, #66b1ff);
}
.score-medium {
  background: linear-gradient(135deg, #e6a23c, #ebb563);
}
.score-pass {
  background: linear-gradient(135deg, #ff9e52, #ffb980);
}
.score-poor {
  background: linear-gradient(135deg, #f56c6c, #f78989);
}

/* 信用分展示 */
.score-display {
  display: flex;
  align-items: center;
  gap: 40px;
}

.score-circle-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.score-circle {
  width: 140px;
  height: 140px;
  border-radius: 50%;
  background: linear-gradient(135deg, #f8fafc, #e2e8f0);
  border: 8px solid;
  border-color: var(--score-color, #409eff);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
}

.score-circle::after {
  content: "";
  position: absolute;
  top: -12px;
  left: -12px;
  right: -12px;
  bottom: -12px;
  border-radius: 50%;
  border: 2px dashed;
  border-color: var(--score-color, #409eff);
  opacity: 0.3;
}

.score-number {
  font-size: 36px;
  font-weight: 700;
  color: #1a1a1a;
}

.score-label {
  font-size: 14px;
  color: #718096;
  margin-top: 4px;
}

.score-progress {
  width: 140px;
}

.progress-labels {
  display: flex;
  justify-content: space-between;
  margin-top: 8px;
  color: #a0aec0;
  font-size: 12px;
}

.score-description {
  flex: 1;
}

.description-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  background: #f8fafc;
  border-radius: 12px;
  margin-bottom: 12px;
  border-left: 4px solid;
}

.description-item:last-child {
  margin-bottom: 0;
}

.description-item i {
  font-size: 18px;
}

/* 状态展示 */
.status-display {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.status-badge {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 12px 24px;
  border-radius: 50px;
  font-weight: 500;
  max-width: fit-content;
}

.status-pending {
  background: rgba(230, 162, 60, 0.1);
  color: #e6a23c;
  border: 1px solid rgba(230, 162, 60, 0.2);
}
.status-active {
  background: rgba(103, 194, 58, 0.1);
  color: #67c23a;
  border: 1px solid rgba(103, 194, 58, 0.2);
}
.status-no-checkin {
  background: rgba(245, 108, 108, 0.1);
  color: #f56c6c;
  border: 1px solid rgba(245, 108, 108, 0.2);
}
.status-temp-leave {
  background: rgba(144, 147, 153, 0.1);
  color: #909399;
  border: 1px solid rgba(144, 147, 153, 0.2);
}
.status-timeout {
  background: rgba(255, 158, 82, 0.1);
  color: #ff9e52;
  border: 1px solid rgba(255, 158, 82, 0.2);
}
.status-completed {
  background: rgba(64, 158, 255, 0.1);
  color: #409eff;
  border: 1px solid rgba(64, 158, 255, 0.2);
}

.status-icon {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: currentColor;
}

.status-time {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #718096;
  font-size: 14px;
}

.status-time i {
  color: #409eff;
}

/* 操作按钮 */
.action-buttons {
  margin-top: 32px;
  text-align: center;
  animation: fadeIn 0.8s ease-out 0.4s both;
}

.back-button {
  padding: 14px 36px;
  border-radius: 50px;
  font-size: 16px;
  font-weight: 500;
  letter-spacing: 0.5px;
  transition: all 0.3s ease;
}

.back-button:hover {
  transform: translateX(-4px);
  box-shadow: 0 8px 24px rgba(64, 158, 255, 0.3);
}

/* 动画 */
@keyframes slideInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* 响应式设计 */
@media (max-width: 768px) {
  .content-container {
    padding: 16px;
  }

  .user-profile-card {
    flex-direction: column;
    text-align: center;
    padding: 24px;
  }

  .user-info-header {
    text-align: center;
  }

  .score-display {
    flex-direction: column;
    gap: 24px;
  }

  .info-grid {
    grid-template-columns: 1fr;
  }

  .avatar-circle {
    width: 80px;
    height: 80px;
  }

  .avatar-text {
    font-size: 28px;
  }

  .user-name {
    font-size: 24px;
  }
}
</style>
