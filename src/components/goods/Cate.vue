<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card>
      <!--添加框-->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddDialog()"
            >添加分类</el-button
          >
        </el-col>
      </el-row>
      <!--表格-->
      <tree-table
        class="tree-table"
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
      >
        <template slot="isOk" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen;"
          ></i>
          <i class="el-icon-error" v-else style="color: red;"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level == 0">一级</el-tag>
          <el-tag v-else-if="scope.row.cat_level == 1" type="success"
            >二级</el-tag
          >
          <el-tag v-else type="warning">三级</el-tag>
        </template>
        <template slot="operation" slot-scope="scope">
          <el-button type="primary" @click="showEditDialog(scope.row)"
            ><i class="el-icon-edit"></i>编辑</el-button
          >
          <el-button type="danger" @click="removeRoleById(scope.row)"
            ><i class="el-icon-delete"></i>删除</el-button
          >
        </template>
      </tree-table>

      <!--分页区域-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[10, 50, 100, 300]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
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
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <span class="demonstration"></span>
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged()"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate()">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑分类的对话框 -->
    <el-dialog
      title="编辑分类"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClose()"
    >
      <el-form
        :model="editForm"
        :rules="addFormRules"
        ref="editFormRef"
        label-width="80px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate()">确 定</el-button>
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
      // 当前页总数据
      total: 0,
      cateList: [],
      parentCateList: [],
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isOk'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'operation'
        }
      ],
      addDialogVisible: false,
      editDialogVisible: false,
      addForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      editForm: {
        cat_id: '',
        cat_name: ''
      },
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true,
        expandTrigger: true
      },
      // 选中的父级分类的数组
      selectedKeys: [],
      addFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    getCateList() {
      this.$http.get('/categories', { params: this.queryInfo }).then(result => {
        let res = this.$resolveResponse(result)
        if (!res) {
          this.$message.error(result.data.meta.msg)
        } else {
          this.cateList = res.result
          this.total = res.total
        }
      })
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    showAddDialog() {
      this.getParentCateList()
      this.addDialogVisible = true
    },
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
      this.selectedKeys = []
      this.addForm.cat_pid = 0
      this.addForm.cat_level = 0
    },
    getParentCateList() {
      this.$http.get('/categories', { params: { type: 2 } }).then(result => {
        let res = this.$resolveResponse(result)
        if (!res) {
          this.$message.error(result.data.meta.msg)
        } else {
          this.parentCateList = res
        }
      })
    },
    parentCateChanged() {
      if (this.selectedKeys.length > 0) {
        this.addForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addForm.cat_level = this.selectedKeys.length
      } else {
        this.addForm.cat_pid = 0
        this.addForm.cat_level = 0
      }
    },
    addCate() {
      this.$refs.addFormRef.validate(valid => {
        if (!valid) {
          return
        }
        this.$http
          .post('/categories', {
            cat_name: this.addForm.cat_name,
            cat_pid: this.addForm.cat_pid,
            cat_level: this.addForm.cat_level
          })
          .then(result => {
            let res = result.data
            if (res.meta.status === 201) {
              this.$message.success(res.meta.msg)
            } else {
              this.$message.error(res.meta.msg)
            }
          })
      })
      this.addDialogVisible = false
    },
    removeRoleById(cateInfo) {
      this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http.delete('/categories/' + cateInfo.cat_id).then(result => {
            let response = result.data
            if (response.meta.status !== 200) {
              return this.$message.error(response.meta.msg)
            } else {
              this.$message({
                type: 'success',
                message: response.meta.msg
              })
              this.getCateList()
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
    showEditDialog(cateInfo) {
      this.editForm.cat_id = cateInfo.cat_id
      this.editForm.cat_name = cateInfo.cat_name
      this.editDialogVisible = true
    },
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },
    editCate() {
      this.$http
        .put(`/categories/${this.editForm.cat_id}`, {
          cat_name: this.editForm.cat_name
        })
        .then(result => {
          let res = result.data
          if (res.meta.status === 200) {
            this.$message.success(res.meta.msg)
            this.getCateList()
          } else {
            this.$message.error(res.meta.msg)
          }
        })
      this.editDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.tree-table {
  margin-top: 15px;
  font-size: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
