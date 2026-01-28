<template>
  <div>
    <navbar title="我的" :left="false"></navbar>

    <!-- 基本信息 -->
    <el-card class="box-card" v-if="user.uid">
      <van-cell title="用户名" :value="user.username" />
      <van-cell title="学号" :value="user.uid" />
      <van-cell title="信用分">
        <span style="color: #409eff">{{ score }} 分</span>
      </van-cell>
    </el-card>

    <!-- 我的预约 -->
    <el-card class="box-card" style="margin-top: 10px">
      <div slot="header">我的预约记录</div>

      <van-empty v-if="rows.length === 0" description="暂无预约记录" />

      <div v-else v-for="item in rows" :key="item.rid">
        <van-cell
          :title="item.areaName + ' ' + item.subName + item.row + item.column"
          :label="getTime(item.startTime) + ' - ' + getTime(item.endTime)"
        />
      </div>
    </el-card>
  </div>
</template>
<script>
import Navbar from "@/components/navbar";
import request from "@/req";

export default {
  name: "Mine",
  components: { Navbar },

  data() {
    return {
      user: {},
      rows: [],
      score: 100,
    };
  },

  methods: {
    getTime(d) {
      const date = new Date(d);
      return `${date.getHours()}:${date.getMinutes() === 0 ? "00" : "30"}`;
    },
  },
  created() {
    this.user = this.$getUser();
    console.log("this.user===", this.user);
    if (!this.user || !this.user.uid) return;

    request
      .post("/user/getScore", {
        uid: this.$getUser().uid,
      })
      .then((res) => {
        this.score = res.score;
      });
  },
};
</script>
