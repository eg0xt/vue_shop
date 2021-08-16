<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>
      <el-table :data="rolesList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template v-slot="slotProps">
            <el-row
              v-for="(item1, index1) in slotProps.row.children"
              :key="item1.id"
              :class="['bdbottom', index1 === 0 ? 'bdtop' : '', 'vcenter']"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(slotProps.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  v-for="(item2, index2) in item1.children"
                  :key="item2.id"
                  :class="[index2 === 0 ? '' : 'bdtop', 'vcenter']"
                >
                  <!-- 渲染二级权限 -->
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(slotProps.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(slotProps.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="roleName" label="角色"> </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"> </el-table-column>
        <el-table-column label="操作" width="300px">
          <template v-slot="slotProps">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditRoleForm(slotProps.row.id)"
              >编辑</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteRole(slotProps.row.id)"
              >删除</el-button
            >
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightDialog(slotProps.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addRoleDialogVisible"
      @closed="addRoleDialogClosed"
    >
      <el-form ref="addRoleFormRef" :model="addRoleForm">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑角色对话框 -->
    <el-dialog title="编辑角色" :visible.sync="editRoleDialogVisible">
      <el-form ref="editRoleFormRef" :model="editRoleForm">
        <el-form-item label="角色名称">
          <el-input v-model="editRoleForm.roleName" disabled></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      @closed="setRightDialogClosed"
    >
      <!-- 树形控件 -->
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
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setRightInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色列表数据
      rolesList: [],
      addRoleDialogVisible: false,
      addRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      editRoleDialogVisible: false,
      editRoleForm: {},
      //   控制是否展示分配权限对话框
      setRightDialogVisible: false,
      //   所有权限数据
      rightsList: [],
      //   树形控件属性绑定对象
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      //   树形控件默认选中的节点id值数组
      defKeys: [],
      currentRoleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) { return this.$message.error('获取角色列表失败') }
      this.rolesList = res.data
    },
    addRoleDialogClosed () {
      this.$refs.addRoleFormRef.resetFields()
    },
    async addRoleInfo () {
      const { data: res } = await this.$http.post('roles', this.addRoleForm)
      if (res.meta.status !== 201) { return this.$message.error('添加角色失败') }
      this.$message.success('添加角色成功')
      this.getRolesList()
      this.addRoleDialogVisible = false
    },
    async deleteRole (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') { return this.$message.info('已取消删除') }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) { return this.$message.error('删除角色失败') }
      this.$message.success('删除角色成功')
      this.getRolesList()
    },
    async showEditRoleForm (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) { return this.$message.error('获取角色信息失败') }
      this.editRoleDialogVisible = true
      this.editRoleForm = res.data
    },
    async editRoleInfo () {
      const { data: res } = await this.$http.put('roles/' + this.editRoleForm.roleId, {
        roleName: this.editRoleForm.roleName,
        roleDesc: this.editRoleForm.roleDesc
      })
      if (res.meta.status !== 200) { return this.$message.error('编辑角色信息失败') }
      this.editRoleDialogVisible = false
      this.getRolesList()
      this.$message.success('编辑角色信息成功')
    },
    // 根据id删除对应权限
    async removeRightById (role, rightId) {
      // 弹框提示用户是否确认删除
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') { return this.$message.info('已取消删除') }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        this.$message.error('删除权限失败')
      }
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      // 获取所有权限列表
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) { return this.$message.error('获取权限列表失败') }
      this.rightsList = res.data
      this.getDefKeys(role, this.defKeys)
      this.currentRoleId = role.id
      this.setRightDialogVisible = true
    },
    // 通过递归的形式获取角色下所有三级权限id数组
    getDefKeys (value, arr) {
      if (!value.children) {
        return arr.push(value.id)
      }
      value.children.forEach(item => {
        this.getDefKeys(item, arr)
      })
    },
    // 监听分配权限对话框关闭事件
    setRightDialogClosed () {
      this.defKeys = []
    },
    async setRightInfo () {
      const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
      const rightsIdStr = keys.join(',')

      const { data: res } = await this.$http.post(`roles/${this.currentRoleId}/rights`, {
        rids: rightsIdStr
      })
      if (res.meta.status !== 200) { return this.$meassage.error('分配权限失败') }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }

}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
