<template>
  <section>
    <!--工具条-->
    <el-row>
      <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
        <el-form size="small" :inline="true" :model="filter" @submit.native.prevent>
          <el-form-item>
            <el-input
              v-model="filter.createdUserName"
              placeholder="操作账号"
              clearable
              @keyup.enter.native="getList"
            >
              <i slot="prefix" class="el-input__icon el-icon-search" />
            </el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="getList">查询</el-button>
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>

    <!--列表-->
    <el-table
      v-loading="listLoading"
      :data="list"
      highlight-current-row
      style="width: 100%;height:100%;"
    >
      <el-table-column prop="id" label="编号" width="80" />
      <el-table-column prop="createdUserName" label="操作账号" width="100">
        <template v-slot="scope">
          {{ scope.row.createdUserName }}<br>{{ scope.row.realName }}
        </template>
      </el-table-column>
      <el-table-column prop="ip" label="IP地址" width="130" />
      <el-table-column prop="apiLabel" label="操作名称" width="" />
      <el-table-column prop="apiPath" label="操作接口" width="" />
      <!-- <el-table-column prop="browser" label="浏览器" width="100" />
      <el-table-column prop="os" label="操作系统" width="100" /> -->
      <el-table-column prop="elapsedMilliseconds" width="70">
        <template slot="header">
          耗时<br>毫秒
        </template>
      </el-table-column>
      <el-table-column prop="status" label="操作状态" width="80">
        <template v-slot="scope">
          <el-tag
            :type="scope.row.status ? 'success' : 'danger'"
            disable-transitions
          >{{ scope.row.status ? '成功' : '失败' }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="msg" label="操作消息" />
      <el-table-column prop="createdTime" label="操作时间" :formatter="formatCreatedTime" width />
    </el-table>

    <!--工具条 prev, pager, next, jumper -->
    <el-row>
      <el-col :span="24" class="pagination">
        <el-pagination
          layout="total, slot, sizes, prev, jumper, next"
          :current-page.sync="currentPage"
          :page-size.sync="pageSize"
          :total="total"
          :page-count="pageCount"
          background
          style="text-align:right;"
        >
          <span class="el-pagination__count">，{{ pageCount }} 页</span>
        </el-pagination>
      </el-col>
    </el-row>
  </section>
</template>

<script>
import { formatTime } from '@/utils'
import { getOprationLogPage } from '@/api/admin/opration-log'
export default {
  name: 'OprationLog',
  data() {
    return {
      filter: {
        name: ''
      },
      list: [],
      total: 0,
      pageSize: 10,
      currentPage: 1,
      listLoading: false
    }
  },
  computed: {
    pageCount() {
      return this.total > 0 && this.pageSize > 0
        ? Math.ceil(this.total / this.pageSize)
        : 1
    }
  },
  watch: {
    currentPage() {
      this.getList()
    },
    pageSize() {
      this.getList()
    }
  },
  mounted() {
    this.getList()
  },
  methods: {
    formatCreatedTime: function(row, column, time) {
      return formatTime(time, 'yyyy-MM-dd hh:mm')
    },
    // 获取列表
    async getList() {
      const para = {
        currentPage: this.currentPage,
        pageSize: this.pageSize,
        filter: this.filter
      }
      this.listLoading = true
      const res = await getOprationLogPage(para)
      this.listLoading = false

      if (!res.success) {
        if (res.msg) {
          this.$message({
            message: res.msg,
            type: 'error'
          })
        }
        return
      }

      this.total = res.data.total
      this.list = res.data.list
    }
  }
}
</script>
