<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="用户名" prop="pkUsername">
      <el-input v-model="dataForm.pkUsername" placeholder="用户名"></el-input>
    </el-form-item>
    <el-form-item label="用时（秒）" prop="pkUserTime">
      <el-input v-model="dataForm.pkUserTime" placeholder="用时（秒）"></el-input>
    </el-form-item>
    <el-form-item label="公里数" prop="pkKilometre">
      <el-input v-model="dataForm.pkKilometre" placeholder="公里数"></el-input>
    </el-form-item>
    <el-form-item label="卡路里" prop="pkCalorie">
      <el-input v-model="dataForm.pkCalorie" placeholder="卡路里"></el-input>
    </el-form-item>
    <el-form-item label="已经打卡数" prop="pkYet">
      <el-input v-model="dataForm.pkYet" placeholder="已经打卡数"></el-input>
    </el-form-item>
    <el-form-item label="未打卡数" prop="pkNot">
      <el-input v-model="dataForm.pkNot" placeholder="未打卡数"></el-input>
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
          pkUsername: '',
          pkUserTime: '',
          pkKilometre: '',
          pkCalorie: '',
          pkYet: '',
          pkNot: ''
        },
        dataRule: {
          pkUsername: [
            { required: true, message: '用户名不能为空', trigger: 'blur' }
          ],
          pkUserTime: [
            { required: true, message: '用时（秒）不能为空', trigger: 'blur' }
          ],
          pkKilometre: [
            { required: true, message: '公里数不能为空', trigger: 'blur' }
          ],
          pkCalorie: [
            { required: true, message: '卡路里不能为空', trigger: 'blur' }
          ],
          pkYet: [
            { required: true, message: '已经打卡数不能为空', trigger: 'blur' }
          ],
          pkNot: [
            { required: true, message: '未打卡数不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/pank/pkuser/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.pkUsername = data.pkUser.pkUsername
                this.dataForm.pkUserTime = data.pkUser.pkUserTime
                this.dataForm.pkKilometre = data.pkUser.pkKilometre
                this.dataForm.pkCalorie = data.pkUser.pkCalorie
                this.dataForm.pkYet = data.pkUser.pkYet
                this.dataForm.pkNot = data.pkUser.pkNot
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
              url: this.$http.adornUrl(`/pank/pkuser/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'pkUsername': this.dataForm.pkUsername,
                'pkUserTime': this.dataForm.pkUserTime,
                'pkKilometre': this.dataForm.pkKilometre,
                'pkCalorie': this.dataForm.pkCalorie,
                'pkYet': this.dataForm.pkYet,
                'pkNot': this.dataForm.pkNot
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
