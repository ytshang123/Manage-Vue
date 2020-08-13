<template>
  <div>
    <head-top></head-top>
    <el-row style="margin-top: 20px;">
      <el-col :span="12" :offset="4">
        <el-form :model="formData" :rules="rules" ref="formData" label-width="110px" class="demo-formData">
          <el-form-item label="用户名" prop="name">
            <el-input v-model="formData.name"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="passwd">
            <el-input v-model="formData.passwd"></el-input>
          </el-form-item>
          <el-form-item label="类型">
            <div>
              <el-radio v-model="formData.type" label="1" border>超级管理员</el-radio>
              <el-radio v-model="formData.type" label="3" border>普通管理员</el-radio>
            </div>
          </el-form-item>
          <el-form-item class="button_submit">
            <el-button type="primary" @click="submitForm()">创建</el-button>
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  import headTop from '@/components/headTop'
  import axios from 'axios'
  import qs from 'qs'

  export default {
    data() {
      return {
        city: {},
        formData: {
          name: '', 
          passwd: '', 
          type: '3',
        },
        rules: {
          name: [
            {required: true, message: '请输入用户名', trigger: 'blur'},
          ],
          passwd: [
            {required: true, message: '请输入密码', trigger: 'blur'}
          ]
        }
      }
    },
    components: {
      headTop,
    },
    methods: {
      submitForm() {
        this.$refs['formData'].validate(async (valid) => {
          if (valid) {
            axios({
              method:'post',
              url:'http://127.0.0.1:9001/admin/addAdmin',
              data:qs.stringify(this.formData),
              withCredentials:true
            }).then(response=>{
              if(response.data.result===1){
                this.$message({
                  type: 'success',
                  message: '添加成功'
                });
                this.formData={
                  name: '', 
                  passwd: '', /
                  type: '3',
                }
              }else{
                this.$message({
                  type: 'error',
                  message: response.data.message
                });
              }
            });
          } else {
            this.$notify.error({
              title: '错误',
              message: '请检查输入是否正确',
              offset: 100
            });
            return false;
          }
        });
      },
    }
  }
}
</script>

<style lang="less">
  @import '../style/mixin';

  .button_submit {
    text-align: center;
  }

  .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }

  .avatar-uploader .el-upload:hover {
    border-color: #20a0ff;
  }

  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 120px;
    height: 120px;
    line-height: 120px;
    text-align: center;
  }

  .avatar {
    width: 120px;
    height: 120px;
    display: block;
  }

  .el-table .info-row {
    background: #c9e5f5;
  }

  .el-table .positive-row {
    background: #e2f0e4;
  }
</style>
