<template>
    <course-list :courses="courses"/>
</template>

<script lang="ts">
import Vue from 'vue'
import {fetch} from '../../utils/fetch'
import CourseList from '../../components/CourseList.vue'
export default Vue.extend({
    components: {
        CourseList
    },
    data() {
        return {
            courses: [] // 课程列表数组
        }
    },
    onLoad() {
        this.getCoursesData()
    },
    methods: {
        async getCoursesData() {
            const result = await fetch({ url: 'course/list' })

            if (result.data.status === 0) {
                this.courses = result.data.message
            }
        }
    }
})
</script>

<style >
	.content {
		text-align: center;
		height: 400upx;
	}
    .logo{
        height: 200upx;
        width: 200upx;
        margin-top: 200upx;
    }
	.title {
		font-size: 36upx;
		color: #8f8f94;
	}
</style>
