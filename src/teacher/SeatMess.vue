<template>
  <div class="seat-management">
    <!-- 使用简洁的Area组件 -->
    <el-card class="seat-card">
      <div class="seat-card-header">
        <!-- 简化版的区域选择器 -->
        <div class="area-selector" v-if="areaRows && areaRows.length > 0">
          <el-select
            v-model="selectedAreaId"
            size="small"
            placeholder="请选择区域"
            @change="handleAreaChange"
            class="area-select"
          >
            <el-option
              v-for="item in areaRows"
              :key="item.aid"
              :label="item.areaName"
              :value="item.aid"
            />
          </el-select>
        </div>

        <!-- 签到码显示区域 -->
        <div class="code-display" v-if="number">
          <span class="code-label">当前签到码：</span>
          <span class="code-value">{{ number }}</span>
        </div>
      </div>

      <!-- 座位显示区域 -->
      <div class="seat-display-container">
        <div class="seat-display-wrapper">
          <Area
            ref="room"
            v-if="seatRows"
            :seat-rows="seatRows"
            :rows="currentAreaRows"
            :columns="currentAreaColumns"
            @seatClick="handleSeatClick"
            class="area-component"
          >
            <!-- 座位菜单插槽 -->
            <div slot="seatMenu" class="seat-menu-popover">
              <div class="menu-content">
                <div class="menu-title">座位信息</div>
                <div class="menu-code">
                  <span>签到码：</span>
                  <span class="code-highlight">{{ number }}</span>
                </div>
              </div>
            </div>
          </Area>

          <!-- 加载提示 -->
          <div v-else class="loading-tip">
            <el-empty description="请选择区域或加载中..." :image-size="100" />
          </div>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script>
import Area from "@/components/Area";
import request from "@/req";

export default {
  name: "SeatMess",
  components: { Area },
  data() {
    return {
      areaRows: null, // 区域列表
      seatRows: null, // 座位数据
      number: "", // 签到码
      selectedAreaId: null, // 当前选中的区域ID
      currentArea: null, // 当前选中的区域信息
    };
  },
  computed: {
    // 计算属性来处理可选链
    currentAreaRows() {
      return this.currentArea && this.currentArea.rows
        ? this.currentArea.rows
        : 0;
    },
    currentAreaColumns() {
      return this.currentArea && this.currentArea.columns
        ? this.currentArea.columns
        : 0;
    },
  },
  methods: {
    // 处理区域变更
    handleAreaChange(areaId) {
      this.currentArea = this.areaRows.find((item) => item.aid === areaId);
      this.loadSeats(areaId);
    },

    // 处理座位点击
    handleSeatClick(index) {
      // 检查座位是否可用
      if (this.seatRows[index].state !== 1) {
        this.$message.warning("该座位当前不可用");
        return;
      }

      // 获取签到码
      request
        .post("/public/getSignedNumber", {
          sid: this.seatRows[index].sid,
        })
        .then((res) => {
          this.number = res.number;

          // 显示成功消息
          this.$message.success({
            message: `签到码已生成：${res.number}`,
            duration: 3000,
          });
        })
        .catch((error) => {
          this.$message.error("获取签到码失败");
        });
    },

    // 加载座位数据
    loadSeats(areaId) {
      this.seatRows = null; // 清空当前座位数据
      request
        .post("/public/getAreaSeats", {
          area: areaId,
        })
        .then((res) => {
          this.seatRows = res.rows || [];
        })
        .catch((error) => {
          this.$message.error("加载座位数据失败");
        });
    },

    // 初始化区域数据
    initAreas() {
      request
        .get("/public/getArea")
        .then((res) => {
          this.areaRows = res.rows || [];

          // 如果有区域数据，默认选择第一个
          if (this.areaRows.length > 0) {
            this.selectedAreaId = this.areaRows[0].aid;
            this.currentArea = this.areaRows[0];
            this.loadSeats(this.selectedAreaId);
          }
        })
        .catch((error) => {
          this.$message.error("加载区域数据失败");
        });
    },
  },
  created() {
    this.initAreas();
  },
};
</script>

