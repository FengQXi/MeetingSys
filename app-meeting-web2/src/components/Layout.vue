<template>
    <div>
        <el-container style="min-height: 100vh; ">

            <el-aside :width="sideWidth + 'px'" style="box-shadow: 2px 0 6px rgb(0 21 41 / 35%);">
                <el-drawer :visible.sync="drawer" :with-header="false" direction="ltr" size="200px">
                    <Aside @asideCollapse="closeCollapse"/>
                </el-drawer>
            </el-aside>

            <el-container>
                <el-header style="border-bottom: 1px solid #ccc">
                    <Header :collapseBtnClass="collapseBtnClass" @asideCollapse="collapse" :employee="employee" />
                </el-header>

                <el-main>
                    <router-view @refreshUser="getUser" />
                </el-main>
            </el-container>
        </el-container>
    </div>
</template>

<script>
import Aside from "@/components/Aside";
import Header from "@/components/Header";

export default {
    name: "Layout",
    data() {
        return {
            drawer: false,
            collapseBtnClass: 'el-icon-s-unfold',
            sideWidth: 0,
            employee: {}
        }
    },
    components: {
        Aside,
        Header
    },
    created() {
        this.getUser()
    },
    methods: {
        //侧边栏收缩
        collapse() {  //点击收缩按钮触发
            this.drawer = true
        },
        closeCollapse() {
            this.drawer = false
        },
        getUser() {
            let username = localStorage.getItem("employee") ? JSON.parse(localStorage.getItem("employee")).username : ""
            //从后台获取数据
            this.request.get("/employee/username/" + username).then(res => {
                console.log(res)
                this.employee = res.data
            })
        }
    }
}
</script>

<style>
.el-aside {
    color: #333;
}
</style>