<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="物资名称" prop="name">
      <el-input v-model="dataForm.name" placeholder="物资名称"></el-input>
    </el-form-item>
    <el-form-item label="物资状态" prop="status">
      <el-input v-model="dataForm.status" placeholder="物资状态"></el-input>
    </el-form-item>
    <el-form-item label="物资数量" prop="num">
      <el-input v-model="dataForm.num" placeholder="物资数量"></el-input>
    </el-form-item>
    <!-- <el-form-item label="出库时间" prop="sendTime">
      <el-input v-model="dataForm.sendTime" placeholder="出库时间"></el-input>
    </el-form-item> -->
    <el-form-item label="出库时间" prop="sendTime">
      <!-- <el-input v-model="dataForm.endTime" placeholder="结束时间"></el-input> -->
      <el-date-picker
      v-model="dataForm.sendTime"
      type="datetime"
      value-format="yyyy-MM-dd HH:mm:ss"
      placeholder="出库时间">
    </el-date-picker>
    </el-form-item>
    <!-- <el-form-item label="入库时间" prop="inTime">
      <el-input v-model="dataForm.inTime" placeholder="入库时间"></el-input>
    </el-form-item> -->
    <el-form-item label="入库时间" prop="sendTime">
      <!-- <el-input v-model="dataForm.inTime" placeholder="结束时间"></el-input> -->
      <el-date-picker
      v-model="dataForm.inTime"
      type="datetime"
      value-format="yyyy-MM-dd HH:mm:ss"
      placeholder="入库时间">
    </el-date-picker>
    </el-form-item>
    <el-form-item label="出库人" prop="sender">
      <el-input v-model="dataForm.sender" placeholder="出库人"></el-input>
    </el-form-item>
    <el-form-item label="入库人" prop="receiver">
      <el-input v-model="dataForm.receiver" placeholder="入库人"></el-input>
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
          status: '',
          num: '',
          sendTime: '',
          inTime: '',
          sender: '',
          receiver: ''
        },
        dataRule: {
          name: [
            { required: true, message: '物资名称不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '物资状态不能为空', trigger: 'blur' }
          ],
          num: [
            { required: true, message: '物资数量不能为空', trigger: 'blur' }
          ],
          sendTime: [
            { required: true, message: '出库时间不能为空', trigger: 'blur' }
          ],
          inTime: [
            { required: true, message: '入库时间不能为空', trigger: 'blur' }
          ],
          sender: [
            { required: true, message: '出库人不能为空', trigger: 'blur' }
          ],
          receiver: [
            { required: true, message: '入库人不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/ipcs/susplies/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.susplies.name
                this.dataForm.status = data.susplies.status
                this.dataForm.num = data.susplies.num
                this.dataForm.sendTime = data.susplies.sendTime
                this.dataForm.inTime = data.susplies.inTime
                this.dataForm.sender = data.susplies.sender
                this.dataForm.receiver = data.susplies.receiver
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
              url: this.$http.adornUrl(`/ipcs/susplies/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'status': this.dataForm.status,
                'num': this.dataForm.num,
                'sendTime': this.dataForm.sendTime,
                'inTime': this.dataForm.inTime,
                'sender': this.dataForm.sender,
                'receiver': this.dataForm.receiver
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
