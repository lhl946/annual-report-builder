<template>
    <div>
        <v-card class="mb-2" outlined flat>
            <v-card-subtitle class="d-flex">
                <span>第一步：选择聊天记录 <span @click="toHelp" class="link">点击查看导出聊天记录的方法</span> </span>
                <v-spacer></v-spacer>
                <span v-if="lineCount > 0">约{{lineCount}}行</span>
            </v-card-subtitle>

            <v-card-text class="d-flex align-center">
                <v-text-field
                    v-model="path"
                    label="点击右侧按钮选择导出的txt格式消息记录文件"
                    outlined
                    dense
                    readonly
                    hide-details
                    class="mr-2"
                ></v-text-field>
                <v-btn @click="selectFile" color="#4caf50" dark>选择聊天记录</v-btn>
            </v-card-text>

        </v-card>
        <v-card class="mb-2" outlined flat>
            <v-card-subtitle class="d-flex">
                <span>第二步：分析的聊天记录时间范围</span>
            </v-card-subtitle>

            <v-card-text class="d-flex flex-column">
                <div class="d-flex align-center">
                    <v-radio-group
                        v-model="setting.type"
                        row
                        dense
                        hide-details
                        class="mt-0"
                    >
                        <v-radio
                            label="年度报告"
                            value="year"
                        ></v-radio>
<!--                        <v-radio-->
<!--                            label="季度报告"-->
<!--                            value="season"-->
<!--                        ></v-radio>-->
                        <v-radio
                            label="月度报告"
                            value="month"
                        ></v-radio>
                        <v-radio
                            label="指定日期"
                            value="ranger"
                        ></v-radio>
                    </v-radio-group>
                </div>

                <div class="mt-2">
                    <section v-if="setting.type === 'year'">
                        <date-picker :clearable="false" v-model="setting.year" value-type="timestamp" type="year" placeholder="选择年份"></date-picker>
                    </section>
                    <section v-if="setting.type === 'month'">
                        <date-picker :clearable="false" v-model="setting.month" value-type="timestamp" type="month" placeholder="选择月份"></date-picker>
                    </section>
                    <section v-if="setting.type === 'ranger'">
                        <date-picker :clearable="false" v-model="setting.ranger" value-type="timestamp" range></date-picker>
                    </section>
                </div>
            </v-card-text>
        </v-card>

        <v-card class="mb-2" outlined flat>
            <v-card-subtitle class="d-flex">
                <span>第三步：选择报告类型</span>
            </v-card-subtitle>

            <v-card-text class="d-flex flex-column">
                <v-radio-group
                    v-model="setting.report"
                    row
                    dense
                    hide-details
                    class="mt-0"
                >
                    <v-radio
                        label="群聊年度报告"
                        value="group"
                    ></v-radio>
                    <v-radio
                        label="个人年度报告"
                        value="person"
                    ></v-radio>
                    <v-radio
                        label="情侣年度报告(开发中)"
                        value="love"
                        disabled
                    ></v-radio>
                </v-radio-group>
                <div v-if="setting.report === 'person'" class="mt-2">
                    <v-text-field
                        v-model="setting.targetQQ"
                        label="需要统计的QQ号，只能输入一个QQ号码"
                        outlined
                        dense
                        hide-details
                    ></v-text-field>
                </div>
            </v-card-text>
        </v-card>

        <v-card outlined flat>
            <v-card-subtitle class="d-flex">
                <span>最后一步</span>
                <v-spacer></v-spacer>
                <a href="javascript:;" @click="toSetting">已排除的QQ号数：{{excludeQQ.length}}，点击查看更多设置</a>
            </v-card-subtitle>
            <v-card-text class="d-flex flex-column">
                <v-btn :loading="showLoadingBar" color="primary" @click="generateReport">立即生成报告</v-btn>
            </v-card-text>
        </v-card>
    </div>
</template>

<script>
const { ipcRenderer } = require('electron')
import DatePicker from 'vue2-datepicker'
import 'vue2-datepicker/index.css'
import 'vue2-datepicker/locale/zh-cn'

const dayjs = require('dayjs')

import { mapState } from "vuex"

export default {
    name: "QQReport",
    components:{
        DatePicker
    },
    data: () => ({
        path: '',
        lineCount: 0,
        setting: {
            type: 'year',
            year: null,
            month: null,
            ranger: [],
            report: 'group',
            targetQQ: null
        },
        defaultYear: null
    }),
    mounted() {
        this.setting.year = new Date().setFullYear(dayjs().year() - 1)
        this.setting.month = new Date().setFullYear(dayjs().year(), dayjs().month())
    },
    methods:{
        async selectFile(){
            let paths = await ipcRenderer.invoke('select-file')
            if (paths && paths.length > 0){
                this.path = paths[0]
                this.lineCount = await ipcRenderer.invoke('count-line', this.path)

            }

        },
        async generateReport(){

            if (this.path.length === 0){
                this.$toast.error('请先选择文件')
                return
            }

            // 生成报告
            let result = await ipcRenderer.invoke('qq-report', this.path, this.lineCount, this.setting, this.$store.state.setting)
            console.log(result)
            // this.$toast.success('Info toast')
        },
        toSetting(){
            this.$store.commit('updateSelectedItem', 3)
            this.$router.replace('/setting')
        },
        toHelp(){
            this.$store.commit('updateSelectedItem', 4)
            this.$router.replace('/help')
        }
    },
    computed:{
        ...mapState(['showLoadingBar']),
        excludeQQ:{
            get(){
                return this.$store.state.setting.excludeQQ
            }
        }
    }
}
</script>

<style scoped lang="scss">
    .link{
        color: #0d47a1;
        text-decoration: underline;
        cursor: pointer;
    }
</style>