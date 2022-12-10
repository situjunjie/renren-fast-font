<template>
  <el-dialog
    title="会员充值"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form  :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="会员姓名" prop="name">
      <el-input v-model="dataForm.name" placeholder="会员姓名" :disabled="true"></el-input>
    </el-form-item>
    <!-- <el-form-item label="会员性别" prop="gender">
      <template>
  <el-radio v-model="dataForm.gender" label="男">男</el-radio>
  <el-radio v-model="dataForm.gender" label="女">女</el-radio>
</template> 
    </el-form-item>-->
    <el-form-item label="电话号码" prop="mobile">
      <el-input v-model="dataForm.mobile" placeholder="电话号码" :disabled="true"></el-input>
    </el-form-item>
    <el-form-item label="充值金额" prop="addPrice">
      <el-input v-model="addPrice" placeholder="充值金额"></el-input>
    </el-form-item>
    
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">充值</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        addPrice:0,
        dataForm: {
          id: 0,
          name: '',
          gender: '',
          mobile: '',
          expirationTime: '',
          age: 0,
          birth: '',
          cardType: '',
          price: 0,
        },
        dataRule: {
          name: [
            { required: true, message: '会员姓名不能为空', trigger: 'blur' }
          ],
          gender: [
            { required: true, message: '会员性别不能为空', trigger: 'blur' }
          ],
          mobile: [
            { required: true, message: '电话号码不能为空', trigger: 'blur' }
          ],
          expirationTime: [
            { required: true, message: '到期时间不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/member/member/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.member.name
                this.dataForm.gender = data.member.gender
                this.dataForm.mobile = data.member.mobile
                this.dataForm.expirationTime = data.member.expirationTime
                this.dataForm.age = data.member.age
                this.dataForm.birth = data.member.birth
                this.dataForm.cardType = data.member.cardType
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.dataForm.price = Number(this.dataForm.price) + Number(this.addPrice)
            this.$http({
              url: this.$http.adornUrl(`/member/member/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'gender': this.dataForm.gender,
                'mobile': this.dataForm.mobile,
                'expirationTime': this.dataForm.expirationTime,
                'age': this.dataForm.age,
                'cardType': this.dataForm.cardType,
                'birth': this.dataForm.birth,
                'price':this.dataForm.price
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
