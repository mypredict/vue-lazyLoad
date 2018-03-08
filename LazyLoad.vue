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
    isLoading: Boolean,
    isWait: Boolean
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
          this.$emit('updata-more')
        }
      } else {
        this.showAnimation = false
      }
    },
    eventListeners () {
      this.timerDebounce(this.pageScroll)
    },
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
