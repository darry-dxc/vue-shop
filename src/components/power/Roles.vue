<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <!-- 添加角色按钮区域 -->
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
                </el-col>
            </el-row>
            <!-- 角色列表区域 -->
            <el-table :data="roleList" border stripe>
                <!-- 展开列 -->
                <el-table-column type="expand">
                  <template slot-scope="scope">
                    <el-row :class="['bdbottom', i1 === 0 ? 'bdtop':'','vcenter']" v-for="(item1, i1) in scope.row.children" :key='item1.id'>
                      <!-- 渲染一级权限 -->
                      <el-col :span="5">
                        <el-tag closable @close='removeRightById(scope.row,item1.id)'>{{item1.authName}}</el-tag>
                        <i class="el-icon-caret-right"></i>
                      </el-col>
                      <!-- 渲染二级和三级权限 -->
                      <el-col :span="19">
                        <!-- 通过for循环嵌套渲染二级权限 -->
                        <el-row :class="[i2 === 0 ?'':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id" >
                          <el-col :span="6">
                            <el-tag type="success" closable @close='removeRightById(scope.row,item2.id)'>{{item2.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                          </el-col>
                          <el-col :span="18">
                            <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable @close='removeRightById(scope.row,item3.id)'>
                              {{item3.authName}}
                            </el-tag>
                          </el-col>
                        </el-row>
                      </el-col>
                    </el-row>
                  </template>
                </el-table-column>
                <el-table-column type="index">
                    <!-- 索引列 -->
                </el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作" width="300px">
                    <template slot-scope="scope">
                         <!-- 修改按钮 -->
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="SetRoleDialog(scope.row.id)">编辑</el-button>
                        <!-- 删除按钮 -->
                        <el-button type="dager" icon="el-icon-delete" size="mini" @click="removeRoleById(scope.row.id)">删除</el-button>
                        <!-- 权限分配按钮 -->
                        <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <!-- 分配权限的对话框 -->
        <el-dialog
          title="分配权限"
          :visible.sync="setRightDialogVisible"
          width="50%"
          @close='setRightDialogClosed'>
          <!-- 树形控件 -->
          <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key='id' default-expand-all :default-checked-keys="defKeys" ref='treeRef'></el-tree>
          <span slot="footer" class="dialog-footer">
            <el-button @click="setRightDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="allowRights">确 定</el-button>
          </span>
        </el-dialog>
        <!-- 添加角色的对话框 -->
        <el-dialog
          title="添加角色"
          :visible.sync="addDialogVisible"
          width="50%"
          @close='addDialogClosed'>
          <el-form :model="addForm" ref="addFormRef" label-width="70px">
            <el-form-item label="角色名称" prop="roleName">
              <el-input v-model="addForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc">
              <el-input v-model="addForm.roleDesc"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addRole">确 定</el-button>
          </span>
        </el-dialog>
        <!-- 编辑角色的对话框 -->
        <el-dialog
          title="编辑角色"
          :visible.sync="editRoleDialogVisible"
          width="50%">
          <el-form :model="editRole" ref="editRoleRef" label-width="70px">
            <el-form-item label="角色名称" prop="roleName">
              <el-input v-model="editRole.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc">
              <el-input v-model="editRole.roleDesc"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="editRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editRoleInfo">确 定</el-button>
          </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色列表数据
      roleList: [],
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 控制编辑角色对话框的显示与隐藏
      editRoleDialogVisible: false,
      // 控制添加角色对话框的显示与隐藏
      addDialogVisible: false,
      // 所有权限的数据
      rightsList: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点的id值
      defKeys: [],
      // 当前即将分配角色权限的id
      roleId: '',
      // 查询到的角色信息对象
      editRole: {},
      // 添加角色的表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      }
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')
      this.roleList = res.data
      this.total = res.data.total
      console.log(res)
    },
    // 根据id删除对应的权限
    async removeRightById (role, rightId) {
      // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') return this.$message.error('取消了删除')
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败')
      // this.getRolesList()
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限数据失败')
      // 获取到的权限数据保存到data中
      this.rightsList = res.data
      console.log(this.rightsList)
      // 递归获取三级节点的id
      this.getLeafKeys(role, this.defKeys)

      this.setRightDialogVisible = true
    },
    // 通过递归的形式获取角色下所有三级权限的id，并保存到defKeys中
    getLeafKeys (node, arr) {
      // 如果当前的node节点不包含children属性，则是三级节点
      if (!node.children) return arr.push(node.id)
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allowRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idstr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idstr })
      if (res.meta.status !== 200) return this.$message.error('分配权限失败')
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    // 展示编辑角色的对话框
    async SetRoleDialog (id) {
      // console.log(id)
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('查询角色信息失败')
      this.editRole = res.data
      this.editRoleDialogVisible = true
    },
    // 修改角色信息并提交
    async editRoleInfo () {
      // 发起修改角色信息的请求
      const { data: res } = await this.$http.put('roles/' + this.editRole.roleId, { roleName: this.editRole.roleName, roleDesc: this.editRole.roleDesc })
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('更新角色信息失败')
      // 关闭对话框
      this.editRoleDialogVisible = false
      // 刷新数据列表
      this.getRolesList()
      // 提示修改角色成功
      this.$message.success('更新角色信息成功')
    },
    // 根据id删除对应的角色信息
    async removeRoleById (id) {
      // 询问用户是否删除数据
      const confirmResult = await this.$confirm('此操作将永久删除角色，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户确认删除，则返回字符串confirm
      // 如果用户取消了删除，则返回字符串cancel
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') return this.$message.info('已经取消删除')
      console.log('确认了删除')
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除用户失败')
      this.$message.success('删除用户成功')
      this.getRolesList()
    },
    // 点击按钮添加新角色
    async addRole () {
      const { data: res } = await this.$http.post('roles', this.addForm)
      if (res.meta.status !== 201) this.$message.error('添加角色失败！')
      this.$message.success('添加角色成功！')
      // 隐藏添加角色的对话框
      this.addDialogVisible = false
      // 重新获取角色列表数据
      this.getRolesList()
    },
    // 监听添加用户对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
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
