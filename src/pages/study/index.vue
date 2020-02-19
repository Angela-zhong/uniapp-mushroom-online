<template>
  <view class="study-container">
    <view class="study-item" v-for="item in studyProgress" :key="item.sid">
      <image :src="item.icon" alt="" />
      <view class="meta">
        <text class="title">{{ item.title }}</text>
        <text class="progress"
          >已学习{{ item.study_hour }}课时/{{ item.total_hour || 0 }}课时</text
        >
      </view>
      <view class="circle">
        <circle
          ref="circleRef"
          :canvasId="item.sid"
          :progress="item.study_progress"
          :width="55"
          :height="55"
        ></circle>
      </view>
    </view>
    <view v-if="isEmpty">
      <text class="no-study-tip">您还没有任何学习记录哦，赶快去学习吧~</text>
    </view>
  </view>
</template>

<script lang="ts">
import Vue from 'vue'
import { fetch } from '../../utils/fetch'
import circle from '../../components/Circle.vue'
export default Vue.extend({
  components: {
    circle
  },
  data () {
    return {
      isEmpty: false, //是否为空
      studyProgress: [] // 学习进度
    }
  },
  onShow () {
    // 获取学习进度数据
    this.getStudyProgressData()
  },
  methods: {
    async getStudyProgressData () {
      const result = await fetch({ url: 'study/progress' })
      console.log(result)
      if (result.data.status === 0) {
        // 在Vue中，我们的视图更新是异步
        this.studyProgress = result.data.message
        this.isEmpty = result.data.message.length === 0
        Vue.nextTick(() => {
          this.$refs.circleRef.forEach(item => {
            console.log('------------nextTick-------------')
            item.drawCircle()
          })
        })
      }
    }
  }
})
</script>

<style lang="less" scoped>
.study-container {
  display: flex;
  flex-direction: column;
  padding-bottom: 32rpx;
  align-items: center;
  position: relative;
}
.study-item {
  margin-top: 32rpx;
  background-color: #fff;
  border-radius: 12rpx;
  width: 686rpx;
  height: 220rpx;
  display: flex;
  align-items: center;
  image {
    width: 108rpx;
    height: 108rpx;
    margin-left: 50rpx;
  }
  .meta {
    flex: 1;
    margin-left: 50rpx;
    .title {
      display: block;
      font-size: 30rpx;
      color: #333333;
    }
    .progress {
      display: block;
      margin-top: 10rpx;
      font-size: 22rpx;
      color: #a8a8a8;
    }
  }
  .circle {
    width: 109rpx;
    height: 109rpx;
    margin-right: 56rpx;
  }
}
.no-study-tip {
  margin-top: 500rpx;
  font-size: 15px;
}
</style>
