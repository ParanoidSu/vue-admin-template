<template>
  <div>
    <el-card>
      <div class="card_header">
        <span>{{ title }}</span>
        <i class="el-icon-info"></i>
      </div>
      <div class="card_content">126060</div>
      <div class="card_charts">
          <slot name="charts"></slot>
      </div>
      <div class="card_footer">
          <slot name="footer"></slot>
      </div>
    </el-card>
  </div>
</template>

<script>
export default {
  name: "Detail",
  props: ["title"],
};
</script>

<style>
.card_header {
  display: flex;
  justify-content: space-between;
  color: #d9d9d9;
}
.card_content {
  font-size: 30px;
  padding: 10px;
}
.card_charts {
    height: 50px ;
    font-size: 15px;
    text-align: center;
}
.card_footer {
    border-top: 1px solid #eee;
    padding-top: 10px;
}
</style>