<template>
  <div class="root">
    <el-card style="margin-top: 10%; position: relative">
      <div slot="header" class="clearfix">
        <ToggleArea
          @changeArea="onAreaChange"
          @areaCreated="onAreaCreated"
          @areaDeleted="refreshAreaList"
          :can-add="canAdd"
          ref="toggleArea"
          :area-rows="areaRows"
          v-if="areaRows"
        ></ToggleArea>
        <HeadTip></HeadTip>
      </div>
      <Area
        class="area-container"
        ref="room"
        v-if="currentArea && currentArea.rows && currentArea.columns"
        :seat-rows="seatRows || []"
        :manageMode="manageMode"
        :rows="currentArea.rows"
        :columns="currentArea.columns"
        @seatClick="seatClick"
        @blankClick="blankClick"
      >
        <div slot="blankMenu" class="blankMenu">
          <div @click="addSeat(0)">添加座位</div>
          <div @click="addSeat(1)">添加桌子</div>
        </div>
        <div slot="seatMenu" class="blankMenu">
          <div @click="deleteSeat"><i class="el-icon-delete"></i>删除</div>
        </div>
      </Area>
    </el-card>
  </div>
</template>

<script>
import Area from "@/components/Area";
import request from "@/req";
import HeadTip from "@/components/HeadTip";
import ToggleArea from "@/components/ToggleArea";

export default {
  name: "SeatCrud",
  components: { ToggleArea, HeadTip, Area },
  data() {
    return {
      areaRows: null,
      currentArea: null,
      canAdd: true,
      manageMode: true,
      seatCur: {
        row: Number,
        column: Number,
      },
      seatRows: null,
      sid: null,
    };
  },
  methods: {
    deleteSeat() {
      request
        .post("/admin/deleteSeat", {
          sid: this.sid,
        })
        .then((res) => {
          this.$message("删除成功");
          this.getSeatRows();
        });
    },
    seatClick(index) {
      //正在使用的座位无法操作
      if (
        this.seatRows[index].state === 1 ||
        this.seatRows[index].state === 2
      ) {
        this.seatRows[index].show = false;
        return;
      }
      this.sid = this.seatRows[index].sid;
    },
    addSeat(type) {
      this.$refs.room.closeLastPop();
      request
        .post("/admin/addSeat", {
          row: this.seatCur.row,
          column: this.seatCur.column,
          area: this.$refs.toggleArea.getArea().aid,
          type: type,
        })
        .then((res) => {
          this.$message("添加成功");
          this.getSeatRows();
        });
    },
    blankClick(index, row, column) {
      this.seatCur.index = index;
      this.seatCur.row = row;
      this.seatCur.column = column;
      console.log("index===", index);
      console.log("row===", row);
      console.log("column===", column);
    },
    onAreaChange(area) {
      console.log("区域切换:", area);
      // area 是 ToggleArea 主动给你的
      this.currentArea = area;
      console.log("currentArea.rows===", this.currentArea.rows);
      request
        .post("/public/getAreaSeats", {
          area: area.aid,
        })
        .then((res) => {
          // 新创建的区域 seatRows 应该是空数组
          this.seatRows = res.rows || [];
          // this.currentArea = res.data;
          console.log("res.rows===", res.rows);
        });
    },
    // 处理区域创建事件，刷新区域列表
    onAreaCreated(newAreaData) {
      console.log("区域创建成功:", newAreaData);
      // 刷新区域列表
      this.refreshAreaList();
    },

    // 刷新区域列表
    refreshAreaList() {
      request.get("/public/getArea").then((res) => {
        this.areaRows = res.rows;
      });
    },
    getSeatRows() {
      this.$nextTick(() => {
        request
          .post("/public/getAreaSeats", {
            area: this.$refs.toggleArea.getArea().aid,
          })
          .then((res) => {
            this.seatRows = res.rows;
          });
      });
    },
  },
  created() {
    request.get("/public/getArea").then((res) => {
      this.areaRows = res.rows;
      this.getSeatRows();
    });
  },
};
</script>

<style scoped>
.root {
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
}
el-card {
  width: 100%;
  height: calc(100vh - 32px);
}
::v-deep .el-card__body {
  height: 100%;
  display: flex;
  flex-direction: column;
}
.area-container {
  flex: 1;
}
</style>
