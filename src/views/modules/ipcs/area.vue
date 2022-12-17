<template>
  <div class="mod-config">
    <!-- <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('ipcs:area:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('ipcs:area:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form> -->
    <el-tree
  :props="props"
  :load="loadNode"
  node-key="id"
  lazy
  >
  <span class="custom-tree-node" slot-scope="{ node, data }">
        <span v-if="node.data.dangerLevel === 0" style="color:black">{{ node.label }} 无风险</span>
        <span v-if="node.data.dangerLevel  === 1" style="color:pink">{{ node.label }} 低风险</span>
        <span v-if="node.data.dangerLevel  === 2" style="color:orange">{{ node.label }} 中风险</span>
        <span v-if="node.data.dangerLevel  === 3" style="color:red">{{ node.label }} 高风险</span>
        <span>
          <el-button
            type="text"
            size="mini"
            v-if="isAuth('ipcs:area:updateDangerLevel')"
            @click="() => addOrUpdateHandle(data.id)">
            设置风险等级
          </el-button>
        </span>
      </span>
</el-tree>
    
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './area-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        props: {
          label: 'label',
          children: 'children',
          id: 'id'
        },
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      // this.getDataList()
    },
    methods: {
      // 获取数据列表
      // getDataList () {
      //   this.dataListLoading = true
      //   this.$http({
      //     url: this.$http.adornUrl('/ipcs/area/list'),
      //     method: 'get',
      //     params: this.$http.adornParams({
      //       'page': this.pageIndex,
      //       'limit': this.pageSize,
      //       'key': this.dataForm.key
      //     })
      //   }).then(({data}) => {
      //     if (data && data.code === 0) {
      //       this.dataList = data.page.list
      //       this.totalPage = data.page.totalCount
      //     } else {
      //       this.dataList = []
      //       this.totalPage = 0
      //     }
      //     this.dataListLoading = false
      //   })
      // },
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
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
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
            url: this.$http.adornUrl('/ipcs/area/delete'),
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
      loadNode(node, resolve) {
        console.log('node',node);
        let parentId = null;
        if(node.data !=null){
          parentId = node.data.id;
        }
        this.$http({
          url: this.$http.adornUrl('/ipcs/area/list'),
          method: 'get',
          params: this.$http.adornParams({
            'parentId': parentId,
          })
        }).then(resp=>{
          return resolve(resp.data.data)
        })
        
      },

    }
  }
</script>
