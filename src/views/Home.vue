<template>
  <div>
    <div class="top"></div>
    <div class="search">
      <el-form
        ref="form"
        :inline="true"
        :model="formInline"
        class="demo-form-inline"
      >
        <el-form-item label="编码" prop="id">
          <el-input v-model="formInline.id" placeholder="请输入编码"></el-input>
        </el-form-item>
        <el-form-item label="供应商" prop="orderPerson">
          <el-input
            v-model="formInline.orderPerson"
            placeholder="请输入供应商"
          ></el-input>
        </el-form-item>
        <el-form-item label="最晚付款时间" prop="time">
          <el-date-picker
            v-model="formInline.time"
            type="datetimerange"
            align="right"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            :default-time="['12:00:00', '08:00:00']"
          >
          </el-date-picker>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" icon="el-icon-search" @click="search"
            >查询</el-button
          >
          <el-button icon="el-icon-edit" @click="resetForm('form')"
            >重置</el-button
          >
        </el-form-item>
      </el-form>
      <div class="addOrDel">
        <el-button
          @click="delCheck(newArr)"
          type="danger"
          icon="el-icon-delete"
          size="small"
          >删除</el-button
        >
        <el-button
          @click="addData"
          type="primary"
          icon="el-icon-star-off"
          size="small"
          >添加商品</el-button
        >
      </div>
      <el-table
        ref="multipleTable"
        :data="tableData"
        border
        style="width: 100%"
        @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="55"> </el-table-column>
        <el-table-column prop="id" label="编码" width="180"> </el-table-column>
        <el-table-column prop="orderPerson" label="供应商" width="180">
        </el-table-column>
        <el-table-column prop="status" label="状态" width="180">
        </el-table-column>
        <el-table-column prop="payTime" label="缴费日期" width="180">
        </el-table-column>
        <el-table-column prop="payMoney" label="缴费金额" width="180">
        </el-table-column>
        <el-table-column label="操作">
          <template v-slot="{ row }">
            <el-button
              @click="del(row.id)"
              type="danger"
              icon="el-icon-delete"
              size="small"
              >删除</el-button
            >
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="small"
              @click="openBox(row)"
              >编辑</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[3, 5, 10, 20]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- 模态框 -->
      <el-dialog
        :title="num === 0 ? '增加信息' : '修改信息'"
        :visible.sync="dialogFormVisible"
      >
        <el-form :model="form">
          <el-form-item label="供应商" :label-width="formLabelWidth">
            <el-input v-model="form.orderPerson" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="状态" :label-width="formLabelWidth">
            <el-input v-model="form.status" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item
            v-show="num === 0 ? true : false"
            label="缴费日期"
            :label-width="formLabelWidth"
          >
            <el-date-picker
              v-model="form.time"
              type="datetimerange"
              align="right"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              :default-time="['12:00:00', '08:00:00']"
            >
            </el-date-picker>
          </el-form-item>
          <el-form-item label="缴费金额" :label-width="formLabelWidth">
            <el-input v-model="form.payMoney" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="saveUpdate">确 定</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  async created() {
    this.tableList()
  },
  data() {
    return {
      newArr: [],
      id: [],
      num: 0,
      searchNo: '',
      formInline: {
        id: '',
        orderPerson: '',
        time: ''
      },
      currentPage: 1,
      pageSize: 5,
      total: 0,
      tableData: [
        {
          payMoney: ''
        }
      ],
      newTable: [],
      newData: [],
      items: [],
      dialogFormVisible: false,
      form: {
        orderPerson: '',
        status: '',
        payMoney: '',
        time: ''
      },
      formLabelWidth: '120px'
    }
  },
  methods: {
    async tableList() {
      await axios
        .get('./../static/goods.json')
        .then((res) => {
          const { data } = res
          data.forEach((item) => {
            this.newData.push(parseFloat(item.payMoney).toLocaleString())
          })
          this.tableData.payMoney = this.newData
          this.tableData = data
          this.id = data[data.length - 1].id
          this.total = data.length
          this.tableData = this.tableData.slice(
            (this.currentPage - 1) * this.pageSize,
            this.currentPage * this.pageSize
          )
        })
        .catch((error) => {
          console.log(error)
        })
    },
    search() {
      this.orderPerson = this.formInline.orderPerson
      this.searchNo = this.formInline.id
      if (this.searchNo !== '' && this.orderPerson === '') {
        const num = this.tableData.find((item) => {
          return item.id === Number(this.searchNo)
        })
        this.newTable.push(num)
        this.tableData = this.newTable
      } else {
        const order = this.tableData.find((item) => {
          return item.orderPerson === this.orderPerson
        })
        this.newTable.push(order)
        this.tableData = this.newTable
      }
      this.searchNo = ''
      this.orderPerson = ''
    },
    addData() {
      this.dialogFormVisible = true
      this.form = {}
      this.num = 0
      const a = 1233434
      const b = parseFloat(a).toLocaleString()
      console.log(b)
    },
    del(id) {
      const i = this.tableData.findIndex((item) => item.id === id)
      this.tableData.splice(i, 1)
      // console.log(id)
    },
    openBox(row) {
      console.log(row)
      this.dialogFormVisible = true
      this.form = row
      this.num = 1
    },
    saveUpdate() {
      if (this.num === 0) {
        this.dialogFormVisible = false
        this.form.time = new Date()
        console.log(this.form.time)
        this.tableData.unshift({
          id: (this.id += 1),
          orderPerson: this.form.orderPerson,
          status: this.form.status,
          payMoney: this.form.payMoney,
          payTime: this.form.time
        })
      } else {
        this.dialogFormVisible = false
        this.tableData.forEach((item) => {
          if (item.id === this.form.id) {
            item.orderPerson = this.form.orderPerson
            item.status = this.form.status
            item.payMoney = this.form.payMoney
          }
        })
      }
    },
    delCheck(data) {
      data.forEach((num) => {
        // console.log(item)
        const i = this.tableData.findIndex((item) => item.id === num)
        this.tableData.splice(i, 1)
      })
    },
    handleSizeChange(val) {
      this.pageSize = this.total / val
      this.tableList()
    },
    handleCurrentChange(val) {
      this.currentPage = val
      this.tableList()
    },
    toggleSelection(rows) {
      if (rows) {
        rows.forEach((row) => {
          this.$refs.multipleTable.toggleRowSelection(row)
        })
      } else {
        this.$refs.multipleTable.clearSelection()
      }
    },
    handleSelectionChange(val) {
      this.multipleSelection = val
      val.forEach((item) => {
        this.newArr.push(item.id)
      })
      // this.tableData.shift(val)
      // console.log(this.newArr)
    },
    resetForm(form) {
      this.$refs[form].resetFields()
    }
  }
}
</script>

<style lang="less" scoped>
.top {
  height: 50px;
  background-color: #333;
}
.search {
  padding: 20px;
  width: 1250px;
  height: 530px;
  box-shadow: 0 0 5px #ccc;
  margin: 100px auto;
}
/deep/.el-form-item {
  margin-right: 10px;
}
.addOrDel {
  box-sizing: border-box;
  width: 100%;
  height: 50px;
  background-color: #eee;
  padding: 10px;
  margin: 20px 0;
}
.el-pagination {
  margin-top: 20px;
  margin-left: 50%;
  transform: translate(-50%);
}
</style>
