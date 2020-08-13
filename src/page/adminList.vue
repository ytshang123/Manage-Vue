<template>
  <div class="fillcontain">
    <head-top></head-top>
    <div class="table_container">
      <el-table
        :data="tableData"
        style="width: 100%">
        <el-table-column
          prop="name"
          label="用户名"
          width="180">
        </el-table-column>
        <el-table-column
          label="类型"
          width="220">
          <template slot-scope="prop">
            <div>{{prop.row.type===1?'超级管理员':'普通管理员'}}</div>
          </template>
        </el-table-column>
        <el-table-column
          label="状态"
          width="180">
          <template slot-scope="prop">
            <el-tag type="success" v-if="prop.row.state===0">正常</el-tag>
            <el-tag type="danger" v-else>禁用</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="操作">
          <template slot-scope="prop">
            <el-button type="danger" size="small" v-if="prop.row.state===0 && prop.row.type===3" @click="manage(prop.row.adminid,1)">禁用</el-button>
            <el-button type="success" size="small" v-if="prop.row.state===1 && prop.row.type===3" @click="manage(prop.row.adminid,0)">恢复</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="Pagination" style="text-align: left;margin-top: 10px;">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-size="20"
          layout="total, prev, pager, next"
          :total="count">
        </el-pagination>
      </div>
    </div>
  </div>
</template>

<script>
  import headTop from '../components/headTop'
  import axios from 'axios'
  import qs from 'qs'

  export default {
    data() {
      return {
        tableData: [],
        currentRow: null,
        offset: 0,
        limit: 20,
        count: 0,
        currentPage: 1,
      }
    },
    components: {
      headTop,
    },
    mounted() {
      this.initData();
    },
    methods: {
      initData() {
        axios({
          method:'post',
          url:'http://127.0.0.1:9001/admin/list',
          data:qs.stringify({}),
          withCredentials:true
        }).then(response=>{
          if(response.data.result===1){
            this.tableData = response.data.data.list
            this.count=this.tableData.length
          }

        });
      },
      manage(id,type){
        axios({
          method:'post',
          url:'http://127.0.0.1:9001/admin/manage',
          data:qs.stringify({id: id, type:type}),
          withCredentials:true
        }).then(response=>{
          if(response.data.result===1){
            this.initData()
          }
        });
      },
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
      },
      handleCurrentChange(val) {
        this.currentPage = val;
        this.offset = (val - 1) * this.limit;
        // this.getAdmin()
      },
    },
  }
}
</script>

<style lang="less">
  @import '../style/mixin';

  .table_container {
    padding: 20px;
  }
</style>


