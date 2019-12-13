<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区 -->
    <el-card>
      <!-- 添加分类按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格区 -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        class="treeTable"
      >
        <!-- 是否有效 -->
        <template slot="isOk" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" style="color: red" v-else></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level=== 0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level=== 1">二级</el-tag>
          <el-tag type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级名称:">
          <!-- options用来指定数据源 -->
          <!-- props用来指定配置对象 -->
          <el-cascader
            expandTrigger="hover"
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { async } from 'q'
export default {
  data() {
    return {
      // getCateList参数设置
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表 默认为空
      cateList: [],
      //   总数据条数 默认为0
      total: 0,
      //   为treeTable指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 将当前列定义为自定义模板列
          type: 'template',
          //   表示当前这一列的使用模板名称
          template: 'isOk'
        },
        {
          label: '排序',
          // 将当前列定义为自定义模板列
          type: 'template',
          //   表示当前这一列的使用模板名称
          template: 'order'
        },
        {
          label: '操作',
          // 将当前列定义为自定义模板列
          type: 'template',
          //   表示当前这一列的使用模板名称
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCateForm: {
        // 将要添加分类的名称
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        // 当前分类等级，默认添加一级分类
        cat_level: 0
      },
      // 添加分类表单的验证规则对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类的列表
      parentCateList: [],
      // 级联选择器配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类的id数组
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //   获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败')
      }
      //   把数据列表赋值给cateList
      this.cateList = res.data.result
      //   为总数据条数赋值
      this.total = res.data.total
    },
    // 监听分页区Pagesize改变的事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pageNum的改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮展示添加分类的对话框
    showAddCateDialog() {
      // 先获取父级分类的数据列表再展示对话框
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })

      if (res.meta.status !== 200) {
        return this.$message.error('获取父级数据列表失败')
      }
      this.parentCateList = res.data
    },
    // cascader选择项发生变化触发这个函数
    parentCateChanged() {
      console.log(this.selectedKeys)
      // 如果selectedKeys数组中的length大于0证明选中了父级分类 反之没有选中任何父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类id绑定
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        // 为当前分类等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 点击确定按钮添加新的分类
    addCate() {
      // 表单校验
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听对话框的关闭事件 重置表单数据
    addCateDialogClosed() {
      // 清空表单
      this.$refs.addCateFormRef.resetFields()
      // 清空级联选择器
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    }
  }
}
</script>

<style lang="less" scoped>
.el-breadcrumb {
  margin-bottom: 20px;
  font-size: 12px;
}

.el-card {
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.15) !important;
}

.treeTable {
  margin-top: 15px;
}

.el-cascader {
  width: 100%;
}
</style>