<template name="component-name">
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
          <el-row>
              <el-col>
                <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
              </el-col>
          </el-row>
          <!-- 表格 -->
          <tree-table class="treeTable" :data="catelist"  :columns="columns" :selection-type="false" :expand-type="false" show-index
        index-text="#" border :show-row-hover="false">
        <!-- 是否有效 -->
            <template slot="isok" slot-scope="scope">
                <i class="el-icon-success" v-if="scope.row.cat_deleted ===false" style="color:lightgreen"></i>
                <i class="el-icon-error" v-else  style="color:red"></i>
            </template>
        <!-- 排序 -->
            <template slot="order" slot-scope="scope">
                <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                <el-tag size="mini" v-else-if="scope.row.cat_level === 1" type="primary">二级</el-tag>
                <el-tag size="mini" v-else type="warning">三级</el-tag>
            </template>
         <!-- 操作 -->
            <template slot="opt">
                <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            </template>

          </tree-table>

          <!-- 分页 -->
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="querInfo.pagenum"
            :page-sizes="[3, 5, 10, 15]"
            :page-size="querInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
          </el-pagination>
        </el-card>

        <!-- 添加分类对话框 -->
        <el-dialog
            title="添加分类"
            :visible.sync="addCatDialogVisible"
            width="50%"
            @close="addCateDialogClose">
            <!-- 添加分类表单 -->
            <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
                <el-form-item label="分类名称：" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类：">
                    <el-cascader
                    v-model="selectedKeys"
                    :options="parentCateList"
                    expand-trigger="hover"
                    :props="cascderProps"
                    @change="parentCateChanged" clearable change-on-select>
                    </el-cascader>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addCatDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addCate">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    return {
      // 商品分类列表
      catelist: [],
      // 查询条件参数
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 所有数据长度
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列的模板名称
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          // 表示当前这一列的模板名称
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          // 表示当前这一列的模板名称
          template: 'opt'
        }
      ],
      // 控制对话框的显示与隐藏
      addCatDialogVisible: false,
      //   添加分类的数据对象
      addCateForm: {
        //   将要添加的分类名称
        cat_name: '',
        // 父级分类的id
        cata_pid: 0,
        // 分类的等级 默认的是一级分类
        cat_level: 0
      },
      //  添加表单的验证规则对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类列表数据
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类的id数组
      selectedKeys: []
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.querInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败! ')
      }
      // 获取到的数据列表赋值给catelist
      this.catelist = res.data.result
      // 为总数据条数赋值
      this.total = res.data.total
    },
    // 监听pagesize改变
    handleSizeChange (newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum的改变
    handleCurrentChange (newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮 展示添加分类的对话框
    showAddCateDialog () {
      this.getParentCateList()
      this.addCatDialogVisible = true
    },
    // 获取父级分类的数据
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级数据失败! ')
      }
      this.parentCateList = res.data
    },
    parentCateChanged () {
      // 选择项触发
      // 如果selectKeys数据中的length大于0 证明选中了父级分类
      // 反之 就说明没有选中
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
        return ''
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    addCate () {
      this.addCatDialogVisible = false
    },
    // 监听dialog关闭 并清空dialog里的数据
    addCateDialogClose () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    }
  }
}
</script>

<style scoped>
.treeTable {
    margin-top: 15px;
}

.el-cascader {
    width: 100%
}
</style>
