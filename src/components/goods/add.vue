<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert
        title="添加商品信息"
        type="info"
        center
        show-icon
        :closable="false"
      >
      </el-alert>
      <!-- 步骤条区域 -->
      <el-steps
        :space="300"
        :active="activeIndex - 0"
        finish-status="success"
        align-center
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <el-form
        ref="addFormRef"
        :model="addForm"
        :rules="addFormRules"
        label-position="top"
        label-width="80px"
      >
        <el-tabs
          :tab-position="'left'"
          v-model="activeIndex"
          :before-leave="beforeLeaveTab"
          @tab-click="tabClicked()"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="cateProps"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item
              :label="item.attr_name"
              v-for="item in manyTableData"
              :key="item.attr_id"
            >
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  border
                  :label="cb"
                  v-for="(cb, i) in item.attr_vals"
                  :key="i"
                >
                </el-checkbox> </el-checkbox-group
            ></el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item
              :label="item.attr_name"
              v-for="item in onlyTableData"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload
              :action="uploadUrl"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              multiple
              list-type="picture"
              :headers="headerObj"
              :on-success="uploadSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!-- bidirectional data binding（双向数据绑定） -->
            <quill-editor
              v-model="addForm.goods_introduce"
              ref="myQuillEditor"
              :options="editorOption"
            >
            </quill-editor>
            <el-button type="primary" class="addBtn" @click="addGood()"
              >添加商品</el-button
            >
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <img :src="previewUrl" alt="" class="previewImg" />
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  data() {
    return {
      activeIndex: '0',
      cateList: [],
      manyTableData: [],
      onlyTableData: [],
      previewVisible: false,
      // 图片组件发送请求的请求头
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      uploadUrl: 'https://www.liulongbin.top:8888/api/private/v1/upload',
      previewUrl: '',
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children',
        expandTrigger: 'hover'
      },
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        // 图片临时地址的数据
        pics: [],
        // 商品详情
        goods_introduce: '',
        attrs: []
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
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
    // 级联选择器修改选项
    handleChange() {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    // 即将发生标签页的切换
    beforeLeaveTab(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类')
        return false
      }
      return true
    },
    // 切换面板时发送请求, 面板成功切换后才会触发
    tabClicked() {
      // 证明访问的是动态参数面板
      if (this.activeIndex === '1') {
        this.$http
          .get(`/categories/${this.cateId}/attributes`, {
            params: { sel: 'many' }
          })
          .then(result => {
            let res = result.data
            if (res.meta.status === 200) {
              this.manyTableData = res.data
              res.data.forEach(item => {
                item.attr_vals =
                  item.attr_vals === '' ? [] : item.attr_vals.split(' ')
              })
            } else {
              this.$message.error(res.meta.msg)
            }
          })
      } else if (this.activeIndex === '2') {
        this.$http
          .get(`/categories/${this.cateId}/attributes`, {
            params: { sel: 'only' }
          })
          .then(result => {
            let res = result.data
            if (res.meta.status === 200) {
              this.onlyTableData = res.data
            } else {
              this.$message.error(res.meta.msg)
            }
          })
      }
    },
    // 图片预览
    handlePreview(file) {
      this.previewUrl = file.response.data.url
      this.previewVisible = true
    },
    // 处理图片移除
    handleRemove(file) {
      const filePath = file.response.data.tmp_path
      const fileIndex = this.addForm.pics.find(x => x === filePath)
      this.addForm.pics.splice(fileIndex, 1)
    },
    uploadSuccess(response) {
      const pic = { pic: response.data.tmp_path }
      this.addForm.pics.push(pic)
    },
    addGood() {
      this.$refs.addFormRef.validate(valid => {
        if (!valid) {
          return this.$message.error('请填写必要的表单项')
        }
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        // 处理动态参数
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        // 发起请求添加商品
        this.$http.post('/goods', form).then(result => {
          let res = result.data
          if (res.meta.status === 201) {
            this.$message.success(res.meta.msg)
            this.$router.push('/goods')
          } else {
            this.$message.error(res.meta.msg)
          }
        })
      })
    }
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 5px !important;
}

.previewImg {
  width: 100%;
}

.addBtn {
  margin-top: 15px;
}
</style>
