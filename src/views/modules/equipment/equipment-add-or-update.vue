<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="器材名称" prop="name">
      <el-input v-model="dataForm.name" placeholder="器材名称"></el-input>
    </el-form-item>
    <el-form-item label="效期日期" prop="expriationDate">
      <!-- <el-input v-model="dataForm.expriationDate" placeholder="效期日期"></el-input> -->
      <el-date-picker
      v-model="dataForm.expriationDate"
      type="date"
      format="yyyy-MM-dd"
      value-format="yyyy-MM-dd"
      placeholder="选择日期时间">
    </el-date-picker>
    </el-form-item>
    <el-form-item label="养护日期" prop="expriationDate">
      <!-- <el-input v-model="dataForm.expriationDate" placeholder="效期日期"></el-input> -->
      <el-date-picker
      v-model="dataForm.maintainDate"
      type="date"
      format="yyyy-MM-dd"
      value-format="yyyy-MM-dd"
      placeholder="选择日期时间">
    </el-date-picker>
    </el-form-item>
    <el-form-item label="清洁日期" prop="expriationDate">
      <!-- <el-input v-model="dataForm.expriationDate" placeholder="效期日期"></el-input> -->
      <el-date-picker
      v-model="dataForm.cleanDate"
      type="date"
      format="yyyy-MM-dd"
      value-format="yyyy-MM-dd"
      placeholder="选择日期时间">
    </el-date-picker>
    </el-form-item>
    <el-form-item label="是否维修" prop="fix">
      <el-input v-model="dataForm.fix" placeholder="是否维修"></el-input>
    </el-form-item>
    <el-form-item label="养护周期" prop="fix">
      <el-input v-model="dataForm.maintainFreq" placeholder="养护周期"></el-input>
    </el-form-item>
    <el-form-item label="生产商" prop="manufacture">
      <el-input v-model="dataForm.manufacture" placeholder="生产商"></el-input>
    </el-form-item>
    <el-form-item label="数量" prop="num">
      <el-input v-model="dataForm.num" placeholder="数量"></el-input>
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
          expriationDate: '',
          manufacture: '',
          cleanDate:'',
          maintainDate:'',
          maintainFreq:0,
          fix:'',
          num: ''
        },
        dataRule: {
          name: [
            { required: true, message: '器材名称不能为空', trigger: 'blur' }
          ],
          expriationDate: [
            { required: true, message: '效期日期不能为空', trigger: 'blur' }
          ],
          manufacture: [
            { required: true, message: '生产商不能为空', trigger: 'blur' }
          ],
          num: [
            { required: true, message: '数量不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/equipment/equipment/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.equipment.name
                this.dataForm.expriationDate = data.equipment.expriationDate
                this.dataForm.manufacture = data.equipment.manufacture
                this.dataForm.num = data.equipment.num
                this.dataForm.cleanDate = data.equipment.cleanDate
                this.dataForm.maintainDate = data.equipment.maintainDate
                this.dataForm.fix = data.equipment.fix
                this.dataForm.maintainFreq = data.equipment.maintainFreq
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
              url: this.$http.adornUrl(`/equipment/equipment/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'expriationDate': this.dataForm.expriationDate,
                'manufacture': this.dataForm.manufacture,
                'num': this.dataForm.num,
                'cleanDate':this.dataForm.cleanDate,
                'maintainDate':this.dataForm.maintainDate,
                'fix':this.dataForm.fix,
                'maintainFreq':this.dataForm.maintainFreq
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
