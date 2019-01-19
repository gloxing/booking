<template>
    <div class='single-page'>
        <div class="booking-container">
            <div class="tab-right-top">
                <el-button type='primary' @click='handleAdd'>新增预约</el-button>
                <el-button type='primary' @click='handleScheduler'>排班信息</el-button>
            </div>
            <el-tabs v-model="activeName" @tab-click="tabClick" v-loading=''>
                <el-tab-pane label="日历模式" name='calendar' class='calendar-container'>
                    <!-- 头部按钮 -->
                    <div class="btns">
                        <el-row :gutter="20">
                            <el-col :span="8">
                                <el-select v-model="selectedStaffIds" placeholder="所有门店员工" multiple clearable>
                                    <el-option v-for='item in staffs' :key=item.id :label="item.staffName" :value="item.id"></el-option>
                                </el-select>
                            </el-col>
                            <el-col :span="12">
                                <div class='inline icon-btn' @click='autoPreDay'><i class="fa fa-angle-left"></i></div>
                                <el-date-picker v-model="selectedDay" placeholder="选择日期" :clearable='false'>
                                </el-date-picker>
                                <div class='inline icon-btn' @click='autoNextDay'><i class="fa fa-angle-right"></i></div>
                                <el-button @click="autoBackToday">今天</el-button>
                                <el-button @click="autoBack7">下周</el-button>
                                <el-button @click="autoBack15">15天</el-button>
                                <el-button @click="autoBack30">30天</el-button>
                            </el-col>
                        </el-row>
                    </div>
                    <!-- 表单 -->
                    <el-table :data="allBookingData" style="width: 100%" height="400" border @cell-click='handClickCell' v-loading="loading" element-loading-text="加载中" element-loading-spinner="el-icon-loading" :cell-class-name='cellClassName'>
                        <el-table-column fixed prop="time" label="时间" width="100">
                            <template slot-scope="scope">
                                <div style='margin-top: 5px;'>{{scope.row.time}}</div>
                            </template>
                        </el-table-column>
                        <el-table-column v-for="(item,index) in selectedStaffs" :key='item.id' :label="item.staffName" :prop="item.id+''" width='200'>
                            <template slot-scope="scope">
                                <div v-if="scope.row[item.id] && scope.row[item.id].position=='start'">
                                    <span>{{scope.row[item.id].text}}</span>
                                </div>
                                <div v-else-if="scope.row[item.id] && scope.row[item.id].position=='other'">
                                </div>
                                <div v-else></div>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <el-tab-pane label="列表模式" name='list' lazy>
                    <!-- 头部按钮 -->
                    <div class="btns">
                        <el-radio v-model="searchDate" label="-1" border>全部</el-radio>
                        <el-radio v-model="searchDate" label="1" border>今日</el-radio>
                        <el-radio v-model="searchDate" label="2" border>本周</el-radio>
                        <el-radio v-model="searchDate" label="3" border>下周</el-radio>
                        <el-radio v-model="searchDate" label="4" border>本月</el-radio>
                        <div class='pull-right'>
                            <el-select v-model="searchStaffId" placeholder="所有门店员工" clearable>
                                <el-option v-for='item in staffs' :key=item.id :label="item.staffName" :value="item.id"></el-option>
                            </el-select>
                            <el-input v-model="searchText" placeholder="筛选姓名/手机号" class="handle-input mr10" @keyup.enter.native='getData'></el-input>
                            <el-button type="primary" icon="search" @click="getData" :loading='loading'>搜索</el-button>
                            <el-button @click='handleDownload' :loading='downloadLoading'>导出Excel</el-button>
                        </div>
                    </div>
                    <!-- 表单 -->
                    <el-table :data="bookingData" style="width: 100%" v-loading="loading" element-loading-text="加载中" element-loading-spinner="el-icon-loading">
                        <el-table-column label="顾客">
                            <template slot-scope="scope">
                                <div>
                                    <div class=''>
                                        <div style='max-width: 130px;display:inline-block'>{{scope.row.custName}}</div>
                                    </div>
                                    <div>{{scope.row.phone}}</div>
                                </div>
                            </template>
                        </el-table-column>
                        <el-table-column prop="bookDate" label="日期">
                        </el-table-column>
                        <el-table-column prop="startTime" label="开始时间">
                        </el-table-column>
                        <el-table-column prop="endTime" label="结束时间">
                        </el-table-column>
                        <el-table-column prop="bookService" label="预约服务">
                        </el-table-column>
                        <el-table-column prop="staffName" label="预约员工">
                        </el-table-column>
                        <el-table-column prop="createTime" label="创建时间">
                        </el-table-column>
                        <el-table-column prop="statusName" label="状态">
                        </el-table-column>
                        <el-table-column label="操作" width="150">
                            <template slot-scope="scope">
                                <el-button size="small" @click="handleEdit(scope.$index, scope.row)">查看修改</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                    <div class="pagination">
                        <el-pagination @current-change="handleCurrentChange" layout="total, sizes,prev, pager, next, jumper" :page-size='pageSize' :page-sizes="[5,7, 10, 20, 50]" :total="total" background @size-change="handleSizeChange">
                        </el-pagination>
                    </div>
                </el-tab-pane>
            </el-tabs>
        </div>
        <cust-booking-dialog ref='custBookingDialog' :onSelect='handleBookingData' :in-form='curDealForm' :staffs='staffs'></cust-booking-dialog>
    </div>
