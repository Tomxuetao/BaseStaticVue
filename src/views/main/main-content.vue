<template>
    <main class="site-content" :class="{ 'site-content--tabs': $route.meta.isTab }">
        <el-card :body-style="siteContentViewHeight">
            <keep-alive>
                <router-view></router-view>
            </keep-alive>
        </el-card>
    </main>
</template>

<script>

export default {
    inject: ['refresh'],
    computed: {
        documentClientHeight: {
            get () {
                return this.$store.state.common.documentClientHeight
            }
        },

        siteContentViewHeight () {
            let height = this.documentClientHeight - 82
            return { minHeight: height + 'px', padding: '24px' }
        }
    }
}
</script>
