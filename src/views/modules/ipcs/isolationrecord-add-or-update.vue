<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="姓名" prop="name">
      <el-input v-model="dataForm.name" placeholder="姓名"></el-input>
    </el-form-item>
    <el-form-item label="性别" prop="gender">
      <el-input v-model="dataForm.gender" placeholder="性别"></el-input>
    </el-form-item>
    <el-form-item label="年龄" prop="age">
      <el-input v-model="dataForm.age" placeholder="年龄"></el-input>
    </el-form-item>
    <el-form-item label="联系电话" prop="mobile">
      <el-input v-model="dataForm.mobile" placeholder="联系电话"></el-input>
    </el-form-item>
    <el-form-item label="隔离地点" prop="isolationAddr">
      <el-input v-model="dataForm.isolationAddr" placeholder="隔离地点"></el-input>
    </el-form-item>
    <el-form-item label="隔离类型" prop="type">
      <el-input v-model="dataForm.type" placeholder="隔离类型"></el-input>
    </el-form-item>
    <el-form-item label="时长(天)" prop="time">
      <el-input v-model="dataForm.time" placeholder="时长(天)"></el-input>
    </el-form-item>
    <el-form-item label="开始时间" prop="beginTime">
      <!-- <el-input v-model="dataForm.beginTime" placeholder="开始时间"></el-input> -->
      <el-date-picker
      v-model="dataForm.beginTime"
      type="datetime"
      value-format="yyyy-MM-dd HH:mm:ss"

      placeholder="开始时间">
    </el-date-picker>
    </el-form-item>
    <el-form-item label="结束时间" prop="endTime">
      <!-- <el-input v-model="dataForm.endTime" placeholder="结束时间"></el-input> -->
      <el-date-picker
      v-model="dataForm.endTime"
      type="datetime"
      value-format="yyyy-MM-dd HH:mm:ss"
      placeholder="结束时间">
    </el-date-picker>
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
          gender: '',
          age: '',
          isolationAddr: '',
          type: '',
          time: '',
          beginTime: '',
          endTime: '',
          mobile:''
        },
        dataRule: {
          name: [
            { required: true, message: '姓名不能为空', trigger: 'blur' }
          ],
          gender: [
            { required: true, message: '性别不能为空', trigger: 'blur' }
          ],
          age: [
            { required: true, message: '性别不能为空', trigger: 'blur' }
          ],
          isolationAddr: [
            { required: true, message: '隔离地点不能为空', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '隔离类型不能为空', trigger: 'blur' }
          ],
          time: [
            { required: true, message: '时长(天)不能为空', trigger: 'blur' }
          ],
          beginTime: [
            { required: true, message: '开始时间不能为空', trigger: 'blur' }
          ],
          endTime: [
            { required: true, message: '结束时间不能为空', trigger: 'blur' }
          ],
          mobile: [
            { required: true, message: '联系电话不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/ipcs/isolationrecord/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.isolationRecord.name
                this.dataForm.gender = data.isolationRecord.gender
                this.dataForm.age = data.isolationRecord.age
                this.dataForm.isolationAddr = data.isolationRecord.isolationAddr
                this.dataForm.type = data.isolationRecord.type
                this.dataForm.time = data.isolationRecord.time
                this.dataForm.beginTime = data.isolationRecord.beginTime
                this.dataForm.endTime = data.isolationRecord.endTime
                this.dataForm.mobile = data.isolationRecord.mobile
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
              url: this.$http.adornUrl(`/ipcs/isolationrecord/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'gender': this.dataForm.gender,
                'age': this.dataForm.age,
                'isolationAddr': this.dataForm.isolationAddr,
                'type': this.dataForm.type,
                'time': this.dataForm.time,
                'beginTime': this.dataForm.beginTime,
                'endTime': this.dataForm.endTime,
                'mobile' : this.dataForm.mobile,
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
