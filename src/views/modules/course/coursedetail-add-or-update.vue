<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="课程" prop="courseId">
      <!-- <el-input v-model="dataForm.courseId" placeholder="课程id"></el-input> -->
      <el-select v-model="dataForm.courseId" placeholder="请选择">
    <el-option
      v-for="item in courseOpt"
      :key="item.id"
      :label="item.name"
      :value="item.id">
    </el-option>
  </el-select>
    </el-form-item>
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
    <el-form-item label="开始时间" prop="beginTime">
      <!-- <el-input v-model="dataForm.beginTime" placeholder="开始时间"></el-input> -->
      <el-date-picker
      v-model="dataForm.beginTime"
      value-format="yyyy-MM-dd HH:mm:ss"
      type="datetime"
      placeholder="选择日期时间">
    </el-date-picker>
    </el-form-item>
    <el-form-item label="结束时间" prop="endTime">
      <!-- <el-input v-model="dataForm.endTime" placeholder="结束时间"></el-input> -->
      <el-date-picker
      v-model="dataForm.endTime"
      value-format="yyyy-MM-dd HH:mm:ss"
      type="datetime"
      placeholder="选择日期时间">
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
        courseOpt:[],
        coachOpt:[],
        dataForm: {
          id: 0,
          courseId: '',
          coachId: '',
          beginTime: '',
          endTime: ''
        },
        dataRule: {
          courseId: [
            { required: true, message: '课程id不能为空', trigger: 'blur' }
          ],
          coachId: [
            { required: true, message: '教练id不能为空', trigger: 'blur' }
          ],
          beginTime: [
            { required: true, message: '开始时间不能为空', trigger: 'blur' }
          ],
          endTime: [
            { required: true, message: '结束时间不能为空', trigger: 'blur' }
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
          url: this.$http.adornUrl('/course/course/listAll'),
          method: 'get'
        }).then(({data}) =>{
          if (data && data.code === 0) {
                this.courseOpt = data.data
          }
        })

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
              url: this.$http.adornUrl(`/course/coursedetail/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.courseId = data.courseDetail.courseId
                this.dataForm.coachId = data.courseDetail.coachId
                this.dataForm.beginTime = data.courseDetail.beginTime
                this.dataForm.endTime = data.courseDetail.endTime
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
              url: this.$http.adornUrl(`/course/coursedetail/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'courseId': this.dataForm.courseId,
                'coachId': this.dataForm.coachId,
                'beginTime': this.dataForm.beginTime,
                'endTime': this.dataForm.endTime
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
