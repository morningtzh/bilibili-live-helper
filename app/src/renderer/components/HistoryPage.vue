<template>
  <div class="history-page">
    <div class="history-danmaku-container" ref="container"
      @mouseenter="enterDanmakuArea"
      @mouseleave="leaveDanmakuArea"
      @mousewheel="scrollDanmakuArea">
      <div class="history-danmaku-scrollbar">
        <div class="history-danmaku-cursor" ref="cursor"></div>
      </div>
      <div class="history-danmaku-list" ref="list">
        <div class="history-danmaku-box" v-for="(danmaku, index) in danmakuList">
          <div v-if="danmaku.type == 'connected'" class="msg-connected">弹幕服务器连接成功...</div>
          <div v-else-if="danmaku.type == 'error'" class="msg-error">连接发生错误，3秒后自动重连...</div>
          <div v-else-if="danmaku.type == 'live'" class="msg-live">开始直播啦！</div>
          <div v-else-if="danmaku.type == 'preparing'" class="msg-preparing">直播已结束，下次再见！</div>
          <div v-else-if="danmaku.type == 'welcome'" class="msg-welcome">
            <span v-if="danmaku.user.isVIP" class="vip-user">爷</span>
            <span v-if="danmaku.user.isSVIP" class="svip-user">爷</span>
            <span class="user-name">{{ danmaku.user.name }}</span>
            <span class="welcome-message">进入直播间</span>
          </div>
          <div v-else-if="danmaku.type == 'welcomeGuard'" class="msg-welcome-guard">
            <span v-if="danmaku.user.guard == 1" class="guard-user" :class="userGuardLevel(danmaku.user.guard)">总督</span>
            <span v-if="danmaku.user.guard == 2" class="guard-user" :class="userGuardLevel(danmaku.user.guard)">提督</span>
            <span v-if="danmaku.user.guard == 3" class="guard-user" :class="userGuardLevel(danmaku.user.guard)">舰长</span>
            <span class="user-name">{{ danmaku.user.name }}</span>
            <span class="welcome-message">进入直播间</span>
          </div>
          <div v-else-if="danmaku.type == 'comment'" class="msg-comment"
            @mouseenter="showDanmakuBtns(index)"
            @mouseleave="hiedeDanmakuBtns(index)">
            <span v-if="danmaku.user.isAdmin" class="admin-user">管</span>
            <span v-if="danmaku.user.guard == 1" class="guard-user" :class="userGuardLevel(danmaku.user.guard)">总督</span>
            <span v-if="danmaku.user.guard == 2" class="guard-user" :class="userGuardLevel(danmaku.user.guard)">提督</span>
            <span v-if="danmaku.user.guard == 3" class="guard-user" :class="userGuardLevel(danmaku.user.guard)">舰长</span>
            <span v-if="danmaku.user.isSVIP" class="svip-user">爷</span>
            <span v-else-if="danmaku.user.isVIP" class="vip-user">爷</span>
            <span v-if="danmaku.user.badge" class="user-badge" :class="userBadgeLevelColor(danmaku.user.badge.level)">{{ danmaku.user.badge.title+danmaku.user.badge.level }}</span>
            <span v-if="danmaku.user.level" class="user-level" :class="userLevelColor(danmaku.user.level)">{{ "UL "+danmaku.user.level }}</span>
            <span class="user-name">{{ danmaku.user.name }}:</span>
            <span class="user-comment">{{ danmaku.comment }}</span>
            <div class="block-btn" v-if="danmakuBtnIndex == index && user" @click="blockUser(danmaku.user.id)"><span>禁言</span></div>
          </div>
          <div v-else-if="danmaku.type == 'gift'" class="msg-gift">
            <span class="user-name">{{ danmaku.user.name }}</span>
            <span class="gift-action">赠送</span>
            <span class="gift-img"><img :src="giftImage(danmaku.gift.id)"></span>
            <span class="user-gift">{{ `${danmaku.gift.name} × ${danmaku.gift.count}` }}</span>
          </div>
          <div v-else-if="danmaku.type == 'guardBuy'" class="msg-guard-buy">
            <span class="user-name">{{ danmaku.user.name }}</span>
            <span class="buy-msg">购买</span>
            <span v-if="danmaku.level == 1" class="guard-user" :class="userGuardLevel(danmaku.level)">总督</span>
            <span v-if="danmaku.level == 2" class="guard-user" :class="userGuardLevel(danmaku.level)">提督</span>
            <span v-if="danmaku.level == 3" class="guard-user" :class="userGuardLevel(danmaku.level)">舰长</span>
            <span class="buy-count">{{ `× ${danmaku.count}` }}</span>
          </div>
          <div v-else-if="danmaku.type == 'block'" class="msg-block">
            <span class="user-name">{{ danmaku.user.name }}</span>
            <span class="block-msg">被管理员禁言</span>
          </div>
          <div v-else-if="danmaku.type == 'newFans'" class="msg-new-fans">
            <span class="user-name">{{ danmaku.user.name }}</span>
            <span class="follow-msg">关注了直播间</span>
          </div>
        </div>
      </div>
    </div>
    <div class="send-message-box">
      <input type="text" class="send-message-input"
        :placeholder="messageBoxPlaceholder"
        v-model="messageContent"
        @keyup.enter="sendMessage">
      <img :src="enterIcon" class="icon">
    </div>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        enterIcon: 'imgs/enter.svg',
        messageContent: '',
        win: null,
        inDanmaku: false,
        danmakuBtnIndex: -1
      }
    },
    computed: {
      danmakuList () {
        return this.$root.danmakuPool.filter((danmaku) => {
          return (danmaku.type === 'welcome' || danmaku.type === 'welcomeGuard' || danmaku.type === 'comment' || danmaku.type === 'gift' || danmaku.type === 'guardBuy' || danmaku.type === 'block' || danmaku.type === 'newFans')
        })
      },
      user () {
        return this.$root.userService
      },
      messageBoxPlaceholder () {
        if (this.user) {
          return '请输入弹幕内容 按回车发送'
        } else {
          return '请先登录'
        }
      }
    },
    watch: {
      danmakuList () {
        if (!this.inDanmaku) {
          this.$nextTick(() => {
            this.$refs.list.scrollTop = this.$refs.list.scrollHeight
            this.scrollCursorStyle()
          })
        }
      }
    },
    mounted () {
      this.$nextTick(() => {
        this.$refs.list.scrollTop = this.$refs.list.scrollHeight
      })
    },
    methods: {
      enterDanmakuArea () {
        this.inDanmaku = true
      },
      leaveDanmakuArea () {
        this.inDanmaku = false
      },
      scrollDanmakuArea (e) {
        this.$refs.list.scrollTop += e.deltaY
        this.scrollCursorStyle()
      },
      scrollCursorStyle () {
        let height = this.$refs.container.clientHeight / this.$refs.list.scrollHeight * 100
        let top = (this.$refs.list.scrollTop + this.$refs.list.clientHeight) / this.$refs.list.scrollHeight * 100
        if (height == 100) {
          this.$refs.cursor.style.backgroundColor = 'transparent'
        } else {
          this.$refs.cursor.style.backgroundColor = 'white'
          this.$refs.cursor.style.height = height + '%'
          this.$refs.cursor.style.top = top - height < 0 ? '0%' : (top - height) + '%'
        }
      },
      userLevelColor (level) {
        return "user-level-"+Math.ceil(Number(level)/10)
      },
      userBadgeLevelColor (level) {
        return "user-badge-level-"+Math.ceil(Number(level)/4)
      },
      userGuardLevel (level) {
        return "guard-user-"+level
      },
      giftImage (id) {
        return `http://static.hdslb.com/live-static/live-room/images/gift-section/gift-${id}.png`
      },
      showDanmakuBtns (index) {
        this.danmakuBtnIndex = index
      },
      hiedeDanmakuBtns (index) {
        this.danmakuBtnIndex = -1
      },
      blockUser (uid) {
        if (!this.user) return
        this.user.blockUser(uid, 720)
      },
      sendMessage () {
        if (!this.user) return
        this.user.sendMessage(this.messageContent)
        this.$nextTick(() => {
          this.messageContent = ''
        })
      }
    }
  }
