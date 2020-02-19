<template>
  <!-- 背景 -->
  <view @click="closeModal" class="modal-mask" v-show="visible">
    <!-- 内容 -->
    <view @click.stop="noImplement" class="modal-content">
      <view class="modal-title" v-if="title">{{ title }} </view>
      <view class="modal-body">
        <slot />
      </view>
      <view @click.stop="postComment" class="modal-action">{{
        confirmText
      }}</view>
    </view>
  </view>
</template>
<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  props: {
    visible: Boolean,
    title: String,
    confirmText: {
      type: String,
      default: '提交'
    }
  },
  methods: {
    closeModal () {
      this.$emit('close')
    },
    // 阻止事件冒泡
    noImplement () {
      // console.log('--noImplement---')
    },
    // 点击了提交按钮
    postComment () {
      this.$emit('postComment')
    }
  }
})
</script>

<style lang="less" scoped>
.modal-mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  z-index: 10;
  transition: all 0.2s ease-in-out;
}
.modal-content {
  margin: 300rpx auto 0;
  width: 600rpx;
  border-radius: 7px;
  padding: 15px 15px 10px 15px;
  // position:relative;
  background-color: #fff;
  border: 1px solid #e0e0e0;
  background-clip: padding-box;
  overflow: hidden;
}
.modal-title {
  padding: 6px 15px 15px;
  margin: 0;
  font-size: 18px;
  line-height: 1;
  color: #1c2438;
  text-align: center;
}
.modal-body {
  margin-bottom: 50rpx;
  font-size: 14px;
  color: #80848f;
  height: 100%;
  line-height: 1.5;
  overflow: auto;
}
.modal-action {
  border-top: 1px solid #e0e0e0;
  margin-top: 30rpx;
  padding-top: 10rpx;
  height: 80rpx;
  line-height: 80rpx;
  color: #ff994d;
  font-size: 16px;
  text-align: center;
}
</style>
