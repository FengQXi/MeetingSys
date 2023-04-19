<template>
    <!-- 我的会议 -->
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right" style="margin-bottom: 20px">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>个人中心</el-breadcrumb-item>
            <el-breadcrumb-item>我的会议</el-breadcrumb-item>
        </el-breadcrumb>

        <!--  搜索框、搜索按钮、重置按钮  -->
        <div style="padding: 10px 0" class="MeetingInput">
            <el-input style="width:250px" suffix-icon="el-icon-search" placeholder="请输入会议名称" v-model="meetingname"
                clearable></el-input>
            <el-button style="margin-left: 3px" type="primary" @click="load">搜索</el-button>
            <el-button style="margin-left: 3px" type="warning" @click="reset">重置</el-button>
        </div>

        <!--  主体数据表格  -->
        <el-table :data="tableData" v-loading="loadingData" border stripe :header-cell-class-name="headerBg"
            v-if="!dataShowMethod">
            <el-table-column prop="meetingname" label="会议名称" width="100px"></el-table-column>
            <el-table-column prop="roomname" label="会议室名称" width="100px"></el-table-column>
            <el-table-column prop="signinstarttime" label="签到开始时间" min-width="140px" sortable></el-table-column>
            <el-table-column prop="signinendtime" label="签到结束时间" min-width="140px" sortable></el-table-column>
            <el-table-column prop="starttime" label="会议开始时间" min-width="140px" sortable></el-table-column>
            <el-table-column prop="endtime" label="会议结束时间" min-width="140px" sortable></el-table-column>
            <el-table-column prop="reservationistname" label="会议预定者" width="100px"></el-table-column>
            <el-table-column prop="description" label="会议描述" min-width="140px"></el-table-column>
            <el-table-column prop="status" label="签到状态" width="100px">
                <template slot-scope="scope">
                    <el-tag size="small" v-if="scope.row.status === 0 && nowDate >= scope.row.signinendtime"
                        type="danger">未签到</el-tag>
                    <el-tag size="small" v-if="scope.row.status === 0 && nowDate <= scope.row.signinstarttime"
                        type="info">签到未开始</el-tag>
                    <el-tag size="small"
                        v-if="scope.row.status === 0 && nowDate <= scope.row.signinendtime && nowDate >= scope.row.signinstarttime">正在签到</el-tag>
                    <el-tag size="small" v-else-if="scope.row.status === 1" type="success">已签到</el-tag>
                </template>
            </el-table-column>
            <el-table-column label="操作" min-width="180px">
                <template slot-scope="scope">
                    <el-button type="success" size="small" @click="signInVideo(scope.row.meetingid)"
                        v-if="scope.row.status === 0 && nowDate <= scope.row.signinendtime && nowDate >= scope.row.signinstarttime">人脸签到
                        <i class="el-icon-camera"></i>
                    </el-button>
                    <el-button type="success" size="small" @click="signInPassword(scope.row.meetingid)"
                        v-if="scope.row.status === 0 && nowDate <= scope.row.signinendtime && nowDate >= scope.row.signinstarttime">密码签到
                        <i class="el-icon-edit-outline"></i>
                    </el-button>
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
                                <div class="time" style="margin-bottom: 6px;">开始时间:&nbsp;{{ item.starttime }}</div>
                                <div class="time">签到状态:&nbsp;
                                    <el-tag size="small" v-if="item.status === 0 && nowDate >= item.signinendtime"
                                        type="danger">未签到</el-tag>
                                    <el-tag size="small" v-if="item.status === 0 && nowDate <= item.signinstarttime"
                                        type="info">签到未开始</el-tag>
                                    <el-tag size="small"
                                        v-if="item.status === 0 && nowDate <= item.signinendtime && nowDate >= item.signinstarttime">正在签到</el-tag>
                                    <el-tag size="small" v-else-if="item.status === 1" type="success">已签到</el-tag>
                                </div>
                                <el-button type="success" icon="el-icon-camera" circle class="button"
                                    @click="signInVideo(item.meetingid)"
                                    v-if="item.status === 0 && nowDate <= item.signinendtime && nowDate >= item.signinstarttime">
                                </el-button>
                                <el-button type="success" icon="el-icon-edit-outline" circle class="button"
                                    @click="signInPassword(item.meetingid)"
                                    v-if="item.status === 0 && nowDate <= item.signinendtime && nowDate >= item.signinstarttime">
                                </el-button>
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

        <!--  人脸签到  -->
        <el-dialog title="人脸签到" :visible.sync="videoVisible" @close="onCancel" :width="sginWidth" center top="20px"
            class="sgin">
            <!-- PC端 -->
            <div class="box" v-if="!dataShowMethod">
                <video id="videoCamera" class="canvas" :width="videoWidth" :height="videoHeight" autoPlay></video>
                <canvas id="canvasCamera" class="canvas" :width="videoWidth" :height="videoHeight" v-show="false"></canvas>
            </div>
            <!-- 移动端 -->
            <div class="box" v-if="dataShowMethod">
                <video id="videoCamera" class="canvas" :width="videoWidth" :height="videoHeight" autoPlay
                    v-show="videoorcanvasShow || !videoorcanvasShow"></video>
                <canvas id="canvasCamera" class="canvas" :width="videoWidth" :height="videoHeight"
                    v-show="!videoorcanvasShow && videoorcanvasShow"></canvas>
            </div>
            <div slot="footer">
                <el-button v-if="open === false" @click="
                    drawImage();
                videoorcanvasShow = false" icon="el-icon-camera" size="small">拍照</el-button>
                <el-button v-if="open" @click="getCompetence" icon="el-icon-video-camera" size="small">打开摄像头</el-button>
                <el-button v-else @click="stopNavigator" icon="el-icon-switch-button" size="small">关闭摄像头</el-button>
                <el-button v-if="open === false" @click="
                    resetCanvas();
                videoorcanvasShow = true" icon="el-icon-refresh" size="small">重置</el-button>
                <el-button @click="onUpload" :loading="submitLoading" type="primary" icon="el-icon-circle-check"
                    size="small" v-show="false">提交</el-button>
                <!-- <el-upload action :http-request="uploadImg" :show-file-list="false"
                    style="display: inline-block;margin-left: 7px">
                    <el-button type="primary" style="margin-left: 5px"><i class="el-icon-picture"></i>图片上传</el-button>
                </el-upload> -->
            </div>
        </el-dialog>
        <!--  密码签到  -->
        <el-dialog title="密码签到" :visible.sync="passwordVisible" width="420px" top="50px" center
            class="inputPassWordVisible">
            <el-input placeholder="请输入内容" v-model="inputPassword" clearable>
            </el-input>
            <div slot="footer" class="dialog-footer">
                <el-button @click="passwordVisible = false">取 消</el-button>
                <el-button type="primary" @click="signPassword">确 定</el-button>
            </div>
        </el-dialog>


    </div>
