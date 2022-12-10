<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="隔离类型" prop="type">
      <el-input v-model="dataForm.type" placeholder="隔离类型"></el-input>
    </el-form-item>
    <el-form-item label="时长(天)" prop="time">
      <el-input v-model="dataForm.time" placeholder="时长(天)"></el-input>
    </el-form-item>
    <el-form-item label="风险等级" prop="dangerLevel">
      <el-input v-model="dataForm.dangerLevel" placeholder="风险等级"></el-input>
    </el-form-item>
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
          type: '',
          time: '',
          dangerLevel: ''
        },
        dataRule: {
          type: [
            { required: true, message: '隔离类型不能为空', trigger: 'blur' }
          ],
          time: [
            { required: true, message: '时长(天)不能为空', trigger: 'blur' }
          ],
          dangerLevel: [
            { required: true, message: '风险等级不能为空', trigger: 'blur' }
          ]
        }
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
              url: this.$http.adornUrl(`/ipcs/isolationrule/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.type = data.isolationRule.type
                this.dataForm.time = data.isolationRule.time
                this.dataForm.dangerLevel = data.isolationRule.dangerLevel
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
              url: this.$http.adornUrl(`/ipcs/isolationrule/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'type': this.dataForm.type,
                'time': this.dataForm.time,
                'dangerLevel': this.dataForm.dangerLevel
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
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