<style scoped>
/* 基础样式 */
.seat-management {
  padding: 16px;
  min-height: calc(100vh - 32px);
  background: linear-gradient(135deg, #f5f7fa 0%, #f1f5f9 100%);
  box-sizing: border-box;
  overflow: hidden; /* 禁止页面整体滚动 */
}

.seat-card {
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  border: 1px solid rgba(226, 232, 240, 0.8);
  background: white;
  height: 100vh;
  display: flex;
  flex-direction: column;
  overflow: hidden; /* 禁止卡片内部滚动 */
}

.seat-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  border-bottom: 1px solid #f0f0f0;
  flex-wrap: wrap;
  gap: 12px;
  flex-shrink: 0; /* 防止头部被压缩 */
}

.area-selector {
  flex: 1;
  min-width: 200px;
}

.area-select {
  width: 100%;
}

.code-display {
  padding: 8px 16px;
  background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%);
  border-radius: 20px;
  color: white;
  font-weight: 600;
  box-shadow: 0 2px 8px rgba(59, 130, 246, 0.3);
  animation: pulse 2s infinite;
  flex-shrink: 0;
  text-align: center;
  min-width: 200px;
}

@keyframes pulse {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.8;
  }
  100% {
    opacity: 1;
  }
}

.code-label {
  font-size: 14px;
  opacity: 0.9;
}

.code-value {
  font-size: 16px;
  font-family: "Courier New", monospace;
  letter-spacing: 2px;
}

/* 座位显示区域 - 关键修改 */
.seat-display-container {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  overflow: visible; /* 桌面端不显示滚动条 */
  min-height: 0; /* 重要：让flex子元素正确收缩 */
}

.seat-display-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  overflow: visible; /* 桌面端不显示滚动条 */
  position: relative;
}

.area-component {
  max-width: 100%;
  max-height: 100%;
  overflow: visible; /* 桌面端不显示滚动条 */
}

/* 确保Area组件内部自适应 */
::v-deep .area-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: visible;
}

::v-deep .room {
  display: grid;
  gap: 2px;
}

/* 调整座位大小适应屏幕 */
::v-deep .seat {
  min-width: 24px;
  min-height: 24px;
  width: 100%;
  height: 100%;
}

.loading-tip {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 300px;
}

/* 座位菜单弹窗样式 */
.seat-menu-popover {
  padding: 12px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
  max-width: 250px;
}

