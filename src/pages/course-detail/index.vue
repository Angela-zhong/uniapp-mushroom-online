<template>
    <view>
        <text>课程详情---id is {{id}}</text>
    </view>
</template>

<script lang="ts">
    import Vue from 'vue'
    export default Vue.extend({
        data() {
            return {
                id: null
            }
        },
        onLoad(options){
            this.id = options.id
        }
    })
</script>