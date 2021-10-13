<template>
  <div>

     <!-- 面包屑导航区 -->
   <el-breadcrumb separator="/">
     <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
     <el-breadcrumb-item>权限管理</el-breadcrumb-item>
     <el-breadcrumb-item>角色列表</el-breadcrumb-item>
   </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>

      <!-- 添加角色按钮区域 -->
      <el-row :gutter="10">
        <el-col :span="6">
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色区域 -->
      <el-table :data="roleList" style="width: 100%" border stripe>

        <!-- 展开列 -->
        <el-table-column type="expand" >
          <template slot-scope="scope">
            <el-row :class="['bdbottom',i1===0? 'bdtop':'','expandRow','vcenter']"  v-for="(item1,i1) in scope.row.children" :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag type="primary" closable @close='removeRightById(scope.row,item1.id)'>{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级权限和三级权限 -->
              <el-col :span="19">
                <!-- for循环嵌套渲染二级权限 -->
                <el-row :class="[i2===0? '':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success" closable @close='removeRightById(scope.row,item2.id)'>{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag type="warning" closable v-for="item in item2.children" :key="item.id" @close='removeRightById(scope.row,item.id)'>{{item.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>

            </el-row>
          </template>
        </el-table-column>

        <!-- 索引列 -->
        <el-table-column type="index">  </el-table-column>
        <el-table-column label="角色名称" prop="roleName">  </el-table-column>
        <el-table-column label="角色描述" prop="roleDesc">  </el-table-column>

        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" >删除</el-button>
            <el-button type="warning" icon="el-icon-share" size="mini"  @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 修改权限的对话框 -->
    <el-dialog  title="修改权限"  :visible.sync="SetRightdialogVisible" width="50%"  @close="setRightDialogClose">

      <!-- 主体内容 -->
      <!-- 树形控件 -->
        <el-tree ref="treeRef" :data="rightList" :props="defaultProps"  show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys"></el-tree>

        <!-- 底部按钮区 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="SetRightdialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色的列表
      roleList: [],
      // 控制对话框显示与隐藏
      SetRightdialogVisible: false,
      // 所有权限列表
      rightList: [],
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中的节点数组
      defKeys: [],
      // 即将分配权限的角色的id
      roleId: ''
    }
  },
  created () {
    this.getRoleList()
  },
  methods: {
    // 获取角色列表
    async getRoleList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }

      this.roleList = res.data
    },
    async removeRightById (role, id) {
      // 弹框提示用户是否删除
      const confRes = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => {
        return err
      })

      if (confRes !== 'confirm') {
        return this.$message.info('用户取消删除')
      }

      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('用户删除权限失败')
      }
      role.children = res.data
      // this.getRoleList()
      this.$message.success('用户删除权限成功')
    },
    // 展示修改权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限失败')
      }
      this.rightList = res.data
      // console.log(this.rightList)
      // 递归获取三级权限
      this.getLeafKeys(role, this.defKeys)
      this.SetRightdialogVisible = true
    },

    // 通过递归的形式，获取角色下所有三级权限的id，并保存到defKeys数组中
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        return this.getLeafKeys(item, arr)
      })
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClose () {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]

      const idStr = keys.join(',')

      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('更新角色权限失败')
      }
      this.$message.success('更新角色权限成功')
      this.getRoleList()
      this.SetRightdialogVisible = false
    }
  }

}
</script>

<style lang="less" scoped>
.el-tag{
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.expandRow{
  margin: 0 50px;
}
.vcenter{
  display: flex;
  align-items: center;
}

</style>
