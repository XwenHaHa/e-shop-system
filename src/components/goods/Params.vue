<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert
        title="注意: 只允许为第三级分类设置相关参数"
        type="warning"
        :closable="false"
        show-icon
      >
      </el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类: </span>
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader
            expand-trigger="hover"
            v-model="selectedKeys"
            :options="catelist"
            :props="CateProps"
            @change="handleChange"
          ></el-cascader>
        </el-col>
      </el-row>
      <!-- Tabs标签页 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数的面板 -->
        <el-tab-pane label="动态参数" name="many">
          <!-- 添加参数的按钮 -->
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="AddDialogVisible = true"
            >添加参数</el-button
          >
          <!-- 动态参数表格 -->
          <el-table :data="manyData" border stripe>
            <!-- 展开列 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环Tag标签 -->
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClosed(i, scope.row)"
                >
                  {{ item }}
                </el-tag>
                <!-- 输入的文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <!-- 添加按钮 -->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column
              label="参数列表"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性的面板 -->
        <el-tab-pane label="静态属性" name="only">
          <!-- 添加属性的按钮 -->
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="AddDialogVisible = true"
            >添加属性</el-button
          >
          <!-- 静态属性表格 -->
          <el-table :data="onlyData" border stripe>
            <!-- 展开列 -->
            <!-- 展开列 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环Tag标签 -->
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClosed(i, scope.row)"
                >
                  {{ item }}
                </el-tag>
                <!-- 输入的文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <!-- 添加按钮 -->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column
              label="属性列表"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="AddDialogVisible"
      width="30%"
      @close="AddDialogClosed"
    >
      <!-- 添加参数的表单 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="AddDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="30%"
      @close="editDialogClosed"
    >
      <!-- 修改参数的表单 -->
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 商品分类列表数据
      catelist: [],
      // 级联选择框的配置对象
      CateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 级联选择框双向绑定到的数组
      selectedKeys: [],
      // 被激活的页签的名称
      activeName: 'many',
      // 动态参数的数据
      manyData: [],
      // 静态属性的数据
      onlyData: [],
      // 控制添加对话框的显示和隐藏
      AddDialogVisible: false,
      // 添加参数的表单数据对象
      addForm: {
        attr_name: ''
      },
      // 添加参数的表单验证规则
      addFormRules: {
        attr_name: [
          {
            required: true,
            message: '请输入参数或属性名称',
            trigger: 'blur'
          }
        ]
      },
      // 控制修改用户表单的显示与隐藏
      editDialogVisible: false,
      // 修改参数的表单数据对象
      editForm: {},
      // 修改参数的表单验证规则
      editFormRules: {
        attr_name: [
          {
            required: true,
            message: '请输入参数或属性名称',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品分类列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类列表失败')
      }
      this.catelist = res.data
      console.log(this.catelist)
    },
    // 级联选择框选中项变化会触发这个函数
    handleChange() {
      this.getParamsData()
    },
    // Tabs标签页的点击处理事件
    handleTabClick() {
      this.getParamsData()
      console.log(this.activeName)
    },
    // 获取参数的列表数据
    async getParamsData() {
      // console.log(this.selectedKeys)
      // 证明选中的不是三级分类
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyData = []
        this.onlyData = []
        return null
      }
      // 证明选中的是三级分类
      console.log(this.selectedKeys)
      // 根据所选分类的Id和当前所处的面板，获取对应的参数
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        this.$message.error('获取参数列表失败')
      }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制文本的显示与隐藏
        item.inputVisible = false
        // 文本框中输入的值
        item.inputValue = ''
      })
      console.log(res.data)
      if (this.activeName === 'many') {
        this.manyData = res.data
      } else {
        this.onlyData = res.data
      }
    },
    // 监听关闭添加对话框的事件
    AddDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮，提交数据
    addParams() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return null
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 201) {
          this.$message.error('添加失败')
        }
        this.$message.success('添加成功')
        this.AddDialogVisible = false
        this.getParamsData()
      })
    },
    // 点击按钮，展示修改的对话框
    async showEditDialog(attrid) {
      // 查询当前参数信息
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attrid}`,
        {
          params: { attr_sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        this.$message.error('获取参数列表失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 监听关闭修改对话框的事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 点击按钮修改数据
    editParams() {
      // 先进行表单预校验
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return null
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改失败!')
        }
        this.$message.success('修改成功')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 根据id删除对应项
    // err=>err捕获错误并且将错误return
    async removeParams(attrid) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该参数, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 用户取消了操作
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // 删除的业务逻辑
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attrid}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败')
      }
      this.$message.success('删除参数成功')
      this.getParamsData()
    },
    // 文本框失去焦点，或者摁下了Enter键都会触发
    async handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return null
      }
      // 如果没有return证明输入的内容需要做后续的处理
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // 需要发起请求，保存这次操作
      this.saveAttrVals(row)
    },
    // 将对attr_vals的操作，保存到数据库
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      if (res.meta.status !== 200) {
        this.$message.error('修改参数失败')
      }
      this.$message.success('修改参数成功')
    },
    // 点击按钮，展示文本输入框
    showInput(row) {
      row.inputVisible = true
      // 让输入框自动获取焦点
      // $nextTick方法的作用，就是页面上元素被重新渲染后，才会执行回调函数的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 删除对应的参数可选项
    handleClosed(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    // 如果按钮需要被禁用，则返回true，否则返回false
    isBtnDisabled() {
      if (this.selectedKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分类的Id
    cateId() {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[2]
      }
      return null
    },
    // 判断title是动态参数还是静态属性
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
  margin-top: 15px;
}
.el-tag {
  margin: 5px;
}
.input-new-tag {
  width: 120px;
}
</style>
