<template>
  <div>
    <navbar title="预约座位" :left="false">
      <Score slot="right"></Score>
    </navbar>

    <el-card style="overflow: scroll">
      <div slot="header" class="clearfix">
        <ToggleArea
          v-if="areaRows"
          ref="toggleArea"
          :area-rows="areaRows"
          @changeArea="getSeatRows"
        />
        <HeadTip />
      </div>

      <Area
        v-if="seatRows"
        ref="room"
        :seat-rows="seatRows"
        @seatClick="seatClick"
      >
        <div slot="seatMenu" class="blankMenu">
          <div @click="clickPop"><i class="el-icon-s-flag"></i>预约</div>
        </div>
      </Area>
    </el-card>

    <el-card>
      <van-cell title="座位" :value="seatName" />
      <TimeSlider ref="timeSlider" />
    </el-card>

    <div class="btn" @click="submit">预约</div>
  </div>
</template>

<script>
import Area from "@/components/Area";
import Navbar from "@/components/navbar";
import request from "@/req";
import HeadTip from "@/components/HeadTip";
import ToggleArea from "@/components/ToggleArea";
import TimeSlider from "@/components/TimeSlider";
import { Toast } from "vant";
import Score from "@/components/Score";

export default {
  name: "Reservation",
  components: {
    Score,
    TimeSlider,
    ToggleArea,
    HeadTip,
    Navbar,
    Area,
  },

  data() {
    return {
      areaRows: null,
      seatRows: null,
      seatCurIndex: 0,
      seatName: "请选择座位",
    };
  },

  mounted() {
    // ① 先加载区域
    request.get("/public/getArea").then((res) => {
      this.areaRows = res.rows;

      // ② 等 ToggleArea 真正渲染出来
      this.$nextTick(() => {
        this.getSeatRows();
      });
    });

    // ③ 检查是否有未完成预约
    this.togglePage();
  },

  methods: {
    /** 提交预约 */
    submit() {
      if (this.seatName === "请选择座位") {
        Toast.fail("请选择座位");
        return;
      }

      if (new Date().getHours() >= 22) {
        Toast.fail("22点之后无法预约");
        return;
      }

      this.$nextTick(() => {
        const slider = this.$refs.timeSlider;
        if (!slider) return;

        const body = {
          startTime: slider.getStartTime(),
          endTime: slider.getEndTime(),
          uid: this.$getUser().uid,
          sid: this.seatRows[this.seatCurIndex].sid,
        };

        if (!body.sid || body.startTime === body.endTime) {
          Toast.fail("时间非法！");
          return;
        }

        request.post("/user/addReservation", body).then((res) => {
          if (res.code === 200) {
            Toast.success("预约成功");
            this.togglePage();
          } else {
            Toast.fail("预约失败，当前已有预约");
          }
        });
      });
    },

    /** 判断是否跳转到签到 / 使用页 */
    togglePage() {
      request
        .post("/user/getReservationByUid", {
          uid: this.$getUser().uid,
        })
        .then((res) => {
          if (!res.rows) return;

          res.rows.forEach((item) => {
            localStorage.setItem("reservation", JSON.stringify(item));

            if (item.state === 0 || item.state === 3) {
              if (this.$route.path !== "/student/seat/toSigned") {
                this.$router.replace("/student/seat/toSigned");
              }
            }

            if (item.state === 1) {
              if (this.$route.path !== "/student/seat/beUse") {
                this.$router.replace("/student/seat/beUse");
              }
            }
          });
        });
    },

    /** 点击座位弹出显示 */
    clickPop() {
      this.$nextTick(() => {
        const toggle = this.$refs.toggleArea;
        if (!toggle || !toggle.getArea) return;

        const area = toggle.getArea();
        const seat = this.seatRows[this.seatCurIndex];
        if (!area || !seat) return;

        this.seatName = area.subName + seat.row + seat.column;
      });
    },

    /** 获取当前区域座位 */
    getSeatRows() {
      this.$nextTick(() => {
        const toggle = this.$refs.toggleArea;
        if (!toggle || !toggle.getArea) return;

        const area = toggle.getArea();
        if (!area || !area.aid) return;

        request
          .post("/public/getAreaSeats", {
            area: area.aid,
          })
          .then((res) => {
            this.seatRows = res.rows;
          });
      });
    },

    /** 点击座位 */
    seatClick(index) {
      const seat = this.seatRows[index];
      if (!seat) return;

      // 不可用座位
      if (seat.type === 1 || seat.state === 1 || seat.state === 2) {
        seat.show = false;
        return;
      }

      this.seatCurIndex = index;
    },
  },
};
</script>

<style scoped></style>
