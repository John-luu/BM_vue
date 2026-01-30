<template>
  <div class="statistics-container">
    <div class="header-section">
      <div class="title-group">
        <h2 class="main-title">系统统计分析</h2>
        <p class="sub-title">实时数据监控与统计概览</p>
      </div>
      <div class="time-indicator">
        <span class="time-label">更新时间</span>
        <span class="time-value">{{ currentTime }}</span>
        <div class="refresh-indicator">
          <div class="pulse-dot"></div>
          <span>实时同步</span>
        </div>
      </div>
    </div>

    <!-- 图表区域 -->
    <div class="charts-section">
      <div class="charts-grid">
        <div class="chart-card">
          <UserCounterListCharts v-if="userList" :list="userList" />
        </div>
        <div class="chart-card">
          <TimeListCharts v-if="timeList" :list="timeList" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import UserCounterListCharts from "@/components/UserCounterListCharts";
import request from "@/req";
import TimeListCharts from "@/components/TimeListCharts";

export default {
  name: "Statistics",
  components: { TimeListCharts, UserCounterListCharts },
  data() {
    return {
      userList: null,
      timeList: null,
      currentTime: this.formatTime(new Date()),
    };
  },
  created() {
    request.get("/admin/getStatistics").then((res) => {
      this.userList = res.userCounter;
      this.timeList = res.timeList;
    });

    // 更新时间显示
    setInterval(() => {
      this.currentTime = this.formatTime(new Date());
    }, 1000);
  },
  methods: {
    formatTime(date) {
      const hours = date.getHours().toString().padStart(2, "0");
      const minutes = date.getMinutes().toString().padStart(2, "0");
      const seconds = date.getSeconds().toString().padStart(2, "0");
      return `${hours}:${minutes}:${seconds}`;
    },
  },
};
</script>

<style scoped>
.statistics-container {
  padding: 24px;
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  min-height: 100vh;
}

/* 头部区域 */
.header-section {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 32px;
  padding: 24px;
  background: white;
  border-radius: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  border: 1px solid rgba(226, 232, 240, 0.8);
}

.title-group {
  flex: 1;
}

.main-title {
  font-size: 28px;
  font-weight: 700;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin: 0 0 8px 0;
  letter-spacing: -0.5px;
}

.sub-title {
  font-size: 14px;
  color: #64748b;
  margin: 0;
  font-weight: 500;
}

.time-indicator {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 8px;
}

.time-label {
  font-size: 12px;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: 600;
}

.time-value {
  font-size: 24px;
  font-weight: 700;
  color: #334155;
  font-family: "Courier New", monospace;
  background: linear-gradient(135deg, #1e293b 0%, #475569 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.refresh-indicator {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 12px;
  background: rgba(74, 222, 128, 0.1);
  border-radius: 20px;
  border: 1px solid rgba(74, 222, 128, 0.3);
}

.pulse-dot {
  width: 8px;
  height: 8px;
  background: #4ade80;
  border-radius: 50%;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.3;
  }
  100% {
    opacity: 1;
  }
}

.refresh-indicator span {
  font-size: 12px;
  color: #16a34a;
  font-weight: 600;
}

/* 图表区域 */
.charts-section {
  margin-top: 32px;
}

.charts-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
}

.chart-card {
  background: white;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  border: 1px solid rgba(226, 232, 240, 0.8);
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .charts-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 768px) {
  .header-section {
    flex-direction: column;
    gap: 16px;
  }

  .time-indicator {
    align-items: flex-start;
  }
}
</style>
