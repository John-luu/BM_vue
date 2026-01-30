<template>
  <div class="score-management-container">
    <div v-for="(item, index) in rows" :key="index" class="card-wrapper">
      <el-card class="management-card" shadow="hover">
        <!-- 卡片顶部信息 -->
        <div class="card-header">
          <div class="user-info">
            <div class="user-avatar">{{ getAvatarText(item.username) }}</div>
            <div class="user-details">
              <div class="user-name-row">
                <span class="user-name">{{ item.username }}</span>
                <span class="status-badge" :class="getStatusClass(item.state)">
                  {{ getState(item.state) }}
                </span>
              </div>
              <div class="reservation-time">
                <i class="el-icon-time"></i>
                {{ getTime(item.startTime) }}
              </div>
            </div>
          </div>
          <div class="reservation-id">
            <span class="id-label">预约ID:</span>
            <span class="id-value">{{ item.rid || "N/A" }}</span>
          </div>
        </div>

        <!-- 时间轴区域 -->
        <div class="time-header">
          <span class="time-title">使用时段</span>

          <span class="time-range">
            {{ formatTimeRange(item.startTime, item.endTime) }}
          </span>
        </div>

        <!-- 操作区域 -->
        <div class="action-section">
          <div class="score-info">
            <div class="score-label">当前状态:</div>
            <div class="score-value" :class="{ scored: item.score }">
              {{ item.score ? `已扣 ${item.score} 分` : "待处理" }}
            </div>
          </div>

          <div class="action-buttons">
            <van-button
              v-if="!item.score"
              type="danger"
              size="small"
              round
              @click.stop="subScore(item)"
              class="deduct-btn"
            >
              <i class="el-icon-minus"></i>
              扣除信用分
            </van-button>
            <div v-else class="scored-status">
              <van-icon name="passed" color="#909399" />
              <span>已处理</span>
            </div>
          </div>
        </div>

        <!-- 卡片底部装饰 -->
        <div class="card-footer">
          <div class="footer-line"></div>
        </div>
      </el-card>
    </div>

    <!-- 空状态 -->
    <div v-if="rows && rows.length === 0" class="empty-state">
      <div class="empty-icon">
        <i class="el-icon-circle-check"></i>
      </div>
      <div class="empty-text">当前无待处理的扣分记录</div>
      <div class="empty-subtext">所有预约记录都已处理完成</div>
    </div>
  </div>
</template>

<script>
import request from "@/req";
import TimeSlider from "@/components/TimeSlider";
import { Toast } from "vant";

export default {
  name: "ScoreManagement",
  components: { TimeSlider },
  data() {
    return {
      rows: null,
    };
  },
  methods: {
    subScore(item) {
      request
        .post("/teacher/subScore", {
          uid: item.uid,
          rid: item.rid,
        })
        .then((res) => {
          Toast.success({
            message: "扣分操作成功",
            duration: 1500,
          });
          this.update();
        });
    },
    getState(state) {
      switch (state) {
        case 2:
          return "预约未签到";
        case 4:
          return "暂离超时未回";
        default:
          return "未知状态";
      }
    },
    getStatusClass(state) {
      return state === 2 ? "status-unchecked" : "status-overtime";
    },
    getTime(d) {
      let date = new Date(d);
      const year = date.getFullYear();
      const month = (date.getMonth() + 1).toString().padStart(2, "0");
      const day = date.getDate().toString().padStart(2, "0");
      const hours = date.getHours().toString().padStart(2, "0");
      const minutes =
        date.getMinutes() === 0
          ? "00"
          : date.getMinutes().toString().padStart(2, "0");
      return `${year}/${month}/${day} ${hours}:${minutes}`;
    },
    formatTimeRange(start, end) {
      const startTime = new Date(start);
      const endTime = new Date(end);

      const format = (date) => {
        const hours = date.getHours().toString().padStart(2, "0");
        const minutes = date.getMinutes().toString().padStart(2, "0");
        return `${hours}:${minutes}`;
      };

      return `${format(startTime)} - ${format(endTime)}`;
    },
    getAvatarText(username) {
      if (!username || username.length === 0) return "?";
      return username.substring(0, 1).toUpperCase();
    },
    update() {
      request.get("/teacher/getReservationNeedSub").then((res) => {
        this.rows = res.rows;
      });
    },
  },
  created() {
    this.update();
  },
};
</script>

<style scoped>
.score-management-container {
  padding: 16px;
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  min-height: 100vh;
}

.card-wrapper {
  margin-bottom: 16px;
  animation: fadeInUp 0.5s ease-out forwards;
  opacity: 0;
}

.card-wrapper:nth-child(1) {
  animation-delay: 0.1s;
}
.card-wrapper:nth-child(2) {
  animation-delay: 0.2s;
}
.card-wrapper:nth-child(3) {
  animation-delay: 0.3s;
}
.card-wrapper:nth-child(4) {
  animation-delay: 0.4s;
}

