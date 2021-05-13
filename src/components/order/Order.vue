<!--  -->
<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input
            placeholder="请输入商品名称"
            v-model="queryInfo.query"
            clearable
            @clear="getOrderList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getOrderList"
            ></el-button>
          </el-input>
        </el-col>
      </el-row>

      <!-- 表格区域 -->
      <el-table :data="orderlist" border stripe style="width: 100%">
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column
          label="订单价格"
          width="120px"
          prop="order_price"
        ></el-table-column>
        <el-table-column label="是否付款" width="120px" prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === 1"
              >已付款</el-tag
            >
            <el-tag type="warning" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="是否发货"
          width="120px"
          prop="is_send"
        ></el-table-column>
        <el-table-column label="下单时间" width="200px" prop="create_time">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200px">
          <template slot-scope="scope">
            <el-button
              @click="showBox"
              type="primary"
              size="mini"
              icon="el-icon-edit"
            ></el-button>
            <el-button
            @click="addressInfo"
              type="success"
              size="mini"
              icon="el-icon-location"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>

      <!-- 编辑按钮弹出框 -->
      <el-dialog title="修改地址" :visible.sync="addressVisible" width="50%" @close="addressDialogClosed">
        <el-form
          :model="addressForm"
          :rules="addressFormRules"
          ref="addressFormRef"
          label-width="100px"
        >
          <el-form-item label="省市区/县" prop="address1">
            <el-cascader
              :options="cityData"
              v-model="addressForm.address1"
              >
            </el-cascader>
          </el-form-item>
          <el-form-item label="详细地址" prop="address2">
            <el-input v-model="addressForm.address2"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addressVisible = false">取 消</el-button>
          <el-button type="primary" @click="addressVisible = false"
            >确 定</el-button
          >
        </span>
      </el-dialog>

      <!-- 物流信息弹出框 -->
      <el-dialog title="物流进度" :visible.sync="addressInfoVisible" width="50%" @close="addressInfoDialogClosed">
        <el-form
          :model="addressInfoForm"
          :rules="addressInfoFormRules"
          ref="addressInfoFormRef"
          label-width="100px"
        >
          <span>暂无物流信息！</span>
        </el-form>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
import cityData from './citydata.js'

export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderlist: [],
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: ""
      },
      addressFormRules: {
        address1: [
          {
            required: true,
            message: '请输入省市区/县',
            trigger: 'blur'
          }
        ],
        address2: [
          {
            required: true,
            message: '请输入详细地址',
            trigger: 'blur'
          }
        ]
      },
      cityData,
      addressInfoVisible: false,
      addressInfoForm: {

      },
      addressInfoFormRules: {

      },
    }
  },

  created() {
    this.getOrderList()
  },

  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get(`orders`, {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败！')
      }
      console.log(res)
      this.orderlist = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    showBox() {
      this.addressVisible = true
    },
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields()
    },
    addressInfoDialogClosed() {
      this.$refs.addressInfoFormRef.resetFields()
    },
   addressInfo() {
      
      this.addressInfoVisible = true
     
    },

  }
}
</script>

<style scoped>
.el-cascader{
  width: 100%;
}
</style>
