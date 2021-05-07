<!--  -->
<template>
  <div>

    <!-- 面包屑导航区 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>

      <!-- 表格视图 -->
      <el-table :data="rightsList" style="width: 70%" border stripe>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <!-- 权限名称列 -->
        <el-table-column  label="权限名称" prop="authName"></el-table-column>
        <!-- 路径 -->
        <el-table-column label="路径" prop="path"></el-table-column>
        <!-- 权限等级 -->
        <el-table-column label="权限等级" prop="level">
          <!-- 权限图标 -->
          <template slot-scope="scope">
            <!-- 一级权限图标 -->
            <el-tag v-if="scope.row.level ==  '0' ">一级权限</el-tag>
            <!-- 二级权限图标 -->
            <el-tag v-else-if="scope.row.level ==  '1'" type="success">二级权限</el-tag>
            <!-- 三级权限图标 -->
            <el-tag v-else type="warning">三级权限</el-tag>
          </template>
        </el-table-column>

      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 权限列表
      rightsList: [],
    };
  },
  methods: {
    // 获取全写列表
    async getRightsList() {
      const {data: res } = await this.$http.get("rights/list")
      if(res.meta.status !== 200) {
        return this.$message.error("获取用户列表失败！")
      }
      this.rightsList = res.data
    }
  },
  created() {
    this.getRightsList();
  },

}

</script>

<style  scoped>
.el-card {
  margin-top: 15px;
}
</style>