<template>
    <div class="search-box" style="display: flex">
        <div class="box-history">
            <el-button type="info" class="history-btn" @click.stop="showHistoryHandle">
                <span>筛选</span>
                <i class="el-icon-caret-bottom"></i>
            </el-button>
            <div id="historyWrapper" class="history-wrapper">
                <template v-if="localList.length > 0 || sessionList.length > 0">
                    <div class="history-local" v-if="localList.length > 0">
                        <div class="local-header">已保存的筛选条件</div>
                        <div class="local-content">
                            <div class="local-item" v-for="(item, index) in localList" :key="index"
                                 @click="() => selectRecordHandle(item)">
                                <div class="item-title">{{item.name}}</div>
                                <div class="item-content">
                                    <el-tooltip :content="item|assembleHandle" placement="right">
                                        <span>{{item|assembleHandle|strSplitHandle}}</span>
                                    </el-tooltip>
                                    <i class="el-icon-delete" @click.stop="() => removeLocalOneHandle(index)"></i>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="history-session" v-if="sessionList.length > 0">
                        <div class="session-header">
                            <div>最近搜索历史记录</div>
                            <el-button class="session-clear" type="text" @click="clearSessionHandle">清空</el-button>
                        </div>
                        <div class="session-content">
                            <div class="session-item" v-for="(item, index) in sessionList" :key="index"
                                 @click="() => selectRecordHandle(item)">
                                <el-tooltip :content="item|assembleHandle" placement="right">
                                    <span>{{item|assembleHandle|strSplitHandle}}</span>
                                </el-tooltip>
                            </div>
                        </div>
                    </div>
                </template>
                <template v-else>
                    <div>123</div>
                </template>
            </div>
        </div>
        <div class="box-wrapper">
            <div class="input-wrapper">
                <ul class="input-container" id="inputContainer">
                    <li class="select-wrapper" v-for="(item, index) in searchList" :key="index">
                        <div class="select-item">
                            <div class="select-name">{{item.title}}</div>
                            <div class="value-box" v-for="(temp, key) in item.list" :key="key">
                                <div class="value-item">{{temp.valueStr}}</div>
                                <div class="remove-icon __condition__" @click="removeItemHandle(index, key)">
                                    <i class="el-icon-close __condition__"></i>
                                </div>
                            </div>
                        </div>
                    </li>
                    <li class="input-box __condition__">
                        <el-input id="searchInput" class="custom-input __condition__" placeholder="搜索或过滤结果..." v-model="inputValue"
                                  @focus="inputFocusHandle" @blur="inputFocus = false" @input="inputChangeHandle"
                                  ref="input"></el-input>
                        <div id="selectCondition" class="select-condition __condition__">
                            <div class="condition-header __condition__" @click="searchHandle">
                                <i class="el-icon-search __condition__"></i>
                                <span class="__condition__">按回车键或点击此处搜索</span>
                            </div>
                            <div class="condition-content __condition__">
                                <div class="condition-item __condition__" v-for="(item, index) in searchOptions"
                                     :key="index">
                                    <div class="condition-title __condition__">
                                        <i class="el-icon-document-copy __condition__"></i>
                                        {{item.title}}
                                    </div>
                                    <div class="item-content __condition__" v-for="(item2, index2) in item.list"
                                         :key="index2"
                                         @click.prevent="() => conditionClickHandle(item.title, item.type, item2)">
                                        <span class="__condition__">{{item2.valueStr}}</span>
                                        <i v-if="item2.check" class="el-icon-circle-check __condition__"
                                           style="margin-left: 12px; font-size: 12px; color: #0091FF"></i>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <!-- 二级条件选择 -->
                        <div id="nextCondition" class="next-condition __condition__">
                            <div class="next-header __condition__" @click="selectOtherHandle">
                                <i class="el-icon-d-arrow-left __condition__"></i>
                                <span class="__condition__">选择其他刷选条件</span>
                            </div>
                            <div class="next-content __condition__">
                                <div class="next-item __condition__" v-for="(item, index) in cloneTempList"
                                     :key="index" @click="() => conditionClickHandle(inputTemp.valueStr, 'select', item)">
                                    {{item.valueStr}}
                                </div>
                            </div>
                        </div>
                    </li>
                </ul>
            </div>
            <div class="input-operate" v-if="searchList.length > 0">
                <el-button class="operate-item" icon="el-icon-error" type="text"
                           @click="() => resetSearchHandle(false)"></el-button>
                <el-popover
                        v-model="popoverVisible"
                        placement="bottom"
                        width="242">
                    <div style="font-size: 14px; color: rgba(23, 35, 61, 1); margin-bottom: 12px; padding-left: 12px;">
                        保存本次搜索条件
                    </div>
                    <el-form :rules="dataRules" :model="dataForm" style="padding-left: 12px">
                        <el-form-item prop="localSearchName">
                            <el-input placeholder="保存搜索名称" v-model="dataForm.localSearchName">
                                <template slot="append">
                                    <el-button type="primary" @click="saveLocalHandle">保存</el-button>
                                </template>
                            </el-input>
                        </el-form-item>
                    </el-form>
                    <el-button class="operate-item" icon="el-icon-s-claim" type="text" slot="reference"></el-button>
                </el-popover>
            </div>
        </div>
        <div style="margin-left: 8px">
            <el-button icon="el-icon-refresh-right" type="info" @click="() => resetSearchHandle(true)">重置</el-button>
        </div>
        <div style="display: none;">
            <el-date-picker v-for="(item, index) in datePickerList" :key="index"
                            popper-class="picker-wrapper"
                            :ref="item.prop"
                            v-model="item.model"
                            @change="() => pickerChangeHandle(item)"
                            type="daterange"
                            value-format="yyyy-MM-dd">
            </el-date-picker>
        </div>
    </div>
