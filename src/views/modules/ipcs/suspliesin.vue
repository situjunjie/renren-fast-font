<template>
    <el-form :model="dataForm" ref="dataForm" label-width="80px">
    <el-form-item label="物资名称" prop="name">
      <el-input v-model="dataForm.name" placeholder="物资名称"></el-input>
    </el-form-item>
    <el-form-item label="物资数量" prop="num">
      <el-input v-model="dataForm.num" placeholder="物资数量"></el-input>
    </el-form-item>
    <el-form-item label="入库人" prop="receiver">
      <el-input v-model="dataForm.receiver" placeholder="入库人"></el-input>
    </el-form-item>
    
    <el-form-item>
    <el-button type="primary" @click="onSubmit">入库</el-button>
  </el-form-item>
    </el-form>
</template>

<script>
export default {
  data(){
    return {
      dataForm:{
        name : '',
        num: '',
        receiver: ''
      },
    }
  },
  methods:{
    onSubmit(){
      this.$http({
              url: this.$http.adornUrl(`/ipcs/susplies/receive`),
              method: 'post',
              data: this.$http.adornData({
                'name': this.dataForm.name,
                'num': this.dataForm.num,
                'receiver': this.dataForm.receiver,
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '入库成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error('入库失败')
              }
            })
    }
  }
}
</script>

<style>

</style>