<template>
  <div v-loading="loading">
    <!-- 搜索栏 模糊查询-->
    <el-form :inline="true" :model="page" class="demo-form-inline" size="mini">
      <el-form-item label="用户名">
        <el-input v-model="page.params.userName" placeholder="输入用户名搜索" clearable />
      </el-form-item>
      <el-form-item label="组别">
        <el-select v-model="page.params.userGroupId" placeholder="组别" clearable filterable>
          <el-option
            v-for="item in groupList"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="角色">
        <el-select v-model="page.params.userPositionId" placeholder="角色" clearable filterable>
          <el-option
            v-for="item in positionList"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="起始日期">
        <el-date-picker
          v-model="page.params.reportDate"
          type="daterange"
          align="right"
          unlink-panels
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :default-time="['00:00:00', '23:59:59']"
          :picker-options="pickerOptions"
          format="yyyy 年 MM 月 dd 日"
          value-format="yyyy-MM-dd HH:mm:ss"
        />
      </el-form-item>
      <el-select
        clearable
        size="small"
        placeholder="状态"
        class="filter-item"
        style="width: 80px;margin-top:-2px;"
      >
        <el-option
          v-for="item in enabledTypeOptions"
          :key="item.key"
          :label="item.display_name"
          :value="item.key"
        />
      </el-select>
      <span>
        <el-form-item style="margin-left:9px;margin-top:-2px">
          <el-button type="primary" size="small" @click="query">查询</el-button>
        </el-form-item>
      </span></el-form>
    <!-- 分割线 -->
    <!-- 增删改查 -->
    <div style="margin-bottom:28px">
      <!-- crud组件 -->
      <button type="button" class="el-button filter-item el-button--primary el-button--mini"><!----><i class="el-icon-plus" /><span>
        新增
      </span>
      </button>
      <button disabled="disabled" type="button" class="el-button filter-item el-button--success el-button--mini is-disabled"><!----><i class="el-icon-edit" /><span>
        修改
      </span>
      </button>
      <button slot="reference" disabled="disabled" type="button" class="el-button filter-item el-button--danger el-button--mini is-disabled"><!----><i class="el-icon-delete" /><span>
        删除
      </span>
      </button>
      <button type="button" class="el-button filter-item el-button--warning el-button--mini"><!---->
        <i class="el-icon-download" /><span>导出</span></button>
    </div>
    <!--  -->

    <el-row :gutter="20">
      <el-col>
        <!-- 用户数据表格 -->
        <el-table
          :data="table"
          style="width: 100%"
        >
          <el-table-column :selectable="checkboxT" type="selection" width="55" />
          <el-table-column prop="id" label="用户ID" width="80" />
          <el-table-column prop="name" label="用户名" />
          <el-table-column prop="userIcon" label="头像" />
          <el-table-column prop="class" label="班级" width="120" />
          <el-table-column prop="group" label="小组" width="120" />
          <el-table-column prop="created_time" label="创建日期" width="180" />
          <el-table-column prop="updated_time" label="更新日期" width="180" />
          <el-table-column prop="status" label="用户状态" width="120" />
          <el-table-column prop="operation" label="操作" />
        </el-table>
        <!-- 分页插件 -->
        <el-form :inline="true" class="demo-form-inline" size="mini" style="margin-left:150px">
          <!--  分页排序-->
          <el-form-item>
            <el-pagination
              align="left"
              class="pagination"
              :current-page="page.currentPage"
              :page-sizes="[2,10,20,50]"
              :page-size="page.pageSize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="page.totalCount"
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
            />
          </el-form-item>
          <!-- 跳转页按钮 -->
          <el-form-item style="margin-top:17px;margin-left:10px">
            <el-button type="primary" sizi="mini" @click="query">确定</el-button>
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>
  </div>

</template>

<script>
export default {
  data() {
    return {
    // 模拟数据
      table: [{
        id: 1,
        name: '南街北巷',
        class: '卓越一班',
        group: 'ZYT-1',
        status: '1',
        created_time: '2016-05-02',
        updated_time: '2020-05-13',
        opearation: ''
      }, {
        id: 2,
        name: '南街北巷',
        class: '卓越一班',
        group: 'ZYT-1',
        status: '1',
        created_time: '2016-05-02',
        updated_time: '2020-05-13',
        opearation: ''
      }, {
        id: 3,
        name: '南街北巷',
        class: '卓越一班',
        group: 'ZYT-1',
        status: '1',
        created_time: '2016-05-02',
        updated_time: '2020-05-13',
        opearation: ''
      }, {
        id: 4,
        name: '南街北巷',
        class: '卓越一班',
        group: 'ZYT-1',
        status: '1',
        created_time: '2016-05-02',
        updated_time: '2020-05-13',
        opearation: ''
      }],
      // 定义page对象
      page: {
        currentPage: 1, // 当前页
        pageSize: 10, // 每页显示条数
        totalPage: 0, // 总页数
        totalCount: 0, // 总条数
        params: {
          'reportType': 0,
          'reportDate': ['', '']
        }, // 查询参数对象
        list: [], // 数据
        sortColumn: 'created_time', // 排序列
        sortMethod: 'asc' // 排序方式
      }
      // 分隔线
    }
  }
}
</script>
