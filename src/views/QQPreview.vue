<template>
    <div>

        <v-card flat class="mb-2">
            <v-card-title class="d-flex">
                <span>导出选项</span>
                <v-spacer></v-spacer>
                <v-btn dark small color="red" @click="toHomepage">退出预览模式</v-btn>
            </v-card-title>

            <v-card-text>

                <div class="d-flex mb-2">
                    <v-text-field
                        dense
                        hide-details
                        label="群昵称"
                        outlined
                        readonly
                        v-model="QQPreviewData.groupName"
                    ></v-text-field>

                    <v-text-field
                        dense
                        hide-details
                        label="时间类型"
                        outlined
                        class="mr-2 ml-2"
                        readonly
                        v-model="timeType.name"
                    ></v-text-field>

                    <v-text-field
                        dense
                        hide-details
                        label="时间区间"
                        outlined
                        readonly
                        v-model="timeType.time"
                    ></v-text-field>
                </div>

                <div class="d-flex">
                    <v-text-field
                        dense
                        hide-details
                        label="报告类型"
                        outlined
                        readonly
                        v-model="productType"
                    ></v-text-field>

                    <v-text-field
                        dense
                        hide-details
                        label="个人报告QQ号"
                        outlined
                        class="mr-2 ml-2"
                        readonly
                        :disabled="QQPreviewData.setting.targetQQ === null"
                        v-model="QQPreviewData.setting.targetQQ"
                    ></v-text-field>

                    <v-text-field
                        dense
                        hide-details
                        label="群聊活跃成员数"
                        outlined
                        readonly
                        v-model="QQPreviewData.members.length"
                    ></v-text-field>
                </div>

            </v-card-text>

            <div class="d-flex">
                <h5 class="ml-3 grey--text">选择导出的类型(用于分享)</h5>
            </div>

            <v-card-actions class="d-flex">
                <div class="productType">
                    <v-radio-group
                        v-model="product"
                        row
                        dense
                        hide-details
                        class="mt-0 d-flex"
                    >
                        <v-radio
                            label="网页版"
                            value="web">
                            <template v-slot:label>
                                <span v-if="product !== 'web'">网页版</span>
                                <v-select
                                    v-else
                                    dense
                                    hide-details
                                    style="width: 158px"
                                    :items="getWebTemplates"
                                    label="选择网页模板"
                                    outlined
                                    item-text="name"
                                    item-value="path"
                                    v-model="webTemplateSelectedPath"
                                ></v-select>
                            </template>
                        </v-radio>
                        <v-radio
                            label="长图版"
                            value="img"
                        >
                            <template v-slot:label>
                                <span v-if="product !== 'img'">长图版</span>
                                <v-select
                                    v-else
                                    dense
                                    hide-details
                                    style="width: 158px"
                                    :items="getImgTemplates"
                                    label="选择长图模板"
                                    outlined
                                    item-text="name"
                                    item-value="path"
                                    v-model="imgTemplateSelectedPath"
                                ></v-select>
                            </template>
                        </v-radio>
                        <v-radio
                            label="视频版"
                            value="video"
                            disabled
                        ></v-radio>
                        <v-radio
                            value="txt"
                        >
                            <template v-slot:label>
                                <span v-if="product !== 'txt'">纯文本</span>
                                <v-select
                                    v-else
                                    dense
                                    hide-details
                                    style="width: 158px"
                                    :items="getTextTemplates"
                                    label="选择纯文本模板"
                                    outlined
                                    item-text="name"
                                    item-value="path"
                                    v-model="textTemplateSelectedPath"
                                ></v-select>
                            </template>
                        </v-radio>
                    </v-radio-group>
                </div>
                <v-spacer></v-spacer>
                <v-btn color="primary" @click="exportProduct">导出</v-btn>
            </v-card-actions>
        </v-card>

        <v-card flat class="mb-2" v-if="outputText.length > 0">
            <v-card-title class="d-flex">
                <span>文本导出</span>
            </v-card-title>

            <v-card-text>
                <v-textarea
                    readonly
                    outlined
                    :value="outputText"
                ></v-textarea>

                <div class="d-flex">
                    <v-spacer></v-spacer>
                    <v-btn color="red" dark @click="copyText">一键复制</v-btn>
                </div>
            </v-card-text>

        </v-card>

        <v-card flat class="mb-2">
            <v-card-title class="d-flex">
                <span>数据预览</span>
            </v-card-title>

            <div>

                <v-divider></v-divider>

                <div>
                    <v-card-subtitle>成员数据统计</v-card-subtitle>
                    <v-data-table
                        :headers="memberHeaders"
                        :items="QQPreviewData.members"
                        :items-per-page="5"
                    ></v-data-table>
                </div>

                <v-divider></v-divider>

                <div class="pb-4">
                    <v-card-subtitle>词云</v-card-subtitle>
                    <div id="wordcloud" style="height: 400px" ref="wordcloud"></div>
                </div>

                <v-divider></v-divider>

                <div>
                    <v-card-subtitle>24小时消息数</v-card-subtitle>
                    <div ref="hoursLineChart" style="height: 400px"></div>
                </div>

                <v-divider></v-divider>

                <div>
                    <v-card-subtitle>所有时间消息数</v-card-subtitle>
                    <div ref="allTimeBarChart" style="height: 400px;"></div>
                </div>

                <v-divider></v-divider>

                <div>
                    <v-card-subtitle>复读排行</v-card-subtitle>
                    <v-data-table
                        :headers="repeaterHeaders"
                        :items="QQPreviewData.sortRepeats"
                        :items-per-page="5"
                    ></v-data-table>
                </div>

                <v-divider></v-divider>

                <div>
                    <v-card-subtitle>最长消息内容</v-card-subtitle>
                    <div class="pa-2">
                        <v-textarea
                            readonly
                            outlined
                            :value="QQPreviewData.longestContent"
                        ></v-textarea>
                    </div>
                </div>
            </div>
        </v-card>

        <v-expansion-panels class="mb-2" flat>
            <v-expansion-panel>
                <v-expansion-panel-header>
                    原始产出数据
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                    <v-textarea
                        readonly
                        outlined
                        :value="JSON.stringify(QQPreviewData, null, 4)"
                    ></v-textarea>
                </v-expansion-panel-content>
            </v-expansion-panel>
        </v-expansion-panels>
    </div>
