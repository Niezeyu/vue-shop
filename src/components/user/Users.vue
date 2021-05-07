<!--  -->
<template>
  <!-- 面包屑导航区 -->
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <!-- 搜索区 -->

      <el-row :gutter="10">
        <el-col :span="10">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getUsersList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUsersList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true"
            >添加用户</el-button
          >
        </el-col>
      </el-row>

      <el-table :data="usersList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="userStatuChange(scope.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="width">
          <template slot-scope="scope">
            <el-button
              size="small"
              type="primary"
              icon="el-icon-edit"
              @click="editDialog(scope.row.id)"
            ></el-button>
            <el-button
              size="small"
              type="danger"
              icon="el-icon-delete"
              @click="removeUserById(scope.row.id)"
            ></el-button>
            <el-tooltip
              :enterable="false"
              class="item"
              effect="dark"
              content="分配角色"
              placement="top"
            >
              <el-button
                @click="setRole(scope.row)"
                size="small"
                type="warning"
                icon="el-icon-setting"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%"
      center
      @close="addDialogClose"
    >
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="70px"
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog
      title="修改用户信息"
      :visible.sync="editDialogVisite"
      width="50%"
      @close="editDialogClose"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="70px"
      >
        <el-form-item label="用户名">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisite = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配角色弹框 -->
    <el-dialog
      title="分配角色"
      :visible.sync="setRoleDialogVisible"
      width="50%"
      @close="setRoleDialogVisibleClose"
    >
      <div>
        <p>当前的用户: {{ userInfo.username }}</p>
        <p>当前的角色: {{ userInfo.role_name }}</p>
        <p>
          分配新角色:
          <el-select v-model="selectROleId" placeholder="请选择新角色">
            <el-option
              v-for="item in rolesLIst"
              :key="item.id"
              :label="item.roleName"
              :value="item.id"
            >
            </el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      usersList: [],
      total: 0,
      addDialogVisible: false,
      editDialogVisite: false,
      editForm: {},
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editFormRules: {
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' }],
        mobile: [{ required: true, message: '请输入邮箱', trigger: 'blur' }]
      },
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' }],
        mobile: [{ required: true, message: '请输入邮箱', trigger: 'blur' }]
      },
      setRoleDialogVisible: false,
      userInfo: {},
      rolesLIst: [],
      selectROleId: ''
    }
  },
  created() {
    this.getUsersList()
  },
  methods: {
    async getUsersList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      ;(this.usersList = res.data.users), (this.total = res.data.total)
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUsersList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUsersList()
    },
    async userStatuChange(userinfo) {
      const { data: res } = await this.$http.put(
        `users/${userinfo.id}/state/${userinfo.mg_state}`
      )
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('设置状态失败！')
      }
      this.$message.success('设置状态成功！')
    },
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    },
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addForm)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')
        this.addDialogVisible = false
        this.getUsersList()
      })
    },
    async editDialog(id) {
      this.editDialogVisite = true
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('修改用户信息失败')
      }
      this.editForm = res.data
      this.getUsersList()
    },
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },
    editUser() {
      this.editDialogVisite = false
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'users/' + this.editForm.id,
          { email: this.editForm.email, mobile: this.editForm.mobile }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('更新用户信息失败')
        }
        this.editDialogClose()
        this.getUsersList()
        this.$message.success('更新用户信息成功')
      })
    },
    async removeUserById(id) {
      // 询问用户是否删除
      const confirmRes = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) return
      this.$message.success('更新用户信息成功')
      this.getUsersList()
    },
    async setRole(userinfo) {
      // 在展示对话之前 获取所有角色列表
      this.userInfo = userinfo
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200)
        return this.$message.error('获取角色列表失败')
      this.rolesLIst = res.data
      this.setRoleDialogVisible = true
    },
    async saveRoleInfo() {
      if(!this.selectROleId) {
        return this.$message.error('请选择要的分配的角色') 
      }
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectROleId})
      console.log(res);
      if (res.meta.status !== 200){
        return this.$message.error('更新角色失败')
      }
      this.$message.success('更新角色成功')
      this.setRoleDialogVisible = false
      this.getUsersList()
    },
    setRoleDialogVisibleClose() {
      this.selectROleId = ""
      this.userInfo = {}
    }
  }
}
</script>

<style scoped>
.el-breadcrumb {
  margin-bottom: 15px;
}
.el-card {
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.15);
}
.el-table {
  margin-top: 15px;
  font-size: 12px;
}
.el-pagination {
  margin-top: 15px;
}
</style>
