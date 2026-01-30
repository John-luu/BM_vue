<template>
  <div class="teacher-management">
    <!-- 顶部搜索区 -->
    <el-card class="toolbar-card" shadow="never">
      <div class="compact-toolbar">
        <div class="search-group">
          <el-input
            v-model="search.number"
            size="mini"
            clearable
            placeholder="学工号"
            class="compact-input"
            prefix-icon="el-icon-user"
          />
          <el-input
            v-model="search.username"
            size="mini"
            clearable
            placeholder="姓名"
            class="compact-input"
            prefix-icon="el-icon-user-solid"
          />
          <div class="button-group">
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-search"
              @click="handleSearch"
              class="search-btn"
            >
              搜索
            </el-button>
            <el-button
              size="mini"
              icon="el-icon-refresh-right"
              @click="resetSearch"
              class="reset-btn"
            >
              重置
            </el-button>
          </div>
        </div>

        <el-button
          type="success"
          size="mini"
          @click="openAddDialog"
          icon="el-icon-plus"
          class="add-btn"
        >
          新增教师
        </el-button>
      </div>
    </el-card>

    <!-- 表格 -->
    <el-card shadow="never" class="table-card">
      <el-table :data="rows" border>
        <!-- 序号 -->
        <el-table-column label="序号" width="70" align="center">
          <template slot-scope="{ $index }">
            {{ (page - 1) * pageSize + $index + 1 }}
          </template>
        </el-table-column>
        <el-table-column label="学工号" prop="number" width="200" />
        <el-table-column label="姓名" prop="username" width="180" />

        <el-table-column label="信用分" width="120">
          <template slot-scope="{ row }">
            <el-tag :type="scoreTagType(row.score)" size="mini">
              {{ row.score }}
            </el-tag>
          </template>
        </el-table-column>

        <el-table-column label="重置密码" min-width="200">
          <template slot-scope="{ row }">
            <div class="pwd-reset-group">
              <el-input
                size="mini"
                v-model="row.newPwd"
                placeholder="新密码"
                class="compact-pwd-input"
                show-password
              />
              <el-button
                type="text"
                size="mini"
                @click="remarkPwd(row)"
                class="reset-pwd-btn"
              >
                重置
              </el-button>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分页 -->
    <BasePagination
      :total="total"
      :page-size="pageSize"
      :current.sync="page"
      @change="fetchData"
    />

    <!-- 新增教师弹窗 -->
    <el-dialog title="新增教师" :visible.sync="addVisible" width="400px">
      <el-form label-width="80px" :model="form" autocomplete="off">
        <el-form-item label="学工号">
          <el-input
            v-model="form.number"
            autocomplete="off"
            size="small"
            placeholder="请输入学工号"
          />
        </el-form-item>
        <el-form-item label="姓名">
          <el-input
            v-model="form.username"
            autocomplete="off"
            size="small"
            placeholder="请输入姓名"
          />
        </el-form-item>
        <el-form-item label="密码">
          <el-input
            v-model="form.password"
            show-password
            autocomplete="new-password"
            size="small"
            placeholder="请输入密码"
          />
        </el-form-item>
      </el-form>

      <span slot="footer">
        <el-button size="small" @click="addVisible = false" class="dialog-btn">
          取消
        </el-button>
        <el-button
          type="primary"
          size="small"
          @click="submitAdd"
          class="dialog-btn"
        >
          确定
        </el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import request from "@/req";
import BasePagination from "@/components/BasePagination";

export default {
  name: "TeacherManagement",
  components: { BasePagination },
  data() {
    return {
      rows: [],
      total: 0,
      page: 1,
      pageSize: 10,

      search: {
        number: "",
        username: "",
      },

      addVisible: false,
      form: {
        number: "",
        username: "",
        password: "",
      },
    };
  },
  created() {
    this.fetchData();
  },
  methods: {
    scoreTagType(score) {
      if (score >= 90) return "success";
      if (score >= 60) return "primary";
      return "danger";
    },

    fetchData() {
      request
        .get("/admin/searchTeacher", {
          params: {
            ...this.search,
            page: this.page,
            pageSize: this.pageSize,
            type: 1, // 教师类型
          },
        })
        .then((res) => {
          this.rows = res.rows.map((r) => ({ ...r, newPwd: "" }));
          this.total = res.total;
        })
        .catch(() => {
          // 如果上面的接口不支持type参数，使用老接口
          this.getTeacherList();
        });
    },

    handleSearch() {
      this.page = 1;
      this.fetchData();
    },

    resetSearch() {
      this.search.number = "";
      this.search.username = "";
      this.page = 1;
      this.fetchData();
    },

    openAddDialog() {
      this.form = { number: "", username: "", password: "" };
      this.addVisible = true;
    },

    submitAdd() {
      const { number, username, password } = this.form;
      if (!number || !username || !password) {
        return this.$message.warning("请填写完整信息");
      }

      request
        .post("/public/register", {
          number,
          username,
          password,
          type: 1, // 教师类型
        })
        .then(() => {
          this.$message.success("添加成功");
          this.addVisible = false;
          this.fetchData();
        });
    },

    remarkPwd(row) {
      if (!row.newPwd) return this.$message.warning("请输入新密码");

      this.$confirm("确认重置该教师密码？", "提示", {
        type: "warning",
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        customClass: "confirm-dialog",
      }).then(() => {
        request
          .post("/admin/updatePwd", {
            uid: row.uid,
            password: row.newPwd,
          })
          .then(() => {
            row.newPwd = "";
            this.$message.success("密码已重置");
          });
      });
    },
  },
};
</script>

