<template>
  <div class="app-container calendar-list-container">

    <!-- 查询和其他操作 -->
    <div class="filter-container">
      <el-input clearable class="filter-item" style="width: 200px;" placeholder="请输入用户ID" v-model="listQuery.userId">
      </el-input>
      <el-input clearable class="filter-item" style="width: 200px;" placeholder="请输入商品ID" v-model="listQuery.valueId">
      </el-input>
      <el-button class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">查找</el-button>
      <el-button class="filter-item" type="primary" icon="el-icon-download" @click="handleDownload" :loading="downloadLoading">导出</el-button>
    </div>

    <!-- 查询结果 -->
    <el-table size="small" :data="list" v-loading="listLoading" element-loading-text="正在查询中。。。" border fit highlight-current-row>

      <el-table-column align="center" label="用户ID" prop="userId">
      </el-table-column>

      <el-table-column align="center" label="商品ID" prop="valueId">
      </el-table-column>

      <el-table-column align="center" label="打分" prop="star">
      </el-table-column>

      <el-table-column align="center" label="评论内容" prop="content">
      </el-table-column>

      <el-table-column align="center" label="评论图片" prop="picUrls">
        <template slot-scope="scope">
          <img v-for="item in scope.row.picUrls" :key="item" :src="item" width="40"/>
        </template>
      </el-table-column>

      <el-table-column align="center" label="时间" prop="addTime">
      </el-table-column>

      <el-table-column align="center" label="操作" width="200" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="handleReply(scope.row)">回复</el-button>
          <el-button type="danger" size="mini"  @click="handleDelete(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <div class="pagination-container">
      <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="listQuery.page"
        :page-sizes="[10,20,30,50]" :page-size="listQuery.limit" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>

    <!-- 评论回复 -->
    <el-dialog title="回复" :visible.sync="replyFormVisible">
      <el-form ref="replyForm" :model="replyForm" status-icon label-position="left" label-width="100px" style='width: 400px; margin-left:50px;'>
        <el-form-item label="回复内容" prop="content">
          <el-input type="textarea" :autosize="{ minRows: 4, maxRows: 8}" v-model="replyForm.content"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="replyFormVisible = false">取消</el-button>
        <el-button type="primary" @click="reply">确定</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
import { listComment, deleteComment } from '@/api/comment'
import { replyComment } from '@/api/order'

export default {
  name: 'Comment',
  data() {
    return {
      list: undefined,
      total: undefined,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
        userId: undefined,
        valueId: undefined,
        sort: 'add_time',
        order: 'desc'
      },
      downloadLoading: false,
      replyForm: {
        commentId: 0,
        content: ''
      },
      replyFormVisible: false
    }
  },
  created() {
    this.getList()
  },
  methods: {
    getList() {
      this.listLoading = true
      listComment(this.listQuery).then(response => {
        this.list = response.data.data.items
        this.total = response.data.data.total
        this.listLoading = false
      }).catch(() => {
        this.list = []
        this.total = 0
        this.listLoading = false
      })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getList()
    },
    handleSizeChange(val) {
      this.listQuery.limit = val
      this.getList()
    },
    handleCurrentChange(val) {
      this.listQuery.page = val
      this.getList()
    },
    handleReply(row) {
      this.replyForm = { commentId: row.id, content: '' }
      this.replyFormVisible = true
    },
    reply() {
      replyComment(this.replyForm).then(response => {
        this.replyFormVisible = false
        this.$notify.success({
          title: '成功',
          message: '回复成功'
        })
      }).catch(response => {
        this.$notify.error({
          title: '失败',
          message: response.data.errmsg
        })
      })
    },
    handleDelete(row) {
      deleteComment(row).then(response => {
        this.$notify({
          title: '成功',
          message: '删除成功',
          type: 'success',
          duration: 2000
        })
        const index = this.list.indexOf(row)
        this.list.splice(index, 1)
      })
    },
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['评论ID', '用户ID', '商品ID', '评论', '评论图片列表', '评论时间']
        const filterVal = ['id', 'userId', 'valueId', 'content', 'picUrls', 'addTime']
        excel.export_json_to_excel2(tHeader, this.list, filterVal, '商品评论信息')
        this.downloadLoading = false
      })
    }
  }
}
</script>
