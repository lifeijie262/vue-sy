<template>
  <div class="orders">
    <!-- 面包屑导航区域 -->
   <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>订单管理</el-breadcrumb-item>
    <el-breadcrumb-item>订单列表</el-breadcrumb-item>
  </el-breadcrumb>

  <!-- 卡片试图区域 -->
  <el-card>
    <el-row>
      <el-col :span="8">
         <el-input placeholder="请输入内容" class="input-with-select">
          <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
      </el-col>
    </el-row>

    <!-- 订单列表数据 -->
    <el-table :data="ordersList" border stripe>
      <el-table-column type="index"></el-table-column>
      <el-table-column label="订单编号" prop="order_number"></el-table-column>
      <el-table-column label="订单价格" prop="order_price"></el-table-column>
      <el-table-column label="是否付款" prop="pay_status">
        <template slot-scope="scope">
          <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
          <el-tag type="danger" v-else>未付款</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="是否发货" prop="is_send"></el-table-column>
      <el-table-column label="下单时间" prop="create_time">
        <template slot-scope="scope">
          {{scope.row.create_time | dateFormat}}
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button size="mini" type="primary" icon="el-icon-edit" @click="showBox"></el-button>
          <el-button size="mini" type="success" icon="el-icon-location" @click="showProgress"></el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页区域 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>

    <!-- 修改地址的对话框 -->
    <el-dialog
      title="修改地址"
      :visible.sync="addressVisible"
      width="50%"
      @close="addressClosed"
      >
      <el-form :model="addressForm" :rules="addressRules" ref="addressFormRef" label-width="100px">
      <el-form-item label="省市区/县" prop="address1">
        <el-cascader :options="cityData" v-model="addressForm.address1"></el-cascader>
      </el-form-item>
       <el-form-item label="详细地址" prop="address2">
        <el-input v-model="addressForm.address2"></el-input>
      </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressVisible = false">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 展示物流信息的对话框 -->
    <el-dialog
      title="物流信息"
      :visible.sync="progressVisible"
      width="50%"
      >
      <el-timeline>
      <el-timeline-item
      v-for="(activity, index) in progressInfo "
      :key="index"
      :timestamp="activity.time">
      {{activity.context}}
    </el-timeline-item>
  </el-timeline>
    </el-dialog>
  </el-card>
  </div>
</template>
<script>
import cityData from './citydata'
export default {
  data: () => ({
    queryInfo: {
      query: '',
      pagenum: 1,
      pagesize: 10
    },
    ordersList: [],
    total: 0,
    addressVisible: false,
    addressForm: {
      address1: [],
      address2: ''
    },
    addressRules: {
      address1: [
         { required: true, message: '请选择省市区县', trigger: 'blur' }
      ],
      address2: [
        { required: true, message: '请填写详细地址', trigger: 'blur' }
      ]
    },
    cityData,
    progressVisible: false,
    progressInfo: []
  }),
  created() {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const { data: { data, meta } } = await this.$http.get('orders', { params: this.queryInfo })
      if (meta.status !== 200) return this.$message.error(meta.msg)
      console.log(data)
      this.ordersList = data.goods
      this.total = data.total
    },
    handleSizeChange(newSize) {
      this.ordersList.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange(newPage) {
      this.ordersList.pagenum = newPage
      this.getOrderList()
    },
    // 控制修改对话框的显示与隐藏
    showBox() {
      this.addressVisible = true
    },
    // 监听修改地址对话框的关闭事件
    addressClosed() {
      this.$refs.addressFormRef.resetFields()
    },
    async showProgress() {
      const { data: { data, meta } } = await this.$http.get(`/kuaidi/${1106975712662}`)
      if (meta.status !== 200) return this.$message.error(meta.msg)
      // console.log(data)
      this.progressInfo = data
      this.progressVisible = true
    }
  }
}
</script>
<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
