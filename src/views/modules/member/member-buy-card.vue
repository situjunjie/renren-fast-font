<template>
  <div class="mod-config">
    <!--办卡弹窗 begin-->
    <el-dialog title="办卡" :visible.sync="buyCardVisible">
  <el-form :model="buyCardForm">
    <el-form-item label="卡类型" :label-width="formLabelWidth">
      <el-select v-model="buyCardForm.cardType" placeholder="请选择卡类型">
        <el-option label="月卡 100元/月" value="1"></el-option>
        <el-option label="季卡 200元/季" value="2"></el-option>
        <el-option label="年卡 600元/年" value="3"></el-option>
      </el-select>
    </el-form-item>
  </el-form>
  <div slot="footer" class="dialog-footer">
    <el-button @click="buyCardVisible = false">取 消</el-button>
    <el-button type="primary" @click="buyCard()">确 定</el-button>
  </div>
</el-dialog>
    <!--办卡弹窗 end-->

    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.name" placeholder="会员姓名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.mobile" placeholder="手机号" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <!-- <el-button v-if="isAuth('member:member:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('member:member:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button> -->
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <!-- <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="主键">
      </el-table-column> -->
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="会员姓名">
      </el-table-column>
      <!-- <el-table-column
        prop="gender"
        header-align="center"
        align="center"
        label="会员性别">
      </el-table-column>
      <el-table-column
        prop="age"
        header-align="center"
        align="center"
        label="年龄">
      </el-table-column>
      <el-table-column
        prop="birth"
        header-align="center"
        align="center"
        label="生日">
      </el-table-column>
      <el-table-column
        prop="cardType"
        header-align="center"
        align="center"
        label="卡类型">
      </el-table-column> -->
      <el-table-column
        prop="mobile"
        header-align="center"
        align="center"
        label="电话号码">
      </el-table-column>
      <el-table-column
        prop="expirationTime"
        header-align="center"
        align="center"
        label="到期时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="openBuyCardForm(scope.row.id)">购卡</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
  </div>
</template>

<script>
  
  export default {
    data () {
      return {
        dataForm: {
          name: '',
          mobile: ''
        },
        buyCardForm:{
          memberId:0,
          cardType:0
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        buyCardVisible: false
      }
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/member/member/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'name': this.dataForm.name,
            'mobile': this.dataForm.mobile
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/member/member/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      },

      //打开购卡弹窗
      openBuyCardForm(id){
        console.log("打开购卡弹窗 memberId = "+id);
        this.buyCardVisible = true
        this.buyCardForm.memberId = id
      }
,
      //请求后端办卡方法
      buyCard(){
        this.$http({
          url: this.$http.adornUrl('/member/member/buyCard'),
          method: 'post',
          data: this.buyCardForm
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.$message.error(data.msg)
          }
          this.dataListLoading = false
        })
        this.buyCardVisible = false
      }
    }
  }
</script>