</template>
<style>
.book-item {
    cursor: pointer;
}

.booked {
    background-color: #e8f1ff
}

.booked-start {
    border-top: solid 4px #00D1B2;
    border-left: solid 4px #00D1B2;
}

.booked-other {
    border-left: solid 4px #00D1B2;
}

.staff-not-work {
    background-color: #F2F6FC;
    cursor: not-allowed !important;
}

@media (max-height:1199px) {
    .calendar-container .el-table {
        height: 760px !important;
    }
}

@media (max-height:991px) {
    .calendar-container .el-table {
        height: 700px !important;
    }
}

@media (max-height:769px) {
    .calendar-container .el-table {
        height: 580px !important;
    }
}

@media (max-height:667px) {
    .calendar-container .el-table {
        height: 360px !important;
    }
}

</style>
<style scoped lang='less'>
.booking-container {
    padding: 0px 30px 30px 30px;
    background: #fff;
}


.btns {
    margin-top: 10px;
    margin-bottom: 10px;
}

.handle-input {
    width: 300px;
    display: inline-block;
}

.icon-btn {
    height: 30px;
    line-height: 30px;
    border-radius: 3px;
    border: 1px solid #d9d9d9;
    font-weight: 400;
    cursor: pointer
}

.icon-btn:focus,
.icon-btn:hover {
    color: #00D1B2;
    border-color: rgb(179, 241, 232);
    background-color: rgb(230, 250, 247);
}

.icon-btn .fa {
    vertical-align: inherit;
}

</style>
<script>
import dateFunc from '../../../../utils/dateFunc'
import custBookingDialog from './custBookingDialog'

