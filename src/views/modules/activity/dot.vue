<!--  -->
<template>
<div class=''>
    <div>
        <el-button type="primary" @click="adddot(0)">新增</el-button>
        <el-button type="danger" @click="deleted()">删除</el-button>
    </div>
    <el-table :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeHandle">
        <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
        <el-table-column label="序号" prop="dname" align="center" type="index"></el-table-column>
        <el-table-column label="点名" prop="dname" align="center"></el-table-column>
        <el-table-column label="描述" prop="ddescribe" align="center"></el-table-column>

        <el-table-column label="经纬度" prop="longitude" align="center"></el-table-column>
        <el-table-column label="标识" prop="dtag" align="center"></el-table-column>
        <el-table-column label="操作" prop="dname" align="center">
            <template slot-scope="scope">
                <el-button type="text" @click="adddot(scope.row.id)">修改</el-button>
                <el-button type="text" @click="disdialog(scope.row)">查看图片</el-button>
            </template>
        </el-table-column>
    </el-table>
    <el-pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" :current-page="pageIndex" :page-sizes="[10, 20, 50, 100]" :page-size="pageSize" :total="totalPage" layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <add-or-update v-if="updatefalg" ref="dotaddupdate" @refreshDataList="getDataList"></add-or-update>
    <el-dialog title="提示" :visible.sync="dialogVisible">
        <el-carousel>
            <el-carousel-item v-for="item in accessories" :key="item.id">

                <el-image :src="item.url"></el-image>
            </el-carousel-item>
        </el-carousel>
    </el-dialog>
    <div class="images" v-viewer="{movable: false}" v-show="falg">
        <img v-for="src in photo" :src="src" :key="src">
    </div>
</div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';
import AddOrUpdate from './dot-add-or-update'
export default {
    //import引入的组件需要注入到对象中才能使用
    components: {
        AddOrUpdate
    },
    data() {
        //这里存放数据
        return {
            accessories: [],
            dialogVisible: false,
            dataListLoading: false,
            updatefalg: false,
            dataList: [],
            pageIndex: 1,
            pageSize: 10,
            totalPage: 0,
            dataListSelections: [],
            srcList: [],
            photo: [],
            falg: false
        };
    },
    //监听属性 类似于data概念
    computed: {},
    //监控data中的数据变化
    watch: {},
    //方法集合
    methods: {
        disdialog(row) {
            //this.falg = true
            // this.dialogVisible = true
            this.photo = []
            //this.falg = true

            this.$nextTick(() => {
                for (var i = 0; i < row.accessories.length; i++) {
                    this.photo.push(row.accessories[i].url)
                }

                this.$forceUpdate()
                const viewer = this.$el.querySelector('.images').$viewer
                viewer.show()
            })

        },
        //生成二维码
        qcode(id) {

        },
        //新增
        adddot(val) {
            this.updatefalg = true
            this.$nextTick(() => {
                this.$refs.dotaddupdate.init(val)
            })
        },
        // 多选
        selectionChangeHandle(val) {
            this.dataListSelections = val
        },
        getDataList() {
            this.dataListLoading = true
            this.$http({
                url: this.$http.adornUrl("/pank/dot/getlist"),
                method: "get",
                params: this.$http.adornParams({
                    'page': this.pageIndex,
                    'limit': this.pageSize,
                })
            }).then(({
                data
            }) => {
                console.log(data)
                this.dataListLoading = false
                if (data.code == 0) {
                    this.dataList = data.page.list
                    this.totalPage = data.page.totalCount
                }
            })
        },
        deleted() {
            let ids = this.dataListSelections.map(item => {
                return item.id
            })
            this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                this.$http({
                    url: this.$http.adornUrl('/pank/dot/deleted'),
                    method: 'post',
                    //params: this.http.adornParams({ })
                    data: this.$http.adornData(ids, false)
                }).then(({
                    data
                }) => {
                    this.getDataList()
                });
            })

        },
        // 每页数
        sizeChangeHandle(val) {
            this.pageSize = val
            this.pageIndex = 1
            this.getDataList()
        },
        // 当前页
        currentChangeHandle(val) {
            this.pageIndex = val
            this.getDataList()
        },
        // 多选
        selectionChangeHandle(val) {
            this.dataListSelections = val
        },
    },

    activated() {
        this.getDataList()
    }, //如果页面有keep-alive缓存功能，这个函数会触发
}
</script>
