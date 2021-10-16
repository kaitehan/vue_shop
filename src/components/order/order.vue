<template>
  <div>
       <!-- 面包屑导航区 -->
   <el-breadcrumb separator="/">
     <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
     <el-breadcrumb-item>订单管理</el-breadcrumb-item>
     <el-breadcrumb-item>订单列表</el-breadcrumb-item>
   </el-breadcrumb>
   <!-- 卡片视图区 -->
   <el-card>
     <el-row :gutter="10">
       <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" class="input-with-select">
             <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
       </el-col>
     </el-row>
     <!-- 订单列表数据 -->
     <el-table
       :data="orderList"
       style="width: 100%" border stripe>
       <el-table-column type="index" > </el-table-column>
       <el-table-column prop="order_number" label="订单编号"> </el-table-column>
       <el-table-column prop="order_price" label="订单价格" width="95px"> </el-table-column>
       <el-table-column prop="pay_status" label="是否付款" width="95px">
         <template slot-scope="scope">
           <el-tag type="success" v-if="scope.row.pay_status==='1'">已付款</el-tag>
           <el-tag type="danger" v-else>未付款</el-tag>
         </template>
       </el-table-column>
       <el-table-column prop="is_send" label="是否发货" width="95px"> </el-table-column>
       <el-table-column prop="create_time" label="下单时间" width="200px">
         <template slot-scope="scope">
           {{scope.row.create_time | dateFormat}}
         </template>
       </el-table-column>
       <el-table-column label="操作" width="180px">
         <template >
           <el-button type="primary" size="mini" @click="showBox">修改</el-button>
           <el-button type="success" size="mini" @click="showProgressBox">物流</el-button>
         </template>
       </el-table-column>
     </el-table>

     <!-- 分页区域 -->
     <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
   </el-card>

   <!-- 修改地址对话框 -->
   <el-dialog
     title="修改地址"
     :visible.sync="addressDialogVisible"
     width="50%"
     @close="addressDialogClosed"
     >
     <el-form ref="editFormRef" :rules="addressFormRules" :model="addressForm" label-width="100px">
       <el-form-item label="省市区/县" prop="address1">
         <el-cascader
           :options="cityData"
           v-model="addressForm.address1"
          >
         </el-cascader>
       </el-form-item>
       <el-form-item label="详细地址" prop="address2">
         <el-input v-model="addressForm.address2"></el-input>
       </el-form-item>

     </el-form>
     <div slot="footer">
       <el-button @click="addressDialogVisible = false">取 消</el-button>
       <el-button type="primary" @click="addressDialogVisible = false">确 定</el-button>
     </div>
   </el-dialog>

   <!-- 物流进度的对话框 -->
   <el-dialog
     title="物流进度"
     :visible.sync="progressDialogVisible"
     width="50%">
     <el-timeline>
       <el-timeline-item :timestamp="item.time" v-for="(item,index) in progressInfo" :key="index" placement="top" >
          <el-card>
            <h4>{{item.context}}</h4>
            <p>更新于{{item.ftime}}</p>
          </el-card>
        </el-timeline-item>
      </el-timeline>
   </el-dialog>
  </div>
</template>
<script>
import cityData from './citydata.js'
export default {
  data () {
    return {
      // 查询对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      orderList: [],
      total: 0,
      addressDialogVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: '请选择省市区/县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请填写详细地址', trigger: 'blur' }
        ]
      },
      cityData,
      progressDialogVisible: false,
      progressInfo:
        [
          {
            time: '2018-05-10 09:39:00',
            ftime: '2018-05-10 09:39:00',
            context: '已签收,感谢使用顺丰,期待再次为您服务',
            location: ''
          },
          {
            time: '2018-05-10 08:23:00',
            ftime: '2018-05-10 08:23:00',
            context: '[北京市]北京海淀育新小区营业点派件员 顺丰速运 95338正在为您派件',
            location: ''
          },
          {
            time: '2018-05-10 07:32:00',
            ftime: '2018-05-10 07:32:00',
            context: '快件到达 [北京海淀育新小区营业点]',
            location: ''
          },
          {
            time: '2018-05-10 02:03:00',
            ftime: '2018-05-10 02:03:00',
            context: '快件在[北京顺义集散中心]已装车,准备发往 [北京海淀育新小区营业点]',
            location: ''
          },
          {
            time: '2018-05-09 23:05:00',
            ftime: '2018-05-09 23:05:00',
            context: '快件到达 [北京顺义集散中心]',
            location: ''
          },
          {
            time: '2018-05-09 21:21:00',
            ftime: '2018-05-09 21:21:00',
            context: '快件在[北京宝胜营业点]已装车,准备发往 [北京顺义集散中心]',
            location: ''
          },
          {
            time: '2018-05-09 13:07:00',
            ftime: '2018-05-09 13:07:00',
            context: '顺丰速运 已收取快件',
            location: ''
          },
          {
            time: '2018-05-09 12:25:03',
            ftime: '2018-05-09 12:25:03',
            context: '卖家发货',
            location: ''
          },
          {
            time: '2018-05-09 12:22:24',
            ftime: '2018-05-09 12:22:24',
            context: '您的订单将由HLA（北京海淀区清河中街店）门店安排发货。',
            location: ''
          },
          {
            time: '2018-05-08 21:36:04',
            ftime: '2018-05-08 21:36:04',
            context: '商品已经下单',
            location: ''
          }]

    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败')
      }
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    // 监听pagesize改变事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
    },
    // 监听pagenum改变事件
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
    },
    // 打开修改对话框
    showBox () {
      this.addressDialogVisible = true
    },
    addressDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    async showProgressBox () {
      // 单号数据有问题，直接用死数据
      // const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      // if (res.meta.status !== 200) {
      //   return this.$message.error('获取物流进度失败')
      // }
      // this.progressInfo = res.data
      this.progressDialogVisible = true
      console.log(this.progressInfo)
    }
  }
}
</script>

<style lang="less" scoped>
.el-cascader{
  width: 100%;
}
</style>
