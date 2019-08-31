<template>
  <div class="wrap">
    <!-- 结构一：下拉标语和动画 设置样式h：0；overflow: hidden; -->
    <div class="pullTopShow" :animation="pullDownAnimation" >
      <div class="tip-text">
        <!-- 动画给git图 -->
        <img class="logo" src="../../../static/images/earthsports.gif"/>
        <!-- 随机提示语 -->
        <span>{{ refreshTips }}</span>
      </div>
    </div>
    <!-- 结构二：刷新结果 设置样式h：0；overflow: hidden-->
    <div class="refreshResult" :animation="showAnimation" >
      <div v-if="pullLength !== 0">您更新了{{ pullLength }}条新内容</div>
    </div>
    <!-- 
      结构三：主题内容：
      当开始拖拽后，触发touchmove，结构一的高度随着拖拽逐渐增加；
      拖拽结束后：触发touchend后，开始调用同步的加载数据方法，执行touchend，
      这时“结构一”高度0.1s后缩小为60，
      1s后“结构一”开始了0.5s的隐藏动画，随后执行回调函数，展示刷新数据结果
    -->
    <div class="body"
      @touchstart="touchstart"
      @touchmove="pullDownFresh"
      @touchend="touchend"
      >
      请往下拉
    </div>
  </div>
</template>
<script>
export default {
  data () {
    return {
      // 只有刷新出数据后，才能提示刷新结果
      pullLength: 8,
      refreshTips: '又看见你了',
      pullDownAnimation: {},
      showAnimation: {},
      pullTips: [
        '今天天气不错哦',
        '别问我为什么要努力，因为心中有未来'
      ],
      changeTips: false,
      touchData: {},
      pullJudgeMove: false,
      refreshMark: false,
      timerF: '',
      timerT: '',
      timerS: ''
    }
  },
  methods: {
    touchstart (e) {
      this.changeTips = true
      let touchs = e.touches[0]
      this.touchData.startX = touchs.pageX
      this.touchData.startY = touchs.pageY
    },
    async touchend (e) {
      if (this.refreshEnd || !this.pullJudgeMove) {
        return
      }
      this.refreshMark = true
      this.refreshEnd = !this.refreshEnd
      this.pullJudgeMove = false
      this.timerF = setTimeout(() => {
        // 下拉列表时，如果没有新增数据时，则没有刷新数据提醒
        if (this.pullLength) {
          this.timerT = setTimeout(() => {
            this.generateAnimation({
              d: 300,
              type: 'showAnimation'
            })
            // 打开可以再次拖拽页面的开关
            this.refreshMark = false
            this.refreshEnd = false
            this.clear()
          }, 2000)
        }
        this.generateAnimation({
          h: 0,
          d: 500,
          func: () => {
            const _t = this
            this.timerS = setTimeout(() => {
              if (_t.pullLength) {
                _t.generateAnimation({
                  h: 33,
                  type: 'showAnimation'
                })
              } else {
                // 打开可以再次拖拽页面的开关
                _t.refreshMark = false
                _t.refreshEnd = false
                _t.clear()
              }
            }, 1000)
          }
        })
      }, 1000)
      // 调用加载数据的方法
      // await listrefresh()
      // 因为文章列表刷新结果直接影响下拉页面后，是否要显示刷新结果
      this.generateAnimation({
        h: 60,
        d: 100
      })
    },
    // touchmove监听方法
    pullDownFresh (e) {
      // 判断当本次动画结束后才能触发下次有效拖拽
      if (!this.refreshMark) {
        let touchs = e.touches[0]
        let pageX = touchs.pageX
        let pageY = touchs.pageY
        const disX = pageX - this.touchData.startX
        const disY = pageY - this.touchData.startY
        // 判断拖拽动作是否有效
        const pullJudge = disY > 50 && !this.refreshMark && disY > disX * 3
        if (pullJudge) {
          // 判断只有发生有效拖拽后才能，触发touchend
          this.pullJudgeMove = true
          // 调用随机语录生成方法
          this.getRefreshTips()
          // 下拉动画
          this.generateAnimation({
            h: disY,
            d: 500
          })
        }
      }
    },
    getRefreshTips () {
      // 生成随机语录
      if (this.changeTips) {
        this.changeTips = false
        const tipsLen = this.pullTips.length
        const randomIndex = Math.floor(Math.random(0, 1) * tipsLen)
        this.refreshTips = this.pullTips[randomIndex]
      }
    },
    // 方法名：generateAnimation
    // 生成动画对象的公共方法
    // h 默认值 0，动画消失
    // d 默认值 500ms 动画运行时间
    // l 动画类别 默认 linear
    // type 动画输出对象 默认赋值给下拉动画
    // func 回调函数 不是函数类型 不执行
    // example:
    // this.generateAnimation({
    //   h: disY,
    //   d: 500
    // })
    generateAnimation (obj) {
      const {
        h = 0,
        d = 500,
        l = 'linear',
        type = 'pullDownAnimation',
        func
      } = obj
      let endAnimation = wx.createAnimation({
        d,
        l
      })
      endAnimation.height(h).step()
      this[type] = endAnimation.export()
      if (Object.prototype.toString.call(func) === '[object Function]') {
        func()
      }
    },
    // 清理定时器
    clear () {
      clearTimeout(this.timerF)
      clearTimeout(this.timerS)
      clearTimeout(this.timerT)
    }
  }
}
</script>
<style>
  .wrap{
    width: 100%;
    height: 100%;
  }
  .pullTopShow{
    width: 100%;
    height: 0;
    overflow: hidden;
    text-align: center;
    position: relative;
  }
  .tip-text {
    width: 100%;
    position: absolute;
    left: 50%;
    position:absolute;
    bottom:0px;
    padding:0px;
    margin:0px;
    transform: translateX(-50%);
    z-index: 1;
  }
  .logo{
    width: 260rpx;
    height: 120rpx;
    vertical-align: middle;
    margin-left: -70px;
  }
  span{
    font-size: 32rpx;
    display: inline-block;
    margin-left: -50rpx;
    font-weight: 500;
    color: #2894FF;
  }
  .refreshResult{
    height: 0rpx;
    width: 100%;
    color: #eee;
    background: #2894FF;
    text-align: center;
    line-height: 30px;
    font-size: 28rpx;
    font-weight:500;
    overflow: hidden;
  }
  .body{
    height: 100vh;
    width: 100%;
    font-size: 18px;
    text-align: center;
    color: #333;
    background: #eee;
    padding: 20vh 0 0 0;
  }

</style>
