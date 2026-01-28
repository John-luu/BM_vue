<template>
  <div class="pagination">
    <el-pagination
      background
      layout="total, prev, pager, next"
      :total="total"
      :page-size="pageSize"
      :current-page="current"
      @current-change="change"
    />
  </div>
</template>

<script>
export default {
  name: "BasePagination",
  props: {
    total: {
      type: Number,
      default: 0,
    },
    pageSize: {
      type: Number,
      default: 10,
    },
    current: {
      type: Number,
      default: 1,
    },
  },
  methods: {
    change(page) {
      this.$emit("update:current", page);
      this.$emit("change");
    },
  },
};
</script>

<style scoped>
.pagination {
  margin-top: 16px;
  display: flex;
  justify-content: flex-end;
}
</style>
