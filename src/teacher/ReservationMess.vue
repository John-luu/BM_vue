<template>
  <div class="reservation-container">
    <div
      v-for="(item, index) in rows"
      :key="index"
      @click="clickItem(item)"
      class="card-wrapper"
    >
      <el-card
        class="reservation-card"
        :class="{
          'active-card': item.state === 1,
          'pending-card': item.state === 0 || item.state === 2,
        }"
        shadow="never"
      >
        <div class="card-content">
          <!-- 左侧用户信息 -->
          <div class="user-section">
            <div class="user-avatar-wrapper">
              <div class="user-avatar">{{ getAvatarText(item.username) }}</div>
              <div
                class="state-indicator"
                :class="getStateClass(item.state)"
              ></div>
            </div>
            <div class="user-info">
              <div class="user-header">
                <span class="user-name">{{ item.username }}</span>
                <span class="state-badge" :class="getStateClass(item.state)">
                  {{ getState(item.state) }}
                </span>
              </div>
              <div class="user-meta">
                <span class="seat-info">
                  <i class="el-icon-location-outline"></i>
                  {{ item.subName }}{{ item.row }}{{ item.column }}
                </span>
                <span class="date-info">
                  <i class="el-icon-date"></i>
                  {{ getTime(item.startTime) }}
                </span>
              </div>
            </div>
          </div>

          <!-- 右侧时间信息 -->
          <div class="time-section">
            <div class="time-slider-wrapper">
              <TimeSlider
                :disable="true"
                :start-time="item.startTime"
                :end-time="item.endTime"
                class="compact-slider"
              ></TimeSlider>
            </div>
          </div>
        </div>
      </el-card>
    </div>
  </div>
</template>

<script>
import request from "@/req";
import TimeSlider from "@/components/TimeSlider";

export default {
  name: "ReservationMess",
  components: { TimeSlider },
  props: {},
  data() {
    return {
      rows: null,
    };
  },
  methods: {
    clickItem(item) {
      localStorage.setItem("student", JSON.stringify(item));
      this.$router.push("/LookStudentMess");
    },
    getState(state) {
      switch (state) {
        case 0:
          return "待签到";
        case 1:
          return "使用中";
        case 2:
          return "未签到";
        case 3:
          return "暂离";
        case 4:
          return "超时";
        case 5:
          return "完成";
        default:
          return "未知";
      }
    },
    getStateClass(state) {
      switch (state) {
        case 0:
          return "state-pending";
        case 1:
          return "state-active";
        case 2:
          return "state-no-checkin";
        case 3:
          return "state-temp-leave";
        case 4:
          return "state-timeout";
        case 5:
          return "state-completed";
        default:
          return "state-default";
      }
    },
    getTime(d) {
      let date = new Date(d);
      const month = date.getMonth() + 1;
      const day = date.getDate();
      return `${month}月${day}日`;
    },
    formatDuration(start, end) {
      const startTime = new Date(start);
      const endTime = new Date(end);

      const format = (date) => {
        const hours = date.getHours().toString().padStart(2, "0");
        const minutes = date.getMinutes().toString().padStart(2, "0");
        return `${hours}:${minutes}`;
      };

      return `${format(startTime)}-${format(endTime)}`;
    },
    getAvatarText(username) {
      if (!username || username.length === 0) return "?";
      return username.substring(0, 1).toUpperCase();
    },
  },
  created() {
    request.get("/teacher/getReservation").then((res) => {
      this.rows = res.rows;
    });
  },
};
</script>

<style scoped>
.reservation-container {
  padding: 12px 16px;
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  min-height: 100vh;
}

.card-wrapper {
  margin-bottom: 12px;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
}

.card-wrapper:hover {
  transform: translateY(-2px);
  filter: drop-shadow(0 4px 12px rgba(0, 0, 0, 0.08));
}

