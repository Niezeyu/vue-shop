<!--  -->
<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 商品卡片区域 -->
    <el-card>
      <div>
        <el-row :gutter="10">
          <el-col :span="6">
            <!-- 添加分类按钮 -->
            <el-button type="primary" @click="showAddCate">添加分类</el-button></el-button>
          </el-col>
        </el-row>
      </div>

      <!-- 表格区域 -->
      <tree-table class="treeTable" :data="cateList" :columns="columns" :selection-type="false" :expand-type="false" show-index border>
        <!-- 是否有效列 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen"></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>

        <!-- 排序列 -->
        <template slot="order" slot-scope="scope">
          <!-- 一级 -->
          <el-tag  size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <!-- 二级 -->
          <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <!-- 三级 -->
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>

      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[3, 5, 10, 15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>

    <!-- 添加分类弹出框 -->
    <el-dialog
  title="添加分类"
  :visible.sync="addCateDialogVisible"
  width="50%" @close="addCateDialogClose">
  <!-- 添加分类的表单 -->

  <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px" class="demo-ruleForm">
    <el-form-item label="分类名称 : " prop="cat_name">
      <el-input v-model="addCateForm.cat_name"></el-input>
    </el-form-item>
    <el-form-item label="父级分类 : ">
      
      <el-cascader expand-trigger="hover"
    v-model="selectedKeys"
    :options="parentCateList"
    :props="cascaderProps"
    @change="parentCateChange" clearable change-on-select ></el-cascader>
      </el-form-item>
  </el-form>

  <span slot="footer" class="dialog-footer">
    <el-button @click="addCateDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      cateList: [],
      // 总数居条数
      total: 0,
      columns: [
        {
        label: "分类名称",
        prop: "cat_name"
        },
        {
        label: "是否有效",
        type: "template",
        template: "isok"
        },
        {
        label: "排序",
        type: "template",
        template: "order"
        },
        
        
      ],
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: "",
        cat_pid: 0,
        cat_level: 0,

      },
      addCateFormRules: {
        cat_name: [
          {
            required: true,
            message: "请输入分类名称",
            trigger: "blur"
          }
        ]
      },
      // 父级分类的列表
      parentCateList: [],
      cascaderProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children"

      },
      selectedKeys: [],

    }
  },
  created() {
    this.getCateLIst()
  },
  methods: {
    // 获取商品分类数据
    async getCateLIst() {
      const { data: res} = await this.$http.get("categories", {params: this.queryInfo})
      if(res.meta.status !== 200) {
        return this.$message.error("获取商品列表失败!")
      }
      this.cateList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateLIst()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateLIst()
    },
    // 点击按钮 添加分类
    showAddCate(){
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", { params: {type: 2} }  )
      if(res.meta.status !== 200) {
        return this.$message.error("获取父级商品泪飙失败!")
      }
      this.parentCateList = res.data 
    },
    parentCateChange() {
      if(this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
        return 
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }

    },
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if(!valid) return
        const {data: res} = await this.$http.post("categories", this.addCateForm)
        if(res.meta.status !== 201) {
          return this.$message.error("添加分类失败!")
        }
        this.$message.success("添加分类成功!")
        this.getCateLIst()
        this.addCateDialogVisible = false

      })
      
    },
    addCateDialogClose() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0

    }
  }
}
</script>

<style scoped>
.el-card{
  margin-top: 15px;

}
.el-tag{
  margin-left: 15px;
}
.treeTable{
  margin-top: 15px;
}
.el-cascader{
  width: 100%;
}
</style>
