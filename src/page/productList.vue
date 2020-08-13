<template>
  <div class="fillcontain">
    <head-top></head-top>
    <div class="table_container">
      <el-table
        :data="tableData"
        style="width: 100%">
        <el-table-column
          label="图片" width="100">
          <template slot-scope="scope">
            <div style="width: 90%">
              <img style="width: 90%" :src="'http://127.0.0.1:9001/product/img/'+scope.row.img" />
            </div>
          </template>
        </el-table-column>
        <el-table-column
          label="名称"
          width="200"
          prop="name">
        </el-table-column>
        <el-table-column
          label="分类"
          prop="type">
        </el-table-column>
        <el-table-column
          label="标签"
          prop="tag">
        </el-table-column>
        <el-table-column
          label="数量"
          prop="amount">
        </el-table-column>
        <el-table-column
          label="单价"
          prop="price">
        </el-table-column>
        <el-table-column
          label="状态">
          <template slot-scope="scope">
            <el-tag type="danger" v-if="scope.row.off===1">已下架</el-tag>
            <el-tag type="success" v-else>上架中</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="160">
          <template slot-scope="scope">
            <el-button
              size="small"
              @click="handleEdit(scope.row)">编辑
            </el-button>
            <el-button
              v-if="scope.row.off===0"
              size="small"
              type="danger"
              @click="manage(scope.row.productid, 1)">下架
            </el-button>
            <el-button
              v-else
              size="small"
              type="success"
              @click="manage(scope.row.productid, 0)">上架
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="Pagination">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-size="20"
          layout="total, prev, pager, next"
          :total="count">
        </el-pagination>
      </div>
      <el-dialog title="修改商品信息" v-model="dialogFormVisible">
        <el-form :model="selectTable">
          <el-form-item label="商品名称" label-width="100px">
            <el-input v-model="selectTable.name" auto-complete="off"></el-input>
          </el-form-item>
          <el-form-item label="商品分类" label-width="100px">
            <el-input v-model="selectTable.type"></el-input>
          </el-form-item>
          <el-form-item label="商品标签" label-width="100px">
            <el-input v-model="selectTable.tag"></el-input>
          </el-form-item>
          <el-form-item label="商品数量" label-width="100px">
            <el-input-number v-model="selectTable.amount"></el-input-number>
          </el-form-item>
          <el-form-item label="商品单价" label-width="100px">
            <el-input-number :precision="2" v-model="selectTable.price"></el-input-number>
          </el-form-item>
          <el-form-item label="商品图片" label-width="100px">
            <el-upload
              class="avatar-uploader"
              action="/ttt"
              :http-request="upload"
              :show-file-list="false">
              <img v-if="selectTable.img" :src="selectTableImg" class="avatar">
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="update">确 定</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<script>
  import headTop from '../components/headTop'
  import {baseUrl, baseImgPath} from '@/config/env'
  import axios from 'axios'
  import qs from 'qs'
  export default {
    data() {
      return {
        baseUrl,
        baseImgPath,
        restaurant_id: null,
        city: {},
        offset: 0,
        limit: 20,
        count: 0,
        tableData: [],
        currentPage: 1,
        selectTable: {},
        dialogFormVisible: false,
        menuOptions: [],
        selectMenu: {},
        selectIndex: null,
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
        specsFormVisible: false,
        expendRow: [],
        file: undefined,
        selectTableImg:undefined
      }
    },
    mounted() {
      // this.restaurant_id = this.$route.query.restaurant_id;
      this.initData();
    },
    computed: {
      specs: function () {
        let specs = [];
        if (this.selectTable.specfoods) {
          this.selectTable.specfoods.forEach(item => {
            specs.push({
              specs: item.specs_name,
              packing_fee: item.packing_fee,
              price: item.price,
            })
          })
        }
        return specs
      }
    },
    components: {
      headTop,
    },
    methods: {
      initData() {
        axios({
          method:'post',
          url:'http://127.0.0.1:9001/admin/product/list',
          data:qs.stringify({}),
          withCredentials:true
        }).then(response=>{
          if(response.data.result===1){
            this.tableData = response.data.data.list
            this.count=this.tableData.length
          }
        });
      },

      tableRowClassName(row, index) {
        if (index === 1) {
          return 'info-row';
        } else if (index === 3) {
          return 'positive-row';
        }
        return '';
      },
      addspecs() {
        this.specs.push({...this.specsForm});
        this.specsForm.specs = '';
        this.specsForm.packing_fee = 0;
        this.specsForm.price = 20;
        this.specsFormVisible = false;
      },
      deleteSpecs(index) {
        this.specs.splice(index, 1);
      },
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
      },
      handleCurrentChange(val) {
        this.currentPage = val;
        this.offset = (val - 1) * this.limit;
      },
      expand(row, status) {
        if (status) {
          this.getSelectItemData(row)
        } else {
          const index = this.expendRow.indexOf(row.index);
          this.expendRow.splice(index, 1)
        }
      },
      handleEdit(row) {
        this.selectTable = row;
        this.dialogFormVisible = true;
        this.selectTableImg = 'http://127.0.0.1:9001/product/img/' + this.selectTable.img;
      },
      handleSelect(index) {
        this.selectIndex = index;
        this.selectMenu = this.menuOptions[index];
      },
      manage(id,type){
        axios({
          method:'post',
          url:'http://127.0.0.1:9001/admin/product/manage',
          data:qs.stringify({id: id, type:type}),
          withCredentials:true
        }).then(response=>{
          if(response.data.result===1){
            this.initData()
          }
        });
      },
      handleServiceAvatarScucess(res, file) {
        if (res.status == 1) {
          this.selectTable.image_path = res.image_path;
        } else {
          this.$message.error('上传图片失败！');
        }
      },
      beforeAvatarUpload(file) {
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
      upload(option){
        this.file = option.file;
        let father = this;
        let reader = new FileReader();
        reader.readAsDataURL(this.file)
        reader.onload = function(e){
          father.selectTableImg = reader.result;
        }
      },
      update(){
        let param = new FormData(); //创建form对象
        if(this.file) param.append('file',this.file);//通过append向form对象添加数据
        for(let key in this.selectTable) param.append(key, this.selectTable[key]);
        param.append("id",this.selectTable.productid)
        axios({
          method:'post',
          url:'http://127.0.0.1:9001/admin/product/update',
          data:param,
          headers:{'Content-Type':'multipart/form-data'},
          withCredentials:true
        }).then(response=>{
          if(response.data.result===1){
            this.$message({
              type: 'success',
              message: '更新商品信息成功'
            });
            this.file=undefined;
            this.dialogFormVisible = false;
            this.initData()
          }
        });
      },
    },
  }
</script>

<style lang="less">
  @import '../style/mixin';

  .demo-table-expand {
    font-size: 0;
  }

  .demo-table-expand label {
    width: 90px;
    color: #99a9bf;
  }

  .demo-table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
  }

  .table_container {
    padding: 20px;
  }

  .Pagination {
    display: flex;
    justify-content: flex-start;
    margin-top: 8px;
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
</style>
