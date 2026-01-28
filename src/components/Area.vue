<template>
  <div class="area-wrapper">
    <div class="room" :style="gridStyle">
      <!--      显示底部网格-->
      <div
        v-for="(item, index) in blankRows"
        class="item"
        @click="blankClick(index)"
      >
        <div v-if="!item.show" class="seat"></div>
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
              :class="index === lastBlankIndex ? 'seatSelect' : ''"
            ></div>
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
  },
  methods: {
    closeLastPop() {
      for (let i = 0; i < this.blankRows.length; i++) {
        this.blankRows[i].show = false;
      }
      for (let i = 0; i < this.seatRows.length; i++) {
        this.seatRows[i].show = false;
      }
    },
    seatClick(index) {
      this.closeLastPop();
      this.seatRows[index].show = true;
      this.$emit("seatClick", index);
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
</style>
