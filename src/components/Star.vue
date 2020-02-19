<template>
  <view class="container">
    <!-- 透明的star -->
    <view class="star-bg">
      <image
        @click="star(index)"
        v-for="(item, index) in 5"
        :key="item"
        src="/static/images/star_empty@2x.png"
        alt
      />
    </view>
    <!-- 实心的star -->
    <view class="star">
      <image
        @click="star(index)"
        v-for="(item, index) in myScore"
        :key="item"
        src="/static/images/star_fill@2x.png"
        alt
      />
    </view>
  </view>
</template>

<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  data () {
    return {
      // 赋值修改组件
      myScore: 0
    }
  },
  // 组件传值
  props: {
    score: {
      type: Number,
      default: 5
    },
    readonly: {
      type: Boolean,
      default: true
    }
  },
  created () {
    //   修改data中的组件赋值
    this.myScore = this.score
  },
  watch: {
    score: function () {
      this.myScore = this.score
    }
  },
  methods: {
    star (index) {
      if (this.readonly) return
      // 把我们 index + 1 之后传递给父组件
      this.$emit('changeScore', index + 1)
    }
  }
})
</script>

<style lang="less">
.container {
  position: relative;
}
.star,
.star-bg {
  float: left;
  display: flex;
  height: 50rpx;
  align-items: center;
  image {
    width: 38rpx;
    height: 36rpx;
  }
}
.star {
  position: absolute;
  top: 0;
  left: 0;
}
</style>
