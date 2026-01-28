<template>
  <div class="area-wrapper">
    <div class="room" :style="gridStyle">
      <!--      显示底部网格-->
      <div
        v-for="(item, index) in blankRows"
        class="item"
        @click="blankClick(index)"
      >
        <div v-if="!item.show" class="seat" :class="getBatchSeatClass(index)">
          <!-- 批量模式下显示选中标记 -->
          <div v-if="isSelectedInBatch(index)" class="batch-selected-indicator">
            ✓
          </div>
        </div>
        <van-popover
          v-else
          v-model="item.show"
          :offset="[0, 0]"
          placement="top"
          theme="dark"
        >
          <slot name="blankMenu"></slot>
          <template #reference>
            <div
              class="seat"
              :class="[
                index === lastBlankIndex ? 'seatSelect' : '',
                getBatchSeatClass(index),
              ]"
            >
              <!-- 批量模式下显示选中标记 -->
              <div
                v-if="isSelectedInBatch(index)"
                class="batch-selected-indicator"
              >
                ✓
              </div>
            </div>
          </template>
        </van-popover>
      </div>
    </div>
    <div class="room overlay" :style="gridStyle">
      <!--      显示座位-->
      <div
        v-for="(item, index) in seatRows"
        @click="seatClick(index)"
        class="item"
        :style="'grid-row:' + item.row + ';grid-column:' + item.column"
      >
        <seat v-if="!item.show" :item="item"></seat>
        <van-popover
          v-else
          v-model="item.show"
          :offset="[0, 0]"
          placement="top"
          theme="dark"
        >
          <slot name="seatMenu"></slot>
          <template #reference>
            <seat :item="item"></seat>
          </template>
        </van-popover>
      </div>
    </div>
  </div>
</template>

<script>
import Seat from "@/components/Seat";

export default {
  name: "Area",
  components: { Seat },
  data() {
    return {
      blankRows: [],
      lastBlankIndex: -1,
    };
  },
  props: {
    manageMode: {
      type: Boolean,
      default: false,
    },
    batchMode: {
      type: Boolean,
      default: false,
    },
    seatRows: {
      type: Array,
    },
    rows: {
      type: Number,
      required: true,
    },
    columns: {
      type: Number,
      required: true,
    },
    batchSelected: {
      // 添加这个prop来接收选中的位置
      type: Array,
      default: () => [],
    },
  },
  methods: {
    // 判断空白格是否被选中
    isSelectedInBatch(index) {
      if (!this.batchMode || !this.batchSelected.length) return false;

      const realIndex = index + 1;
      let row, column;

      if (realIndex % this.columns === 0) {
        row = Math.trunc(realIndex / this.columns);
        column = this.columns;
      } else {
        row = Math.trunc(realIndex / this.columns) + 1;
        column = realIndex % this.columns;
      }

      return this.batchSelected.some(
        (item) => item.row === row && item.column === column,
      );
    },
    closeLastPop() {
      for (let i = 0; i < this.blankRows.length; i++) {
        this.blankRows[i].show = false;
      }
      for (let i = 0; i < this.seatRows.length; i++) {
        this.seatRows[i].show = false;
      }
    },
    seatClick(index) {
      // 如果是批量模式，不显示菜单
      if (this.batchMode) {
        this.$emit("seatClick", index);
        return;
      }
      this.closeLastPop();
      this.seatRows[index].show = true;
      this.$emit("seatClick", index);
    },
    // 获取批量模式下的样式类
    getBatchSeatClass(index) {
      if (!this.batchMode) return "";

      return this.isSelectedInBatch(index) ? "batch-selected" : "";
    },
    closeOnOutside(ev) {
      if (
        ev.target.getAttribute("class") === null ||
        ev.target.getAttribute("class").indexOf("seat") === -1
      ) {
        this.closeLastPop();
      }
    },
    blankClick(index) {
      console.log("111");
      console.log("blankClick(index)===", index);
      console.log("this.manageMode===", this.manageMode);
      if (!this.manageMode) return;
      console.log("222");
      this.lastBlankIndex = index;
      // 如果是批量模式，不显示菜单
      if (this.batchMode) {
        const realIndex = index + 1;
        let row, column;

        if (realIndex % this.columns === 0) {
          row = Math.trunc(realIndex / this.columns);
          column = this.columns;
        } else {
          row = Math.trunc(realIndex / this.columns) + 1;
          column = realIndex % this.columns;
        }

        this.$emit("blankClick", realIndex, row, column);
        return;
      }

      // 非批量模式才显示菜单
      this.closeLastPop();
      this.blankRows[index].show = true;

      const realIndex = index + 1;
      let row, column;

      if (realIndex % this.columns === 0) {
        row = Math.trunc(realIndex / this.columns);
        column = this.columns;
      } else {
        row = Math.trunc(realIndex / this.columns) + 1;
        column = realIndex % this.columns;
      }

      this.$emit("blankClick", realIndex, row, column);
    },
    initBlankRows() {
      this.blankRows = [];
      const total = this.rows * this.columns;
      console.log("total===", total);
      console.log("this.rows===", this.rows);
      console.log("this.columns===", this.columns);
      for (let i = 0; i < total; i++) {
        this.blankRows.push({ show: false });
      }
      console.log("this.blankRows==", this.blankRows);
    },
  },
  created() {
    //定义空白方块数组
    if (this.rows && this.columns) {
      console.log("111");
      console.log("this.rows===", this.rows);
      console.log("this.columns===", this.columns);
      this.initBlankRows();
    }
    console.log("this.rows===", this.rows);
    console.log("this.columns===", this.columns);

    window.addEventListener("click", this.closeOnOutside);
  },
  destroyed() {
    window.removeEventListener("click", this.closeOnOutside);
  },
  watch: {
    rows() {
      this.initBlankRows();
    },
    columns() {
      this.initBlankRows();
    },
    seatRows() {
      this.lastBlankIndex = -1;
    },
    batchSelected() {
      // 当选中的位置变化时，强制更新视图
      this.$forceUpdate();
    },
  },
  computed: {
    gridStyle() {
      const style = {
        gridTemplateRows: `repeat(${this.rows}, 32px)`,
        gridTemplateColumns: `repeat(${this.columns}, 32px)`,
      };
      console.log("gridStyle:", style);
      console.log("预期尺寸:", this.rows * 32, "px ×", this.columns * 32, "px");
      return style;
    },
  },
};
</script>

<style scoped>
/* .room {
  display: grid;
}

.item {
  width: 32px;
  height: 32px;
} */
/* 添加调试样式 */
.area-wrapper {
  position: relative;
  width: max-content; /* 根据 grid 实际尺寸撑开 */
  height: max-content;
}
.room {
  display: grid;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none; /* 允许点击穿透到下层 */
}

/* 但座位本身需要响应点击 */
.overlay .item {
  pointer-events: auto;
}
.overlay .seat {
  pointer-events: auto;
}
.seat {
  width: 100%;
  height: 100%;
  background-color: #f0f0f0; /* 灰色背景，方便查看 */
}
/* 批量选中的样式 */
.batch-selected {
  background-color: #409eff !important;
  border: 2px solid #67c23a !important;
  box-sizing: border-box;
}

.batch-selected-indicator {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-weight: bold;
  font-size: 16px;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
  z-index: 10;
}
/* 确保座位组件正确显示 */
::v-deep .seat-component {
  width: 100%;
  height: 100%;
  position: relative;
}
</style>
