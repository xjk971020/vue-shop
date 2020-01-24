<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card class="card">
      <el-row :gutter="20">
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>
      <!--用户列表区域-->
      <el-table :data="rolesList" border stripe style="width: 100%">
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bd-bottom', i1 === 0 ? 'bd-top' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!--渲染一级权限-->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!--渲染二级和三级权限-->
              <el-col :span="19">
                <el-row
                  :class="[i2 === 0 ? '' : 'bd-top', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <!--二级权限-->
                  <el-col :span="6">
                    <el-tag
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      type="success"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!--三级权限-->
                  <el-col :span="18">
                    <el-tag
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                    >
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="roleName" label="角色名称"> </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"> </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="small"
              @click="showEditDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="small"
              @click="removeRoleById(scope.row.id)"
              >删除</el-button
            >
            <el-tooltip effect="dark" content="分配权限" placement="top">
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="small"
                @click="showSetRightDialog(scope.row)"
                >分配权限</el-button
              >
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色的对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClose()"
    >
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="80px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole()">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑角色的对话框 -->
    <el-dialog
      title="编辑角色"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClose()"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="80px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole()">确 定</el-button>
      </span>
    </el-dialog>

    <!--分配权限对话框-->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed()"
    >
      <el-tree
        :data="rightsList"
        :props="rightTreeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defaultKeys"
        ref="rightTreeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rolesList: [],
      rightsList: [],
      defaultKeys: [],
      // 当前即将分配权限的角色id
      roleId: '',
      // 树型空间属性绑定
      rightTreeProps: {
        label: 'authName',
        children: 'children'
      },
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      editForm: {},
      addDialogVisible: false,
      editDialogVisible: false,
      setRightDialogVisible: false,
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }
        ]
      },
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.rolesList = this.getRoleList()
  },
  methods: {
    getRoleList() {
      this.$http.get('/roles').then(result => {
        if (result.data.meta.status !== 200) {
          this.$message.error(result.data.meta.msg)
        } else {
          this.rolesList = result.data.data
        }
      })
    },
    // 展示修改用户的对话框
    showEditDialog(id) {
      this.$http.get(`/roles/${id}`).then(result => {
        let response = result.data
        if (response.meta.status !== 200) {
          return this.$message.error(response.meta.msg)
        } else {
          this.editForm = response.data
          console.log(this.editForm)
          this.editDialogVisible = true
        }
      })
    },
    removeRoleById(id) {
      this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http.delete('/roles/' + id).then(result => {
            let response = result.data
            if (response.meta.status !== 200) {
              return this.$message.error(response.meta.msg)
            } else {
              this.$message({
                type: 'success',
                message: response.meta.msg
              })
              this.getRoleList()
            }
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    addRole() {
      //   this.$refs.addFormRef.validate(valid => {
      //     if (!valid) {
      //       return
      //     }
      this.$http.post('roles', this.addForm).then(result => {
        let response = result.data
        if (response.meta.status !== 201) {
          return this.$message.error(response.meta.msg)
        } else {
          this.$message.success(response.meta.msg)
          this.addDialogVisible = false
          this.getRoleList()
        }
      })
      //   })
    },
    editRole() {
      this.$http
        .put('roles/' + this.editForm.roleId, {
          roleName: this.editForm.roleName,
          roleDesc: this.editForm.roleDesc
        })
        .then(result => {
          let response = result.data
          if (response.meta.status !== 200) {
            return this.$message.error(response.meta.msg)
          } else {
            this.$message.success('修改成功')
            this.editDialogVisible = false
            this.getRoleList()
          }
        })
    },
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    },
    // 对话框关闭时需要清除在对话框上的数据
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },
    removeRightById(roleInfo, rightId) {
      console.log(roleInfo)
      this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http
            .delete(`roles/${roleInfo.id}/rights/${rightId}`)
            .then(result => {
              let response = result.data
              if (response.meta.status !== 200) {
                return this.$message.error(response.meta.msg)
              } else {
                this.$message.success(response.meta.msg)
                roleInfo.children = response.data
              }
            })
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
        .catch(() => {
          this.$message.info({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 展示分配权限的对话框
    showSetRightDialog(roleInfo) {
      this.roleId = roleInfo.id
      // 获取所有权限数据
      this.$http.get('/rights/tree').then(result => {
        let response = result.data
        if (response.meta.status !== 200) {
          return this.$message.error(response.meta.msg)
        } else {
          this.rightsList = response.data
          this.getLeafKeys(roleInfo, this.defaultKeys)
        }
      })
      this.setRightDialogVisible = true
    },
    // 通过递归的形式获取所有三级权限id
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defaultKeys = []
    },
    // 分配权限
    allotRights() {
      // ...拓展运算符
      const keys = [
        ...this.$refs.rightTreeRef.getCheckedKeys(),
        ...this.$refs.rightTreeRef.getHalfCheckedNodes()
      ]
      const idStr = keys.join(',')
      this.$http
        .post(`/roles/${this.roleId}/rights`, { rids: idStr })
        .then(result => {
          let response = result.data
          if (response.meta.status !== 200) {
            return this.$message.error(response.meta.msg)
          } else {
            this.$message.success(response.meta.msg)
            this.getRoleList()
            this.setRightDialogVisible = false
          }
        })
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bd-top {
  border-top: 1px solid #eee;
}

.bd-bottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
