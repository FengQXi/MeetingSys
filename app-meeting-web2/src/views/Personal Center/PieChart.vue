<template>
    <div ref="mychart" style="height: 350px; width: 300px;"></div>
</template>

<script>
import * as echarts from "echarts"

export default {
    name: "PieChart",
    props: ["meetingid"],
    data() {
        return {
            totalNumber: '',
            signedNumber: '',
        }
    },
    methods: {
        async getData() {
            let result1 = await this.request.post("/bookings/queryNum/" + this.meetingid)
            let result2 = await this.request.post("/bookings/signed/" + this.meetingid)
            // console.log(result1)
            // console.log(result2)
            if (result1.code == 200 && result2.code == 200) {
                this.totalNumber = result1.data
                this.signedNumber = result2.data

                let pieChart = echarts.init(this.$refs.mychart)
                pieChart.setOption({
                    tooltip: {
                        trigger: 'item'
                    },
                    legend: {
                        top: '5%',
                        left: 'center'
                    },
                    series: [
                        {
                            name: '签到状况',
                            type: 'pie',
                            radius: ['40%', '70%'],
                            avoidLabelOverlap: false,
                            itemStyle: {
                                borderRadius: 10,
                                borderColor: '#fff',
                                borderWidth: 2
                            },
                            label: {
                                show: false,
                                position: 'center'
                            },
                            emphasis: {
                                label: {
                                    show: true,
                                    fontSize: 20,
                                    fontWeight: 'bold'
                                }
                            },
                            labelLine: {
                                show: false
                            },
                            data: [
                                { value: this.totalNumber - this.signedNumber, name: '已签到' },
                                { value: this.signedNumber, name: '未签到' },
                            ]
                        }
                    ]
                })
            }
        }
    },
    mounted() {
        this.getData()
    },
}
</script>

<style></style>