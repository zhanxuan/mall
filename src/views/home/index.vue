<template>
  <van-pull-refresh v-model="refreshing" @refresh="onRefresh">
    <div class="home">

      <!-- <Header /> -->
      <Swiper :banner="banner" />
      <Category :cate-list="cateList" />
      <Session :session-list="sessionList" />
      <Goods
        v-model="loading"
        :goods-list="list"
        :is-finished="finished"
        @onReachBottom="onReachBottom"
      />
      <back-top />
      <Skeleton v-if="isSkeletonShow" />

    </div>
  </van-pull-refresh>
</template>

<script>
import { getBanner, getCategory, getSession, getList } from '@/api/home'
import Header from './modules/Header'
import Swiper from './modules/Swiper'
import Category from './modules/Category'
import Session from './modules/Session'
import Goods from './modules/Goods'
import Skeleton from './modules/Skeleton'

export default {
  name: 'Home',
  components: {
    Header,
    Swiper,
    Category,
    Session,
    Goods,
    Skeleton
  },
  data() {
    return {
      banner: [],
      cateList: [],
      sessionList: [],
      list: [],
      pageNo: 0,
      pageSize: 4,
      loading: false,
      finished: false,
      refreshing: false,
      isSkeletonShow: true
    }
  },
  mounted() {
    Promise.all([this.getBanner(), this.getCategory(), this.getSession()]).then(
      () => {
        this.isSkeletonShow = false
      }
    )
  },
  methods: {
    // banner
    getBanner() {
      return new Promise((resolve) => {
        getBanner().then((res) => {
          this.banner = res.entry
          resolve()
        })
      })
    },
    // category
    getCategory() {
      return new Promise((resolve) => {
        getCategory().then((res) => {
          const data = res.entry
          this.cateList = data
          resolve()
        })
      })
    },
    // session
    getSession() {
      return new Promise((resolve) => {
        getSession().then((res) => {
          this.sessionList = res.entry
          resolve()
        })
      })
    },
    // goods-list
    getGoodsList() {
      getList({
        pageSize: this.pageSize,
        pageNo: this.pageNo
      }).then((res) => {
        const data = res.entry
        if (this.refreshing) {
          this.list = data
          this.refreshing = false
          this.finished = false
        } else {
          this.list = [...this.list, ...data]
          if (data.length < this.pageSize) this.finished = true
        }
        this.loading = false
      })
    },
    // reach-bottom
    onReachBottom() {
      if (!this.finished) {
        this.loading = true
        this.pageNo += 1
        this.getGoodsList()
      }
    },

    // pull-refresh
    onRefresh() {
      if (!this.loading) {
        this.refreshing = true
        this.pageNo = 1
        this.getBanner()
        this.getCategory()
        this.getSession()
        this.getGoodsList()
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.home {
  background: #f5f5f5;
  min-height: 100vh;
  .logo {
    display: block;
    width: 200px;
    margin: 0 auto;
    padding-top: 40px;
  }
  .menu__right {
    width: 50px;
  }
}
</style>