export default {
    name: 'custBooking',
    data: function() {
        return {
            loading: false,
            loaded: false,
            loadErr: false,


            activeName: 'calendar',
            selectedStaffIds: [],
            staffs: [],
            selectedDay: new Date(),

            bookingData: [], //数据库保存

            allBookingData: [
                { time: '09:30', },
                { time: '10:00', },
                { time: '10:30', },
                { time: '11:00', },
                { time: '11:30', },
                { time: '12:00', },
                { time: '12:30', },
                { time: '13:00', },
                { time: '13:30', },
                { time: '14:00', },
                { time: '14:30', },
                { time: '15:00', },
                { time: '15:30', },
                { time: '16:00', },
                { time: '16:30', },
                { time: '17:00', },
                { time: '17:30', },
                { time: '18:00', },
                { time: '18:30', },
                { time: '19:00', },
                { time: '19:30', },
                { time: '20:00', },
                { time: '20:30', },
            ], //界面上展示
            curDealForm: {},

            //list
            total: 0,
            cur_page: 1,
            pageSize: this.$CST.PAGE_SIZE,
            downloadLoading: false,
            searchDate: '4',
            searchText: '',
            searchStaffId: '',

            //排班关联
            shiftInfos: [],
            schedulerData: []
        }
    },
    created() {
        if (screen.height >= 860) {
            this.pageSize = 10;
        } else if (screen.height >= 760) {
            this.pageSize = 8;
        }
        this.originalAllBookingData = JSON.parse(JSON.stringify(this.allBookingData));
        this.queryStaff().then(() => { this.getData(); })
        this.getShiftInfo()
            .then(() => { this.getSchedulerPlan().then(() => { this.updateAllBookingData(); }) })


    },
    components: {
        custBookingDialog,
    },
    computed: {
        selectedStaffs() {
            if (this.selectedStaffIds.length == 0) {
                return this.staffs;
            } else {
                return this.staffs.filter((item) => {
                    if (this.selectedStaffIds.indexOf(item.id) != -1) {
                        return true;
                    } else {
                        return false;
                    }
                })
            }
        }
    },
    watch: {
        selectedDay(newVal) {
            this.getData();
            if (this.activeName == 'calendar') {
                this.getSchedulerPlan().then(() => { this.updateAllBookingData() });
            }
        },
        selectedStaffIds() {
            this.getData();
        },
        searchDate() {
            this.getData();
        },
        searchStaffId() {
            this.getData();
        }
    },
    methods: {
        //查班次时间
        getShiftInfo() {
            return new Promise((resolve, reject) => {
                this.$fetch(this.$CST.API_SHIFTINFO, 'post', {}).then((res) => {
                    this.shiftInfos = res.data;
                    resolve();
                }).catch((reason) => {
                    reject(reason);
                    alert('失败：' + reason);
                }).finally(() => {})
            });
        },

        //查排班表
        getSchedulerPlan() {
            return new Promise((resolve, reject) => {
                this.loading = true;
                let params = {};
                Object.assign(params, {
                    scheduleDate: dateFunc.format(this.selectedDay, 'YYYYMMDD'),
                    orgId: this.$store.getters.orgId
                });
                this.$fetch(this.$CST.API_SCHEDULER, 'post', params).then((res) => {
                    this.dealSchedulerResponse(res.data)
                    this.schedulerData = res.data;
                    resolve();
                }).catch((reason) => {
                    reject(reason);
                    alert('失败：' + reason);
                }).finally(() => {
                    this.loading = false;
                })
            });
        },

        //将班次转成时间
        dealSchedulerResponse(data) {
            data.forEach((item) => {
                item.startTime = '';
                item.endTime = '';
                if (item.scheduler.length > 0) {
                    const shiftInfo = this.shiftInfos.filter((shiftInfo) => {
                        if (shiftInfo.id == item.scheduler[0].shiftId) {
                            return true
                        } else {
                            return false;
                        }
                    })
                    item.startTime = dateFunc.format(shiftInfo[0].startTime, 'HH:mm');
                    item.endTime = dateFunc.format(shiftInfo[0].endTime, 'HH:mm');
                }
            })
        },

        queryStaff() {
            return new Promise((resolve, reject) => {
                this.loading = true;
                this.$fetch(this.$CST.API_ORG_STAFF_REL, 'post', {
                    orgId: this.$store.getters.orgId
                }).then((res) => {
                    this.staffs = res.data;
                    resolve();
                }).catch((reason) => {
                    reject(reason)
                    alert('失败：' + reason);
                    this.loading = false;
                }).finally(() => {})
            });
        },

        trim(s) {
            return s.replace(/(^\s*)|(\s*$)/g, "");
        },

        isPhone(text) {
            let numCount = 0; //数字数量,认定数字大于3为手机号
            for (var key in text) {
                if (text[key] && !isNaN(text[key])) {
                    numCount++
                }
            }
            if (numCount > 3) {
                return true;
            } else {
                return false;
            }
        },

        getData() {
            let params = {};
            this.loading = true;
            if (this.activeName == 'calendar') {
                Object.assign(params, {
                    bookDate: dateFunc.format(this.selectedDay, 'YYYY-MM-DD'),
                    orgId: this.$store.getters.orgId,
                })
            } else {
                //TODO:时间范围
                Object.assign(params, {
                    pageNo: this.cur_page,
                    pageSize: this.pageSize,
                })

                if (this.searchText) {
                    if (this.isPhone(this.searchText)) {
                        Object.assign(params, { 'phone': this.trim(this.searchText) })
                    } else {
                        Object.assign(params, { 'custName': this.trim(this.searchText) })
                    }
                }

                if (this.searchStaffId) {
                    Object.assign(params, { 'staffId': this.searchStaffId })
                }

                if (this.searchDate == '1') {
                    Object.assign(params, {
                        bookDate: dateFunc.format(new Date(), 'YYYY-MM-DD')
                    })
                } else if (this.searchDate == '2') {
                    //一天的毫秒数   
                    const millisecond = 1000 * 60 * 60 * 24;
                    //获取当前时间   
                    const currentDate = new Date();
                    //返回date是一周中的某一天
                    const week = currentDate.getDay();
                    //减去的天数   
                    const minusDay = week != 0 ? week - 1 : 6;
                    //获得当前周的第一天   
                    const currentWeekFirstDay = new Date(currentDate.getTime() - (millisecond * minusDay));
                    //获得当前周的最后一天
                    const currentWeekLastDay = new Date(currentWeekFirstDay.getTime() + (millisecond * 6));
                    Object.assign(params, {
                        bookDateFrom: dateFunc.format(currentWeekFirstDay, 'YYYY-MM-DD'),
                        bookDateTo: dateFunc.format(currentWeekLastDay, 'YYYY-MM-DD'),
                    })
                } else if (this.searchDate == '3') {
                    //起止日期数组   
                    let startStop = new Array();
                    //一天的毫秒数   
                    let millisecond = 1000 * 60 * 60 * 24;
                    //获取当前时间   
                    let currentDate = new Date();
                    //相对于当前日期AddWeekCount个周的日期
                    currentDate = new Date(currentDate.getTime() + (millisecond * 7 * 1));
                    //返回date是一周中的某一天
                    let week = currentDate.getDay();
                    //返回date是一个月中的某一天   
                    let month = currentDate.getDate();
                    //减去的天数   
                    let minusDay = week != 0 ? week - 1 : 6;
                    //获得当前周的第一天   
                    let currentWeekFirstDay = new Date(currentDate.getTime() - (millisecond * minusDay));
                    //获得当前周的最后一天
                    let currentWeekLastDay = new Date(currentWeekFirstDay.getTime() + (millisecond * 6));
                    Object.assign(params, {
                        bookDateFrom: dateFunc.format(currentWeekFirstDay, 'YYYY-MM-DD'),
                        bookDateTo: dateFunc.format(currentWeekLastDay, 'YYYY-MM-DD'),
                    })
                } else if (this.searchDate == '4') {

                    const date = new Date(),
                        y = date.getFullYear(),
                        m = date.getMonth();
                    const firstDay = new Date(y, m, 1);
                    const lastDay = new Date(y, m + 1, 0);
                    Object.assign(params, {
                        bookDateFrom: dateFunc.format(firstDay, 'YYYY-MM-DD'),
                        bookDateTo: dateFunc.format(lastDay, 'YYYY-MM-DD'),
                    })
                }
            }

            this.$fetch(this.$CST.API_BOOKING, 'post', params).then((res) => {
                this.total = res.total;

                for (let key in res.data) {
                    res.data[key].bookDate = res.data[key].bookDate.substring(0, 10);
                    res.data[key].startTime = res.data[key].startTime.substring(11, 16);
                    res.data[key].endTime = res.data[key].endTime.substring(11, 16);
                }

                res = res.data;


                if (this.activeName == 'calendar') {
                    this.bookingData = res;
                    this.updateAllBookingData();
                } else {
                    res.forEach((item) => {
                        if (item.status == '1') {
                            Object.assign(item, { statusName: '已预约' });
                        } else {
                            Object.assign(item, { statusName: '已取消' });
                        }
                    })
                    this.bookingData = res;
                }
            }).catch((reason) => {
                alert('失败：' + reason);
            }).finally(() => {
                this.loaded = true;
                this.loading = false;
            })

        },
        //根据bookingData更新
        updateAllBookingData() {

            this.allBookingData = JSON.parse(JSON.stringify(this.originalAllBookingData));

            //处理预约
            this.bookingData.forEach((item) => {
                this.allBookingData.forEach((singleBookItem) => {
                    if (singleBookItem.time >= item.startTime &&
                        singleBookItem.time <= item.endTime) {
                        //找到time,设置staffId:text
                        singleBookItem[item.staffId] = Object.assign({}, item, { text: item.custName + item.bookService })
                        //设置样式
                        if (singleBookItem.time == item.startTime) {
                            singleBookItem[item.staffId].position = 'start';
                        } else {
                            singleBookItem[item.staffId].position = 'other';
                        }
                    }
                })
            })
            //处理排班
            this.schedulerData.forEach((item) => {
                this.allBookingData.forEach((singleBookItem) => {
                    if (singleBookItem.time >= item.startTime &&
                        singleBookItem.time <= item.endTime) {
                        if (!singleBookItem[item.staffId]) {
                            singleBookItem[item.staffId] = {}
                        }
                        singleBookItem[item.staffId].work = true;
                    } else {
                        if (!singleBookItem[item.staffId]) {
                            singleBookItem[item.staffId] = {}
                        }
                        singleBookItem[item.staffId].work = false;
                    }
                })
            })

        },

        // 分页导航
        handleCurrentChange(val) {
            this.cur_page = val;
            this.getData();
        },
        handleSizeChange(val) {
            this.cur_page = 1;
            this.pageSize = val;
            this.getData();
        },


        cellClassName({ row, column, rowIndex, columnIndex }) {
            //第一列不处理
            if (column.label == '时间') {
                return;
            }

            let ret = 'book-item ';
            //预约样式
            if (row[column.property] && row[column.property].position == 'start') {
                ret += "booked booked-start";
            } else if (row[column.property] && row[column.property].position == 'other') {
                ret += "booked booked-other";
            }
            //排班样式
            if (!row[column.property] || !row[column.property].work) {
                ret += ' staff-not-work'
            }

            return ret;
        },
        tabClick(tab, event) {
            this.getData();
        },
        autoBackToday() {
            this.selectedDay = new Date();
        },

        autoBack7() {
            this.selectedDay = dateFunc.getRelativeDay(new Date(), -7);
        },

        autoBack15() {
            this.selectedDay = dateFunc.getRelativeDay(new Date(), -15);
        },

        autoBack30() {
            this.selectedDay = dateFunc.getRelativeDay(new Date(), -30);
        },

        autoPreDay() {
            this.selectedDay = dateFunc.getRelativeDay(this.selectedDay, 1);
        },
        autoNextDay() {
            this.selectedDay = dateFunc.getRelativeDay(this.selectedDay, -1);
        },
        handClickCell(row, column) {
            //第一列不处理
            if (column.label == '时间') {
                return;
            }

            //如果没有排班不处理
            if (!row[column.property].work) {
                return
            }

            if (!row[column.property].startTime) {
                //找staff
                for (let key in this.staffs) {
                    if (this.staffs[key].id == column.property) {
                        row[column.property] = {
                            staffId: this.staffs[key].id,
                            staffName: this.staffs[key].staffName,
                            startTime: row.time,
                            bookDate: dateFunc.format(this.selectedDay, 'YYYY-MM-DD')
                        };
                        break;
                    }
                }
            }
            this.curDealForm = row[column.property]
            this.$refs.custBookingDialog.isShow = true;
        },
        handleEdit(index, row) {
            this.curDealForm = row;
            this.$refs.custBookingDialog.isShow = true;
        },
        handleScheduler() {
            let routeData = this.$router.resolve({
                name: 'schedulerList',
                params: { refresh: true }
            });
            window.open(routeData.href, '_blank');
        },
        handleAdd() {
            this.curDealForm = { bookDate: dateFunc.format(this.selectedDay, 'YYYY-MM-DD') };
            this.$refs.custBookingDialog.isShow = true;
        },

        handleBookingData(item) {
            this.getData();
        },

        getDownLoadData() {
            let params = {};
            //TODO:时间范围
            Object.assign(params, {})
            return new Promise((resolve, reject) => {
                this.$fetch(this.$CST.API_BOOKING, 'post', params).then((res) => {
                    for (let key in res.data) {
                        res.data[key].bookDate = res.data[key].bookDate.substring(0, 10);
                        res.data[key].startTime = res.data[key].startTime.substring(11, 16);
                        res.data[key].endTime = res.data[key].endTime.substring(11, 16);
                    }
                    resolve(res.data);
                }).catch((reason) => {
                    reject(reason)
                    alert('失败：' + reason);
                }).finally(() => {})
            });
        },

        handleDownload() {
            this.downloadLoading = true;
            this.getDownLoadData().then((res) => {
                import('@/utils/Export2Excel').then(excel => {
                    const tHeader = ['身份', '名称', '手机', '预约日期', '开始时间', '结束时间', '预约服务', '员工'];

                    const filterVal = ['identity', 'custName', 'phone', 'bookDate', 'startTime', 'endTime', 'bookService', 'staffName']

                    const list = res;
                    const data = this.formatJson(filterVal, list)

                    excel.export_json_to_excel({
                        header: tHeader,
                        data,
                        filename: dateFunc.format(new Date(), 'YYYY-MM-DD') + '导出预约数据',
                        autoWidth: true,
                        bookType: 'xlsx'
                    })

                })
            }).catch((reason) => {
                alert('失败：' + reason);
            }).finally(() => {
                this.downloadLoading = false
            })
        },

        formatJson(filterVal, jsonData) {
            return jsonData.map(v => filterVal.map(j => {
                if (j === 'identity') {
                    return v[j] == '1' ? '会员预约' : '散客预约'
                } else {
                    return v[j]
                }
            }))
        }
    }
}

</script>
