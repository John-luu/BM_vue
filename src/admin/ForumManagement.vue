<template>
  <div class="forum-management">
    <el-card class="forum-card" shadow="never">
      <el-table
        :data="rows"
        style="width: 100%"
        class="forum-table"
        row-class-name="forum-row"
      >
        <!-- 标题 -->
        <el-table-column label="标题" min-width="260">
          <template slot-scope="scope">
            <div class="title-cell">
              {{ scope.row.title }}
            </div>
          </template>
        </el-table-column>

        <!-- 内容 -->
        <el-table-column label="内容" min-width="320">
          <template slot-scope="scope">
            <div class="content-cell">
              {{ scope.row.content }}
            </div>
          </template>
        </el-table-column>

        <!-- 时间 -->
        <el-table-column label="发布时间" width="180">
          <template slot-scope="scope">
            <span class="time-text">
              {{ $formatTime(scope.row.datetime) }}
            </span>
          </template>
        </el-table-column>

        <!-- 状态 -->
        <el-table-column label="状态" width="120">
          <template slot-scope="scope">
            <span
              class="status-dot"
              :class="scope.row.status === 1 ? 'success' : 'offline'"
            ></span>
            <span class="status-text">
              {{ scope.row.status === 1 ? "正常" : "已下架" }}
            </span>
          </template>
        </el-table-column>

        <!-- 操作 -->
        <el-table-column label="操作" width="180" align="right">
          <template slot-scope="scope">
            <!-- 查看 -->
            <el-button
              type="text"
              size="mini"
              class="action-primary"
              @click="openDetail(scope.row)"
            >
              查看
            </el-button>

            <!-- 下架 -->
            <el-button
              v-if="scope.row.status === 1"
              type="text"
              size="mini"
              class="action-danger"
              @click="offlineArticle(scope.$index, scope.row)"
            >
              下架
            </el-button>

            <!-- 恢复 -->
            <el-button
              v-else
              type="text"
              size="mini"
              class="action-success"
              @click="restoreArticle(scope.row)"
            >
              恢复
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 帖子详情 -->
    <el-dialog title="帖子详情" :visible.sync="dialogVisible" width="600px">
      <h3 class="detail-title">{{ currentRow.title }}</h3>
      <p class="detail-content">
        {{ currentRow.content }}
      </p>
      <span slot="footer">
        <el-button @click="dialogVisible = false">关闭</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import request from "@/req";

export default {
  name: "ForumManagement",
  data() {
    return {
      rows: [],
      dialogVisible: false,
      currentRow: {},
    };
  },
  created() {
    request.get("/forum/admin/getArticle").then((res) => {
      this.rows = res.rows;
    });
  },
  methods: {
    openDetail(row) {
      this.currentRow = row;
      this.dialogVisible = true;
    },
    offlineArticle(_, row) {
      this.$confirm("确认下架该帖子吗？", "提示", {
        type: "warning",
      }).then(() => {
        request.post("/forum/admin/offlineArticle", { id: row.id }).then(() => {
          row.status = 0;
          this.$message.success("下架成功");
        });
      });
    },
    restoreArticle(row) {
      this.$confirm("确认恢复该帖子吗？", "提示", {
        type: "info",
      }).then(() => {
        request.post("/forum/admin/onlineArticle", { id: row.id }).then(() => {
          row.status = 1;
          this.$message.success("恢复成功");
        });
      });
    },
  },
};
</script>

<style scoped>
/* 卡片 */
.forum-card {
  border-radius: 12px;
  padding: 8px;
}

/* 表头 */
.forum-table ::v-deep th {
  background: #f9fafb;
  color: #374151;
  font-weight: 600;
  border-bottom: none;
}

/* 表格行 */
.forum-table ::v-deep td {
  border-bottom: 1px solid #f1f5f9;
}

.forum-row:hover {
  background: #f9fafb;
}

/* 标题 */
.title-cell {
  font-weight: 600;
  color: #1f2937;
  line-height: 1.4;
}

/* 内容 */
.content-cell {
  font-size: 13px;
  color: #6b7280;
  line-height: 1.6;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* 时间 */
.time-text {
  font-size: 12px;
  color: #9ca3af;
}

/* 状态 */
.status-dot {
  display: inline-block;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  margin-right: 6px;
}

.status-dot.success {
  background: #22c55e;
}

.status-dot.offline {
  background: #9ca3af;
}

.status-text {
  font-size: 13px;
}

/* 操作 */
.action-primary {
  color: #3b82f6;
  margin-right: 8px;
}
.action-danger {
  color: #ef4444;
  font-weight: 500;
  margin-left: 8px;
}

.action-success {
  color: #22c55e;
  font-weight: 500;
  margin-left: 8px;
}
.more-action {
  cursor: pointer;
  color: #9ca3af;
  font-size: 18px;
}

/* 详情弹窗 */
.detail-title {
  margin-bottom: 12px;
  color: #111827;
}

.detail-content {
  white-space: pre-wrap;
  line-height: 1.8;
  color: #374151;
}
</style>
