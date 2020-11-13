<template>
<div class="mod-config">
    <el-button type="danger" @click="cleanpaiming">清除排名</el-button>
    <br>
    <el-table :data="dataarray" border v-loading="dataListLoading" style="width: 100%;">

        <el-table-column prop="paiming" header-align="center" align="center" label="排名" type="index">

        </el-table-column>
        <el-table-column prop="nickName" header-align="center" align="center" label="用户名">
        </el-table-column>
        <el-table-column prop="pkYet" header-align="center" align="center" label="用户头像">
            <template slot-scope="scope">
                <el-image :src="scope.row.headimg"></el-image>
            </template>
        </el-table-column>
        <el-table-column prop="gongli" header-align="center" align="center" label="公里数">
        </el-table-column>
        <el-table-column prop="kaluli" header-align="center" align="center" label="卡路里">
        </el-table-column>
        <el-table-column prop="mintue" header-align="center" align="center" label="使用分钟">
        </el-table-column>
        <el-table-column prop="mintuegongl" header-align="center" align="center" label="配速">
        </el-table-column>

    </el-table>
    <el-pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" :current-page="pageIndex" :page-size="pageSize" :total="totalPage" layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
</div>
</template>

<script>
import AddOrUpdate from './pkuser-add-or-update'
export default {
    data() {
        return {
            dataForm: {
                key: ''
            },
            dataList: [],
            pageIndex: 1,
            pageSize: 10,
            totalPage: 0,
            dataListLoading: false,
            dataListSelections: [],
            addOrUpdateVisible: false,
            dataarray: []
        }
    },
    components: {
        AddOrUpdate
    },
    activated() {
        this.getData2()
    },
    methods: {
        // 获取数据列表
        getDataList() {
            this.dataListLoading = true
            this.$http({
                url: this.$http.adornUrl('/pank/pkuser/list'),
                method: 'get',
                params: this.$http.adornParams({
                    'page': this.pageIndex,
                    'limit': this.pageSize,
                    'key': this.dataForm.key
                })
            }).then(({
                data
            }) => {
                console.log(data)
                if (data && data.code === 0) {
                    this.dataList = data.page.list
                    this.totalPage = data.page.totalCount
                } else {
                    this.dataList = []
                    this.totalPage = 0
                }
                this.dataListLoading = false
            })
        },
        // 每页数
        sizeChangeHandle(val) {
            this.pageSize = val
            this.pageIndex = 1
            this.getData2()
        },
        // 当前页
        currentChangeHandle(val) {
            this.pageIndex = val
            this.getData2()
        },
        cleanpaiming() {

            this.$confirm('此操作将永久删除该内容, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                console.log(1)
                this.$http({
                    url: this.$http.adornUrl('/pank/pkuser/deletePaiming'),
                    method: 'get',
                    //params: this.http.adornParams({ })
                    // data: this.$http.adornData(data, false)
                }).then(({
                    data
                }) => {
                    this.getData2()
                });
            }).catch(() => {

            });

        },
        // 多选
        // selectionChangeHandle(val) {
        //     this.dataListSelections = val
        // },
        // 新增 / 修改
        // addOrUpdateHandle(id) {
        //     this.addOrUpdateVisible = true
        //     this.$nextTick(() => {
        //         this.$refs.addOrUpdate.init(id)
        //     })
        // },
        // // 删除
        // deleteHandle(id) {
        //     var ids = id ? [id] : this.dataListSelections.map(item => {
        //         return item.id
        //     })
        //     this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        //         confirmButtonText: '确定',
        //         cancelButtonText: '取消',
        //         type: 'warning'
        //     }).then(() => {
        //         this.$http({
        //             url: this.$http.adornUrl('/pank/pkuser/delete'),
        //             method: 'post',
        //             data: this.$http.adornData(ids, false)
        //         }).then(({
        //             data
        //         }) => {
        //             if (data && data.code === 0) {
        //                 this.$message({
        //                     message: '操作成功',
        //                     type: 'success',
        //                     duration: 1500,
        //                     onClose: () => {
        //                         this.getDataList()
        //                     }
        //                 })
        //             } else {
        //                 this.$message.error(data.msg)
        //             }
        //         })
        //     })
        // },
        getData2() {
            this.$http({
                url: this.$http.adornUrl('/pank/pkuser/rankinglist'),
                method: 'get',
                params: this.$http.adornParams({})
            }).then(({
                data
            }) => {
                this.dataarray = []
                console.log(data)
                this.dataList = data.page
                for (var i = 0; i < data.page.length; i++) {
                    this.dataList[i].paiming = i + 1
                    this.dataList[i].gongli = data.page[i].gongli.toFixed(2)
                }
                let falg = this.pageIndex * this.pageSize
                let start = falg - this.pageSize
                console.log(falg)
                this.totalPage = this.dataList.length
                for (var i = 0; i < this.dataList.length; i++) {
                    if (i + 1 < falg && i + 1 > start) {

                        this.dataarray.push(this.dataList[i])
                    }
                }
                console.log(this.dataarray)
            })
        }
    },

}
</script>
