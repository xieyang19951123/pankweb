<template>
<div>
    <viewer :options="options" :images="images" @inited="inited" class="viewer">
        <template>
            <img v-for="src in images" :src="src" :key="src" class="images" />
        </template>
    </viewer>
</div>
</template>

<script>
import 'viewerjs/dist/viewer.css'
import Viewer from 'v-viewer/src/component.vue'

export default {
    components: {
        Viewer
    },
    props: {
        images: {
            type: Array,
            default: function () {
                return []
            }
        }
    },
    data() {
        return {
            options: {
                'navbar': false
            }
        }
    },
    created() {
        // console.log(this.images)
    },
    methods: {
        // 这个初始化会在页面初始的时候就执行一次，后续每次传入图片也会自动执行，所有用来转发事件
        inited(viewer) {
            this.$viewer = viewer
            this.$emit('getViewer', viewer)
        }
    }
}
</script>

<style scoped>
.images {
    width: 100px;
    height: 100px;
    margin: 10px;
}
</style>