.menu-content {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.menu-title {
  font-size: 15px;
  font-weight: 600;
  color: #1e293b;
  padding-bottom: 6px;
  border-bottom: 1px solid #e2e8f0;
}

.menu-code {
  display: flex;
  flex-direction: column;
  gap: 4px;
  font-size: 13px;
  color: #475569;
}

.code-highlight {
  font-size: 16px;
  font-weight: 700;
  color: #3b82f6;
  font-family: "Courier New", monospace;
  letter-spacing: 1px;
  background: #eff6ff;
  padding: 6px 10px;
  border-radius: 4px;
  border: 1px solid #bfdbfe;
  word-break: break-all;
  text-align: center;
}

/* 响应式设计 */
/* 平板设备 */
@media (max-width: 1024px) {
  .seat-management {
    padding: 12px;
  }

  .seat-card-header {
    padding: 12px 16px;
  }

  .seat-display-container {
    padding: 16px;
  }

  ::v-deep .seat {
    min-width: 22px;
    min-height: 22px;
  }
}

/* 移动设备 - 关键修改 */
@media (max-width: 768px) {
  .seat-management {
    padding: 8px;
    overflow: auto; /* 移动端允许页面滚动 */
  }

  .seat-card {
    border-radius: 8px;
    overflow: visible; /* 移动端不限制内部滚动 */
  }

  .seat-card-header {
    flex-direction: column;
    align-items: stretch;
    gap: 10px;
    padding: 12px;
  }

  .area-selector {
    min-width: 100%;
  }

  .code-display {
    min-width: 100%;
    padding: 10px;
  }

  .seat-display-container {
    padding: 12px;
    min-height: auto; /* 移动端自适应高度 */
    overflow: auto; /* 移动端显示滚动条 */
    display: block; /* 改为块级显示 */
  }

  .seat-display-wrapper {
    padding: 8px;
    display: block; /* 改为块级显示 */
    overflow: auto; /* 移动端显示滚动条 */
    width: auto; /* 自动宽度 */
    height: auto; /* 自动高度 */
  }

  .area-component {
    overflow: auto; /* 移动端显示滚动条 */
  }

  ::v-deep .area-wrapper {
    display: block; /* 改为块级显示 */
    overflow: auto; /* 移动端显示滚动条 */
  }

  ::v-deep .room {
    gap: 1px;
    display: grid;
    width: max-content; /* 网格宽度由内容决定 */
    margin: 0 auto; /* 居中显示 */
  }

  ::v-deep .seat {
    min-width: 20px;
    min-height: 20px;
    font-size: 10px;
  }

  .seat-menu-popover {
    max-width: 200px;
    padding: 10px;
  }

  .menu-title {
    font-size: 14px;
  }

  .code-highlight {
    font-size: 14px;
    padding: 4px 8px;
  }
}

/* 小屏幕手机 */
@media (max-width: 480px) {
  .seat-management {
    padding: 6px;
  }

  .seat-card-header {
    padding: 10px;
  }

  .code-label {
    font-size: 12px;
  }

  .code-value {
    font-size: 14px;
  }

  .seat-display-container {
    padding: 8px;
  }

  ::v-deep .seat {
    min-width: 18px;
    min-height: 18px;
  }

  .loading-tip {
    min-height: 250px;
  }

  ::v-deep .el-empty__description p {
    font-size: 12px;
  }
}

/* 超小屏幕手机 */
@media (max-width: 360px) {
  .seat-management {
    padding: 4px;
  }

  .seat-card {
    border-radius: 6px;
  }

  ::v-deep .seat {
    min-width: 16px;
    min-height: 16px;
  }

  .seat-menu-popover {
    max-width: 180px;
    padding: 8px;
  }

  .menu-title {
    font-size: 13px;
  }

  .code-highlight {
    font-size: 13px;
  }
}

/* 横屏优化 */
@media (max-height: 600px) and (orientation: landscape) {
  .seat-management {
    padding: 8px;
    overflow: auto; /* 横屏时允许滚动 */
  }

  .seat-card {
    overflow: visible;
  }

  .seat-card-header {
    padding: 8px 12px;
  }

  .seat-display-container {
    min-height: 300px;
    padding: 12px;
    overflow: auto; /* 横屏时允许滚动 */
  }

  ::v-deep .seat {
    min-width: 20px;
    min-height: 20px;
  }
}

/* 大屏幕桌面端优化 */
@media (min-width: 1400px) {
  .seat-display-container {
    padding: 30px;
  }

  ::v-deep .seat {
    min-width: 28px;
    min-height: 28px;
  }
}

/* 超大屏幕优化 */
@media (min-width: 1920px) {
  .seat-management {
    max-width: 1800px;
    margin: 0 auto;
  }

  ::v-deep .seat {
    min-width: 32px;
    min-height: 32px;
  }
}

/* 打印样式 */
@media print {
  .seat-card {
    box-shadow: none;
    border: 1px solid #ddd;
  }

  .code-display {
    background: #f0f0f0;
    color: #333;
    box-shadow: none;
    animation: none;
  }
}

/* 防止网格内容被截断 */
@media (max-width: 768px) {
  ::v-deep .room {
    overflow: visible;
  }
}
</style>
