<template name="component-name">
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品列表</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 搜索框 按钮 -->
        <el-card>
          <el-row :gutter="20">
            <el-col :span="8">
              <el-input clearable placeholder="请输入内容" v-model="queryInfo.query" @clear="getGoodsList"><el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button></el-input>
            </el-col>
            <el-col :span="4"><el-button type="primary" @click="goAddpage">添加商品</el-button></el-col>
          </el-row>

          <el-table :data="goodslist" border stripe>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="商品名称" prop="goods_name"></el-table-column>
            <el-table-column label="商品价格(元)" prop="goods_price" width="90"></el-table-column>
            <el-table-column label="商品重量" prop="goods_weight" width="70"></el-table-column>
            <el-table-column label="创建时间" prop="add_time" width="140">
              <template slot-scope="scope">
                 {{scope.row.add_time | dateformat}}
              </template>
            </el-table-column>
            <el-table-column label="操作" width="130">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(scope.row.goods_id)"></el-button>
              </template>
            </el-table-column>
          </el-table>

           <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[5, 10, 15, 20]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total" background>
          </el-pagination>
        </el-card>

    </div>
</template>

<script>
export default {
  data () {
    return {
      searchValue: '',
      // 查询数据参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 商品列表
      goodslist: [],
      // 数据条数
      total: 0
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error(' 获取商品数据失败! ')
      }
      this.goodslist = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    async removeById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.error('已取消删除')
      }
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error(' 删除商品数据失败! ')
      }
      this.getGoodsList()
    },
    goAddpage () {
      this.$router.push('/goods/add')
    }
  }
}
</script>
<style scoped>

</style>
