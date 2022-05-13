<template>
  <el-card class="box-card">
    <div slot="header" class="clearfix">
      <el-tabs class="tab" v-model="activeName">
        <el-tab-pane label="销售额" name="sale"></el-tab-pane>
        <el-tab-pane label="访问量" name="visits"></el-tab-pane>
      </el-tabs>
      <div class="right">
        <span @click="getDay">今日</span>
        <span @click="getWeek">本周</span>
        <span @click="getMonth">本月</span>
        <span @click="getYear">本年</span>
        <div class="block">
          <el-date-picker
            type="datetimerange"
            range-separator="-"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            size="mini"
            class="data"
            v-model="date"
            value-format="yyyy-MM-dd"
          >
          </el-date-picker>
        </div>
      </div>
    </div>
    <div>
      <el-row :gutter="10">
        <el-col :span="18">
          <div class="charts" ref="charts"></div>
        </el-col>
        <el-col :span="6">
          <h3>门店{{ title }}排名</h3>
          <ul>
            <li>
              <span class="rindex">1</span>
              <span class="rtardemark">肯德基</span>
              <span class="rvalue">123456</span>
            </li>
            <li>
              <span class="rindex">2</span>
              <span class="rtardemark">麦当劳</span>
              <span class="rvalue">123456</span>
            </li>
            <li>
              <span class="rindex">3</span>
              <span class="rtardemark">汉堡王</span>
              <span class="rvalue">123456</span>
            </li>
            <li>
              <span class="rindex">4</span>
              <span class="rtardemark">汉堡王</span>
              <span class="rvalue">123456</span>
            </li>
            <li>
              <span class="rindex">5</span>
              <span class="rtardemark">汉堡王</span>
              <span class="rvalue">123456</span>
            </li>
            <li>
              <span class="rindex">6</span>
              <span class="rtardemark">汉堡王</span>
              <span class="rvalue">123456</span>
            </li>
          </ul>
        </el-col>
      </el-row>
    </div>
  </el-card>
</template>

<script>
import * as echarts from "echarts";
import dayjs, { Dayjs } from "dayjs";
import { mapState } from "vuex";
export default {
  name: "Sale",
  data() {
    return {
      activeName: "sale",
      myCharts:{},
      date: [],
    };
  },
  mounted() {
    this.myCharts = echarts.init(this.$refs.charts);
    this.myCharts.setOption({
      title: {
        text: this.title + "趋势",
      },
      tooltip: {
        trigger: "axis",
        axisPointer: {
          type: "shadow",
        },
      },
      grid: {
        left: "3%",
        right: "4%",
        bottom: "3%",
        containLabel: true,
      },
      xAxis: [
        {
          type: "category",
          data: [],
          axisTick: {
            alignWithLabel: true,
          },
        },
      ],
      yAxis: [
        {
          type: "value",
        },
      ],
      series: [
        {
          name: "Direct",
          type: "bar",
          barWidth: "60%",
          data: [],
          color: "yellowgreen",
        },
      ],
    });
  },
  computed: {
    title() {
      return this.activeName == "sale" ? "销售额" : "访问量";
    },
    ...mapState({
      listState: (state) => state.home.list,
    }),
  },
  watch: {
    title: {
      handler() {
        this.myCharts.setOption({
          title: {
            text: this.title + "趋势",
          },
          xAxis: {
            data:
              this.title == "销售额"
                ? this.listState.orderFullYearAxis
                : this.listState.userFullYearAxis,
          },
          series: [
            {
              name: "Direct",
              type: "bar",
              barWidth: "60%",
              data:
                this.title == "销售额"
                  ? this.listState.orderFullYear
                  : this.listState.userFullYear,
              color: "yellowgreen",
            },
          ],
        });
         
      },
    
    },
    listState() {
      this.myCharts.setOption({
        title: {
          text: this.title + "趋势",
        },
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "shadow",
          },
        },
        grid: {
          left: "3%",
          right: "4%",
          bottom: "3%",
          containLabel: true,
        },
        xAxis: [
          {
            type: "category",
            data: this.listState.orderFullYearAxis,
            axisTick: {
              alignWithLabel: true,
            },
          },
        ],
        yAxis: [
          {
            type: "value",
          },
        ],
        series: [
          {
            name: "Direct",
            type: "bar",
            barWidth: "60%",
            data: this.listState.orderFullYear,
            color: "yellowgreen",
          },
        ],
      });
    },
  },
  methods: {
    getDay() {
      let day = dayjs().format("YYYY-MM-DD");
      console.log(day);
      this.date = [day, day];
    },
    getWeek() {
      let start = dayjs().day(1).format("YYYY-MM-DD");
      let end = dayjs().day(7).format("YYYY-MM-DD");
      //   console.log(start,end);
      this.date = [start, end];
    },
    getMonth() {
      let start = dayjs().startOf("month").format("YYYY-MM-DD");
      let end = dayjs().endOf("month").format("YYYY-MM-DD");
      this.date = [start, end];
    },
    getYear() {
      let start = dayjs().startOf("year").format("YYYY-MM-DD");
      let end = dayjs().endOf("year").format("YYYY-MM-DD");
      this.date = [start, end];
    },
  },
};
</script>
<style >
.el-card__header {
  border-bottom: none;
}
</style>

<style scoped>
.clearfix {
  display: flex;
  justify-content: space-between;
  position: relative;
}
.tab {
  width: 100%;
}
.right {
  position: absolute;
  right: 0;
  display: flex;
}
.right span {
  margin: 0 10px;
  line-height: 28px;
}
.data {
  width: 300px;
  margin: 0 20px;
}
.charts {
  width: 100%;
  height: 300px;
}
ul {
  list-style: none;
  width: 100%;
  height: 300px;
  padding: 0;
}
li {
  margin-left: 10px;
  margin-top: 20px;
}
.rindex {
  border-radius: 50%;
  background-color: black;
  color: white;
  width: 20px;
  text-align: center;
  float: left;
}

.rvalue {
  float: right;
}
.rtardemark {
  margin-left: 20px;
}
</style>