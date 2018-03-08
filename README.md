# vue-lazyload
vue组件,用于懒加载
## 还是先上源码
```
<template lang="pug">
  div.lazy-load
    ul(v-if="showAnimation===true")
      li.loading-circle
      li.loading-circle
      li.loading-circle
    span(v-else) 没有更多了...
</template>

<script type="text/javascript">
export default {
  name: 'LazyLoad',
  props: {
    isLoading: Boolean,   // 来自父组件的信息是否到底了
    isWait: Boolean       // 来自父组件的信息是否等待加载
  },
  data () {
    return {
      showAnimation: true,
      scrollTops: '',
      scrollHeights: ''
    }
  },
  computed: {
    clientHeights () {
      return document.documentElement.clientHeight
    }
  },
  mounted () {
    this.$nextTick(() => {
      window.addEventListener('scroll', this.eventListeners)
    })
  },
  destroyed () {
    window.removeEventListener('scroll', this.eventListeners)
  },
  methods: {
    pageScroll () {
      this.scrollTops = document.documentElement.scrollTop || document.body.scrollTop
      this.scrollHeights = document.documentElement.scrollHeight
      let goBase = this.scrollHeights - this.scrollTops - 200
      if (goBase < this.clientHeights && this.isLoading) {
        this.showAnimation = true
        if (this.isWait) {
          this.$emit('updata-more')   //触发父组件到底了进行提取数据(子组件向父组件传参)
        }
      } else {
        this.showAnimation = false
      }
    },
    eventListeners () {
      this.timerDebounce(this.pageScroll)
    },
    // 利用基本的函数去抖
    timerDebounce (cont) {
      clearTimeout(cont.tId)
      cont.tId = setTimeout(() => {
        this.pageScroll()
      }, 200)
    }
  }
}
</script>

<style lang="scss" scoped>
.lazy-load {
  width: 100%;
  height: 3em;
  line-height: 3;
  text-align: center;
  ul {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 4em;
    height: 100%;
    margin: 0 auto;
    li.loading-circle {
      width: 0.8em;
      height: 0.8em;
      border-radius: 50%;
      animation: loading-circle linear 1s infinite;
      background: $font2;
    }
    .loading-circle:nth-child(1) {
      animation-delay: 0s;
    }
    .loading-circle:nth-child(2) {
      animation-delay: 0.2s;
    }
    .loading-circle:nth-child(3) {
      animation-delay: 0.4s;
    }
    @keyframes loading-circle {
      0%,60%,100% { transform: scale(1); }
      30% { transform: scale(1.5); }
    }
  }
  span {
    color: $font2;
  }
}
</style>

```
## 使用方法
```
html
  LazyLoad(:is-loading="isUpdata", :is-wait="isWait", @updata-more="loadArticle")   //使用组件

<script type="text/javascript">
import LazyLoad from '@/components/content/common/LazyLoad'
import axios from 'axios'   //例子使用了axios

export default {
  name: 'TopLife',
  components: {
    LazyLoad    //注册组件
  },
  data () {
    return {
      lifeLists_data: [],   //存放以从服务器添加的数据
      dataListNum: 1,     //这里只是个简单的判断方法,记录共加载了多少组信息,用来决定is-loading的值
      isUpdata: true,   //如果为true则是提醒子组件这里还有数据,下次落地后继续加载数据
      isWait: true    //当消息没加载完毕时告诉子组件等着,不要连续触发
    }
  },
  computed: {
    lifeListLength () {
      return Math.trunc(this.lifeLists_data.length / 3)
    }
  },
  created () {
    axios.get('http://localhost:8080/static/data1.json')
      .then(response => {
        this.lifeLists_data = response.data
      })
      .catch(error => {
        console.log(error)
      })
  },
  methods: {
    loadArticle () {
      this.isWait = false
      this.dataListNum++
      axios.get(`http://localhost:8080/static/data${this.dataListNum}.json`)
        .then(response => {
          this.lifeLists_data.push(...response.data)
          if (this.lifeListLength < 4) {
            this.isWait = true
          } else {
            this.isUpdata = false
          }
        })
        .catch(error => {
          console.log(error)
        })
    }
  }
}
</script>
```