<style scoped>
.teacher-management {
  padding: 20px;
}

.toolbar-card {
  margin-bottom: 20px;
  background: linear-gradient(135deg, #f5f7fa 0%, #f8f9fa 100%);
  border: 1px solid #e4e7ed;
  border-radius: 8px;
  padding: 16px 20px;
}

.compact-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.search-group {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}

.compact-input {
  width: 160px !important;
}

.compact-input >>> .el-input__inner {
  height: 32px;
  line-height: 32px;
  border-radius: 6px;
  border: 1px solid #dcdfe6;
  transition: all 0.3s ease;
  font-size: 13px;
}

.compact-input >>> .el-input__inner:focus {
  border-color: #409eff;
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.1);
}

.compact-input >>> .el-input__prefix {
  display: flex;
  align-items: center;
  left: 8px;
}

.button-group {
  display: flex;
  gap: 8px;
}

.search-btn,
.reset-btn,
.add-btn {
  height: 32px;
  padding: 7px 15px;
  font-size: 13px;
  border-radius: 6px;
  transition: all 0.3s ease;
  font-weight: 500;
}

.search-btn {
  background: linear-gradient(135deg, #409eff 0%, #66b1ff 100%);
  border: none;
}

.search-btn:hover {
  background: linear-gradient(135deg, #66b1ff 0%, #409eff 100%);
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(64, 158, 255, 0.3);
}

.reset-btn {
  border: 1px solid #dcdfe6;
  background: #ffffff;
  color: #606266;
}

.reset-btn:hover {
  border-color: #c0c4cc;
  background: #f5f7fa;
  color: #409eff;
  transform: translateY(-1px);
}

.add-btn {
  background: linear-gradient(135deg, #67c23a 0%, #85ce61 100%);
  border: none;
  padding: 7px 20px;
}

.add-btn:hover {
  background: linear-gradient(135deg, #85ce61 0%, #67c23a 100%);
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(103, 194, 58, 0.3);
}

.table-card {
  margin-top: 20px;
  border-radius: 8px;
  border: 1px solid #e4e7ed;
}

.el-table {
  border-radius: 8px;
}

.el-table >>> th {
  background: #f5f7fa;
  color: #303133;
  font-weight: 600;
  font-size: 13px;
  height: 44px;
}

.el-table >>> td {
  font-size: 13px;
  height: 52px;
}

.el-table >>> .cell {
  padding: 0 12px;
}

.pwd-reset-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.compact-pwd-input {
  width: 140px !important;
}

.compact-pwd-input >>> .el-input__inner {
  height: 28px;
  line-height: 28px;
  border-radius: 4px;
  font-size: 12px;
  border: 1px solid #dcdfe6;
}

.reset-pwd-btn {
  font-size: 12px;
  color: #409eff;
  padding: 5px 8px;
  transition: all 0.2s ease;
  border-radius: 4px;
}

.reset-pwd-btn:hover {
  background: rgba(64, 158, 255, 0.1);
  color: #66b1ff;
}

.el-tag {
  min-width: 40px;
  height: 24px;
  line-height: 22px;
  font-size: 12px;
  border-radius: 4px;
  font-weight: 500;
}

.dialog-btn {
  padding: 7px 20px;
  border-radius: 6px;
  font-size: 13px;
  transition: all 0.3s ease;
}

.dialog-btn:first-child:hover {
  border-color: #c0c4cc;
  background: #f5f7fa;
  color: #409eff;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .compact-toolbar {
    flex-direction: column;
    align-items: stretch;
    gap: 12px;
  }

  .search-group {
    justify-content: space-between;
  }

  .compact-input {
    width: calc(50% - 6px) !important;
  }

  .button-group {
    width: 100%;
    justify-content: flex-end;
  }
}
</style>
