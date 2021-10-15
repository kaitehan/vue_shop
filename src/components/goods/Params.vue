<template>
  <div>
      <!-- 面包屑导航区 -->
   <el-breadcrumb separator="/">
     <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
     <el-breadcrumb-item>商品管理</el-breadcrumb-item>
     <el-breadcrumb-item>分类参数</el-breadcrumb-item>
   </el-breadcrumb>
   <!-- 卡片视图区 -->
   <el-card>
     <!-- 头部警告区域 -->
       <el-alert title="注意：只允许为第三级分类设置相关参数"  type="warning" :closable='false' show-icon></el-alert>
       <!-- 选择商品分类区域 -->
       <el-row  class="cat_opt">
         <el-col >
           <span>选择商品分类：</span>
           <!-- 选择商品的级联选择框 -->
           <el-cascader
             :options="cateList"
             v-model="selectedKeys"
             :props="cateProps"
             @change="selectedCateChange"
             clearable>
           </el-cascader>
         </el-col>
       </el-row>
       <!-- tab页签区域 -->
       <el-tabs v-model="activeName" @tab-click="handleTabClick">
         <!-- 添加动态参数面板 -->
          <el-tab-pane label="动态参数" name="many" :disabled='isBtnDisabled'>
            <el-button type="primary" @click="addDialogVisible=true">添加参数</el-button>
            <!-- 动态参数表格 -->
            <el-table :data="manyTableData" style="width: 100%" border>
              <!-- 展开行 -->
              <el-table-column type="expand" >
                <template slot-scope="scope">
                  <el-tag closable  v-for="(item,index) in scope.row.attr_vals" :key="index" @close="handleTagClose(index,scope.row)">{{item}}</el-tag>
                  <!-- 输入文本框 -->
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
                  <!-- 输入按钮 -->
                  <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                </template>
              </el-table-column>
              <!-- 索引列 -->
              <el-table-column type="index" label="#"></el-table-column>
              <el-table-column prop="attr_name" label="参数名称"></el-table-column>
              <el-table-column  label="操作">
                <template slot-scope="scope">
                  <el-button type="primary" size="mini" @click="showEditDialog(scope.row.attr_id)">修改</el-button>
                  <el-button type="danger" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
          <!-- 添加静态属性面板 -->
          <el-tab-pane label="静态属性" name="only" :disabled='isBtnDisabled'>
            <el-button type="primary"  @click="addDialogVisible=true">添加属性</el-button>
            <!-- 静态属性表格 -->
            <el-table :data="onlyTableData" style="width: 100%" border>
              <!-- 展开行 -->
              <el-table-column type="expand" >
                <template slot-scope="scope">
                  <el-tag closable  v-for="(item,index) in scope.row.attr_vals" :key="index" @close="handleTagClose(index,scope.row)">{{item}}</el-tag>
                  <!-- 输入文本框 -->
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
                  <!-- 输入按钮 -->
                  <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                </template>
              </el-table-column>
              <!-- 索引列 -->
              <el-table-column type="index" label="#"></el-table-column>
              <el-table-column prop="attr_name" label="参数名称"></el-table-column>
              <el-table-column  label="操作">
                <template slot-scope="scope">
                  <el-button type="primary" size="mini"  @click="showEditDialog(scope.row.attr_id)">修改</el-button>
                  <el-button type="danger" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>

       </el-tabs>

   </el-card>

   <!-- 添加参数对话框 -->

   <el-dialog
     :title="'添加'+titleText"
     :visible.sync="addDialogVisible"
     width="50%"
     @close="addDialogClosed"
      >
      <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>

      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
   </el-dialog>
   <!-- 修改参数对话框 -->

   <el-dialog
     :title="'修改'+titleText"
     :visible.sync="editDialogVisible"
     width="50%"
     @close="editDialogClosed"
      >
      <el-form ref="editFormRef" :model="editForm" :rules="editFormRules" label-width="100px">
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
  data () {
    return {
      // 保存二级分类列表数据
      cateList: [],
      // 已选中的分类id
      selectedKeys: [],
      // 级联选择器的配置对象
      cateProps: {
        // 选中值
        value: 'cat_id',
        // 展示标签
        label: 'cat_name',
        // 子节点
        children: 'children',
        // 触发展开的方式
        expandTrigger: 'hover'

      },
      // 被激活的标签页名称
      activeName: 'many',
      // 动态参数列表
      manyTableData: [],
      // 静态属性列表
      onlyTableData: [],
      // 控制对话框的显隐
      addDialogVisible: false,
      editDialogVisible: false,
      // 添加参数的表单数据对象
      addForm: {
        attr_name: ''
      },
      editForm: {
      },
      // 添加表单的验证规则对象
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    // 获取二级分类权限
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.cateList = res.data
    },
    // 监听级联选择框选中项改变事件
    selectedCateChange () {
      this.getParamsData()
    },
    // 获取参数列表数据
    async getParamsData () {
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // 根据所选id，和当前所处的面板，获取对应的参数
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数失败')
      }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(',') : []
        // 控制文本框的显示与隐藏
        item.inputVisible = false
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // tab页签被点击的事件处理函数
    handleTabClick () {
      this.getParamsData()
    },
    // 对话框关闭后清空表单
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮，添加参数
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败')
        }
        this.getParamsData()
        this.$message.success('添加参数成功')
        this.addDialogVisible = false
      })
    },
    // 打开编辑对话框
    async showEditDialog (id) {
      // this.editForm = cate
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${id}`, {
        params: { attr_sel: this.activeName }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数失败')
      }
      this.editForm = res.data

      this.editDialogVisible = true
    },
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 点击按钮修改参数
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败')
        }
        this.getParamsData()
        this.$message.success('修改参数成功')
        this.editDialogVisible = false
      })
    },
    // 点击按钮删除参数
    async removeParams (id) {
      const confRes = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confRes !== 'confirm') {
        return this.$message.info('用户取消了此删除操作')
      }
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败')
      }
      this.$message.success('删除参数成功')
      this.getParamsData()
    },
    // 文本框失去焦点或者点击事件都触发此事件
    handleInputConfirm (role) {
      if (role.inputValue.trim().length === 0) {
        role.inputValue = ''
        role.inputVisible = false
        return
      }
      // 如果没有return，则证明输入的内容，需要后续处理
      role.attr_vals.push(role.inputValue.trim())
      role.inputValue = ''
      this.saveAttrVals(role)
    },
    // 修改参数中便签中数据
    async saveAttrVals (role) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${role.attr_id}`, {
        attr_name: role.attr_name,
        attr_sel: this.activeName,
        attr_vals: role.attr_vals.join(',')
      })
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数属性失败')
      }
      this.$message.success('修改参数属性成功')
      role.inputVisible = false
    },
    // 显示文本框
    showInput (role) {
      role.inputVisible = true
      // $nextTick 方法的作用，就是当页面上的元素被重新渲染后，才会执行回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 监听tag 的关闭事件
    handleTagClose (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    isBtnDisabled () {
      if (this.selectedKeys.length !== 3) {
        return true
      } else {
        return false
      }
    },
    cateId () {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[2]
      }
      return null
    },
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }

}
</script>

<style lang="less" scoped>
.cat_opt{
  margin: 15px 0;
}
.el-tag{
  margin: 10px;
}
.input-new-tag{
  width: 150px;
}
</style>
