<!--  -->
<template>
<div class=''>
    <el-button @click="dialogVisible=true">上传</el-button>

    <el-dialog :visible.sync="dialogVisible" width="30%">
        <el-upload class="upload-demo" ref="upload" :file-list="fileList" list-type="picture" action="#" :http-request="uploadfuncation" :on-remove="handleRemove" :multiple="true">
            <el-button size="small" type="primary">选择图片</el-button>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
        </el-upload>
        <span slot="footer" class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="insertAccessory()">确 定</el-button>
        </span>

    </el-dialog>
    <el-carousel :interval="5000" arrow="always" @click.native="disdialog()">
        <el-carousel-item v-for="a in datalist" :key="a.id">
            <el-image :src="a.url"></el-image>
        </el-carousel-item>
    </el-carousel>
    <div class="images" v-viewer="{movable: false}" v-show="falg">
        <img v-for="src in photo" :src="src" :key="src">
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
            fileList: [],
            dialogVisible: false,
            datalist: [],
            falg: false,
            photo: []
        };
    },
    //监听属性 类似于data概念
    computed: {},
    //监控data中的数据变化
    watch: {},
    //方法集合
    methods: {
        disdialog() {
            //this.falg = true
            // this.dialogVisible = true
            this.photo = []
            //this.falg = true
            //console.log(this.fileList)
            this.$nextTick(() => {
                for (var i = 0; i < this.fileList.length; i++) {
                    this.photo.push(this.fileList[i].url)
                }
                //console.log(this.photo)
                this.$forceUpdate()
                const viewer = this.$el.querySelector('.images').$viewer
                viewer.show()
            })

        },
        //删除图片
        handleRemove(file, flieList) {
            console.log(file)
            this.$http({
                url: this.$http.adornUrl("/pank/accessory/deleteAccessory"),
                method: "post",
                data: this.$http.adornData({
                    'id': file.id,
                    'name': file.name,
                    'url': file.url,
                    'urlMin': file.urlMin
                })
            }).then(({
                data
            }) => {
                console.log(flieList)
                this.fileList = flieList
            })
        },
        uploadfuncation(file) {
            //console.log(file)
            let formdata = new FormData()
            formdata.append("file", file.file)
            this.$http({
                url: this.$http.adornUrl(`/pank/accessory/insertAccessory`),
                method: "post",
                data: formdata
            }).then(({
                data
            }) => {
                if (data.code === 0) {
                    data.page.map(item => {
                        this.fileList.push({
                            name: item.name,
                            url: item.url,
                            id: item.id,
                            urlMin: item.urlMin
                        })
                    })

                }
                //console.log(data)
            })

        },
        insertAccessory() {
            console.log(this.fileList)
            let ids = this.fileList.map(item => {
                return item.id
            })
            this.$http({
                url: this.$http.adornUrl('/pank/editImg'),
                method: 'post',
                //params: this.http.adornParams({ })
                data: this.$http.adornData(ids, false)
            }).then(({
                data
            }) => {
                console.log(data)
                this.dialogVisible = false
                this.getlist()
            });
        },
        getlist() {
            this.$http({
                url: this.$http.adornUrl('/pank/getimglist'),
                method: 'get',
                params: this.$http.adornParams({})
            }).then(({
                data
            }) => {
                console.log(data.page[0].accessories)
                this.datalist = data.page[0].accessories
                this.fileList = data.page[0].accessories
            })
        }
    },
    //生命周期 - 创建完成（可以访问当前this实例）
    created() {

    },
    //生命周期 - 挂载完成（可以访问DOM元素）
    mounted() {
        this.getlist()
    },
    beforeCreate() {}, //生命周期 - 创建之前
    beforeMount() {}, //生命周期 - 挂载之前
    beforeUpdate() {}, //生命周期 - 更新之前
    updated() {}, //生命周期 - 更新之后
    beforeDestroy() {}, //生命周期 - 销毁之前
    destroyed() {}, //生命周期 - 销毁完成
    activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
}
</script>
