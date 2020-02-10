<template>
  <view class="home-container">
    <!-- 搜索条，渲染 -->
    <search-bar tips="请输入课程名称" />
    <!-- 轮播图 -->
    <swiper autoplay circular indicator-active-color="#fff" :interval="3000">
      <navigator
        :url="'/pages/course-detail/index?id=' + item.course_id"
        v-for="item in swipers"
        :key="item.id"
      >
        <swiper-item>
          <image :src="item.img_url"></image>
        </swiper-item>
      </navigator>
    </swiper>
    <!-- 推荐课程 -->
    <view>
      <view class="tips">
        <text class="tip">推荐课程</text>
        <image
          @click="goToCoursePage"
          src="/static/images/arrow@2x.png"
          alt=""
        />
      </view>
      <scroll-view scroll-x class="course-container">
        <navigator
          :url="'/pages/course-detail/index?id=' + item.relation_id"
          class="course-item"
          v-for="item in courses"
          :key="item.id"
        >
          <image :src="item.icon" alt="" />
        </navigator>
      </scroll-view>
    </view>
    <!-- 热门视频 -->
    <view>
      <view class="tips">
        <text class="tip">热门视频</text>
        <image
          @click="goToCourseDetail(3)"
          src="/static/images/arrow@2x.png"
          alt=""
        />
      </view>
      <view class="hot-video">
        <navigator
          :url="'/pages/course-detail/index?id=' + item.course_id"
          class="video-item"
          v-for="item in videos"
          :key="item.id"
        >
          <image :src="item.cover_photo_url" alt="" />
          <text class="title">{{ item.name }}</text>
          <text class="subtitle">{{ item.view_count }}人已观看</text>
        </navigator>
      </view>
    </view>
  </view>
</template>

<script lang="ts">
import Vue from 'vue'
import { fetch } from '../../utils/fetch.js'
import SearchBar from '../../components/SearchBar.vue'
export default Vue.extend({
  // 注册
  components: {
    // "search-bar":SearchBar
    SearchBar
  },
  data () {
    return {
      swipers: [], // 轮播图
      courses: [], // 推荐课程
      videos: [] // 热门视频
    }
  },
  onLoad () {
    // 获取轮播图数据
    this.getSwipersData()
    // 获取推荐课程数据
    this.getCoursesData()
    // 获取热门视频数据
    this.getVideosData()
  },
  methods: {
    async getSwipersData () {
      const result = await fetch({
        url: 'home/swipers'
      })
      if (result.data.status === 0) {
        this.swipers = result.data.message
      }
    },
    async getCoursesData () {
      const result = await fetch({
        url: 'home/course'
      })
      if (result.data.status === 0) {
        this.courses = result.data.message
      }
    },
    async getVideosData () {
      const result = await fetch({
        url: 'home/video'
      })
      if (result.data.status === 0) {
        this.videos = result.data.message
      }
    },
    // 跳转到课程tab页面
    goToCoursePage () {
      uni.switchTab({
        url: '/pages/course/index'
      })
    },
    // 跳转到课程详情页面
    goToCourseDetail (id) {
      console.log(id)
      uni.navigateTo({
        url: `/pages/course-detail/index?id=${id}`
      })
    }
  }
})
</script>

<style lang="less" scoped>
.home-container {
  padding: 20rpx;
  background-color: #fff;
}
swiper {
  width: 100%;
  height: 342rpx;
  swiper-item {
    width: 100%;
    height: 100%;
  }
  image {
    width: 100%;
    height: 100%;
  }
}
.tips {
  height: 120rpx;
  display: flex;
  align-items: center;
  justify-content: space-between;
  text {
    font-size: 36rpx;
    font-weight: 700;
    color: #212121;
  }
  image {
    width: 48rpx;
    height: 48rpx;
  }
}
.course-container {
  height: 170rpx;
  white-space: nowrap;
  .course-item {
    width: 296rpx;
    height: 168rpx;
    display: inline-block;
    margin-right: 32rpx;
    image {
      width: 100%;
      height: 100%;
    }
  }
}
.hot-video {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  .video-item {
    width: 340prx;
    image {
      width: 340rpx;
      height: 210rpx;
      border-radius: 12rpx;
    }
    .title {
      display: block;
      margin-top: 15rpx;
      color: #262626;
      font-size: 26rpx;
    }
    .subtitle {
      display: block;
      margin-top: 15rpx;
      margin-bottom: 15rpx;
      color: #959595;
      font-size: 18rpx;
    }
  }
}
</style>
