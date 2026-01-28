<template>
  <div class="student-management">
    <!-- 顶部搜索区 -->
    <el-card class="toolbar-card" shadow="never">
      <el-form :inline="true" class="search-form">
        <el-form-item label="学工号">
          <el-input
            v-model="search.number"
            clearable
            placeholder="请输入学工号"
          />
        </el-form-item>

        <el-form-item label="姓名">
          <el-input
            v-model="search.username"
            clearable
            placeholder="请输入姓名"
          />
        </el-form-item>

        <el-form-item>
          <el-button type="primary" icon="el-icon-search" @click="handleSearch"
            >搜索</el-button
          >
          <el-button icon="el-icon-refresh" @click="resetSearch"
            >重置</el-button
          >
        </el-form-item>

        <el-form-item style="margin-left: auto">
          <el-button type="success" @click="openAddDialog" icon="el-icon-plus">
            新增学生
          </el-button>
        </el-form-item>
      </el-form>
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
            <el-input
              size="small"
              v-model="row.newPwd"
              placeholder="新密码"
              style="width: 140px; margin-right: 8px"
            />
            <el-button type="text" size="small" @click="remarkPwd(row)">
              重置
            </el-button>
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

    <!-- 新增学生弹窗 -->
    <el-dialog title="新增学生" :visible.sync="addVisible" width="400px">
      <el-form label-width="80px" :model="form" autocomplete="off">
        <el-form-item label="学工号">
          <el-input v-model="form.number" autocomplete="off" />
        </el-form-item>
        <el-form-item label="姓名">
          <el-input v-model="form.username" autocomplete="off" />
        </el-form-item>
        <el-form-item label="密码">
          <el-input
            v-model="form.password"
            show-password
            autocomplete="new-password"
          />
        </el-form-item>
      </el-form>

      <span slot="footer">
        <el-button @click="addVisible = false">取消</el-button>
        <el-button type="primary" @click="submitAdd">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import request from "@/req";
import BasePagination from "@/components/BasePagination";

export default {
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
        .get("/admin/searchUser", {
          params: {
            ...this.search,
            page: this.page,
            pageSize: this.pageSize,
          },
        })
        .then((res) => {
          this.rows = res.rows.map((r) => ({ ...r, newPwd: "" }));
          this.total = res.total;
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
      // 每次打开新增弹窗时，表单清空
      this.form = { number: "", username: "", password: "" };
      console.log("form.username==", this.form.username);
      console.log("form.password==", this.form.password);
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
          type: 0,
        })
        .then(() => {
          this.$message.success("添加成功");
          this.addVisible = false;
          this.fetchData();
        });
    },

    remarkPwd(row) {
      if (!row.newPwd) return this.$message.warning("请输入新密码");

      this.$confirm("确认重置该学生密码？", "提示", { type: "warning" }).then(
        () => {
          request
            .post("/admin/updatePwd", {
              uid: row.uid,
              password: row.newPwd,
            })
            .then(() => {
              row.newPwd = "";
              this.$message.success("密码已重置");
            });
        },
      );
    },
  },
};
</script>

<style scoped>
.student-management {
  padding: 20px;
}

.toolbar-card {
  margin-bottom: 20px;
}

.search-form .el-form-item {
  margin-right: 20px;
}

.el-input {
  width: 180px;
}

.el-button {
  font-size: 14px;
  margin-left: 10px;
}

.table-card {
  margin-top: 20px;
}

.el-table {
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.el-table-column {
  font-size: 14px;
}

.el-tag {
  margin: 0 5px;
}

.el-button[type="text"] {
  font-size: 12px;
}

.el-dialog {
  border-radius: 8px;
}

.el-input[placeholder] {
  color: #909399;
}

.el-form-item {
  margin-bottom: 15px;
}
</style>
