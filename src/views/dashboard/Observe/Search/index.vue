<template>
  <div>
    <el-card>
      <div class="header">
          <span>线上热门搜索</span>
          <el-dropdown>
            <span class="el-dropdown-link">
              <i class="el-icon-more"></i>
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item>黄金糕</el-dropdown-item>
              <el-dropdown-item>狮子头</el-dropdown-item>
              <el-dropdown-item>螺蛳粉</el-dropdown-item>
              <el-dropdown-item disabled>双皮奶</el-dropdown-item>
              <el-dropdown-item divided>蚵仔煎</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
      </div>
      <div class="main">
        <div>
          <el-row :gutter="24">
            <el-col :span="12">
              <lineChart></lineChart>
            </el-col>
            <el-col :span="12">
              <lineChart></lineChart>
            </el-col>
          </el-row>
          <el-table
            style="width: 100%"
            :default-sort="{ prop: 'date', order: 'descending' }"
            border
            class="table"
          >
            <el-table-column label="排名" width="180"> </el-table-column>
            <el-table-column label="搜索关键字" width="180"> </el-table-column>
            <el-table-column label="用户数" sortable> </el-table-column>
            <el-table-column label="周涨幅" sortable> </el-table-column>
          </el-table>
        </div>
      </div>
      <div class="footer">
        <el-pagination
          layout="prev, pager, next"
          :total="1000"
          class="pagination"
        >
        </el-pagination>
      </div>
    </el-card>
  </div>
</template>

<script>
import lineChart from "./lineChart";
export default {
  name: "Search",
  components: { lineChart },
};
</script>

<style scoped>
.el-dropdown-link {
  cursor: pointer;
}
.el-icon-arrow-down {
  font-size: 12px;
}
.header {
  border-bottom: 1px solid #eee;
  height: 50px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.footer {
  display: flex;
  justify-content: center;
  height: 32px;
  align-items: center;
  margin-top: 20px;
}
.table{
    margin-top: 20px;
}
</style>