</script>

<style lang="stylus">
.history-danmaku-container
  position absolute
  left 0
  top 0
  width 100%
  border-radius 5px
  overflow hidden
  height calc(100% - 33px)
  background-color rgba(25,25,25,.8)

.history-danmaku-scrollbar
  position absolute
  top 0
  right 0
  width 4px
  height 100%
  background-color rgba(0,0,0,.6)
  .history-danmaku-cursor
    position absolute
    left 0
    width 4px
    background-color white
    z-index 99

.history-danmaku-list
  width 100%
  height 100%
  overflow hidden
  background-color transparent

.history-danmaku-box
  padding 4px 8px
  font-size 14px
  line-height 20px
  user-select none
  cursor default
  color #fff
  position relative
  &:hover
    background-color: rgba(255,255,255,.2);
  .admin-user
    padding 1px 2px
    border-radius 4px
    background-color #ea9336
  .vip-user
    padding 1px 2px
    border-radius 4px
    background-color #f25d8e
  .svip-user
    padding 1px 2px
    border-radius 4px
    background-color #ffb100
  .user-name
    color #4fc1e9
  .user-level
    padding 1px 2px
    border-radius 4px
    border-width 1px
    border-style solid
  .user-level-1
    color #939393
  .user-level-2
    color #5dbb57
  .user-level-3
    color #5595d9
  .user-level-4
    color #9a65ed
  .user-level-5
    color #fc84ae
  .user-level-6
    color #fc953a
  .user-badge
    padding 1px 2px
    border-radius 4px
  .user-badge-level-1
    background-color #61decb
  .user-badge-level-2
    background-color #5896de
  .user-badge-level-3
    background-color #a068f1
  .user-badge-level-4
    background-color #ff86b2
  .user-badge-level-5
    background-color #f6be18
  .guard-user
    padding 1px 2px
    border-radius 4px
    border-width 1px
    border-style solid
  .guard-user-1
    color #e1dfcc
  .guard-user-2
    color #e3efef
  .guard-user-3
    color #e6c28b
  .msg-gift
    .gift-img > img
      height 32px
    .user-name
      color #ff8f34
  .block-btn
    position absolute
    top 0
    right 0
    height 100%
    display flex
    justify-content center
    align-items center
    padding 0 8px
    background-color #ff4545
.send-message-box
  position absolute
  bottom 0
  padding 2px 0
  width 100%
  background-color rgba(25,25,25,.8)
  border-radius 5px
  overflow hidden
  .icon
    position absolute
    top 0
    right 0
    width 16px
    padding-top 10px
    margin-right 2px
  input[type="text"]
    padding 6px 16px
    border 0
    width 100%
    font-size 14px
    line-height 16px
    outline 0
    color #fff
    &::-webkit-input-placeholder
      color #fff
</style>
