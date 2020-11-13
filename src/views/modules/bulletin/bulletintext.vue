<!--  -->
<template>
<div class=''>
    <div>
        <el-button @click="insert()">新增</el-button>
        <el-button @click="deleted()">删除</el-button>
        <el-table :data="datalist" border @selection-change="selectionShange">
            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column align="center" label="序号" type="index"></el-table-column>
            <el-table-column align="center" label="公告" prop="mytext"></el-table-column>
            <el-table-column align="center" label="修改" prop="mytext">
                <template slot-scope="scope">
                    <el-button type="text" @click="edit(scope.row.id)">修改</el-button>
                </template>
            </el-table-column>
        </el-table>
    </div>
</div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
    //import引入的组件需要注入到对象中才能使用
    components: {},
    data() {
        //这里存放数据
        return {
            datalist: [],
            selectList: []
        };
    },
    //监听属性 类似于data概念
    computed: {},
    //监控data中的数据变化
    watch: {},
    //方法集合
    methods: {
        insert() {
            this.$prompt('请输入邮箱', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                inputPattern: /\S/,
                inputErrorMessage: '邮箱格式不正确'
            }).then(({
                value
            }) => {
                this.$http({
                    url: this.$http.adornUrl('/pank/insert'),
                    method: 'post',
                    //params: this.http.adornParams({ })
                    data: this.$http.adornData({
                        "mytext": value
                    })
                }).then(({
                    data
                }) => {

                    this.getlist()
                });
            })
        },
        getlist() {
            this.$http({
                url: this.$http.adornUrl('/pank/getlist'),
                method: 'get',
                params: this.$http.adornParams({})
            }).then(({
                data
            }) => {
                console.log(data)
                this.datalist = data.page
            })
        },
        selectionShange(val) {
            this.selectList = val
        },
        deleted() {
            let ids = this.selectList.map(item => {
                return item.id
            })
            this.$http({
                url: this.$http.adornUrl('/pank/deleted'),
                method: 'post',
                //params: this.http.adornParams({ })
                data: this.$http.adornData(ids, false)
            }).then(({
                data
            }) => {
                this.getlist()
            });
        },
        edit(id) {
            this.$prompt('请输入公告', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                inputPattern: /\S/,
                inputErrorMessage: '公告不能为空'
            }).then(({
                value
            }) => {
                this.$http({
                    url: this.$http.adornUrl('/pank/edit'),
                    method: 'post',
                    // params: this.http.adornParams({})
                    data: this.$http.adornData({
                        "id": id,
                        "mytext": value
                    })
                }).then(({
                    data
                }) => {

                    this.getlist()
                });
            })
        }
    },

    activated() {
        this.getlist()
    }, //如果页面有keep-alive缓存功能，这个函数会触发
}
</script>
