<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert
        title="注意：只允许为第三级分类设置相关参数！"
        type="warning"
        :closable="false"
        show-icon
      >
      </el-alert>
      <!--选择商品分类-->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader
            :options="cateList"
            :props="cateProps"
            v-model="selectedCateList"
            @change="handleChange"
          >
          </el-cascader>
        </el-col>
      </el-row>

      <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button
            :disabled="isBtnDisabled"
            type="primary"
            size="mini"
            @click="addDialogVisible = true"
            >添加参数</el-button
          >
          <!--动态表格参数-->
          <el-table :data="manyTableData" style="width: 100%">
            <el-table-column type="index" label="#"> </el-table-column>
            <el-table-column prop="attr_name" label="参数名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="small"
                  @click="showEditDialog(scope.row)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="small"
                  @click="removeParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button
            :disabled="isBtnDisabled"
            type="primary"
            size="mini"
            @click="addDialogVisible = true"
            >添加属性</el-button
          >
          <!--静态属性参数-->
          <el-table :data="onlyTableData" style="width: 100%">
            <el-table-column type="index" label="#"> </el-table-column>
            <el-table-column prop="attr_name" label="属性名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="small"
                  @click="showEditDialog(scope.row)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="small"
                  @click="removeParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!--添加对话框-->
    <el-dialog
      :title="titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      :before-close="addDialogClose"
    >
      <el-form
        :rules="addFormRules"
        ref="addFormRef"
        :model="addForm"
        label-width="80px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams()">确 定</el-button>
      </span>
    </el-dialog>

    <!--编辑对话框-->
    <el-dialog
      :title="titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      :before-close="editDialogClose"
    >
      <el-form
        :rules="addFormRules"
        ref="editFormRef"
        :model="editForm"
        label-width="80px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cateList: [],
      selectedCateList: [],
      // 控制添加对话框演示隐藏
      addDialogVisible: false,
      editDialogVisible: false,
      // 切换标签中选中的标签名
      activeName: 'many',
      addForm: {
        attr_name: ''
      },
      editForm: {
        attr_id: '',
        attr_name: ''
      },
      manyTableData: [],
      onlyTableData: [],
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children',
        expandTrigger: 'hover'
      },
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    getCateList() {
      this.$http.get('/categories').then(result => {
        let response = result.data
        if (response.meta.status === 200) {
          this.cateList = response.data
        } else {
          this.$message.error(response.meta.msg)
        }
      })
    },
    // 选择商品分类的级联选择
    handleChange() {
      this.getParamsData()
    },
    // 切换标签页
    handleClick() {
      this.getParamsData()
    },
    // 获取需要渲染到表格中的分类数据
    getParamsData() {
      // 证明选中的不是三级分类
      if (this.selectedCateList.length !== 3) {
        this.selectedCateList = []
      } else {
        // 根据所选分类id和所处的标签发送请求
        this.$http
          .get(`/categories/${this.cateId}/attributes`, {
            params: { sel: this.activeName }
          })
          .then(result => {
            let response = result.data
            if (response.meta.status === 200) {
              this.activeName === 'many'
                ? (this.manyTableData = response.data)
                : (this.onlyTableData = response.data)
            } else {
              this.$message.error(response.meta.msg)
            }
          })
      }
    },
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    },
    // 添加参数
    addParams() {
      this.$refs.addFormRef.validate(valid => {
        if (!valid) {
          return
        }
        this.$http
          .post(`/categories/${this.cateId}/attributes`, {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          })
          .then(result => {
            let res = result.data
            if (res.meta.status === 201) {
              this.$message.success(res.meta.msg)
              this.addDialogVisible = false
              this.getParamsData()
            } else {
              this.$message.error(res.meta.msg)
            }
          })
      })
    },
    showEditDialog(cateInfo) {
      this.$http
        .get(`/categories/${this.cateId}/attributes/${cateInfo.attr_id}`, {
          params: { attr_sel: this.activeName }
        })
        .then(result => {
          let res = result.data
          if (res.meta.status === 200) {
            this.editForm = res.data
          } else {
            this.$message.error(res.meta.msg)
          }
        })
      this.editDialogVisible = true
    },
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },

    editParams() {
      this.$http
        .put(`/categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName
        })
        .then(result => {
          let res = result.data
          if (res.meta.status === 200) {
            this.$message.success(res.meta.msg)
            this.editDialogVisible = false
            this.getParamsData()
          } else {
            this.$message.error(res.meta.msg)
          }
        })
    },
    removeParams(attrId) {
      this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http
            .delete(`/categories/${this.cateId}/attributes/${attrId}`)
            .then(result => {
              let response = result.data
              if (response.meta.status !== 200) {
                return this.$message.error(response.meta.msg)
              } else {
                this.$message({
                  type: 'success',
                  message: response.meta.msg
                })
                this.getParamsData()
              }
            })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    }
  },
  computed: {
    isBtnDisabled() {
      if (this.selectedCateList.length !== 3) {
        return true
      } else {
        return false
      }
    },
    cateId() {
      if (this.selectedCateList.length === 3) {
        return this.selectedCateList[this.selectedCateList.length - 1]
      } else {
        return null
      }
    },
    // 动态计算添加框的标题
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px;
}
</style>
