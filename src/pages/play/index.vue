<template>
  <view class="play-container" v-if="course_detail">
    <!-- 视频 -->
    <view class="cover_image">
      <video @play="onPlay" id="videoId" :src="playingUrl" controls></video>
    </view>
    <!-- 基本信息渲染 -->
    <view class="introduction">
      <view class="title-and-info">
        <text class="title">{{ course_detail.course.title }}</text>
        <text class="info">
          <text>{{ course_detail.course.study_count }}人学过</text>
          <!-- 通过计算属性的方式(推荐) -->
          <!-- <text>难度:{{ level }}</text> -->
          <!-- 通过过滤器的方式 -->
          <text>难度:{{ course_detail.course.level | getLevel }}</text>
          <text>时长:{{ course_detail.course.course_duration }}</text>
        </text>
      </view>
      <!-- 子传父组件 -->
      <Modal
        :visible="isShowModal"
        @close="closeModal"
        @postComment="postComment"
      >
        <view class="comment-content">
          <textarea
            placeholder="请输入要评价的内容哦~"
            rows="5"
            v-model="content"
          ></textarea>
        </view>
        <view style="margin-top:10rpx;">
          <text>评分：</text>
          <view style="float:right;margin-right:300rpx;margin-top:-5rpx;">
            <Star @changeScore="getScore" :readonly="false" :score="score" />
          </view>
        </view>
      </Modal>
      <view class="comment">
        <image @click="evaluate" src="/static/images/evaluate@2x.png" alt />
      </view>
    </view>
    <!-- 课程进度 -->
    <view class="course-progress">
      <view class="title">课程进度</view>
      <view class="catelog-container">
        <view
          @click="playOnVideo(index)"
          v-for="(item, index) in course_detail.videos"
          :key="item.id"
          class="course-item"
        >
          <text :class="[index === playingIndex ? 'active' : '']"
            >{{ index + 1 }}.{{ item.name }}</text
          >
          <text v-if="item.is_study === 1" class="studied">已学习</text>
          <text :class="[index === playingIndex ? 'active' : '']" v-else>{{
            item.duration
          }}</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script lang="ts">
import Vue from 'vue'
import { fetch } from '../../utils/fetch'
// 导入子组件
import Modal from '../../components/Modal.vue'
import Star from '../../components/Star.vue'
export default Vue.extend({
  components: {
    Modal,
    Star
  },
  data () {
    return {
      id: null, //课程id
      course_detail: null, // 课程详情
      playingUrl: null, // 当前正在播放的url
      playingIndex: 0, //当前播放索引
      isValidate: false, // 是否授权
      isShowModal: false, // 是否显示Modal
      content: '', // 评价的内容
      score: 5 // 评价的分数
    }
  },
  onLoad (options) {
    this.id = options.id
    this.getCourseDetailData()
  },
  methods: {
    async getCourseDetailData () {
      const res = await fetch({
        url: `course/play/${this.id}`
      })
      console.log('视频渲染', res)
      if (res.data.status === 0) {
        this.course_detail = res.data.message
        // 给playingUrl赋值
        this.playingUrl = res.data.message.videos[0].video_url
      }
    },
    // 关闭自定义Modal框
    closeModal () {
      this.isShowModal = false
    },
    // 星星交互
    getScore (score) {
      this.score = score
    },
    // 提交
    async postComment () {
      // 判断评价内容是否为空
      if (this.content.trim().length === 0) {
        uni.showToast({
          title: '内容不能为空',
          icon: 'none',
          duration: 500
        })

        return
      }
      const result = await fetch({
        url: 'comment/create',
        method: 'POST',
        data: {
          course_id: this.id,
          content: this.content,
          score: this.score
        }
      })

      if (result.data.status === 0) {
        this.content = ''
        this.score = 5
        this.closeModal()

        uni.showToast({
          title: result.data.message,
          icon: 'none',
          duration: 500
        })
      }
    },
    // 评价
    async evaluate () {
      const res = await fetch({
        url: 'study/complete',
        data: {
          course_id: this.id
        }
      })
      console.log(res)

      if (res.data.status === 0) {
        if (!res.data.complete) {
          uni.showModal({
            title: '提示',
            content: '请先学习完毕，再来评价',
            showCancel: false,
            confirmColor: '#ff9a29'
          })
          return
        }
        // 学习完，弹出我们自定义的评价模态框
        this.isShowModal = true
      }
    },
    onPlay () {
      if (!this.isValidate) {
        this.playOnVideo(0)
      }
    },
    // 播放当前index的视频
    async playOnVideo (index) {
      // 切换选中的索引
      this.playingIndex = index

      // 去鉴权，如果支付了，则正常播放，如果没有，则提示要他跳转支付页面
      const isCanPlay = await this.validatePlayRight()
      if (!isCanPlay) {
        // 提示要去到支付页面
        uni.showModal({
          title: '提示',
          content: '您还没有购,买请先购买后,\n再来播放哦~',
          confirmText: '去购买',
          confirmColor: '#ff9a29',
          success: res => {
            if (res.confirm) {
              uni.navigateTo({
                url: `/pages/pay/index?id=${this.id}&title=${this.course_detail.course.title}
                            &price=${this.course_detail.course.price}&cover_image_url=${this.course_detail.course.cover_image_url}`
              })
            }
          }
        })
        return
      }
      // 有权限之后，播放刚刚点击的那个视频
      const videoObj = this.course_detail.videos[index]
      // 给playingUrl赋值
      this.playingUrl = videoObj.video_url
      setTimeout(() => {
        uni.createVideoContext('videoId').play()
      }, 200)
      // 发请求给后台记录播放了该视频
      const result = await fetch({
        url: 'study/video',
        method: 'POST',
        data: {
          course_id: this.id,
          video_id: videoObj.id
        }
      })
      if (result.data.status === 0) {
        // 更改模型之后，就会更新视图
        videoObj.is_study = 1
      }
    },
    // 发请求，查看当前的课程是否购买过
    async validatePlayRight () {
      const result = await fetch({
        url: 'order/info',
        data: {
          course_id: this.id
        }
      })
      if (result.data.status === 0) {
        if (result.data.message.pay_status === 0) {
          return Promise.resolve(false)
        } else {
          this.isValidate = true

          return Promise.resolve(true)
        }
      } else {
        return Promise.resolve(false)
      }
    }
  },
  // 计算属性  基本信息
  // computed: {
  //   level () {
  //     if (!this.course_detail) return null
  //     switch (this.course_detail.course.level) {
  //       case 1: {
  //         return '初级'
  //         break
  //       }

  //       case 2: {
  //         return '中级'
  //         break
  //       }

  //       case 3: {
  //         return '高级'
  //         break
  //       }

  //       default:
  //         return '无级别'
  //         break
  //     }
  //   }
  // }

  //过滤器
  filters: {
    getLevel (level) {
      switch (level) {
        case 1: {
          return '初级'
          break
        }

        case 2: {
          return '中级'
          break
        }

        case 3: {
          return '高级'
          break
        }

        default:
          return '无级别'
          break
      }
    }
  }
})
</script>

