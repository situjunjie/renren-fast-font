<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="教练" prop="coachId">
      <!-- <el-input v-model="dataForm.coachId" placeholder="教练id"></el-input> -->
      <el-select v-model="dataForm.coachId" placeholder="请选择">
    <el-option
      v-for="item in coachOpt"
      :key="item.id"
      :label="item.name"
      :value="item.id">
    </el-option>
  </el-select>
    </el-form-item>
    <el-form-item label="评分" prop="score">
      <el-rate
    v-model="dataForm.score"
    :colors="colors">
  </el-rate>  
</el-form-item>
    <el-form-item label="评价详情" prop="content">
      <el-input v-model="dataForm.content" placeholder="评价详情"></el-input>
    </el-form-item>
    <el-form-item label="评价人" prop="createBy">
      <el-input v-model="dataForm.createBy" placeholder="评价人"></el-input>
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
        coachOpt:[],
        dataForm: {
          id: 0,
          coachId: '',
          score: 0,
          content: '',
          createBy: ''
        },
        colors: ['#99A9BF', '#F7BA2A', '#FF9900'],
        dataRule: {
          coachId: [
            { required: true, message: '教练id不能为空', trigger: 'blur' }
          ],
          score: [
            { required: true, message: '评分不能为空', trigger: 'blur' }
          ],
          content: [
            { required: true, message: '评价详情不能为空', trigger: 'blur' }
          ],
          createBy: [
            { required: true, message: '评价人不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true

        //获取下拉框数据 begin

        this.$http({
          url: this.$http.adornUrl('/coach/coach/listAll'),
          method: 'get'
        }).then(({data}) =>{
          if (data && data.code === 0) {
                this.coachOpt = data.data
          }
        })
        //获取下拉框数据 end
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/coach/coachrange/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.coachId = data.coachRange.coachId
                this.dataForm.score = data.coachRange.score
                this.dataForm.content = data.coachRange.content
                this.dataForm.createBy = data.coachRange.createBy
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
              url: this.$http.adornUrl(`/coach/coachrange/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'coachId': this.dataForm.coachId,
                'score': this.dataForm.score,
                'content': this.dataForm.content,
                'createBy': this.dataForm.createBy
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
