<template>
  <!-- 加载 -->
  <div v-loading="loading">

    <!-- 搜索栏 模糊查询-->
    <el-form :inline="true" :model="page" class="demo-form-inline" size="mini">
      <el-form-item label="评论内容">
        <el-input v-model="page.params.commentContent" placeholder="评论内容" clearable />
      </el-form-item>
      <el-form-item label="评论类型">
        <el-select v-model="page.params.commentType" placeholder="请求状态" clearable filterable>
          <el-option label="文章评论" :value="0" />
          <el-option label="评论评论" :value="1" />
        </el-select>
      </el-form-item>
      <el-form-item label="父级id">
        <el-input v-model="page.params.pid" placeholder="父级id" clearable />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" sizi="mini" @click="getByPage">查询</el-button>
      </el-form-item>
    </el-form>
    <!-- 分割线 -->
    <el-divider />

    <!-- 列表 -->
    <!--
      1. :data v-bind:model="page.list" 绑定数据 分页对象的的list数据
      2. show-overflow-tooltip 超出部分隐藏
      3. @selection-change="handleSelectionChange" selection-change	当选择项发生变化时会触发该事件
      4. @sort-change="changeSort" sort-change 事件回中可以获取当前排序的字段名[prop]和排序顺序[order]
     -->
    <el-table
      :data="page.list"
      border
      style="width: 100%"
      @sort-change="changeSort"
    >
      <el-table-column type="index" fixed="left" label="#" width="60" align="center" />
      <el-table-column prop="commentContent" align="center" label="评论内容" width="250" show-overflow-tooltip />
      <el-table-column prop="commentUser" label="评论人" width="120" align="center" />
      <el-table-column prop="articleTitle" label="文章标题" width="160" sortable="custom" align="center" />
      <el-table-column prop="commentCount" label="评论数" width="100" sortable="custom" align="center" />
      <el-table-column prop="commentGood" label="点赞数" width="100" sortable="custom" align="center" />
      <el-table-column prop="commentTime" label="评论时间" width="200" sortable="custom" align="center" />
      <el-table-column prop="commentType" label="评论类型" width="120" align="center">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.commentType === 0">文章的评论</el-tag>
          <el-tag v-if="scope.row.commentType === 1" type="success">评论的评论</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="isEnable" label="状态" width="100" align="center">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.isEnabled === 1">启用</el-tag>
          <el-tag v-else type="info">弃用</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" fixed="right" width="120" align="center">
        <template slot-scope="scope">
          <el-dropdown>
            <el-button type="primary" size="mini">
              操作
              <i class="el-icon-arrow-down el-icon--right" />
            </el-button>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item>
                <el-button size="mini" type="primary" @click="toRead(scope.row.commentId)">查看</el-button>
              </el-dropdown-item>
              <el-dropdown-item>
                <el-button v-if="scope.row.isEnabled === 0" size="mini" type="success" disabled @click="toEnable(scope.row.commentId)">启用</el-button>
              </el-dropdown-item>
              <el-dropdown-item>
                <el-button v-if="scope.row.isEnabled === 1" size="mini" type="warning" disabled @click="toDisable(scope.row.commentId)">弃用</el-button>
              </el-dropdown-item>
              <el-dropdown-item>
                <el-button size="mini" type="danger" @click="toDelete(scope.row.commentId)">删除</el-button>
              </el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>
    </el-table>

    <!--
      分页组件-最完整版
      class : 分页组件
      current-page : 当前页 此处为动态绑定page对象的currentPage属性
      page-sizes : 每页显示个数选择器的选项设置
      page-size : 每页大小
      layout : 组件布局
      total : 总条目数 此处动态绑定page对象的totalCount属性
      @size-change="handleSizeChange"  pageSize 改变时会触发  参数:每页条数
      @current-change="handleCurrentChange" currentPage 改变时会触发 参数:当前页
     -->
    <el-pagination
      align="center"
      class="pagination"
      :current-page="page.currentPage"
      :page-sizes="[5,10,20,50]"
      :page-size="page.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="page.totalCount"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
    <!-- 阅读弹窗 -->
    <el-dialog title="评论列表" :visible.sync="readDialog" width="50%">
      <comment-read :comment="comment" @closeReadDialog="closeReadDialog" @getByPage="getByPage" />
    </el-dialog>

  </div>
</template>

<script>
// 导入api接口定义的方法 接收变量为 articleApi
import commentApi from '@/api/article/comment'
// 导入组件
import CommentRead from './comment-read'
export default {
  //  定义添加的组件 子组件/私有组件
  components: {
    CommentRead
  },
  data() {
    return {
      // 定义page对象
      page: {
        currentPage: 1, // 当前页
        pageSize: 10, // 每页显示条数
        totalPage: 0, // 总页数
        totalCount: 0, // 总条数
        params: {}, // 查询参数对象
        list: [], // 数据
        sortColumn: 'commentTime', // 排序列
        sortMethod: 'asc' // 排序方式
      },
      comment: {
        articleTitle: '',
        commentArticle: '',
        commentContent: '',
        commentCount: '',
        commentGood: '',
        commentId: '',
        commentList: [],
        commentTime: '',
        commentType: '',
        commentUser: '',
        pid: ''
      },
      loading: true, // 控制是否显示加载效果
      readDialog: false // 控制阅读弹窗显示
    }
  },
  // 初始化函数
  created() {
    this.getByPage()
  },
  // 定义方法
  methods: {
    // 每页大小改变 参数 value 为每页大小(pageSize)
    handleSizeChange(val) {
      this.page.pageSize = val
      // 重新请求,刷新页面
      this.getByPage()
    },
    // 当前页跳转 参数 value 当前页(currentPage)
    handleCurrentChange(val) {
      this.page.currentPage = val
      this.getByPage()
    },
    // 分页方法 调用封装的方法 getByPage()
    getByPage() {
      commentApi.getByPage(this.page).then(res => {
        this.page = res.data
        this.loading = false
      })
    },
    // 条件排序 e 和 val 都行
    changeSort(e) {
      if (e.order) {
        this.page.sortColumn = e.prop
        this.page.sortMethod = e.order
      } else {
        this.page.sortColumn = ''
        this.page.sortMethod = 'asc'
      }
      this.$message.success('操作成功!')
      this.getByPage()
    },
    // 启用
    toEnable(id) {
      this.$confirm('是否启用？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        commentApi.enable(id).then(res => {
          this.$message.success(res.msg)
          this.getByPage()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消启用'
        })
      })
    },
    // 弃用
    toDisable(id) {
      this.$confirm('是否弃用?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        commentApi.disable(id).then(res => {
          this.$message.success(res.msg)
          this.getByPage()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消弃用'
        })
      })
    },
    // 删除
    toDelete(id) {
      this.$confirm('是否删除?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        commentApi.delete(id).then(res => {
          this.$message.success(res.msg)
          this.getByPage()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    // 阅读
    toRead(id) {
      commentApi.get(id).then(res => {
        this.comment = res.data
        console.log(this.comment)
        // console.log(this.comment.commentType)
        this.readDialog = true
      })
    },
    closeReadDialog() {
    // 关闭阅读弹窗
      this.readDialog = false
    }
  }
}
</script>

