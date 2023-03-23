<template>
    <el-menu style="min-height: 100%;overflow-x: hidden" background-color="#304156" text-color="#bfcbd9"
        active-text-color="#409EFF" :default-active="$route.path"
        router>
        <div style="height: 60px;line-height: 60px;text-align: center">
            <img src="../assets/logo.png" alt="" style="width: 20px;position: relative;top: 5px;margin-right: 5px">
            <b style="color: #fff">会议签到系统</b>
        </div>
        <div v-for="item in normalMenus" :key="item.id">
            <div v-if="item.path">
                <el-menu-item @click="collapse" :index="item.path">
                    <i :class="item.icon"></i>
                    <span slot="title">{{ item.name }}</span>
                </el-menu-item>
            </div>
            <div v-else>
                <el-submenu :index="item.id + ''">
                    <template slot="title">
                        <i :class="item.icon"></i>
                        <span>{{ item.name }}</span>
                    </template>
                    <div v-for="subItem in item.children" :key="subItem.id">
                        <el-menu-item @click="collapse" :index="subItem.path">
                            <i :class="subItem.icon"></i>
                            <span slot="title">{{ subItem.name }}</span>
                        </el-menu-item>
                    </div>
                </el-submenu>
            </div>
        </div>
        <!-- <div v-if="adminMenus.length" style="height: 2px; width: calc(100% - 40px); background-color: rgb(151, 145, 145); margin: auto; border-radius: 2px;"></div>
        <div v-for="item in adminMenus" :key="item.id">
            <div v-if="item.path">
                <el-menu-item :index="item.path">
                    <i :class="item.icon"></i>
                    <span slot="title">{{ item.name }}</span>
                </el-menu-item>
            </div>
            <div v-else>
                <el-submenu :index="item.id + ''">
                    <template slot="title">
                        <i :class="item.icon"></i>
                        <span>{{ item.name }}</span>
                    </template>
                    <div v-for="subItem in item.children" :key="subItem.id">
                        <el-menu-item :index="subItem.path">
                            <i :class="subItem.icon"></i>
                            <span slot="title">{{ subItem.name }}</span>
                        </el-menu-item>
                    </div>
                </el-submenu>
            </div>
        </div> -->
    </el-menu>
</template>

<script>
export default {
    name: "Aside",
    props: {
        isCollapse: Boolean,
        logoTextShow: Boolean
    },
    data() {
        return {
            admin: ['数据报表', '系统管理员', '权限管理'],
            menus: localStorage.getItem("menus") ? JSON.parse(localStorage.getItem("menus")) : [],
            opens: localStorage.getItem("menus") ? JSON.parse(localStorage.getItem("menus")).map(v => v.id + '') : []
        }
    },
    methods: {
        collapse() {
            this.$emit("asideCollapse")
        },
    },
    computed: {
        normalMenus() {
            return this.menus.filter(item => {
                if(this.admin.indexOf(item.name) > -1) {
                    return false
                }
                else return true
            })
        },
        adminMenus() {
            return this.menus.filter(item => {
                if(this.admin.indexOf(item.name) > -1) {
                    return true
                }
                else return false
            })
        }
    },
}
</script>

<style>
/*解决收缩菜单文字不消失问题*/
.el-menu--collapse span {
    visibility: hidden;
}

.aaaaa {
    background-color: rgb(151, 145, 145)
}
</style>