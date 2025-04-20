<template>
  <div class="dashboard-container">
    <div class="container">
      <div class="tableBar">
        <label style="margin-right:5px;">员工姓名：</label><el-input v-model="name" placeholder="请输入员工姓名"
          style="width: 200px" />
        <el-button type="primary" style="margin-left: 10px;" @click="pageQuery()">查询</el-button>
        <el-button type="primary" style="float: right;" @click="handleAddEmp()">+添加员工</el-button>
      </div>
      <el-table :data="records" stripe style="width: 100%">
        <el-table-column prop="name" label="员工姓名" width="180">
        </el-table-column>
        <el-table-column prop="username" label="账号">
        </el-table-column>
        <el-table-column prop="phone" label="手机号">
        </el-table-column>
        <el-table-column prop="status" label="账号状态">
          <template slot-scope="scope">
            {{ scope.row.status === 1 ? '启用' : '禁用' }}
          </template>
        </el-table-column>
        <el-table-column prop="updateTime" label="最后操作时间">
          <template slot-scope="scope">
            {{ scope.row.updateTime | formatDate }}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button type="text" @click="handleEdit(scope.row)" :disabled="scope.row.username === 'admin'">
              修改
            </el-button>
            <el-button type="text" :disabled="scope.row.username === 'admin'">{{ scope.row.status === 0 ? '启用' : '禁用'
              }}</el-button>
          </template>
        </el-table-column>

      </el-table>
      <el-pagination @size-change="handleSizeChange" class="pageList" @current-change="handleCurrentChange"
        :current-page="page" :page-sizes="[10, 20, 30, 40]" :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>
  </div>
</template>
<script lang="ts">
import { getEmployeeList } from "@/api/employee";
export default {
  data() {
    return {
      name: '', // 员工姓名
      page: 1, // 当前页码
      pageSize: 10, // 每页条数
      total: 0, // 总条数
      records: [] // 分页数据
    }
  },
  methods: {
    // 分页查询
    pageQuery() {
      // 参数准备
      const params = { name: this.name, page: this.page, pageSize: this.pageSize }
      getEmployeeList(params).then(res => {
        if (res && res.data && res.data.code === 1) {
          this.total = res.data.data.total
          this.records = res.data.data.records
        }
      }).catch(err => {
        this.$message.error('请求出错了：' + err.message)
      })
    },

    // 当每页条数发生改变
    handleSizeChange(pageSize) {
      this.pageSize = pageSize;
      this.pageQuery();
    },
    // 实现翻页功能
    handleCurrentChange(page) {
      this.page = page;
      this.pageQuery();
    },

    // 新增员工（通过路由跳转）
    handleAddEmp() {
      this.$router.push({
        path: '/employee/add'
      })
    },
    // 修改员工（通过路由跳转）
    handleEdit(employee) {
      this.$router.push({
        path: '/employee/add',
        query: { id: employee.id }
      })
    }
  },


  created() {
    this.pageQuery()
  }
}
</script>

<style lang="scss" scoped>
.disabled-text {
  color: #bac0cd !important;
}

.pageList {
  text-align: center;
  margin-top: 30px;
}
</style>
