<template>
    <div class="outer_box">
        <div class="filterImg"></div>
        <div class="login_box">
            <h1><img src="../../../public/avater.png" alt="会议签到系统"></h1>
            <el-form :model="ruleForm" :rules="rules" ref="ruleForm" class="login_input">
                <el-form-item prop="username">
                    <el-input prefix-icon="el-icon-user" v-model="ruleForm.username" placeholder="请输入账号"></el-input>
                </el-form-item>
                <el-form-item prop="password">
                    <el-input prefix-icon="el-icon-lock" type="password" show-password v-model="ruleForm.password"
                        placeholder="请输入密码"></el-input>
                </el-form-item>
                <div class="register2"><router-link to="/register">免费注册</router-link></div>
                <el-form-item class="btns">
                    <el-button type="primary" @click="toFaceLogin" class="btnFace" plain size="small">人脸登录</el-button>
                    <el-button type="primary" :loading="loading" @click.native.prevent="submitForm('ruleForm')"
                        class="btnLongin" size="small">
                        <span v-if="!loading">登录</span>
                        <span v-else>登录中...</span>
                    </el-button>
                    <el-button @click="resetForm('ruleForm')" class="btnReset" size="small">重置</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>

<script>

import { setRoutes } from "@/router";

export default {
    name: "Login",
    data() {
        return {
            ruleForm: {
                username: '',
                password: '',
            },
            loading: false,
            rules: {
                username: [
                    { required: true, message: '请输入账号', trigger: 'blur' },
                    { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }
                ],
                password: [
                    { required: true, message: '请输入密码', trigger: 'change' },
                    { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
                ],
            },
        };
    },
    mounted() {
        // 获取地理位置
        let geolocation = new BMapGL.Geolocation();
        geolocation.getCurrentPosition(function (r) {
            if (this.getStatus() === BMAP_STATUS_SUCCESS) {
                const province = r.address.province
                const city = r.address.city
                localStorage.setItem("location", JSON.stringify(province + city))
            }
        });
    },
    // computed: {
    //   dataShowMethod() {
    //     if (document.documentElement.clientWidth <= 500)
    //       return true
    //     else
    //       return false
    //   },
    // },
    methods: {
        submitForm(ruleForm) {
            this.$refs['ruleForm'].validate((valid) => {
                if (valid) {
                    this.loading = true
                    this.request.post("/employee/login", this.ruleForm).then(res => {
                        if (res.code === 200) {
                            localStorage.setItem("employee", JSON.stringify(res.data))  //把用户信息存在浏览器
                            localStorage.setItem("menus", JSON.stringify(res.data.menus))
                            localStorage.setItem("token", JSON.stringify(res.data.token))
                            this.$notify.success({
                                title: "登录成功",
                                message: `欢迎回来~ 亲爱的${res.data.employeename}`
                            });
                            //设置当前用户的路由
                            setRoutes()
                            this.$router.push("/")
                        } else {
                            this.$message({
                                showClose: true,
                                message: res.msg,
                                type: 'error',
                                duration: 1000
                            })
                        }
                        this.loading = false
                    }, error => {
                        // console.log(error);
                        this.$message({
                            showClose: true,
                            message: error.message || "ERROR",
                            type: 'error',
                            duration: 1500
                        })
                        this.loading = false
                    })
                }
            })
        },
        resetForm(ruleForm) {
            this.$refs['ruleForm'].resetFields();
        },
        toFaceLogin() {
            this.$router.push('/faceLogin')
        }
    }
}
</script>

<style scoped>
* {
    margin: 0px;
    padding: 0px;
    color: #fff;
}

.filterImg {
    width: 100%;
    height: 100%;
    background-image: url('../../../public/backimg.jpg');
    background-size: 100% 100%;
}

.outer_box {
    width: 100%;
    position: relative;
    height: calc(100vh);
}

.login_box {
    width: 320px;
    height: 400px;
    border: 1px solid #fff;
    background-color: rgba(189, 156, 156, 0.233);
    border-radius: 10px;
    position: absolute;
    left: 75%;
    top: 47%;
    transform: translate(-50%, -50%);
}

/* 用户头像 */
.login_box h1 {
    width: 100%;
    height: 112px;
    margin: 25px 0px;
    text-align: center;
}

.login_box h1 img {
    width: 120px;
    height: 120px;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0px 0px 5px #fff;
    border: 1px solid #fff;
}

/* 注册与忘记密码 */
.register2 {
    width: 260px;
    font-size: 12px;
    margin: 0px 30px;
    display: flex;
    justify-content: flex-end;
}

.login_input {
    position: absolute;
    bottom: 10px;
    width: 100%;
    padding: 20px;
    box-sizing: border-box;
}

.btns {
    display: flex;
    justify-content: space-around;
}

.btnLongin {
    width: 85px;
    height: 35px;
    margin-top: 20px;
}

.btnFace {
    width: 85px;
    height: 35px;
    margin-top: 20px;

}

.btnReset {
    width: 85px;
    height: 35px;
    margin-top: 20px;
    color: skyblue;
}

a {
    text-decoration: none;
    color: #cccccc;
}

@media screen and (max-width: 500px) {
    .login_box {
        left: 50%;
    }
}
</style>