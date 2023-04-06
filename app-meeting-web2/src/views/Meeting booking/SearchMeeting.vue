<template>
  <!-- 会议室管理 -->
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right" style="margin-bottom: 20px">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>会议预定</el-breadcrumb-item>
      <el-breadcrumb-item>搜索会议</el-breadcrumb-item>
    </el-breadcrumb>

    <!--  搜索框、搜索按钮、重置按钮  -->
    <div style="padding: 10px 0" class="searchInput">
      <div class="searchOptionInput">
        <el-input style="width:250px;" suffix-icon="el-icon-search" placeholder="请输入会议名称" v-model="meetingname" clearable>
        </el-input>
        <el-input style="width:250px;" suffix-icon="el-icon-search" placeholder="请输入会议室名称" v-model="roomname" clearable>
        </el-input>
        <el-input style="width:250px;" suffix-icon="el-icon-search" placeholder="请输入预定者姓名" v-model="reservationistname"
          clearable>
        </el-input>
      </div>
      <div>
        <div class="searchScheduledDate">
          <span>预定日期：</span>
          <el-date-picker style="margin-top: 10px" v-model="reservationtime" type="datetime" placeholder="选择预定日期时间"
            value-format="yyyy-MM-dd HH:mm:ss">
          </el-date-picker>
        </div>
        <div class="searchMeetingDate">
          <span>会议日期：</span>
          <el-date-picker v-model="starttime" type="datetime" placeholder="选择会议开始日期" value-format="yyyy-MM-dd HH:mm:ss">
          </el-date-picker>
        </div>
        <div class="searchtoDate">
          <span>至</span>
          <el-date-picker v-model="starttime" type="datetime" placeholder="选择会议结束日期" value-format="yyyy-MM-dd HH:mm:ss">
          </el-date-picker>
        </div>
        <el-button style="margin-left: 3px" type="primary" @click="load">搜索</el-button>
        <el-button style="margin-left: 3px" type="warning" @click="reset">重置</el-button>
      </div>
    </div>

    <!--  主体数据表格  -->
    <el-table style="margin-top: 5px" :data="tableData" v-loading="loading" border stripe
      :header-cell-class-name="headerBg" :default-sort="{ prop: 'roomid' }" v-if="!dataShowMethod">
      <el-table-column prop="meetingid" label="会议ID" sortable width="90">
      </el-table-column>
      <el-table-column prop="meetingname" label="会议名称" width="100">
      </el-table-column>
      <el-table-column prop="roomname" label="会议室名称" width="100">
      </el-table-column>
      <el-table-column prop="signinstarttime" label="签到开始时间" min-width="140px">
      </el-table-column>
      <el-table-column prop="signinendtime" label="签到结束时间" min-width="140px">
      </el-table-column>
      <el-table-column prop="starttime" label="会议开始时间" min-width="140px">
      </el-table-column>
      <el-table-column prop="endtime" label="会议结束时间" min-width="140px">
      </el-table-column>
      <el-table-column prop="reservationistname" label="预定者" width="100px">
      </el-table-column>
      <el-table-column label="操作" min-width="320px" header-align="center">
        <template slot-scope="scope">
          <el-button type="primary" size="small" @click="meetingLook(scope.row.meetingid)">查看详情<i
              class="el-icon-s-promotion"></i></el-button>
          <el-button type="primary" size="small" @click="participants(scope.row.meetingid)">查看参会员工<i
              class="el-icon-user"></i></el-button>
          <el-button type="danger" size="small" @click="handleDelete(scope.row.meetingid)"
            v-if="scope.row.reservationistid === employee.employeeid">删除<i class="el-icon-delete"></i></el-button>
        </template>
      </el-table-column>
    </el-table>
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
                      @click="handleDelete(item.meetingid)"
                      v-if="item.reservationistid === employee.employeeid"></el-button>
                    <el-button 
                      type="primary" 
                      icon="el-icon-user" 
                      circle 
                      class="button" 
                      @click="participants(item.meetingid)"></el-button>
                    <el-button 
                      type="primary" 
                      icon="el-icon-s-promotion" 
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
    <div style="padding-top: 15px" v-if="!dataShowMethod">
      <el-pagination align="center" @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="pageNum" :page-sizes="[5, 10, 15, 20]" :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>

    <!--查看会议详情-->
    <el-dialog title="会议信息" :visible.sync="dialogFormVisible" width="70%">
      <el-descriptions :direction="directionWithWidth" :column="columnWithWidth" border>
        <el-descriptions-item>
          <template slot="label"><i class="el-icon-monitor"></i>&nbsp;会议名称</template>{{ form.meetingname }}
        </el-descriptions-item>
        <el-descriptions-item>
          <template slot="label"><i class="el-icon-s-custom"></i>&nbsp;预计参与人数</template>{{ form.numberofparticipants }}
        </el-descriptions-item>
        <el-descriptions-item>
          <template slot="label"><i class="el-icon-s-home"></i>&nbsp;会议室房间号</template>{{ form.roomnum }}
        </el-descriptions-item>
        <el-descriptions-item>
          <template slot="label"><i class="el-icon-tickets"></i>&nbsp;会议室名称</template>
          <el-tag size="small">{{ form.roomname }}</el-tag>
        </el-descriptions-item>
        <el-descriptions-item>
          <template slot="label"><i class="el-icon-alarm-clock"></i>&nbsp;会议预定时间</template>{{ form.reservationtime }}
        </el-descriptions-item>
        <el-descriptions-item>
          <template slot="label"><i class="el-icon-postcard"></i>&nbsp;会议说明</template>{{ form.description }}
        </el-descriptions-item>
      </el-descriptions>
    </el-dialog>

    <!--  查看参会员工的弹窗  -->
    <el-dialog title="参会员工信息" :visible.sync="dialogVisible" center width="80%">
      <el-table :data="employeeData" border stripe :header-cell-class-name="headerBg">
        <el-table-column prop="employeename" label="员工姓名"></el-table-column>
        <el-table-column prop="phone" label="手机号"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="departmentname" label="所属部门"></el-table-column>
      </el-table>
    </el-dialog>

  </div>