</template>
  
<script>
import dayjs from 'dayjs' //时间格式转换插件
export default {
    name: "Meeting",
    data() {
        return {
            nowDate: dayjs().format('YYYY-MM-DD HH:mm:ss'),
            employee: localStorage.getItem("employee") ? JSON.parse(localStorage.getItem("employee")) : {},
            tableData: [],
            total: 0,
            pageNum: 1,
            pageSize: 5,
            meetingname: '',
            headerBg: 'headerBg',
            imgSrc: "",
            videoVisible: false,//弹窗
            passwordVisible: false,//弹窗
            loading: false,//上传按钮加载
            open: false,//控制摄像头开关
            thisVideo: null,
            thisContext: null,
            thisCancas: null,
            videoWidth: 500,
            videoHeight: 400,
            meetingid: '',
            loadingData: false,
            location: localStorage.getItem("location"),
            count: 0,
            submitLoading: false,
            sginWidth: '1050px',
            videoorcanvasShow: true,
            inputPassword: ''
        }
    },
    //进入页面刷新数据
    created() {
        //请求分页查询数据
        if (!this.dataShowMethod) this.load()
    },
    computed: {
        dataShowMethod() {
            if (document.documentElement.clientWidth <= 500) {
                this.videoWidth = document.documentElement.clientWidth * 0.95 - 50
                this.videoHeight = document.documentElement.clientWidth * 1.1
                this.sginWidth = document.documentElement.clientWidth * 0.95 + 'px'
                return true
            }
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
            this.loadingData = true
            this.request.get("/bookings/myMeeting/" + this.employee.employeeid, {
                params: {
                    pageNum: this.pageNum,
                    pageSize: this.pageSize,
                    meetingname: this.meetingname,
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
                    this.loadingData = false
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
        //签到
        signInVideo(meetingid) {
            this.meetingid = meetingid
            this.videoVisible = true
            this.getCompetence()
        },
        signInPassword(meetingid) {
            this.meetingid = meetingid
            this.passwordVisible = true
            this.getCompetence()
        },
        onCancel() {
            this.visible = false;
            this.resetCanvas();
            this.stopNavigator();
        },
        //图片上传签到
        handleFileUploadSuccess(res) {
            if (res.code === 400) {
                this.$message({
                    showClose: true,
                    message: res.msg,
                    type: 'error',
                })
            } else {
                this.$message({
                    showClose: true,
                    message: res.data,
                    type: 'success'
                })
            }
            this.load()
        },
        // 密码签到
        signPassword() {
            if (this.inputPassword) {
                const employee = JSON.parse(localStorage.getItem("employee"))
                let formData = new FormData()
                formData.append("uuid", this.inputPassword);
                formData.append("meetingid", this.meetingid)
                formData.append("employeeid", employee.employeeid)
                formData.append("location", this.location)
                this.loading = true
                console.log(formData);
                this.request.post("/SignInByImage2", formData).then(res => {
                    if (res.code === 200) {
                        this.$message({
                            showClose: true,
                            type: 'success',
                            message: res.data,
                        })
                        this.tableData.forEach(item => {
                            if(item.meetingid = this.meetingid) {
                                item.status = 1
                            }
                        })
                        this.submitLoading = false
                        this.visible = false
                        this.load()
                    } else {
                        this.$message({
                            showClose: true,
                            type: 'error',
                            message: res.msg,
                        })
                        this.submitLoading = false
                    }
                })
            }
            else {
                this.$message({
                    message: '请输入',
                    type: 'warning',
                    duration: 1000
                });
            }
        },
        //base64转成文件后上传
        onUpload() {
            if (this.imgSrc) {
                let signedMeetings = localStorage.getItem("signedMeetings")
                if(signedMeetings != null && signedMeetings.indexOf(this.meetingid) != -1) {
                    this.$message({
                        type: "warning",
                        message: "当前会议一台设备只能签到一次",
                    })
                    return
                }

                const employee = JSON.parse(localStorage.getItem("employee"))
                let formData = new FormData()
                formData.append("file", this.base64ToFile(this.imgSrc, "png"));
                formData.append("flag", "videoImg"); // 额外参数
                formData.append("meetingid", this.meetingid)
                // formData.append("employeeid",localStorage.getItem("employeeid"))
                formData.append("employeeid", employee.employeeid)
                formData.append("location", this.location)
                this.loading = true
                console.log(formData);
                this.request.post("/SingIn", formData).then(res => {
                    if (res.code === 200) {
                        this.$message({
                            showClose: true,
                            type: 'success',
                            message: res.data,
                        })
                        this.tableData.forEach(item => {
                            if(item.meetingid = this.meetingid) {
                                item.status = 1
                            }
                        })

                        if(signedMeetings == null) {
                            signedMeetings = new Array(this.meetingid)
                        }
                        else signedMeetings.push(this.meetingid)
                        localStorage.setItem("signedMeetings", signedMeetings)

                        this.submitLoading = false
                        this.visible = false
                        this.load()
                    } else {
                        this.$message({
                            showClose: true,
                            type: 'error',
                            message: res.msg,
                        })
                        this.submitLoading = false
                    }
                })
            }
            else {
                this.$message({
                    message: '请点击拍照',
                    type: 'warning',
                    duration: 1000
                });
            }
        },
        //图片上传
        uploadImg(fileObj) {
            let formData = new FormData();
            formData.append("file", fileObj.file)
            formData.append("meetingid", this.meetingid)
            formData.append("employeeid", localStorage.getItem("employeeid"))
            formData.append("location", this.location)
            this.request.post("/SingIn", formData).then(res => {
                if (res.code === 200) {
                    this.$message({
                        showClose: true,
                        type: 'success',
                        message: res.data,
                    })
                    this.loading = false
                    this.visible = false
                    this.load()
                } else {
                    this.$message({
                        showClose: true,
                        type: 'error',
                        message: res.msg,
                    })
                }
            })
        },
        // 调用摄像头权限
        getCompetence() {
            //必须在model中render后才可获取到dom节点,直接获取无法获取到model中的dom节点
            this.$nextTick(() => {
                const _this = this;
                this.open = false;//切换成关闭摄像头
                this.thisCancas = document.getElementById('canvasCamera');
                this.thisContext = this.thisCancas.getContext('2d');
                this.thisVideo = document.getElementById('videoCamera');
                // 旧版本浏览器可能根本不支持mediaDevices，我们首先设置一个空对象
                if (navigator.mediaDevices === undefined) {
                    navigator.mediaDevices = {}
                }
                // 一些浏览器实现了部分mediaDevices，我们不能只分配一个对象
                // 使用getUserMedia，因为它会覆盖现有的属性。
                // 这里，如果缺少getUserMedia属性，就添加它。
                if (navigator.mediaDevices.getUserMedia === undefined) {
                    navigator.mediaDevices.getUserMedia = function (constraints) {
                        // 首先获取现存的getUserMedia(如果存在)
                        let getUserMedia = navigator.webkitGetUserMedia || navigator.mozGetUserMedia || navigator.getUserMedia;
                        // 有些浏览器不支持，会返回错误信息
                        // 保持接口一致
                        if (!getUserMedia) {
                            return Promise.reject(new Error('getUserMedia is not implemented in this browser'))
                        }
                        // 否则，使用Promise将调用包装到旧的navigator.getUserMedia
                        return new Promise(function (resolve, reject) {
                            getUserMedia.call(navigator, constraints, resolve, reject)
                        })
                    }
                }
                const constraints = {
                    audio: false,
                    video: { width: _this.videoWidth, height: _this.videoHeight, transform: 'scaleX(-1)' }
                };
                navigator.mediaDevices.getUserMedia(constraints).then(function (stream) {
                    // 旧的浏览器可能没有srcObject
                    if ('srcObject' in _this.thisVideo) {
                        _this.thisVideo.srcObject = stream
                    } else {
                        // 避免在新的浏览器中使用它，因为它正在被弃用。
                        _this.thisVideo.src = window.URL.createObjectURL(stream)
                    }
                    _this.thisVideo.onloadedmetadata = function (e) {
                        _this.thisVideo.play()
                    }
                }).catch(err => {
                    this.$message({
                        showClose: true,
                        message: '没有开启摄像头权限或浏览器版本不兼容.',
                        type: 'warning'
                    });
                });
            });
        },
        //绘制图片
        drawImage() {
            // 点击，canvas画图
            this.thisContext.drawImage(this.thisVideo, 0, 0, this.videoWidth, this.videoHeight);
            // 获取图片base64链接
            this.imgSrc = this.thisCancas.toDataURL('image/png');
            this.onUpload()
        },
        // base64 转为 file
        base64ToFile(urlData, fileName) {
            let arr = urlData.split(",");
            let mime = arr[0].match(/:(.*?);/)[1];
            let bytes = atob(arr[1]); // 解码base64
            let n = bytes.length;
            let ia = new Uint8Array(n);
            while (n--) {
                ia[n] = bytes.charCodeAt(n);
            }
            return new File([ia], fileName, { type: mime });
        },
        //清空画布
        clearCanvas(id) {
            let c = document.getElementById(id);
            let cxt = c.getContext("2d");
            cxt.clearRect(0, 0, c.width, c.height);
        },
        //重置画布
        resetCanvas() {
            this.imgSrc = "";
            this.clearCanvas('canvasCamera');
        },
        //关闭摄像头
        stopNavigator() {
            if (this.thisVideo) {
                this.thisVideo.srcObject.getTracks()[0].stop();
                this.open = true;//切换成打开摄像头
            }
        }
    }
}
</script>
  
<style scoped>
.headerBg {
    background: #eee !important;
}

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


@media screen and (max-width: 600px) {

    /* 搜索框 */
    .MeetingInput .el-input--small {
        display: block;
    }

    .MeetingInput .el-button {
        margin-top: 10px;
    }

    .sgin .el-button {
        margin-top: 10px;
    }

}
</style>
<style>
@media screen and (max-width: 600px) {

    .inputPassWordVisible .el-dialog {
        width: 95% !important;
    }
}
</style>