<style lang="less" scoped>
.play-container {
  background-color: #efefef;
  position: relative;
  .cover_image {
    width: 750rpx;
    height: 434rpx;
    display: flex;
    justify-content: center;
    align-items: center;
    image {
      width: 750rpx;
      height: 434rpx;
    }
    video {
      width: 750rpx;
      height: 434rpx;
    }
  }
  .introduction {
    z-index: 3;
    position: absolute;
    margin-top: -10rpx;
    border-radius: 16px 16px 0px 0px;
    border-bottom: 1px solid #f5f5f5;
    left: 0;
    right: 0;
    height: 192rpx;
    background-color: #ffffff;
    padding: 38rpx 32rpx;
    display: flex;
    align-items: center;
    .title-and-info {
      flex: 1;
      .title {
        color: #343434;
        font-size: 24px;
        font-weight: bold;
      }
      .info {
        margin-top: 12rpx;
        height: 34rpx;
        line-height: 34rpx;
        color: #818181;
        font-size: 12px;
        display: block;
        text {
          padding-right: 15rpx;
        }
        text:nth-child(2),
        text:nth-child(3) {
          padding-left: 15rpx;
        }
        text:nth-child(2) {
          position: relative;
          &::before {
            position: absolute;
            content: '';
            background-color: #ebebeb;
            left: 0rpx;
            width: 1px;
            top: 8rpx;
            height: 20rpx;
          }
          &::after {
            position: absolute;
            content: '';
            background-color: #ebebeb;
            right: 0rpx;
            width: 1px;
            top: 8rpx;
            height: 20rpx;
          }
        }
      }
    }
    .comment {
      margin-top: 32rpx;
      image {
        width: 198rpx;
        height: 80rpx;
      }
    }
  }
  .course-progress {
    margin-top: 222rpx;
    padding: 44rpx 32rpx;
    background-color: #fff;
    .title {
      color: #343434;
      font-size: 30rpx;
      font-family: PingFang SC;
      font-weight: 800;
    }
    .catelog-container {
      .course-item {
        display: flex;
        justify-content: space-between;
      }
      text {
        color: #636363;
        font-size: 30rpx;
        height: 80rpx;
        line-height: 80rpx;
      }
      .studied {
        float: right;
        margin-top: 16rpx;
        font-size: 10px;
        text-align: center;
        color: #aaaaaa;
        background-color: #f5f5f5;
        border-radius: 12rpx;
        width: 98rpx;
        height: 48rpx;
        line-height: 48rpx;
      }
      .time {
        float: right;
      }
      .active {
        color: #ff5e00;
      }
    }
  }
}
.comment-content {
  label {
    width: 100rpx;
  }
  textarea {
    width: 100%;
    height: 200rpx;
  }
}
</style>
