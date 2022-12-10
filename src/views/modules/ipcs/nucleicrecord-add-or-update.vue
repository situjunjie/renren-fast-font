<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="姓名" prop="name">
      <el-input v-model="dataForm.name" placeholder="姓名"></el-input>
    </el-form-item>
    <el-form-item label="身份证号" prop="idNo">
      <el-input v-model="dataForm.idNo" placeholder="身份证号"></el-input>
    </el-form-item>
    <el-form-item label="采集时间" prop="sampleTime">
      <el-input v-model="dataForm.sampleTime" placeholder="采集时间"></el-input>
    </el-form-item>
    <el-form-item label="状态" prop="status">
      <el-input v-model="dataForm.status" placeholder="状态"></el-input>
    </el-form-item>
    <el-form-item label="结果" prop="result">
      <el-input v-model="dataForm.result" placeholder="结果"></el-input>
    </el-form-item>
    <el-form-item label="报告时间" prop="reportTime">
      <el-input v-model="dataForm.reportTime" placeholder="报告时间"></el-input>
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
          idNo: '',
          sampleTime: '',
          status: '',
          result: '',
          reportTime: ''
        },
        dataRule: {
          name: [
            { required: true, message: '姓名不能为空', trigger: 'blur' }
          ],
          idNo: [
            { required: true, message: '身份证号不能为空', trigger: 'blur' }
          ],
          sampleTime: [
            { required: true, message: '采集时间不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '状态不能为空', trigger: 'blur' }
          ],
          result: [
            { required: true, message: '结果不能为空', trigger: 'blur' }
          ],
          reportTime: [
            { required: true, message: '报告时间不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/ipcs/nucleicrecord/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.nucleicRecord.name
                this.dataForm.idNo = data.nucleicRecord.idNo
                this.dataForm.sampleTime = data.nucleicRecord.sampleTime
                this.dataForm.status = data.nucleicRecord.status
                this.dataForm.result = data.nucleicRecord.result
                this.dataForm.reportTime = data.nucleicRecord.reportTime
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
              url: this.$http.adornUrl(`/ipcs/nucleicrecord/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'idNo': this.dataForm.idNo,
                'sampleTime': this.dataForm.sampleTime,
                'status': this.dataForm.status,
                'result': this.dataForm.result,
                'reportTime': this.dataForm.reportTime
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