.reservation-card {
  border-radius: 14px !important;
  border: none !important;
  background: white;
  transition: all 0.3s ease;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.reservation-card::before {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 3px;
  background: linear-gradient(180deg, var(--state-color, #409eff), transparent);
  opacity: 0.8;
  transition: all 0.3s ease;
}

.active-card::before {
  --state-color: #67c23a;
}

.pending-card::before {
  --state-color: #e6a23c;
}

.card-content {
  display: flex;
  align-items: stretch;
  padding: 16px;
  gap: 20px;
}

/* 左侧用户信息 */
.user-section {
  display: flex;
  align-items: flex-start;
  gap: 14px;
  flex: 0 0 160px;
  min-width: 0;
}

.user-avatar-wrapper {
  position: relative;
  flex-shrink: 0;
}

.user-avatar {
  width: 40px;
  height: 40px;
  border-radius: 12px;
  background: linear-gradient(135deg, #0ea5e9 0%, #38bdf8 100%);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  font-size: 16px;
  box-shadow: 0 4px 10px rgba(102, 126, 234, 0.25);
}

.state-indicator {
  position: absolute;
  bottom: -4px;
  right: -4px;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: 2px solid white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.state-pending .state-indicator {
  background: #e6a23c;
}
.state-active .state-indicator {
  background: #67c23a;
}
.state-no-checkin .state-indicator {
  background: #f56c6c;
}
.state-temp-leave .state-indicator {
  background: #909399;
}
.state-timeout .state-indicator {
  background: #ff9e52;
}
.state-completed .state-indicator {
  background: #409eff;
}

.user-info {
  flex: 1;
  min-width: 0;
}

.user-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
}

.user-name {
  font-size: 16px;
  font-weight: 600;
  color: #1e293b;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  flex: 1;
  min-width: 0;
}

.state-badge {
  padding: 3px 8px;
  border-radius: 8px;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.3px;
  white-space: nowrap;
  flex-shrink: 0;
}

.state-pending .state-badge {
  background: rgba(230, 162, 60, 0.12);
  color: #d48806;
  border: 1px solid rgba(230, 162, 60, 0.2);
}
.state-active .state-badge {
  background: rgba(103, 194, 58, 0.12);
  color: #389e0d;
  border: 1px solid rgba(103, 194, 58, 0.2);
}
.state-no-checkin .state-badge {
  background: rgba(245, 108, 108, 0.12);
  color: #cf1322;
  border: 1px solid rgba(245, 108, 108, 0.2);
}
.state-temp-leave .state-badge {
  background: rgba(144, 147, 153, 0.12);
  color: #595959;
  border: 1px solid rgba(144, 147, 153, 0.2);
}
.state-timeout .state-badge {
  background: rgba(255, 158, 82, 0.12);
  color: #d46b08;
  border: 1px solid rgba(255, 158, 82, 0.2);
}
.state-completed .state-badge {
  background: rgba(64, 158, 255, 0.12);
  color: #096dd9;
  border: 1px solid rgba(64, 158, 255, 0.2);
}

.user-meta {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.seat-info,
.date-info {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  color: #64748b;
  white-space: nowrap;
}

.seat-info i,
.date-info i {
  font-size: 12px;
  opacity: 0.7;
}

.seat-info {
  font-weight: 500;
  color: #475569;
  font-size: 14px;
}

/* 右侧时间信息 */
.time-section {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  gap: 8px;
}

.time-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 4px;
}

.time-label {
  font-size: 13px;
  font-weight: 500;
  color: #64748b;
  display: flex;
  align-items: center;
  gap: 6px;
}

.time-label::before {
  content: "";
  display: inline-block;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: #94a3b8;
}

.duration {
  font-size: 14px;
  font-weight: 600;
  color: #1e293b;
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  letter-spacing: -0.3px;
}

.time-slider-wrapper {
  position: relative;
  padding: 8px 0;
}

.compact-slider {
  --slider-height: 6px;
  --slider-radius: 3px;
}

.compact-slider::v-deep .slider-track {
  border-radius: 3px;
  height: 6px;
  background: linear-gradient(90deg, #f1f5f9, #e2e8f0);
}

.compact-slider::v-deep .slider-progress {
  border-radius: 3px;
  height: 6px;
  background: linear-gradient(90deg, #6366f1, #8b5cf6);
}

.time-markers {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 4px;
  border-top: 1px dashed #e2e8f0;
}

.time-markers span {
  font-size: 11px;
  color: #94a3b8;
  font-weight: 500;
  position: relative;
}

.time-markers span::before {
  content: "";
  position: absolute;
  top: -10px;
  left: 50%;
  transform: translateX(-50%);
  width: 1px;
  height: 6px;
  background: #e2e8f0;
}

/* 动画效果 */
@keyframes cardAppear {
  from {
    opacity: 0;
    transform: translateY(8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.card-wrapper {
  animation: cardAppear 0.4s ease-out forwards;
  opacity: 0;
}

.card-wrapper:nth-child(1) {
  animation-delay: 0.1s;
}
.card-wrapper:nth-child(2) {
  animation-delay: 0.15s;
}
.card-wrapper:nth-child(3) {
  animation-delay: 0.2s;
}
.card-wrapper:nth-child(4) {
  animation-delay: 0.25s;
}
.card-wrapper:nth-child(5) {
  animation-delay: 0.3s;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .reservation-container {
    padding: 10px 12px;
  }

  .card-content {
    padding: 14px;
    gap: 16px;
    flex-direction: column;
  }

  .user-section {
    flex: none;
    width: 100%;
  }

  .time-section {
    width: 100%;
  }

  .time-markers {
    font-size: 10px;
  }

  .time-markers span {
    font-size: 10px;
  }
}

/* 空状态 */
.reservation-container:empty::before {
  content: "暂无预约记录";
  display: flex;
  justify-content: center;
  align-items: center;
  height: 200px;
  color: #94a3b8;
  font-size: 14px;
  font-weight: 500;
}
</style>
