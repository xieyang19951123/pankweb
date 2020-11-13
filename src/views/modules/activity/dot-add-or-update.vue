<!--  -->
<template>
<div class=''>

    <el-dialog :visible.sync="dialogVisible" :close-on-click-modal="false" :title="!dataForm.id ?'新增':'修改'" width="40%" :close="disloagclose">
        <el-form :model="dataForm" :rules="dataRule" label-width="80px" size="small" ref="dataForm">
            <el-form-item prop="dName" label="点位名称">
                <el-input v-model="dataForm.dName"></el-input>
            </el-form-item>
            <el-form-item prop="dDescribe" label="点位介绍">
                <el-input v-model="dataForm.dDescribe" type="textarea"></el-input>
            </el-form-item>
            <el-form-item label="点位图片">
                <el-upload class="upload-demo" ref="upload" :file-list="fileList" list-type="picture" action="#" :http-request="uploadfuncation" :on-preview="handlePreview" :on-remove="handleRemove" :multiple="true">
                    <el-button size="small" type="primary">选择图片</el-button>
                    <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                </el-upload>
            </el-form-item>
            <el-form-item label="经纬度" prop="longitude">
                <el-col :span="9">
                    <el-input placeholder="请输入经纬度" v-model="dataForm.longitude"></el-input>
                </el-col>
                <el-col :span="3" :offset="1">
                    <el-link href="http://api.map.baidu.com/lbsapi/getpoint/?qq-pf-to=pcqq.c2c" type="primary" target="_blank">选点</el-link>
                </el-col>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="insertdot">确 定</el-button>
        </span>
    </el-dialog>
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
            dialogVisible: false,
            dataForm: {
                id: 0,
                dName: "",
                dDescribe: "",
                longitude: "",
                //aId: undefined,
            },
            fileList: [

            ],

            dataRule: {
                dName: [{
                    required: true,
                    message: '用户名不能为空',
                    trigger: 'blur'
                }],
                dDescribe: [{
                    required: true,
                    message: '用户名不能为空',
                    trigger: 'blur'
                }],
                longitude: [{
                    required: true,
                    message: '用户名不能为空',
                    trigger: 'blur'
                }]
            }
        };
    },

    //监听属性 类似于data概念
    computed: {},
    //监控data中的数据变化
    watch: {},
    //方法集合
    methods: {
        disloagclose() {
            this.$refs['dataForm'].resetFields()
            this.$refs['upload'].clearFiles()
        },
        init(val) {
            console.log(val)
            this.dialogVisible = true
            this.dataForm.id = val || 0
            this.fileList = []
            console.log(this.dataForm)
            this.$nextTick(() => {
                this.$refs['dataForm'].resetFields()
                this.$refs['upload'].clearFiles()
                if (this.dataForm.id) {
                    this.$refs['dataForm'].resetFields()
                    this.$refs['upload'].clearFiles()
                    //console.log(this.dataForm)
                    this.$http({
                        url: this.$http.adornUrl(`/pank/dot/info/${this.dataForm.id}`),
                        method: 'get',
                        params: this.$http.adornParams({})
                    }).then(({
                        data
                    }) => {
                        //console.log(data)
                        if (data && data.code == 0 && val != 0) {
                            this.dataForm.id = data.page.id
                            this.dataForm.dName = data.page.dname
                            this.dataForm.dDescribe = data.page.ddescribe
                            this.dataForm.longitude = data.page.longitude
                            this.fileList = data.page.accessories
                        }
                    })
                }
            })

        },
        handlePreview(file, flieList) {
            console.log(this.fileList)
        },
        //删除图片
        handleRemove(file, flieList) {
            //console.log(file)
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
                console.log(data)
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
        insertdot() {
            this.$refs['dataForm'].validate((vali) => {
                if (vali) {
                    let ids = this.fileList.map(item => {
                        return item.id
                    })
                    this.$http({
                        url: this.$http.adornUrl(`/pank/dot/${!this.dataForm.id ?'insetrDot':'editDot'}`),
                        method: "post",
                        data: this.$http.adornData({
                            "id": this.dataForm.id ? this.dataForm.id : undefined,
                            "dname": this.dataForm.dName,
                            "ddescribe": this.dataForm.dDescribe,
                            "aids": ids.length > 0 ? ids : undefined,
                            "longitude": this.dataForm.longitude
                        })
                    }).then(({
                        data
                    }) => {
                        //console.log(data)
                        if (data.code === 0) {
                            this.$refs['dataForm'].resetFields()
                            this.$refs['upload'].clearFiles()
                            this.flieList = []
                            this.dialogVisible = false
                            this.$emit('refreshDataList')
                        } else {
                            this.$message(data.msg)
                        }
                    })
                } else {
                    return false;
                }
            })
        },
        //查询所有的点位

    },
    //生命周期 - 创建完成（可以访问当前this实例）
    created() {

    },
    //生命周期 - 挂载完成（可以访问DOM元素）
    mounted() {

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
