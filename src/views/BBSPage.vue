<template>
  <el-row :gutter="20"  class="el-row" type="flex" >
    <el-col :span="8">
      <el-card class="box-card" style="min-height: 20px;" align="left" onclick="">
        <search></search>
        <chatlist></chatlist>
      </el-card>
    </el-col>
    <el-col :span="16">
      <div class="chatbox">
        <message></message>
        <v-text></v-text>
      </div>
    </el-col>
  </el-row>
</template>

<script>
import search from '../views/bbs/search'
import chatlist from '../views/bbs/chatlist'
import message from '../views/bbs/message'
import vText from '../views/bbs/text'
import Search from './bbs/search.vue'
import Chatlist from './bbs/chatlist.vue'
export default {
  components: {
    search,
    chatlist,
    message,
    vText
  }
}
</script>

<style type="text/css" scoped>
  .el-card {
    min-width: 100%;
    height: 100%;
    margin-right: 20px;
  }
  .box-card {
    background-color:rgb(230,230,230);
  }
  .el-card:hover{
    margin-top: -5px;
  }
  .el-row {
    margin-bottom: 20px;
    display: flex;
    flex-wrap: wrap;
    height: 100%;
  }
  .el-col {
    border-radius: 4px;
    align-items: stretch;
    margin-bottom: 40px;
  }
  .chat-box {
    display: flex
  }
</style>