</template>

<script>
export default {
  name: "SearchMeeting",
  data() {
    return {
      tableData: [],
      employeeData: [],
      total: 0,
      pageNum: 1,
      pageSize: 5,
      meetingname: '',
      roomname: '',
      reservationistname: '',
      reservationtime: '',
      starttime: '',
      form: [],
      headerBg: 'headerBg',
      dialogFormVisible: false,
      dialogVisible: false,
      dialogEditVisible: false,
      loading: false,
      employee: localStorage.getItem("employee") ? JSON.parse(localStorage.getItem("employee")) : {},
      count: 0,
      loadingdate: false
    }
  },
  //进入页面刷新数据
  created() {
    //请求分页查询数据
    if(!this.dataShowMethod) this.load()

  },
  computed: {
    columnWithWidth() {
      if (document.documentElement.clientWidth <= 600)
        return 1
      else
        return 4
    },
    directionWithWidth() {
      if (document.documentElement.clientWidth <= 600)
        return "horizontal"
      else
        return "vertical"
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
    //分页查询
    load() {
      this.loadingdate = true
      this.request.get("/meeting/page", {
        params: {
          pageNum: this.pageNum,
          pageSize: this.pageSize,
          meetingname: this.meetingname,
          roomname: this.roomname,
          reservationistname: this.reservationistname,
          reservationtime: this.reservationtime,
          starttime: this.starttime,
        }
      }).then(res => {
        if (this.dataShowMethod) {
          this.loading = true
          setTimeout(() => {
            this.tableData = [...this.tableData, ...res.data.records]
            this.total = res.data.total
            this.count += 5
            this.pageNum += 1
            this.loading = false
          }, 1000)
        }
        else {
          this.tableData = res.data.records
          this.total = res.data.total
          this.loadingdate = false
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
      this.roomname = ""
      this.reservationistname = ""
      this.reservationtime = ""
      this.starttime = ""
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
    //查看参会人员
    participants(meetingid) {
      this.dialogVisible = true
      this.meetingid = meetingid
      this.request.post("/bookings/participants/" + meetingid).then(res => {
        console.log(res)
        this.employeeData = res.data
      })
      this.employeeData = []
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
.searchInput .searchOptionInput {
  display: inline-block;
}

.searchInput .searchOptionInput .el-input--small:nth-child(2) {
  margin-left: 10px;
}

.searchInput .searchOptionInput .el-input--small:nth-child(3) {
  margin-left: 10px;
}

.searchInput .searchScheduledDate {
  display: inline-block;
}

.searchInput .searchMeetingDate {
  margin-left: 20px;
  display: inline-block;
}

.searchInput .searchtoDate {
  display: inline-block;
}
.searchInput .searchtoDate span{
  margin: 0px 3px;
  display: inline-block;
}
@media screen and (max-width: 600px) {
  .searchInput .searchOptionInput {
    margin: 10px 0px 0px 0px !important;
    display: block;
  }

  .searchInput .searchOptionInput .el-input:nth-child(2){
    margin: 10px 0px 0px 0px !important;
    display: none;
  }
  .searchInput .searchOptionInput .el-input:nth-child(3){
    margin: 10px 0px 0px 0px !important;
    display: none;
  }

  .searchInput .searchScheduledDate {
    display: none;
  }

  .searchInput .searchMeetingDate {
    display: none;
  }

  .searchInput .searchtoDate {
    display: none;
  }
  .searchInput .el-button {
    margin-top: 10px;
  }
}
</style>