</template>

<script>
import cloneDeep from 'lodash/cloneDeep'
import isEqual from 'lodash/isEqual'

export default {
    name: 'index',
    data () {
        return {
            selectConditionDom: {},
            nextConditionDom: {},
            historyWrapperDom: {},
            inputValue: null,
            inputFocus: false,
            // 搜索框内容
            searchList: [],
            // 临时选中对象
            inputTemp: {},
            // 选中对象的list深度拷贝值
            cloneTempList: [],
            // 存放临时（最近）搜索记录 {name: '', list: []}
            sessionList: [],
            // 存放保存的搜素记录  {name: '', list: []}
            localList: [],
            // 存放时间选择器内容
            datePickerList: [],
            dataForm: {
                localSearchName: ''
            },
            popoverVisible: false,
            searchOptions: [
                {
                    title: '库存状态',
                    type: 'select',
                    list: [{
                        prop: 'status0',
                        value: '',
                        valueStr: '在库',
                        check: false
                    }, {
                        prop: 'status0',
                        value: '',
                        valueStr: '已出库',
                        check: false
                    }]
                },
                {
                    title: '入库原因',
                    type: 'select',
                    list: [{
                        prop: 'status1',
                        value: '',
                        valueStr: '库存备货',
                        check: false
                    }, {
                        prop: 'status1',
                        value: '',
                        valueStr: '项目备货',
                        check: false
                    }]
                },
                {
                    title: '时间',
                    type: 'time',
                    list: [{
                        prop: 'time',
                        value: '',
                        valueStr: '退库时间',
                        check: false
                    }, {
                        prop: 'time1',
                        value: '',
                        valueStr: '采购日期',
                        check: false
                    }, {
                        prop: 'time2',
                        value: '',
                        valueStr: '退换期限',
                        check: false
                    }, {
                        prop: 'time3',
                        value: '',
                        valueStr: '质保期限',
                        check: false
                    }]
                }, {
                    title: '品牌',
                    type: 'select',
                    list: [{
                        prop: 'pt2',
                        value: '',
                        valueStr: '高锦',
                        check: false
                    }, {
                        prop: 'pt2',
                        value: '',
                        valueStr: '其他',
                        check: false
                    }, {
                        prop: 'pt2',
                        value: '',
                        valueStr: '中性产品',
                        check: false
                    }]
                },
                {
                    title: '模糊查询',
                    type: 'list',
                    list: [{
                        prop: 'name',
                        value: '',
                        valueStr: '名称',
                        check: false,
                        list: [{
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛2',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛3',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛4',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛5',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛6',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛7',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛8',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛9',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛10',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛11',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛12',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛13',
                            check: false
                        }]
                    }, {
                        prop: 'sort',
                        value: '',
                        valueStr: '型号',
                        check: false,
                        list: [{
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛1',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛11',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛12',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛13',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛14',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛15',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛16',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛17',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛18',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛19',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛20',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛21',
                            check: false
                        }]
                    }, {
                        prop: 'sort2',
                        value: '',
                        valueStr: '序列号',
                        check: false,
                        list: [{
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛11',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛21',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛31',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛41',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛51',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛61',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛71',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛81',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛91',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛421',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛4561',
                            check: false
                        }, {
                            prop: 'name',
                            value: '',
                            valueStr: '王雪涛891',
                            check: false
                        }]
                    }]
                }
            ],
            dataRules: {
                localSearchName: [{ required: true, message: '搜索名称不能为空', trigger: 'blur' }]
            }
        }
    },

    created () {
        sessionStorage.setItem('searchArray', JSON.stringify([]))
        localStorage.setItem('searchArray', JSON.stringify([]))
        let tempDateList = this.searchOptions.filter(item => item.type === 'time')
        if (tempDateList.length > 0) {
            this.datePickerList = tempDateList[0].list
            this.datePickerList.map(item => {
                this.$set(item, 'model', '')
            })
        }
    },

    mounted () {
        this.$nextTick(() => {
            this.selectConditionDom = document.getElementById('selectCondition')
            this.nextConditionDom = document.getElementById('nextCondition')
            this.historyWrapperDom = document.getElementById('historyWrapper')
            document.addEventListener('keyup', this.keyupListenerHandle)
            document.addEventListener('click', this.clickListenerHandle)
        })
    },

    beforeDestroy () {
        document.removeEventListener('keyup', event => {
        }, false)
        document.removeEventListener('click', event => {
        }, false)
    },

    filters: {
        assembleHandle: function (item) {
            let tempString = ''
            if (item) {
                item.list.map(temp => {
                    tempString += temp.title + '：' + temp.list.reduce((accumulator, currentValue) => {
                        if (accumulator === '') {
                            return accumulator + currentValue.valueStr
                        } else {
                            return accumulator + '，' + currentValue.valueStr
                        }
                    }, '') + '；'
                })
            }
            return tempString
        },

        strSplitHandle: function (string) {
            if (string.length > 30) {
                return string.substr(0, 30) + '...'
            }
            return string
        }
    },

    methods: {
        // 移除一项
        removeItemHandle (index, key) {
            this.searchList[index].list.splice(key, 1)
            if (this.searchList[index].list.length === 0) {
                this.searchList.splice(index, 1)
            }
            this.setPositionHandle()
        },

        // 重置搜素框人
        resetSearchHandle (reset) {
            this.searchList = []
            this.inputTemp = {}
            this.cloneTempList = []
            if (reset) {
                this.$emit('resetFormHandle')
            }
            this.setPositionHandle()
        },

        // 输入框获取焦点事件
        inputFocusHandle () {
            this.inputFocus = true
            this.historyWrapperDom.style.display = 'none'
            if (Object.keys(this.inputTemp).length > 0) {
                this.nextConditionDom.style.display = 'block'
            } else {
                this.selectConditionDom.style.display = 'block'
            }
            this.setPositionHandle()
        },

        // 在 Input 值改变时触发事件
        inputChangeHandle (value) {
            if (Object.keys(this.inputTemp).length > 0) {
                this.cloneTempList = cloneDeep(this.inputTemp.list)
            }
            if (value) {
                if (this.cloneTempList.length > 0) {
                    this.cloneTempList = this.cloneTempList.filter(item => {
                        return item.valueStr.includes(value)
                    })
                }
            } else {
                this.inputValue = ''
            }
        },

        // 时间选择框处理
        pickerChangeHandle (item) {
            let temp = this.searchList.find(data => {
                return data.title === item.valueStr
            })
            if (temp) {
                if (!temp.list.find(element => { return element.valueStr === item.valueStr })) {
                    temp.list = [{
                        value: item.model.join(' - '),
                        valueStr: item.model.join(' - ')
                    }]
                }
            } else {
                this.searchList.push({
                    title: item.valueStr,
                    prop: item.prop,
                    list: [{
                        value: item.model.join(' - '),
                        valueStr: item.model.join(' - ')
                    }]
                })
            }
        },

        // 面板条件点击事件
        conditionClickHandle (title, type, select) {
            switch (type) {
            case 'select': {
                let temp = this.searchList.find(item => {
                    return item.title === title
                })
                if (temp) {
                    if (!temp.list.find(element => { return element.valueStr === select.valueStr })) {
                        temp.list.push({
                            value: select.value,
                            valueStr: select.valueStr
                        })
                    }
                } else {
                    this.searchList.push({
                        title: title,
                        prop: select.prop,
                        list: [{
                            value: select.value,
                            valueStr: select.valueStr
                        }]
                    })
                }
                break
            }
            case 'list': {
                this.selectConditionDom.style.display = 'none'
                this.nextConditionDom.style.display = 'block'
                this.inputTemp = select
                this.cloneTempList = cloneDeep(this.inputTemp.list)
                break
            }
            case 'time': {
                this.selectConditionDom.style.display = 'none'
                this.$refs[select.prop][0].focus()
                break
            }
            }
            this.setPositionHandle()
        },

        // 切换选择条件
        selectOtherHandle () {
            this.nextConditionDom.style.display = 'none'
            this.selectConditionDom.style.display = 'block'
            this.inputTemp = {}
            this.cloneTempList = []
        },

        // 父组件通过自定义事件，真正执行查询
        searchHandle () {
            if (this.searchList.length > 0) {
                this.$emit('executeSearchHandle', this.convertHandle())
            } else {
                this.$message.error('查询条件为空！')
            }
        },

        // 点击搜素记录
        selectRecordHandle (record) {
            this.historyWrapperDom.style.display = 'none'
            this.selectConditionDom.style.display = 'none'
            this.searchList = cloneDeep(record.list)
            this.$emit('executeSearchHandle', this.convertHandle())
        },

        // 转换查询条件
        convertHandle () {
            let searchForm = {}
            let cloneTemp = {
                name: '',
                list: cloneDeep(this.searchList)
            }
            let sessionTemp = JSON.parse(sessionStorage.getItem('searchArray'))
            let flagArray = []
            // 重复搜索条件不保存
            sessionTemp.forEach(item => {
                flagArray.push(isEqual(item, cloneTemp))
            })
            if (!flagArray.includes(true)) {
                sessionTemp.push(cloneTemp)
                sessionStorage.setItem('searchArray', JSON.stringify(sessionTemp))
                this.sessionList.push(cloneTemp)
            }

            // 组装搜索条件
            cloneTemp.list.map(item => {
                searchForm[item.prop] = item.list.map(element => {
                    return element.valueStr
                })
            })
            return searchForm
        },

        // 清空临时的搜素记录
        clearSessionHandle () {
            this.sessionList = []
            sessionStorage.setItem('searchArray', JSON.stringify([]))
        },

        // 移除一条已保存的搜索记录
        removeLocalOneHandle (index) {
            this.localList.splice(index, 1)
            let storageList = JSON.parse(localStorage.getItem('searchArray'))
            storageList.splice(index, 1)
            localStorage.setItem('searchArray', JSON.stringify(storageList))
        },

        // 保存查询条件
        saveLocalHandle () {
            if (this.dataForm.localSearchName !== '') {
                let cloneTemp = {
                    name: this.dataForm.localSearchName,
                    list: cloneDeep(this.searchList)
                }

                let localTemp = JSON.parse(localStorage.getItem('searchArray'))
                let flagArray = []
                // 重复搜索条件不保存
                localTemp.forEach(item => {
                    flagArray.push(isEqual(item, cloneTemp))
                })
                if (!flagArray.includes(true)) {
                    localTemp.push(cloneTemp)
                    localStorage.setItem('searchArray', JSON.stringify(localTemp))
                    this.localList.push(cloneTemp)
                }
                this.popoverVisible = false
                this.dataForm.localSearchName = ''
            }
        },

        // 展示搜索历史面板
        showHistoryHandle () {
            this.historyWrapperDom.style.display = 'block'
            this.selectConditionDom.style.display = 'none'
            this.nextConditionDom.style.display = 'none'
        },

        // 调整条件选择框的位置
        setPositionHandle () {
            this.$nextTick(() => {
                if (document.getElementById('searchInput').clientWidth < 700) {
                    this.selectConditionDom.style.right = '-38px'
                } else {
                    this.selectConditionDom.style.right = ''
                }
                console.log(document.getElementById('searchInput').clientWidth)
            })
        },

        keyupListenerHandle (event) {
            // 按下Enter执行查询操作
            if (event.key === 'Enter') {
                this.searchHandle()
            }
            // 按下Backspace执行删除一项
            if (event.key === 'Backspace') {
                if (this.inputFocus && this.inputValue === null) {
                    let searchLen = this.searchList.length
                    if (searchLen > 0) {
                        let listLen = this.searchList[searchLen - 1].list.length
                        this.removeItemHandle(searchLen - 1, listLen - 1)
                    }
                } else {
                    if (this.inputValue === '') {
                        this.inputValue = null
                    }
                }
            }
        },

        clickListenerHandle (event) {
            if (!event.target.classList.contains('__condition__') && !event.target.classList.contains('el-input__inner')) {
                this.selectConditionDom.style.display = 'none'
                this.nextConditionDom.style.display = 'none'
                this.historyWrapperDom.style.display = 'none'
            }
        }
    }
}
</script>

