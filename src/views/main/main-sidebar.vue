<template>
    <aside class="site-sidebar" :class="'site-sidebar--' + sidebarLayoutSkin">
        <div class="site-sidebar__inner">
            <el-menu
                :default-active="menuActiveName || 'home'"
                :collapse="sidebarFold"
                :collapseTransition="false"
                :unique-opened="true"
                class="site-sidebar__menu">
                <el-menu-item index="home" @click="$router.push({ name: 'home' })">
                    <icon-svg name="shouye" class="site-sidebar__menu-icon"></icon-svg>
                    <span slot="title">首页</span>
                </el-menu-item>
                <sidebar-sub-menu
                    v-for="menu in menuList"
                    :key="menu.menuId"
                    :menu="menu"
                    :dynamicMenuRoutes="dynamicMenuRoutes">
                </sidebar-sub-menu>
            </el-menu>
        </div>
    </aside>
</template>

<script>
import SidebarSubMenu from './main-sidebar-sub-menu'

export default {
    data () {
        return {
            dynamicMenuRoutes: []
        }
    },
    components: {
        SidebarSubMenu
    },
    computed: {
        sidebarLayoutSkin: {
            get () {
                return this.$store.state.common.sidebarLayoutSkin
            }
        },
        sidebarFold: {
            get () {
                return this.$store.state.common.sidebarFold
            }
        },
        menuList: {
            get () {
                return this.$store.state.common.menuList
            },
            set (val) {
                this.$store.commit('common/updateMenuList', val)
            }
        },
        menuActiveName: {
            get () {
                return this.$store.state.common.menuActiveName
            },
            set (val) {
                this.$store.commit('common/updateMenuActiveName', val)
            }
        }
    },
    watch: {
        $route: 'routeHandle'
    },
    created () {
        this.menuList = JSON.parse(sessionStorage.getItem('menuList') || '[]')
        this.dynamicMenuRoutes = JSON.parse(sessionStorage.getItem('dynamicMenuRoutes') || '[]')
        this.routeHandle(this.$route)
    },
    methods: {
        // 路由操作
        routeHandle (route) {
        }
    }
}
</script>
