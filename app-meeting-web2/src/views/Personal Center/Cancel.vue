<template>
  <!-- 取消的会议 -->
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right" style="margin-bottom: 20px">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>个人中心</el-breadcrumb-item>
      <el-breadcrumb-item>已取消的会议</el-breadcrumb-item>
    </el-breadcrumb>

    <!--  搜索框、搜索按钮、重置按钮  -->
    <div style="padding: 10px 0" class="CancelInput">
      <el-input style="width:250px" suffix-icon="el-icon-search" placeholder="请输入会议名称" v-model="meetingname"
        clearable></el-input>
      <el-button style="margin-left: 3px" type="primary" @click="load">搜索</el-button>
      <el-button style="margin-left: 3px" type="warning" @click="reset">重置</el-button>
    </div>

    <!--  主体数据表格  -->
    <!-- PC端 -->
    <el-table :data="tableData" v-loading="loading" border stripe :header-cell-class-name="headerBg"
      v-if="!dataShowMethod">
      <el-table-column prop="roomname" label="会议室名称" width="100px"></el-table-column>
      <el-table-column prop="signinstarttime" label="签到开始时间" min-width="140px" sortable></el-table-column>
      <el-table-column prop="signinendtime" label="签到结束时间" min-width="140px" sortable></el-table-column>
      <el-table-column prop="starttime" label="会议开始时间" min-width="140px" sortable></el-table-column>
      <el-table-column prop="endtime" label="会议结束时间" min-width="140px" sortable></el-table-column>
      <el-table-column prop="canceledtime" label="取消会议时间" min-width="140px"></el-table-column>
      <el-table-column prop="canceledreason" label="取消会议原因" min-width="140px"></el-table-column>
      <el-table-column label="操作" width="200">
        <template slot-scope="scope">
          <el-button type="primary" size="small" @click="meetingLook(scope.row.meetingid)">查看详情<i
              class="el-icon-s-promotion"></i></el-button>
          <el-button type="danger" size="small" @click="handleDelete(scope.row.meetingid)">删除<i
              class="el-icon-delete"></i></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!--移动端 -->
    <div class="infinite-list-wrapper" style="overflow:auto" v-if="dataShowMethod">
      <ul class="list" v-infinite-scroll="load" infinite-scroll-disabled="disabled">
        <li v-for="item in tableData" class="list-item" style="margin-bottom: 20px;">
              <el-card :body-style="{ padding: '7px' }" shadow="never">
                <div style="padding: 14px;">
                  <span>{{ item.meetingname }}</span>
                  <div class="bottom clearfix">
                    <div class="time" style="margin-bottom: 6px;">会议地点:&nbsp;{{ item.roomname }}</div>
                    <div class="time">开始时间:&nbsp;{{ item.starttime }}</div>
                    <el-button 
                      type="danger" 
                      icon="el-icon-delete" 
                      circle 
                      class="button" 
                      @click="handleDelete(item.meetingid)"></el-button>
                    <el-button 
                      type="primary" 
                      icon="el-icon-view" 
                      circle 
                      class="button" 
                      @click="meetingLook(item.meetingid)"></el-button>
                  </div>
                </div>
              </el-card>
        </li>
      </ul>
      <p v-if="loading">
        <el-alert title="加载中..." type="success" :closable="false" center></el-alert>
      </p>
      <p v-if="noMore">
        <el-alert title="没有更多了" type="info" :closable="false" center></el-alert>
      </p>
    </div>

    <!--  分页  -->
    <div style="padding-top: 15px" class="paginationShow" v-if="!dataShowMethod">
      <el-pagination align="center" @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="pageNum" :page-sizes="[5, 10, 15, 20]" :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>

    <!--查看会议详情-->
    <el-dialog title="会议信息" :visible.sync="dialogFormVisible" width="90%">
      <el-descriptions class="margin-top" :column="columnWithWidth" :size="size" border>
        <el-descriptions-item>
          <template slot="label"><i class="el-icon-monitor"></i>&nbsp;会议名称</template>{{ form.meetingname }}
        </el-descriptions-item>
        <el-descriptions-item>
          <template slot="label"><i class="el-icon-alarm-clock"></i>&nbsp;会议预定时间</template>{{ form.reservationtime }}
        </el-descriptions-item>
        <el-descriptions-item>
          <template slot="label"><i class="el-icon-s-home"></i>&nbsp;会议室房间号</template>{{ form.roomnum }}
        </el-descriptions-item>
        <el-descriptions-item>
          <template slot="label"><i class="el-icon-tickets"></i>&nbsp;会议室名称</template>
          <el-tag size="small">{{ form.roomname }}</el-tag>
        </el-descriptions-item>
        <el-descriptions-item>
          <template slot="label"><i class="el-icon-postcard"></i>&nbsp;会议说明</template>{{ form.description }}
        </el-descriptions-item>
      </el-descriptions>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "Cancel",
  data() {
    return {
      tableData: [],
      headerBg: 'headerBg',
      employee: localStorage.getItem("employee") ? JSON.parse(localStorage.getItem("employee")) : {},
      dialogFormVisible: false,
      form: {},
      meetingid: '',
      employeeData: [],
      total: 0,
      pageNum: 1,
      pageSize: 5,
      meetingname: '',
      loading: false,
      // srceenWidth: document.documentElement.clientWidth,
      count: 0,
      loading: false
    }
  },
  //进入页面刷新数据
  created() {
    this.load()
  },
  computed: {
    columnWithWidth() {
      if (document.documentElement.clientWidth <= 500)
        return 1
      else
        return 2
    },
    dataShowMethod() {
      if (document.documentElement.clientWidth <= 500)
        return true
      else
        return false
    },
    noMore() {
      return this.count > this.total
    },
    disabled() {
      return this.loading || this.noMore
    }
  },
  methods: {
    //请求分页查询数据
    load() {
      this.loading = true
      this.request.get("/notifications/cancelMeeting/" + this.employee.employeeid, {
        params: {
          pageNum: this.pageNum,
          pageSize: this.pageSize,
          meetingname: this.meetingname
        }
      }).then(res => {
        if (this.dataShowMethod && !this.noMore) {
          this.loading = true
          setTimeout(() => {
            this.tableData = [...this.tableData, ...res.data.records]
            this.total = res.data.total
            this.count += 5
            this.loading = false
          }, 2000)
        }
        else {
          this.tableData = res.data.records
          this.total = res.data.total
          this.loading = false
        }
      });
    },
    //页数
    handleSizeChange(pageSize) {
      this.pageSize = pageSize
      this.load()
    },
    //页码
    handleCurrentChange(pageNum) {
      this.pageNum = pageNum
      this.load()
    },
    //搜索重置按钮
    reset() {
      this.meetingname = ""
      this.load()
    },
    //查看详情
    meetingLook(meetingid) {
      this.request.post("/meeting/meetingById/" + meetingid, this.form).then(res => {
        console.log(res)
        this.dialogFormVisible = true
        this.form = res.data[0]
      })
    },
    //根据ID删除
    handleDelete(meetingid) {
      this.$confirm('确定删除此会议吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.request.delete("/meeting/delete/" + meetingid).then(res => {
          if (res.code === 200) {
            this.$message({
              showClose: true,
              type: "success",
              message: "删除成功",
            })
            this.load()
          } else {
            this.$message({
              showClose: true,
              type: "error",
              message: "删除失败",
            })
          }
        })
      })

    },
  }
}
</script>

<style scoped>
.headerBg {
  background: #eee !important;
}

.time {
  font-size: 13px;
  color: #999;
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
}

.button {
  padding: 8px;
  margin-left: 3px;
  float: right;
}

.image {
  width: 50%;
  display: block;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}

.clearfix:after {
  clear: both
}

@media screen and (max-width: 500px) {
  .CancelInput .el-input--small {
    display: block;
  }

  .CancelInput .el-button {
    margin-top: 10px;
  }
}
</style>