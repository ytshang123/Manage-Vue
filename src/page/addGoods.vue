<template>
  <div>
    <head-top></head-top>
    <el-row style="margin-top: 20px;">
      <el-col :span="14" :offset="4">
        <header class="form_header">添加商品</header>
        <el-form :model="foodForm" :rules="foodrules" ref="foodForm" label-width="110px" class="form food_form">
          <el-form-item label="商品名称" label-width="100px">
            <el-input v-model="foodForm.name" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="商品分类" label-width="100px">
            <el-input v-model="foodForm.type"></el-input>
          </el-form-item>
          <el-form-item label="商品标签" label-width="100px">
            <el-input v-model="foodForm.tag"></el-input>
          </el-form-item>
          <el-form-item label="商品数量" label-width="100px">
            <el-input-number v-model="foodForm.amount"></el-input-number>
          </el-form-item>
          <el-form-item label="商品单价" label-width="100px">
            <el-input-number :precision="2" v-model="foodForm.price"></el-input-number>
          </el-form-item>
          <el-form-item label="商品图片" label-width="100px">
            <el-upload
              class="avatar-uploader"
              action="/ttt"
              :http-request="upload"
              :show-file-list="false">
              <img v-if="foodFormImg" :src="foodFormImg" class="avatar">
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="addFood('foodForm')">确认添加商品</el-button>
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  import headTop from '@/components/headTop'
  import {getCategory, addCategory, addFood} from '@/api/getData'
  import {baseUrl, baseImgPath} from '@/config/env'
  import axios from 'axios'
  import qs from 'qs'

  export default {
    data() {
      return {
        baseUrl,
        baseImgPath,
        foodFormImg: undefined,
        restaurant_id: 1,
        categoryForm: {
          categoryList: [],
          categorySelect: '',
          name: '',
          description: '',
        },
        foodForm: {

        },
        foodrules: {
          name: [
            {required: true, message: '请输入商品名称', trigger: 'blur'},
          ],
        },
        showAddCategory: false,
        foodSpecs: 'one',
        dialogFormVisible: false,
        specsForm: {
          specs: '',
          packing_fee: 0,
          price: 20,
        },
        specsFormrules: {
          specs: [
            {required: true, message: '请输入规格', trigger: 'blur'},
          ],
        },
        file: undefined
      }
    },
    components: {
      headTop,
    },
    methods: {
      upload(option){
        this.file = option.file;
        let father = this;
        let reader = new FileReader();
        reader.readAsDataURL(this.file)
        reader.onload = function(e){
          father.foodFormImg = reader.result;
        }
      },
      addCategoryFun() {
        this.showAddCategory = !this.showAddCategory;
      },
      submitcategoryForm(categoryForm) {
        this.$refs[categoryForm].validate(async (valid) => {
          if (valid) {
            const params = {
              name: this.categoryForm.name,
              description: this.categoryForm.description,
              restaurant_id: this.restaurant_id,
            }
            try {
              const result = await addCategory(params);
              if (result.status == 1) {
                this.initData();
                this.categoryForm.name = '';
                this.categoryForm.description = '';
                this.showAddCategory = false;
                this.$message({
                  type: 'success',
                  message: '添加成功'
                });
              }
            } catch (err) {
              console.log(err)
            }
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
      uploadImg(res, file) {
        if (res.status === 1) {
          this.foodForm.image_path = res.image_path;
        } else {
          this.$message.error('上传图片失败！');
        }
      },
      beforeImgUpload(file) {
        const isRightType = (file.type === 'image/jpeg') || (file.type === 'image/png');
        const isLt2M = file.size / 1024 / 1024 < 2;

        if (!isRightType) {
          this.$message.error('上传头像图片只能是 JPG 格式!');
        }
        if (!isLt2M) {
          this.$message.error('上传头像图片大小不能超过 2MB!');
        }
        return isRightType && isLt2M;
      },
      addspecs() {
        this.foodForm.specs.push({...this.specsForm});
        this.specsForm.specs = '';
        this.specsForm.packing_fee = 0;
        this.specsForm.price = 20;
        this.dialogFormVisible = false;
      },
      handleDelete(index) {
        this.foodForm.specs.splice(index, 1);
      },
      tableRowClassName(row, index) {
        if (index === 1) {
          return 'info-row';
        } else if (index === 3) {
          return 'positive-row';
        }
        return '';
      },
      addFood(foodForm) {
        this.$refs[foodForm].validate(async (valid) => {
          if (valid) {
            let param = new FormData(); //创建form对象
            if(this.file) param.append('file',this.file);//通过append向form对象添加数据
            for(let key in this.foodForm) param.append(key, this.foodForm[key]);
            axios({
              method:'post',
              url:'http://127.0.0.1:9001/admin/product/add',
              data:param,
              headers:{'Content-Type':'multipart/form-data'},
              withCredentials:true
            }).then(response=>{
              if(response.data.result===1){
                this.$message({
                  type: 'success',
                  message: '添加商品信息成功'
                });
                this.file=undefined;
                this.foodForm={}
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
      }
    }
  }
}
</script>

<style lang="less">
  @import '../style/mixin';

  .form {
    min-width: 400px;
    margin-bottom: 30px;

    &:hover {
      box-shadow: 0 0 8px 0 rgba(232, 237, 250, .6), 0 2px 4px 0 rgba(232, 237, 250, .5);
      border-radius: 6px;
      transition: all 400ms;
    }
  }

  .food_form {
    border: 1px solid #eaeefb;
    padding: 10px 10px 0;
  }

  .form_header {
    text-align: center;
    margin-bottom: 10px;
  }

  .category_select {
    border: 1px solid #eaeefb;
    padding: 10px 30px 10px 10px;
    border-top-right-radius: 6px;
    border-top-left-radius: 6px;
  }

  .add_category_row {
    height: 0;
    overflow: hidden;
    transition: all 400ms;
    background: #f9fafc;
  }

  .showEdit {
    height: 185px;
  }

  .add_category {
    background: #f9fafc;
    padding: 10px 30px 0px 10px;
    border: 1px solid #eaeefb;
    border-top: none;
  }

  .add_category_button {
    text-align: center;
    line-height: 40px;
    border-bottom-right-radius: 6px;
    border-bottom-left-radius: 6px;
    border: 1px solid #eaeefb;
    border-top: none;
    transition: all 400ms;

    &:hover {
      background: #f9fafc;

      span, .edit_icon {
        color: #20a0ff;
      }
    }

    span {
      .sc(14px, #999);
      transition: all 400ms;
    }

    .edit_icon {
      color: #ccc;
      transition: all 400ms;
    }
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

  .cell {
    text-align: center;
  }
</style>
