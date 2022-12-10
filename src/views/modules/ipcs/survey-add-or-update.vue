<template>
  <el-dialog :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
      label-width="300px">
      <el-form-item label="姓名" prop="name">
        <el-input v-model="dataForm.name" placeholder="姓名"></el-input>
      </el-form-item>
      <el-form-item label="年龄" prop="age">
        <el-input v-model="dataForm.age" placeholder="年龄"></el-input>
      </el-form-item>
      <el-form-item label="性别" prop="gender">
        <el-radio v-model="dataForm.gender" label="男">男</el-radio>
        <el-radio v-model="dataForm.gender" label="女">女</el-radio>
      </el-form-item>
      <el-form-item label="职业" prop="occupy">
        <el-input v-model="dataForm.occupy" placeholder="职业"></el-input>
      </el-form-item>
      <el-form-item label="住址" prop="address">
        <el-input v-model="dataForm.address" placeholder="住址"></el-input>
      </el-form-item>
      <el-form-item label="是否发热" prop="fever">
        <el-radio v-model="dataForm.fever" label="是">是</el-radio>
        <el-radio v-model="dataForm.fever" label="否">否</el-radio>
      </el-form-item>
      <el-form-item label="体温" prop="temperature">
        <el-input v-model="dataForm.temperature" placeholder="体温"></el-input>
      </el-form-item>
      <el-form-item label="近14天是否有中、高风险区域旅居史" prop="content1">
        <el-radio v-model="dataForm.content1" label="是">是</el-radio>
        <el-radio v-model="dataForm.content1" label="否">否</el-radio>
      </el-form-item>
      <el-form-item label="是否密接、次密接" prop="content2">
        <el-radio v-model="dataForm.content2" label="是">是</el-radio>
        <el-radio v-model="dataForm.content2" label="否">否</el-radio> </el-form-item>
      <el-form-item label="是否有与次密接、密接患者有时空接触" prop="content3">
        <el-radio v-model="dataForm.content3" label="是">是</el-radio>
        <el-radio v-model="dataForm.content3" label="否">否</el-radio> </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        id: 0,
        name: '',
        age: '',
        gender: '',
        occupy: '',
        address: '',
        fever: '',
        temperature: '',
        content1: '',
        content2: '',
        content3: ''
      },
      dataRule: {
        name: [
          { required: true, message: '姓名不能为空', trigger: 'blur' }
        ],
        age: [
          { required: true, message: '年龄不能为空', trigger: 'blur' }
        ],
        gender: [
          { required: true, message: '性别不能为空', trigger: 'blur' }
        ],
        occupy: [
          { required: true, message: '职业不能为空', trigger: 'blur' }
        ],
        address: [
          { required: true, message: '住址不能为空', trigger: 'blur' }
        ],
        fever: [
          { required: true, message: '是否发热不能为空', trigger: 'blur' }
        ],
        temperature: [
          { required: true, message: '体温不能为空', trigger: 'blur' }
        ],
        content1: [
          { required: true, message: '近14天是否有中、高风险区域旅居史不能为空', trigger: 'blur' }
        ],
        content2: [
          { required: true, message: '是否密接、次密接不能为空', trigger: 'blur' }
        ],
        content3: [
          { required: true, message: '是否有与次密接、密接患者有时空接触不能为空', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/ipcs/survey/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.survey.name
              this.dataForm.age = data.survey.age
              this.dataForm.gender = data.survey.gender
              this.dataForm.occupy = data.survey.occupy
              this.dataForm.address = data.survey.address
              this.dataForm.fever = data.survey.fever
              this.dataForm.temperature = data.survey.temperature
              this.dataForm.content1 = data.survey.content1
              this.dataForm.content2 = data.survey.content2
              this.dataForm.content3 = data.survey.content3
            }
          })
        }
      })
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(`/ipcs/survey/${!this.dataForm.id ? 'save' : 'update'}`),
            method: 'post',
            data: this.$http.adornData({
              'id': this.dataForm.id || undefined,
              'name': this.dataForm.name,
              'age': this.dataForm.age,
              'gender': this.dataForm.gender,
              'occupy': this.dataForm.occupy,
              'address': this.dataForm.address,
              'fever': this.dataForm.fever,
              'temperature': this.dataForm.temperature,
              'content1': this.dataForm.content1,
              'content2': this.dataForm.content2,
              'content3': this.dataForm.content3
            })
          }).then(({ data }) => {
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
