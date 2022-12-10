<template>
  <div class="mod-config">
    <!--选择教练 begin-->
    <el-dialog
    title="教练安排"
    :close-on-click-modal="false"
    :visible.sync="coachArrangeVisible">
    <el-transfer @change="arrangeCoachChange()" v-model="coachIds" :data="coach"></el-transfer>
  </el-dialog>
  <!--选择教练 end-->
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('course:course:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('course:course:delete')" type="danger" @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column type="selection" header-align="center" align="center" width="50">
      </el-table-column>

      <el-table-column prop="name" header-align="center" align="center" label="课程名称">
      </el-table-column>
      <el-table-column prop="time" header-align="center" align="center" label="课程时长(分钟)">
      </el-table-column>
      <el-table-column prop="tips" header-align="center" align="center" label="注意事项">
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="coachArrangeHandle(scope.row.id)">教练安排</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]" :page-size="pageSize" :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>

  </div>
</template>

<script>
import { SSL_OP_SSLEAY_080_CLIENT_DH_BUG } from 'constants'
import AddOrUpdate from './course-add-or-update'
import { log } from 'util'
export default {
  data() {
    return {
      dataForm: {
        key: ''
      },
      coachRelationForm:{
        coachIds:[],
        courseId:0
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      coachArrangeVisible: false,
      coachIds:[], //已选择的教练列表
      coach:[]

    }
  },
  components: {
    AddOrUpdate
  },
  activated() {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/course/course/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'key': this.dataForm.key
        })
      }).then(({ data }) => {
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
    sizeChangeHandle(val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val
      this.getDataList()
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id)
      })
    },
    // 删除
    deleteHandle(id) {
      var ids = id ? [id] : this.dataListSelections.map(item => {
        return item.id
      })
      this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/course/course/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
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
    //安排教练
    coachArrangeHandle(id) {
      console.log('教练安排',id);
      this.coachArrangeVisible = true
      this.coachRelationForm.courseId = id
      this.$http({
                url:this.$http.adornUrl(`/course/coursecoachrelation/transferData`),
                data:id,
                method:'post'
            }).then((data) => {
              console.log(data);
                if (data.data && data.data.code === 0) {
                  console.log('教练列表',data.data.data.leftData);
                  this.coach = data.data.data.leftData;
                  console.log('已选择的教练列表',data.data.data.rigthData);
                  // this.coachIds = Object.keys(data.data.data.rightData).map(c=>{c.key});
                  this.coachIds = []
                  for (let index = 0; index < data.data.data.rigthData.length; index++) {
                    const element = data.data.data.rigthData[index];
                    this.coachIds.push(element.key)
                  }
                  console.log('coachIds',this.coachIds);
              }else{
                this.$message.error(data.data.msg)
              }
            });
    },
    //选择教练落库
    arrangeCoachChange(){
      console.log('穿梭框change方法');
      this.coachRelationForm.coachIds = this.coachIds
      this.$http({
                url:this.$http.adornUrl(`/course/coursecoachrelation/updateBath`),
                data:this.coachRelationForm,
                method:'post'
            }).then((data) => {
              console.log(data);
                if (data.data && data.data.code === 0) {
                  this.$message.success('操作成功')
              }else{
                this.$message.error(data.data.msg)
              }
            });
    }
  }
}
</script>
