<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改风险等级'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="风险等级" prop="dangerLevel">
      <!-- <el-input v-model="dataForm.dangerLevel" placeholder="风险等级"></el-input> -->
      <!-- <el-radio v-model="dataForm.dangerLevel" label="0">无风险</el-radio>
  <el-radio v-model="dataForm.dangerLevel" label="1">低风险</el-radio>
  <el-radio v-model="dataForm.dangerLevel" label="2">中风险</el-radio>
  <el-radio v-model="dataForm.dangerLevel" label="3">高风险</el-radio> -->
  <el-select v-model="dataForm.dangerLevel" placeholder="请选择">
    <el-option
      v-for="item in options"
      :key="item.value"
      :label="item.label"
      :value="item.value">
    </el-option>
  </el-select>
    </el-form-item>
    <el-form-item label="是否覆盖子区域等级" prop="overChildren">
      <el-switch
  v-model="dataForm.overChildren"
  active-color="#13ce66"
  inactive-color="#ff4949">
</el-switch>
    </el-form-item>
    <!-- <el-form-item label="层级" prop="level">
      <el-input v-model="dataForm.level" placeholder="层级"></el-input>
    </el-form-item> -->
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          dangerLevel:0,
          overChildren:true
        },
        dataRule: {
          
        },
        options: [{
          value: 0,
          label: '无风险'
        }, {
          value: 1,
          label: '低风险'
        }, {
          value: 2,
          label: '中风险'
        }, {
          value: 3,
          label: '高风险'
        }]
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/ipcs/area/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                console.log('data',data.area);
                this.dataForm.name = data.area.name
                this.dataForm.parentId = data.area.parentId
                this.dataForm.level = data.area.level
                this.dataForm.dangerLevel = data.area.dangerLevel
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/ipcs/area/${!this.dataForm.id ? 'save' : 'updateDangerLevel'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'dangerLevel': this.dataForm.dangerLevel,
                'overChildren': this.dataForm.overChildren
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 500,
                  onClose: () => {
                    this.visible = false
                    location.reload()
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
