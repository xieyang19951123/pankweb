<template>
<div class="mod-demo-echarts">

    <el-row :gutter="20">

        <el-col :span="24">
            <el-card>
                <div id="J_chartBarBox" class="chart-box"></div>
            </el-card>
        </el-col>
        <el-col :span="12">
            <el-card>
                <div id="J_chartPieBox" class="chart-box"></div>
            </el-card>
        </el-col>
    </el-row>

</div>
</template>

<script>
import echarts from 'echarts'
export default {
    data() {
        return {

            chartBar: null,

        }
    },
    mounted() {

        this.getPunchup()
        this.getAllDot()

    },
    activated() {
        // 由于给echart添加了resize事件, 在组件激活时需要重新resize绘画一次, 否则出现空白bug

        if (this.chartBar) {
            this.chartBar.resize()
        }
        if (this.chartPie) {
            this.chartPie.resize()
        }
    },
    methods: {
        getAllDot() {
            this.$http({
                url: this.$http.adornUrl('/pank/dot/getAllDot'),
                method: 'get',
                params: this.$http.adornParams({})
            }).then(({
                data
            }) => {
                this.initChartBar(data.page)
            })
        },
        getPunchup() {
            this.$http({
                url: this.$http.adornUrl('/pank/dot/getPunchup'),
                method: 'get',
                params: this.$http.adornParams({})
            }).then(({
                data
            }) => {
                this.initChartPie(data.page)
            })
        },
        // 柱状图
        initChartBar(data) {
            var option = {
                color: ["#3398DB"],
                title: {
                    text: '参加人数'
                },
                tooltip: {},
                legend: {
                    data: ['参加人数']
                },
                xAxis: {
                    data: data.keys
                },
                yAxis: {},
                series: [{
                    name: '',
                    type: 'bar',
                    data: data.vals
                }]
            };
            this.chartBar = echarts.init(document.getElementById('J_chartBarBox'))
            this.chartBar.setOption(option)
            window.addEventListener('resize', () => {
                this.chartBar.resize()
            })
        },

        // 饼状
        initChartPie(data) {
            let vas = []
            for (var i = 0; i < data.length; i++) {
                if (data[i].value != 0) {
                    vas.push(data[i])
                }
            }
            var option = {
                backgroundColor: '#2c343c',
                title: {
                    text: '打卡点人数分布',
                    left: 'center',
                    top: 20,
                    textStyle: {
                        color: '#ccc'
                    }
                },
                tooltip: {
                    trigger: 'item',
                    formatter: '{a} <br/>{b} : {c} ({d}%)'
                },
                visualMap: {
                    show: false,
                    min: 80,
                    max: 600,
                    inRange: {
                        colorLightness: [0, 1]
                    }
                },
                series: [{
                    name: '人数',
                    type: 'pie',
                    radius: '55%',
                    center: ['50%', '50%'],
                    data: vas.sort(function (a, b) {
                        return a.value - b.value
                    }),
                    roseType: 'radius',
                    label: {
                        normal: {
                            textStyle: {
                                color: 'rgba(255, 255, 255, 0.3)'
                            }
                        }
                    },
                    labelLine: {
                        normal: {
                            lineStyle: {
                                color: 'rgba(255, 255, 255, 0.3)'
                            },
                            smooth: 0.2,
                            length: 10,
                            length2: 20
                        }
                    },
                    itemStyle: {
                        normal: {
                            color: '#c23531',
                            shadowBlur: 200,
                            shadowColor: 'rgba(0, 0, 0, 0.5)'
                        }
                    },
                    animationType: 'scale',
                    animationEasing: 'elasticOut',
                    animationDelay: function (idx) {
                        return Math.random() * 200
                    }
                }]
            }
            this.chartPie = echarts.init(document.getElementById('J_chartPieBox'))
            this.chartPie.setOption(option)
            window.addEventListener('resize', () => {
                this.chartPie.resize()
            })
        },

    }
}
</script>

<style lang="scss">
.mod-demo-echarts {
    >.el-alert {
        margin-bottom: 10px;
    }

    >.el-row {
        margin-top: -10px;
        margin-bottom: -10px;

        .el-col {
            padding-top: 10px;
            padding-bottom: 10px;
        }
    }

    .chart-box {
        min-height: 400px;
    }
}
</style>
