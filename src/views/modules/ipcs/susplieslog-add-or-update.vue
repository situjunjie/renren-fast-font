<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="物资名称" prop="name">
      <el-input v-model="dataForm.name" placeholder="物资名称"></el-input>
    </el-form-item>
    <el-form-item label="数量" prop="num">
      <el-input v-model="dataForm.num" placeholder="数量"></el-input>
    </el-form-item>
    <el-form-item label="入出库时间" prop="logTime">
      <el-input v-model="dataForm.logTime" placeholder="入出库时间"></el-input>
    </el-form-item>
    <el-form-item label="入出库动作" prop="type">
      <el-input v-model="dataForm.type" placeholder="入出库动作"></el-input>
    </el-form-item>
    <el-form-item label="操作人" prop="operator">
      <el-input v-model="dataForm.operator" placeholder="操作人"></el-input>
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
          name: '',
          num: '',
          logTime: '',
          type: '',
          operator: ''
        },
        dataRule: {
          name: [
            { required: true, message: '物资名称不能为空', trigger: 'blur' }
          ],
          num: [
            { required: true, message: '数量不能为空', trigger: 'blur' }
          ],
          logTime: [
            { required: true, message: '入出库时间不能为空', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '入出库动作不能为空', trigger: 'blur' }
          ],
          operator: [
            { required: true, message: '操作人不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/ipcs/susplieslog/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.suspliesLog.name
                this.dataForm.num = data.suspliesLog.num
                this.dataForm.logTime = data.suspliesLog.logTime
                this.dataForm.type = data.suspliesLog.type
                this.dataForm.operator = data.suspliesLog.operator
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
              url: this.$http.adornUrl(`/ipcs/susplieslog/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'num': this.dataForm.num,
                'logTime': this.dataForm.logTime,
                'type': this.dataForm.type,
                'operator': this.dataForm.operator
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
