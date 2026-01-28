<template>
  <div>
    <el-select
      v-if="areaRows"
      size="mini"
      v-model="areaName"
      @change="changeArea"
    >
      <el-option
        v-for="item in areaRows"
        :key="item.aid"
        :label="item.areaName"
        :value="item.aid"
      >
      </el-option>
      <!-- 增加区域的选项 -->
      <el-option v-if="canAdd" label="增加区域" value="addArea" />
    </el-select>
    <!-- 弹窗，用于增加区域 -->
    <el-dialog :visible.sync="dialogVisible" title="添加区域">
      <el-form :model="form" ref="form" label-width="80px">
        <el-form-item label="区域名称">
          <el-input v-model="form.areaName" placeholder="请输入区域名称" />
        </el-form-item>
        <el-form-item label="区域缩写">
          <el-input v-model="form.subName" placeholder="请输入区域缩写" />
        </el-form-item>
        <el-form-item label="空白表格尺寸">
          <!-- 增大输入框 -->
          <el-input-number
            v-model="form.rows"
            :min="1"
            label="行数"
            placeholder="请输入行数"
            style="width: 140px; font-size: 16px"
            controls-position="right"
            :step="1"
          />
          <el-input-number
            v-model="form.columns"
            :min="1"
            label="列数"
            placeholder="请输入列数"
            style="width: 140px; font-size: 16px; margin-left: 10px"
            controls-position="right"
            :step="1"
          />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="createArea">创建</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import request from "@/req";
export default {
  name: "ToggleArea",
  props: {
    areaRows: {
      type: Array,
    },
    canAdd: {
      type: Boolean,
      default: false, // 默认不允许新增
    },
  },
  data() {
    return {
      areaName: null,
      curIndex: 0,
      dialogVisible: false,
      form: {
        areaName: "",
        subName: "",
        rows: 1,
        columns: 1,
      },
    };
  },
  methods: {
    getArea() {
      return this.areaRows[this.curIndex];
    },
    changeArea(value) {
      if (value === "addArea") {
        this.dialogVisible = true;
      } else {
        this.curIndex = this.areaRows.findIndex((item) => item.aid === value);
        this.$emit("changeArea", this.getArea());
      }
    },
    createArea() {
      // 提交表单数据到后端接口
      request
        .post("/admin/addArea", {
          areaName: this.form.areaName,
          subName: this.form.subName,
          rows: this.form.rows,
          columns: this.form.columns,
        })
        .then((res) => {
          this.$message.success("区域创建成功");
          this.dialogVisible = false;
          // 关键：后端应该返回新创建的区域信息

          // 将新创建的区域对象直接传递给父组件
          // 注意：res.data 应该包含 aid, areaName, subName, rows, columns
          this.$emit("changeArea", res.data);
          console.log("res.data===", res.data);
          // 同时触发 areaCreated 事件，让父组件刷新区域列表
          this.$emit("areaCreated", res.data);
          // 清空表单
          this.form = {
            areaName: "",
            subName: "",
            rows: 1,
            columns: 1,
          };
        })
        .catch((error) => {
          this.$message.error("区域创建失败");
        });
    },
  },
  created() {
    if (this.areaRows && this.areaRows.length > 0) {
      this.areaName = this.areaRows[0].aid;
      this.curIndex = 0;
    }
  },
  watch: {
    areaRows: {
      immediate: true,
      handler(val) {
        if (val && val.length > 0) {
          this.areaName = val[0].aid;
          this.curIndex = 0;
          this.$emit("changeArea", this.getArea());
        }
      },
    },
  },
};
</script>

<style scoped></style>