<style lang="scss">
    ::-webkit-scrollbar {
        width: 0 !important;
    }

    .search-box {
        display: flex;

        .box-history {
            .history-btn {
                border-radius: 3px 0 0 3px;
                border: 1px solid #e5e5e5;
                border-right: 0;
                background-color: rgba(244, 245, 247, 1);
            }

            .history-wrapper {
                display: none;
                position: absolute;
                width: 437px;
                height: 375px;
                margin-top: 2px;
                overflow-y: auto;
                border-radius: 2px;
                z-index: 2003;
                background-color: rgba(255, 255, 255, 1);
                box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);

                .history-local {
                    .local-header {
                        height: 35px;
                        line-height: 35px;
                        width: 100%;
                        padding-left: 16px;
                        background-color: rgba(244, 245, 247, 1);
                        font-size: 13px;
                        color: rgba(115, 119, 128, 1);
                    }

                    .local-content {
                        .local-item {
                            cursor: pointer;
                            height: 60px;
                            padding-left: 16px;
                            font-size: 13px;
                            color: rgba(19, 26, 38, 1);
                            border-bottom: 1px solid rgba(231, 234, 236, 1);

                            .item-title {
                                font-size: 13px;
                                margin-top: 16px;
                                color: rgba(19, 26, 38, 1);
                            }

                            .item-content {
                                font-size: 12px;
                                margin-top: 12px;
                                color: rgba(130, 138, 153, 1);

                                > i {
                                    cursor: pointer;
                                    float: right;
                                    padding-right: 20px;
                                    font-size: 14px;
                                    color: rgba(71, 132, 179, 1);
                                }
                            }
                        }
                    }
                }

                .history-session {
                    .session-header {
                        display: flex;
                        height: 35px;
                        line-height: 35px;
                        width: 100%;
                        padding-left: 16px;
                        background-color: rgba(244, 245, 247, 1);
                        font-size: 13px;
                        color: rgba(115, 119, 128, 1);

                        .session-clear {
                            margin-left: 270px;
                            color: rgba(71, 132, 179, 1);
                        }

                    }

                    .session-content {
                        .session-item {
                            cursor: pointer;
                            height: 40px;
                            line-height: 40px;
                            padding-left: 16px;
                            font-size: 13px;
                            color: rgba(19, 26, 38, 1);
                            border-bottom: 1px solid rgba(231, 234, 236, 1);
                        }
                    }
                }
            }
        }

        .box-wrapper {
            cursor: pointer;
            width: 100%;
            min-width: 0;
            border: 1px solid #e5e5e5;
            background-color: #fff;
            position: relative;
            flex: 1;
            display: flex;
            border-radius: 0 4px 4px 0;

            .input-wrapper {
                flex: 1;
                position: relative;
                min-width: 0;

                .input-container {
                    flex-wrap: wrap;
                    padding-left: 12px;
                    margin: 0;
                    display: flex;
                    position: relative;
                    flex: 1;

                    li {
                        list-style: none;
                    }

                    .select-wrapper {
                        display: flex;
                        position: relative;
                        flex-shrink: 0;
                        max-width: 100%;
                        margin-top: 3px;
                        margin-bottom: 3px;
                        white-space: nowrap;

                        .select-item {
                            display: flex;
                            position: relative;

                            .select-name {
                                display: inline-block;
                                padding: 2px 7px;
                                background-color: rgba(244, 245, 247, 1);
                                color: rgba(130, 138, 153, 1);
                                border-radius: 2px 0 0 2px;
                                margin: 1px 1px 1px 0;
                                text-transform: capitalize;
                            }

                            .value-box {
                                display: flex;
                                flex: 1;
                                background-color: rgba(244, 245, 247, 1);
                                color: rgba(19, 26, 38, 1);
                                border-radius: 0 2px 2px 0;
                                margin: 1px 1px 1px 0;
                                padding-right: 8px;

                                .value-item {
                                    display: inline-block;
                                    padding: 2px 8px 2px 14px;
                                }

                                .remove-icon {
                                    display: inline-block;
                                    padding-left: 4px;
                                    padding-right: 0;
                                    line-height: 22px;
                                }
                            }

                            .value-box:hover {
                            }
                        }
                    }

                    .input-box {
                        flex: auto;

                        .custom-input {
                            width: 100%;
                        }

                        .select-condition {
                            display: none;
                            position: absolute;
                            margin-top: 2px;
                            border-radius: 2px;
                            z-index: 2003;
                            background-color: rgba(255, 255, 255, 1);
                            box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);

                            .condition-header {
                                height: 35px;
                                line-height: 35px;
                                width: 100%;
                                padding-left: 16px;
                                background-color: rgba(244, 245, 247, 1);
                                font-size: 13px;
                                color: rgba(115, 119, 128, 1);
                            }

                            .condition-content {
                                display: flex;
                                padding: 12px;

                                .condition-item {
                                    .condition-title {
                                        height: 26px;
                                        width: 130px;
                                        line-height: 26px;
                                        font-size: 14px;
                                        color: rgba(130, 138, 153, 1);
                                    }

                                    .item-content {
                                        height: 26px;
                                        width: 130px;
                                        line-height: 26px;
                                        padding-left: 18px;
                                        font-size: 14px;
                                        color: rgba(58, 64, 77, 1);
                                    }

                                    .item-content:hover {
                                        background-color: rgba(240, 248, 255, 1);
                                    }
                                }
                            }
                        }

                        .next-condition {
                            display: none;
                            width: 250px;
                            position: absolute;
                            margin-top: 2px;
                            border-radius: 2px;
                            z-index: 2003;
                            background-color: rgba(255, 255, 255, 1);
                            box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);

                            .next-header {
                                height: 35px;
                                line-height: 35px;
                                width: 100%;
                                padding-left: 16px;
                                background-color: rgba(244, 245, 247, 1);
                                font-size: 13px;
                                color: rgba(115, 119, 128, 1);
                            }

                            .next-content {
                                height: 350px;
                                overflow-x: auto;

                                .next-item {
                                    height: 35px;
                                    line-height: 35px;
                                    width: 100%;
                                    padding-left: 16px;
                                    font-size: 13px;
                                    color: rgba(130, 138, 153, 1);
                                }

                                .next-item:hover {
                                    background-color: rgba(240, 248, 255, 1);
                                }
                            }
                        }

                        .el-input__inner {
                            border: 0 solid #E1E2E6 !important;
                            height: 30px;
                            line-height: 30px;
                        }
                    }
                }
            }

            .input-operate {
                height: 100%;
                padding-right: 10px;
                margin: auto;

                .operate-item {
                    color: rgba(172, 179, 191, 1);
                    font-size: 12px;
                }
            }
        }
    }

    .picker-wrapper {
        top: 180px !important;
        right: 24px;
        left: auto !important;
    }
</style>
