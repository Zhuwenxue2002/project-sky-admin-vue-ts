<template>
  <div class="addBrand-container">
    <div class="container">
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="180px">
        <el-form-item label="账号" prop="username">
          <el-input v-model="ruleForm.username"></el-input>
        </el-form-item>
        <el-form-item label="员工姓名" prop="name">
          <el-input v-model="ruleForm.name"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="phone">
          <el-input v-model="ruleForm.phone"></el-input>
        </el-form-item>
        <el-form-item label="性别" prop="sex">
          <el-radio v-model="ruleForm.sex" label="1">男</el-radio>
          <el-radio v-model="ruleForm.sex" label="2">女</el-radio>
        </el-form-item>
        <el-form-item label="身份证号" prop="idNumber">
          <el-input v-model="ruleForm.idNumber"></el-input>
        </el-form-item>
        <div class="subBox">
          <el-button type="primary" @click="submitForm('ruleForm', false)">保存</el-button>
          <el-button v-if="this.optType === 'add'" type="primary" @click="submitForm('ruleForm', true)">保存并继续添加员工
          </el-button>
          <el-button @click="() => this.$router.push('/employee')">返回</el-button>
        </div>
      </el-form>
    </div>
  </div>
</template>

<script lang="ts">
import { addEmployee, getEmployeeById, updateEmployee } from "@/api/employee";
export default {
  data() {
    return {
      optType: '', // 操作类型，add：新增，update：修改
      ruleForm: {
        username: '', // 账号
        name: '', // 员工姓名
        phone: '', // 手机号
        sex: '1', // 性别
        idNumber: '', // 身份证号
      },
      rules: {
        username: [
          { required: true, message: '请输入账号', trigger: 'blur' },
        ],
        name: [
          { required: true, message: '请输入员工姓名', trigger: 'blur' },
        ],
        phone: [
          { required: true, pattern: /^1[3-9]\d{9}$/, message: '请输入正确的手机号', trigger: 'blur' },
        ],
        sex: [
          { required: true, message: '请选择性别', trigger: 'blur' },
        ],
        // 身份证校验没写明白XD
        // idNumber: [
        //   {
        //     required: true,
        //     message: '请输入正确的身份证号码',
        //     pattern: /^[1-9]\d{5}(?:18|19|20)\d{2}(?:0[1-9]|1[0-2])(?:0[1-9]|[12]\d|3[01])\d{3}[\dXx]$/,  // 18位
        //     trigger: 'blur'
        //   },
        //   {
        //     message: '请输入正确的身份证号码',
        //     pattern: /^[1-9]\d{5}\d{2}(?:0[1-9]|1[0-2])(?:0[1-9]|[12]\d|3[01])\d{3}$/,  // 15位
        //     trigger: 'blur'
        //   }
        // ]
      }
    }
  },
  methods: {
    submitForm(formName, isContinue) {
      this.$refs[formName].validate((valid) => {
        if (valid === true) {
          if (this.optType === 'add') {
            addEmployee(this.ruleForm).then(res => {
              if (res.data.code === 1) {
                this.$message.success('添加成功')
                if (isContinue) {
                  this.ruleForm = {
                    username: '', // 账号
                    name: '', // 员工姓名
                    phone: '', // 手机号
                    sex: '1', // 性别
                    idNumber: '', // 身份证号
                  }
                }
                else {
                  this.$router.push('/employee')
                }
              }
              else {
                this.$message.error('添加失败')
              }
            })
          }
          else {
            updateEmployee(this.ruleForm).then(res => {
              if (res.data.code === 1) {
                this.$message.success('修改成功')
                this.$router.push('/employee')
              } else {
                this.$message.error('修改失败')
              }
            })
          }
        }
      })

    }
  },
  created() {
    // 使用生命周期钩子判断操作是新增还是修改
    this.optType = this.$route.query.id ? 'update' : 'add'

    // 修改时获取数据回显
    if (this.optType === 'update') {
      getEmployeeById(this.$route.query.id).then(res => {
        if (res.data.code === 1) {
          this.ruleForm = res.data.data
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.addBrand {
  &-container {
    margin: 30px;
    margin-top: 30px;

    .HeadLable {
      background-color: transparent;
      margin-bottom: 0px;
      padding-left: 0px;
    }

    .container {
      position: relative;
      z-index: 1;
      background: #fff;
      padding: 30px;
      border-radius: 4px;

      // min-height: 500px;
      .subBox {
        padding-top: 30px;
        text-align: center;
        border-top: solid 1px $gray-5;
      }
    }

    .idNumber {
      margin-bottom: 39px;
    }

    .el-form-item {
      margin-bottom: 29px;
    }

    .el-input {
      width: 293px;
    }
  }
}
</style>
