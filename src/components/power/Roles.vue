<!--  -->
<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片试图 -->
    <el-card>
      <!-- 添加角色按钮区 -->
      <el-row>
        <el-col>
          <!-- 添加角色按钮 -->
          <el-button type="primary" @click="addDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>

      <!-- 添加用户弹框 -->
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
          <el-form-item label="名称" prop="roleName">
            <el-input v-model="addForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="描述" prop="roleDesc">
            <el-input v-model="addForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addRole">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 编辑角色弹框 -->
      <el-dialog
        title="编辑角色信息"
        :visible.sync="editDialogVisite"
        width="70%"
        @close="editDialogClose"
      >
        <el-form
          :model="editForm"
          :rules="editFormRules"
          ref="editFormRef"
          label-width="50px"
        >
          <el-form-item label="角色ID" prop="roleId" label-width="100px">
            <el-input v-model="editForm.roleId" disabled></el-input>
          </el-form-item>
          <el-form-item label="角色名称" prop="roleName" label-width="100px">
            <el-input v-model="editForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc" label-width="100px">
            <el-input v-model="editForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisite = false">取 消</el-button>
          <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>

    <!-- 表格区域 -->
    <el-table :data="rolesList" style="width: 100%" border stripe>
      <!-- 展开列 -->
      <el-table-column type="expand">
        <template slot-scope="scope">
          <el-row
            class="first_row"
            :gutter="10"
            v-for="(item1, i1) in scope.row.children"
            :key="item1.id"
          >
            <!-- 渲染一级权限 -->
            <el-col :span="5">
              <el-tag closable @close="removeRightsById(scope.row, item1.id)">{{
                item1.authName
              }}</el-tag>
              <i class="el-icon-caret-right"></i>
            </el-col>
            <!-- 渲染二级三级权限 -->
            <el-col :span="19">
              <el-row
                class="second_row"
                v-for="(item2, i2) in item1.children"
                ::key="item2.id"
              >
                <el-col :span="6">
                  <el-tag
                    type="success"
                    closable
                    @close="removeRightsById(scope.row, item2.id)"
                    >{{ item2.authName }}</el-tag
                  >
                  <i class="el-icon-caret-right"></i>
                </el-col>

                <el-col :span="18">
                  <el-tag
                    closable
                    @close="removeRightsById(scope.row, item3.id)"
                    v-for="(item3, i3) in item2.children"
                    :key="item3.id"
                    type="warning"
                    >{{ item3.authName }}</el-tag
                  >
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <!-- 索引列 -->
      <el-table-column type="index"></el-table-column>
      <!-- 角色名称列 -->
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <!-- 角色描述 -->
      <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
      <!-- 操作 -->
      <el-table-column label="操作" width="300px">
        <!-- 操作图标 -->
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            icon="el-icon-edit"
            @click="editDialog(scope.row.id)"
            >编辑</el-button
          >
          <el-button
            size="mini"
            type="danger"
            icon="el-icon-delete"
            @click="removeRolesById(scope.row.id)"
            >删除</el-button
          >
          <el-button
            size="mini"
            type="warning"
            icon="el-icon-setting"
            @click="showRightDialog(scope.row)"
            >分配权限</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightsDialogVisible"
      width="50%"
    >
      <el-tree
        :data="rightsList"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allKeys">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      rolesList: [],
      editDialogVisite: false,
      editDialogVisite: false,
      editForm: {},
      editFormRules: {
        roleId: [{ required: true, message: '请输入用户ID', trigger: 'blur' }],
        roleName: [
          { required: true, message: '请输入用户角色', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }
        ]
      },
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      addFormRules: {
        roleName: [
          { required: true, message: '请输入用户角色', trigger: 'blur' },
          { min: 0, max: 10, message: '长度在 0 到 10 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入用户描述', trigger: 'blur' },
          { min: 0, max: 15, message: '长度在 0 到 15 个字符', trigger: 'blur' }
        ]
      },
      addDialogVisible: false,
      // 分配权限对话框显示
      setRightsDialogVisible: false,
      rightsList: [],
      // 树形控件的数据绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点id值
      defKeys: [],
      roleId: ''
    }
  },
  methods: {
    // 获取所有角色列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
    },
    addRole() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败')
        }
        this.$message.success('添加角色成功')
        this.addDialogVisible = false
        this.getRolesList()
      })
    },
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    },
    // 编辑角色
    async editDialog(id) {
      this.editDialogVisite = true
      const { data: res } = await this.$http.get('roles/' + id)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('修改角色信息失败')
      }
      this.editForm = res.data
      this.getRolesList()
    },
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },
    editUser() {
      this.editDialogVisite = false
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'roles/' + this.editForm.roleId,
          { roleName: this.editForm.roleName, roleDesc: this.editForm.roleDesc }
        )
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('更新角色信息失败')
        }
        this.editDialogClose()
        this.getRolesList()
        this.$message.success('更新角色信息成功')
      })
    },
    // 删除角色
    async removeRolesById(id) {
      // 询问用户是否删除
      const confirmRes = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
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
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return
      this.$message.success('更新用户信息成功')
      this.getRolesList()
    },
    // 删除权限
    async removeRightsById(role, rightId) {
      const confirmRes = await this.$confirm(
        '此操作将永久删除该权限, 是否继续?',
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
      if (confirmRes !== 'confirm') {
        returnthis.$message, info('取消了删除')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status !== 200) return
      this.$message.success('更新权限成功')
      role.children = res.data
    },
    async showRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限失败')
      // 获取到的权限保存到数据中
      this.rightsList = res.data
      // 获取三级节点的id
      this.defKeys = []
      this.getRightsId(role, this.defKeys)
      this.setRightsDialogVisible = true
    },
    // 获取三级权限的id值
    getRightsId(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getRightsId(item, arr))
    },
    async allKeys() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status !== 200) return this.$message.error('分配权限失败')
      this.$message.success('分配权限成功')
      this.setRightsDialogVisible = false
      this.getRolesList()
    }
  },
  created() {
    this.getRolesList()
  }
}
</script>

<style scoped>
.el-card {
  margin-top: 15px;
}
.el-tag {
  margin: 15px;
}
.first_row {
  border-bottom: 1px solid #eee;
}
.first_row:first-child {
  border-top: 0.5px solid #eee;
}
.second_row {
  border-bottom: 1px solid #eee;
}
</style>
