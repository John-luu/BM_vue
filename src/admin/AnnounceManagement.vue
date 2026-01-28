<template>
  <div class="announce-page">
    <!-- 顶部操作栏 -->
    <div class="page-header">
      <div class="title">📢 公告管理</div>
      <el-button
        type="primary"
        icon="el-icon-plus"
        @click="dialogVisible = true"
      >
        新增公告
      </el-button>
    </div>

    <!-- 内容卡片 -->
    <el-card class="content-card" shadow="never">
      <el-table
        :data="rows"
        stripe
        style="width: 100%"
        row-class-name="table-row"
      >
        <el-table-column label="标题" width="200">
          <template slot-scope="scope">
            <div class="title-cell">{{ scope.row.title }}</div>
          </template>
        </el-table-column>

        <el-table-column label="内容">
          <template slot-scope="scope">
            <div class="content-cell">{{ scope.row.content }}</div>
          </template>
        </el-table-column>

        <el-table-column label="发布时间" width="180">
          <template slot-scope="scope">
            {{ $formatTime(scope.row.datetime) }}
          </template>
        </el-table-column>

        <el-table-column label="操作" width="120">
          <template slot-scope="scope">
            <el-button
              type="danger"
              size="mini"
              plain
              @click="handleDelete(scope.$index, scope.row)"
            >
              删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 新增公告弹窗 -->
    <el-dialog title="新增公告" :visible.sync="dialogVisible" width="480px">
      <el-form :model="form" label-width="60px">
        <el-form-item label="标题">
          <el-input v-model="form.title" />
        </el-form-item>

        <el-form-item label="内容">
          <el-input type="textarea" :rows="5" v-model="form.content" />
        </el-form-item>
      </el-form>

      <span slot="footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submitAnnounce">发布</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import request from "@/req";

export default {
  name: "AnnounceManagement",
  data() {
    return {
      rows: [],
      dialogVisible: false,
      form: {
        title: "",
        content: "",
      },
    };
  },
  methods: {
    loadData() {
      request.get("/admin/getAnnounce").then((res) => {
        this.rows = res.rows;
      });
    },
    handleDelete(index, row) {
      this.$confirm("确认删除该公告吗？", "提示", {
        type: "warning",
      }).then(() => {
        request.post("/admin/deleteAnnounce", { id: row.id }).then(() => {
          this.rows.splice(index, 1);
          this.$message.success("删除成功");
        });
      });
    },
    submitAnnounce() {
      if (!this.form.title || !this.form.content) {
        this.$message.warning("标题和内容不能为空");
        return;
      }
      request.post("/admin/addAnnounce", this.form).then(() => {
        this.$message.success("发布成功");
        this.dialogVisible = false;
        this.form.title = "";
        this.form.content = "";
        this.loadData();
      });
    },
  },
  created() {
    this.loadData();
  },
};
</script>

<style scoped>
.announce-page {
  padding: 24px;
  background: #f5f7fb;
  min-height: 100vh;
}

/* 顶部栏 */
.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.page-header .title {
  font-size: 20px;
  font-weight: 600;
}

/* 卡片 */
.content-card {
  border-radius: 8px;
}

/* 表格内容 */
.title-cell {
  font-weight: 500;
}

.content-cell {
  color: #666;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.table-row:hover {
  background: #f0f7ff !important;
}
</style>