.management-card {
  border-radius: 18px !important;
  border: none !important;
  overflow: hidden;
  background: white;
  position: relative;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.management-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.12) !important;
}

.management-card::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, #f56c6c, #ff9e52);
  opacity: 0.8;
}

/* 卡片头部 */
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 20px 20px 16px;
  border-bottom: 1px solid #f0f2f5;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 14px;
}

.user-avatar {
  width: 40px;
  height: 40px;
  border-radius: 12px;
  background: linear-gradient(135deg, #f56c6c 0%, #ff9e52 100%);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  font-size: 18px;
  box-shadow: 0 6px 16px rgba(245, 108, 108, 0.25);
  flex-shrink: 0;
}

.user-details {
  flex: 1;
}

.user-name-row {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 6px;
}

.user-name {
  font-size: 16px;
  font-weight: 600;
  color: #1e293b;
}

.status-badge {
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.5px;
  white-space: nowrap;
}

.status-unchecked {
  background: rgba(230, 162, 60, 0.15);
  color: #e6a23c;
  border: 1px solid rgba(230, 162, 60, 0.3);
}

.status-overtime {
  background: rgba(245, 108, 108, 0.15);
  color: #f56c6c;
  border: 1px solid rgba(245, 108, 108, 0.3);
}

.reservation-time {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  color: #64748b;
}

.reservation-time i {
  color: #409eff;
}

.reservation-id {
  text-align: right;
  flex-shrink: 0;
}

.id-label {
  display: block;
  font-size: 12px;
  color: #94a3b8;
  margin-bottom: 4px;
}

.id-value {
  font-size: 14px;
  font-weight: 600;
  color: #475569;
  font-family: "Courier New", monospace;
  letter-spacing: 0.5px;
}

/* 时间区域 */
.time-section {
  padding: 20px;
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  margin: 0 20px;
  border-radius: 14px;
  border: 1px solid #edf2f7;
}
.time-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}
.time-title {
  font-size: 14px;
  font-weight: 600;
  color: #475569;
  display: flex;
  align-items: center;
  gap: 8px;
}

.time-title::before {
  content: "";
  display: inline-block;
  width: 4px;
  height: 16px;
  background: linear-gradient(180deg, #409eff, #66b1ff);
  border-radius: 2px;
}
.time-range {
  font-size: 14px;
  font-weight: 500;
  color: #1e293b;
  padding: 4px 12px;
  background: rgba(64, 158, 255, 0.08);
  border-radius: 8px;
  white-space: nowrap;
}

/* 操作区域 */
.action-section {
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 1px solid #f0f2f5;
}

.score-info {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.score-label {
  font-size: 13px;
  color: #64748b;
}

.score-value {
  font-size: 16px;
  font-weight: 600;
  color: #f56c6c;
}

.score-value.scored {
  color: #909399;
}

.action-buttons {
  flex-shrink: 0;
}

.deduct-btn {
  padding: 10px 24px;
  font-weight: 500;
  letter-spacing: 0.5px;
  background: linear-gradient(135deg, #f56c6c, #ff7875);
  border: none;
  box-shadow: 0 4px 16px rgba(245, 108, 108, 0.3);
  transition: all 0.3s ease;
}

.deduct-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(245, 108, 108, 0.4);
}

.deduct-btn i {
  margin-right: 6px;
  font-weight: bold;
}

.scored-status {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  background: #f8f9fa;
  border-radius: 20px;
  color: #909399;
  font-size: 14px;
  font-weight: 500;
}

.scored-status i {
  font-size: 16px;
}

/* 卡片底部 */
.card-footer {
  padding: 0 20px 16px;
}

.footer-line {
  height: 2px;
  background: linear-gradient(
    90deg,
    rgba(64, 158, 255, 0.1) 0%,
    rgba(64, 158, 255, 0.3) 50%,
    rgba(64, 158, 255, 0.1) 100%
  );
  border-radius: 1px;
}

/* 空状态 */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80px 20px;
  text-align: center;
}

.empty-icon {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background: linear-gradient(135deg, #67c23a, #85ce61);
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 24px;
  box-shadow: 0 8px 24px rgba(103, 194, 58, 0.25);
}

.empty-icon i {
  font-size: 36px;
  color: white;
}

.empty-text {
  font-size: 18px;
  font-weight: 600;
  color: #1e293b;
  margin-bottom: 8px;
}

.empty-subtext {
  font-size: 14px;
  color: #64748b;
}

/* 动画 */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 响应式 */
@media (max-width: 768px) {
  .score-management-container {
    padding: 12px;
  }

  .card-header {
    flex-direction: column;
    gap: 16px;
    align-items: stretch;
  }

  .reservation-id {
    text-align: left;
  }

  .action-section {
    flex-direction: column;
    gap: 16px;
    align-items: stretch;
  }

  .action-buttons {
    align-self: flex-end;
  }

  .deduct-btn {
    width: 100%;
  }
}
</style>