</template>

<script>

import {mapState} from "vuex"

import * as echarts from 'echarts'

import 'echarts-wordcloud'
import getWordcloud from "@/util/chart/wordcloud"
import getActiveHoursChart from "@/util/chart/activeHoursChart"
import getAllTimeChart from "@/util/chart/allTimeChart"

import textConverter from "@/template/text/converter"
import webConverter from '@/template/web/converter'
import imgConverter from '@/template/image/converter'

const { ipcRenderer } = require('electron')

import {getSelectedTimeDetail} from "@/util/tool"

export default {
    name: "QQPreview",
    data: () => ({
        product: 'web',
        memberHeaders: [
            {
                text: 'QQ号',
                align: 'start',
                sortable: false,
                value: 'qq',
            },
            {text: '昵称', sortable: false, value: 'name'},
            {text: '消息数', value: 'msgs'},
            {text: '发图数', value: 'imgs'},
            {text: '活跃天数', value: 'actives'},
            {text: '总字数', value: 'words'},
            {text: '复读机', value: 'repeats'},
        ],
        repeaterHeaders: [
            {
                text: '复读内容',
                align: 'start',
                sortable: false,
                value: 'content',
            },
            {text: '复读次数', value: 'count'},
        ],
        textTemplateSelectedPath: '',
        // 导出的文字
        outputText: '',
        // 选择的网页模板
        webTemplateSelectedPath: '',
        // 选择的图片模板
        imgTemplateSelectedPath: ''
    }),
    mounted() {
        console.log("所有数据", this.QQPreviewData)

        let wordcloud = echarts.init(this.$refs.wordcloud)
        wordcloud.setOption(this.wordcloudData)

        let activeHours = echarts.init(this.$refs.hoursLineChart)
        activeHours.setOption(this.activeHoursData)

        let allTime = echarts.init(this.$refs.allTimeBarChart)
        allTime.setOption(this.getAllTimeData)

        // 窗口发生变化，重新渲染
        window.onresize = function resize() {
            wordcloud.resize()
            activeHours.resize()
            allTime.resize()
        };

        if (this.getTextTemplates.length > 0){
            this.textTemplateSelectedPath = this.getTextTemplates[0].path
        }
        if (this.getWebTemplates.length > 0){
            this.webTemplateSelectedPath = this.getWebTemplates[0].path
        }

        if (this.getImgTemplates.length > 0){
            this.imgTemplateSelectedPath = this.getImgTemplates[0].path
        }
    },
    methods: {
        toHomepage() {
            this.$router.replace('/')
            this.$store.commit('updateSelectedItem', 0)
        },
        exportProduct(){

            if (this.QQPreviewData.members.length === 0){
                this.$toast.error('没有分析数据，请确保聊天记录内容格式符合要求，或加群反馈')
                return
            }

            switch (this.product) {
                case 'txt':
                    ipcRenderer.invoke('read-txt', this.textTemplateSelectedPath).then( res => {
                        this.outputText = textConverter(res, this.QQPreviewData)
                    })
                    break
                case 'web':
                    ipcRenderer.invoke('select-dir').then( res => {
                        if (res){
                            // 导出
                            webConverter(res[0], this.QQPreviewData, this.webTemplateSelectedPath,this.getWebTemplates, {
                                wordcloudData: this.wordcloudData,
                                activeHoursData: this.activeHoursData,
                                getAllTimeData: this.getAllTimeData
                            }).then( () => {
                                this.$toast.success('导出成功')
                            })
                        }
                    })
                    break
                case 'img':
                    imgConverter(this.QQPreviewData, this.imgTemplateSelectedPath,this.getImgTemplates, {
                        wordcloudData: this.wordcloudData,
                        activeHoursData: this.activeHoursData,
                        getAllTimeData: this.getAllTimeData
                    }).then( () => {
                        this.$toast.success('截图成功，请手动关闭窗口')
                    })

                    break
            }
        },
        copyText(){
            ipcRenderer.invoke('copy-text', this.outputText)
            this.$toast.success('复制成功')
        }
    },
    computed: {
        ...mapState(['QQPreviewData', 'TemplatesConfig']),
        timeType() {
            return getSelectedTimeDetail(this.QQPreviewData.setting)
        },
        productType() {
            switch (this.QQPreviewData.commonSetting.product) {
                case 'group':
                    return '群聊年度报告'
                case 'person':
                    return '群聊个人年度报告'
                case 'love':
                    return '情侣年度报告'
            }
            return '年度报告'
        },
        wordcloudData() {
            return getWordcloud(this.QQPreviewData)
        },
        activeHoursData() {
            return getActiveHoursChart(this.QQPreviewData)
        },
        getAllTimeData() {
            return getAllTimeChart(this.QQPreviewData)
        },
        getTextTemplates(){
            return this.TemplatesConfig.text.filter(template => template.type === this.QQPreviewData.setting.report)
        },
        getWebTemplates(){
            return this.TemplatesConfig.web.filter(template => template.type === this.QQPreviewData.setting.report)
        },
        getImgTemplates(){
            return this.TemplatesConfig.img.filter(template => template.type === this.QQPreviewData.setting.report)
        }
    }
}
</script>

<style scoped>

</style>