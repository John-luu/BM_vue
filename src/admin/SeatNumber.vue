<template>
  <div class="root">
    <el-card style="margin-top: 10%; position: relative">
      <div slot="header" class="clearfix">
        <ToggleArea
          @changeArea="onAreaChange"
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
        :rows="currentArea.rows"
        :columns="currentArea.columns"
        @seatClick="seatClick"
      >
        <div slot="seatMenu" class="blankMenu">
          <div>签到码：{{ number }}</div>
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
  name: "SeatNumber",
  components: { ToggleArea, HeadTip, Area },
  data() {
    return {
      areaRows: null,
      seatRows: null,
      currentArea: null,
      number: "000000",
    };
  },
  methods: {
    seatClick(index) {
      //正在使用的座位无法操作
      if (this.seatRows[index].state !== 1) {
        this.seatRows[index].show = false;
        return;
      }
      request
        .post("/public/getSignedNumber", {
          sid: this.seatRows[index].sid,
        })
        .then((res) => {
          this.number = res.number;
        });
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
    // getSeatRows() {
    //   this.$nextTick(() => {
    //     request
    //       .post("/public/getAreaSeats", {
    //         area: this.$refs.toggleArea.getArea().aid,
    //       })
    //       .then((res) => {
    //         this.seatRows = res.rows;
    //       });
    //   });
    // },
  },
  created() {
    request.get("/public/getArea").then((res) => {
      this.areaRows = res.rows;
      // this.getSeatRows();
    });
  },
};
</script>

<style scoped>
.root {
  width: 100%;
  height: 100vh;
  padding: 16px;
  box-sizing: border-box;
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
