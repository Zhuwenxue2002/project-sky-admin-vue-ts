<template>
  <div class="dashboard-container">
    <div class="container">
      <div class="tableBar">
        <label style="margin-right: 5px"> 套餐名称： </label>
        <el-input v-model="name" placeholder="请输入套餐名称" style="width: 15%"></el-input>

        <label style="margin-right: 5px"> 套餐分类： </label>
        <el-select v-model="seatmealCategory" placeholder="请选择" clearable>
          <el-option v-for="item in seatmealCategoryArr" :key="item.id" :label="item.name" :value="item.id">
          </el-option>
        </el-select>
        <label style="margin-right: 5px"> 售卖状态： </label>
        <el-select v-model="saleStatus" placeholder="请选择" clearable>
          <el-option v-for="item in saleStatusArr" :key="item.value" :label="item.label" :value="item.value">
          </el-option>
        </el-select>

        <el-button type="primary" style="margin-left: 10px" @click="pageQuery()">查询</el-button>
        <div style="float: right">
          <el-button type="danger" @click="handleDelete('B', checkList)">批量删除</el-button>
          <el-button type="info" @click="handleAdd()">+ 新建套餐</el-button>
        </div>
      </div>
      <el-table :data="records" stripe class="tableBox" @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="25" />
        <el-table-column prop="name" label="套餐名称" />
        <el-table-column label="图片">
          <template slot-scope="scope">
            <el-image style="width: 80px; height: 40px; border: none" :src="scope.row.image"></el-image>
          </template>
        </el-table-column>
        <el-table-column prop="categoryName" label="套餐分类" />
        <el-table-column prop="price" label="套餐价" />
        <el-table-column label="售卖状态">
          <template slot-scope="scope">
            <div class="tableColumn-status" :class="{ 'stop-use': scope.row.status === 0 }">
              {{ scope.row.status === 0 ? '停售' : '启售' }}
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="updateTime" label="最后操作时间">
          <template slot-scope="scope">
            {{ scope.row.updateTime | formatDate }}
          </template>
        </el-table-column>
        <el-table-column label="操作" align="center" width="250px">
          <template slot-scope="scope">
            <!-- <el-button type="text" size="small"> 修改 </el-button> -->
            <el-button type="text" size="small" @click="handleStartOrStop(scope.row)">
              {{ scope.row.status == '1' ? '停售' : '启售' }}
            </el-button>
            <el-button type="text" size="small" @click="handleDelete('S', scope.row.id)"> 删除 </el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination class="pageList" :page-sizes="[10, 20, 30, 40]" :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper" :total="total" @size-change="handleSizeChange"
        @current-change="handleCurrentChange" />
    </div>
  </div>
</template>

<script lang="ts">
import { getCategoryByType } from '@/api/category'
import { getSetmealPage, enableOrDisableSetmeal, deleCategory, deleteSetmeal } from '@/api/setMeal'
import { Row } from 'element-ui'
export default {
  data() {
    return {
      name: '', // 套餐名称
      page: 1, // 当前页码
      pageSize: 10, // 每页条数
      total: 0, // 总条数
      records: [], // 分页数据
      saleStatusArr: [
        { label: '售卖', value: 1 },
        { label: '停售', value: 0 }
      ],
      saleStatus: '',
      seatmealCategoryArr: [],
      seatmealCategory: "",
      checkList: []
    }
  },
  methods: {
    // 分页查询
    pageQuery() {
      // 组织参数
      const params = { page: this.page, pageSize: this.pageSize, name: this.name, categoryId: this.seatmealCategory, status: this.saleStatus }
      getSetmealPage(params).then(res => {
        if (res && res.data && res.data.code === 1) {
          this.records = res.data.data.records
          this.total = res.data.data.total
        }
      })
    },
    handleSizeChange(pageSize) {
      this.pageSize = pageSize
      this.pageQuery()
    },
    handleCurrentChange(page) {
      this.page = page
      this.pageQuery()
    },

    // 停售启售
    handleStartOrStop(row) {
      this.$confirm('确认更改该套餐状态?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const params = { id: row.id, status: !row.status ? 1 : 0 }
        enableOrDisableSetmeal(params).then(res => {
          if (res.data && res.data.code === 1) {
            this.$message.success('套餐状态更改成功！')
            this.pageQuery()
          }
        })
      })

    },
    // 批量删除
    handleDelete(type: string, id: string) {
      let params = "";
      if (type === "B") {
        const arr = new Array();
        this.checkList.forEach((element) => {
          arr.push(element.id);
        });
        params = arr.join(",");
        console.log(params);

      } else if (type === "S") {
        params = id;
      }
      deleteSetmeal(params)
        .then((res) => {
          if (res.data && res.data.code === 1) {
            this.$message.success("套餐删除成功！");
            this.pageQuery();
          }
        })
        .catch((error) => {
          this.$message.error(error.message);
        });
    },
    // 复选框选中逻辑
    handleSelectionChange(val) {
      this.checkList = val
      console.log(val)
    },
    //新增套餐，跳转到新增页面（组件）
    handleAdd() {
      this.$router.push('/setmeal/add')
    }
  },
  created() {
    // 获取套餐分类
    getCategoryByType({ type: 2 }).then(res => {
      if (res && res.data && res.data.code === 1) {
        this.seatmealCategoryArr = res.data.data
      }
    })
    this.pageQuery()
  },
}




</script>
<style lang="scss">
.el-table-column--selection .cell {
  padding-left: 10px;
}
</style>
<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 30px;

    .container {
      background: #fff;
      position: relative;
      z-index: 1;
      padding: 30px 28px;
      border-radius: 4px;

      .tableBar {
        margin-bottom: 20px;

        .tableLab {
          float: right;

          span {
            cursor: pointer;
            display: inline-block;
            font-size: 14px;
            padding: 0 20px;
            color: $gray-2;
          }
        }
      }

      .tableBox {
        width: 100%;
        border: 1px solid $gray-5;
        border-bottom: 0;
      }

      .pageList {
        text-align: center;
        margin-top: 30px;
      }

      //查询黑色按钮样式
      .normal-btn {
        background: #333333;
        color: white;
        margin-left: 20px;
      }
    }
  }
}
</style>
