<template>
    <div>
        <el-select
                popper-class="custom-select"
                v-model="value2"
                multiple
                collapse-tags
                style="margin-left: 20px;"
                placeholder="请选择">
            <el-option
                    v-for="item in options"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
            </el-option>
        </el-select>
    </div>
</template>

<script>
export default {
    name: 'index2',
    data () {
        return {
            value2: '',
            options: [{
                value: '选项1',
                label: '黄金糕'
            }, {
                value: '选项2',
                label: '双皮奶'
            }, {
                value: '选项3',
                label: '蚵仔煎'
            }, {
                value: '选项4',
                label: '龙须面'
            }, {
                value: '选项5',
                label: '北京烤鸭'
            }, {
                value: '选项1',
                label: '黄金糕'
            }, {
                value: '选项2',
                label: '双皮奶'
            }, {
                value: '选项3',
                label: '蚵仔煎'
            }, {
                value: '选项4',
                label: '龙须面'
            }, {
                value: '选项5',
                label: '北京烤鸭'
            }]
        }
    },

    mounted () {
        this.$nextTick(() => {
            const temp = document.querySelector('.custom-select ul')
            console.log(temp)
            const li = document.createElement('li')
            li.innerText = '+'
            li.style.color = '#828A99'
            li.style.fontSize = '18px'
            li.style.height = '34px'
            li.style.lineHeight = '34px'
            li.style.textAlign = 'center'
            li.style.cursor = 'pointer'
            li.style.backgroundColor = '#F4F5F7'
            li.onclick = () => {
                this.test()
            }
            temp.appendChild(li)
        })
    },

    methods: {
        test () {
            this.$router.push({ name: 'sys-index3' })
        }
    }
}
</script>

<style scoped>
    .test {
        height: 34px;
        cursor: pointer;
        line-height: 34px;
    }
</style>
