<template>
  <el-row :gutter="20">
    <el-col :span="6">
      <el-card shadow="hover" :body-style="{ padding: '0px' }">
        <div class="card-content">
          <div class="card-left">
            <el-icon><user /></el-icon>
          </div>
          <div class="card-right">
            <div class="card-num">{{ userCount }}</div>
            <div>用户总数</div>
          </div>
        </div>
      </el-card>
    </el-col>
    <el-col :span="6">
      <el-card shadow="hover" :body-style="{ padding: '0px' }">
        <div class="card-content">
          <div class="card-left">
            <el-icon><calendar /></el-icon>
          </div>
          <div class="card-right">
            <div class="card-num">{{ courseCount }}</div>
            <div>课程总数</div>
          </div>
        </div>
      </el-card>
    </el-col>
    <el-col :span="6">
      <el-card shadow="hover" :body-style="{ padding: '0px' }">
        <div class="card-content">
          <div class="card-left">
            <el-icon><document /></el-icon>
          </div>
          <div class="card-right">
            <div class="card-num">{{ channelCount }}</div>
            <div>论坛数量</div>
          </div>
        </div>
      </el-card>
    </el-col>
    <el-col :span="6">
      <el-card shadow="hover" :body-style="{ padding: '0px' }">
        <div class="card-content">
          <div class="card-left">
            <el-icon><ChatDotRound /></el-icon>
          </div>
          <div class="card-right">
            <div class="card-num">{{ messageCount }}</div>
            <div>消息条数</div>
          </div>
        </div>
      </el-card>
    </el-col>
  </el-row>
  <el-row :gutter="20">
    <el-col :span="12">
      <h3>用户性别比例</h3>
      <el-card class="cav-info" shadow="hover" :body-style="{ padding: '0px' }" id="userSex"></el-card>
    </el-col>
    <el-col :span="12">
      <h3>课程类型</h3>
      <el-card class="cav-info" shadow="hover" :body-style="{ padding: '0px' }" id="courseStyle"></el-card>
    </el-col>
  </el-row>
  <br/>
  <h6 align="center">上海财经大学 | 联系我们 | <a style="text-decoration" href="https://beian.miit.gov.cn/">宁ICP备2022000586号</a></h6>
</template>
<script lang="ts" setup>
// import { ref, reactive, getCurrentInstance } from "vue";
import { ref, reactive } from "vue";
import * as echarts from "echarts";
import { Calendar, Document, ChatDotRound, User} from "@element-plus/icons-vue";
import { HttpManager } from "@/api/index";

const userCount = ref(0);
const courseCount = ref(0);
const channelCount = ref(0);
const messageCount = ref(0);
const userSex = reactive({
  series: [
    {
      type: "pie",
      data: [
        {
          value: 0,
          name: "男",
        },
        {
          value: 0,
          name: "女",
        },
      ],
    },
  ],
});
const courseStyle = reactive({
  xAxis: {
    type: "category",
    data: ["必修课", "选修课", "通识模块", "英语模块", "通识选修", "通识限选"],
  },
  yAxis: {
    type: "value",
  },
  series: [
    {
      data: [0, 0, 0, 0, 0, 0],
      type: "bar",
      barWidth: "20%",
    },
  ],
});

function setSex(sex, arr) {
  let value = 0;
  const name = sex === 0 ? "男" : "女";
  for (let item of arr) {
    if (sex === item.sex) {
      value++;
    }
  }
  return { value, name };
}
HttpManager.getAllUser().then((res) => {
  const result = res as ResponseBody;
  userSex.series[0].data.push(setSex(0, result.data));
  userSex.series[0].data.push(setSex(1, result.data));

  const userSexChart = echarts.init(document.getElementById("userSex"));
  userSexChart.setOption(userSex);
});

HttpManager.getUsers().then((res) => {
  userCount.value = (res as ResponseBody).data.length;

})

HttpManager.getAllCourse().then((res) => {
  const result = res as ResponseBody;
  courseCount.value = result.data.length;
  for (let item of result.data) {
    for (let i = 0; i < courseStyle.xAxis.data.length; i++) {
      if (courseStyle.xAxis.data[i] == "通识限选" && item.type.includes("（")) {
        courseStyle.series[0].data[i]++;
      } else if (item.type.includes(courseStyle.xAxis.data[i])) {
        courseStyle.series[0].data[i]++;
      }
    }
  }
  const songStyleChart = echarts.init(document.getElementById("courseStyle"));
  songStyleChart.setOption(courseStyle);
});

HttpManager.getChannels().then((res) => {
  const result = res as ResponseBody;
  channelCount.value = result.data.length;
  for (var i = 0; i < channelCount.value; i++) {
    HttpManager.getMessage(result.data[i].cid).then((res) => {
      messageCount.value += (res as ResponseBody).data.length;
    });
  }
});

</script>

<style scoped>
.card-content {
  display: flex;
  align-items: center;
  justify-content: space-around;
  height: 100px;
  padding-left: 20%;
  text-align: center;
}

.card-left {
  display: flex;
  font-size: 3rem;
}

.card-right {
  flex: 1;
  font-size: 14px;
}

.card-num {
  font-size: 30px;
  font-weight: bold;
}

h3 {
  margin: 10px 0;
  text-align: center;
}
.cav-info {
  border-radius: 6px;
  overflow: hidden;
  height: 250px;
  background-color: white;
}
</style>
