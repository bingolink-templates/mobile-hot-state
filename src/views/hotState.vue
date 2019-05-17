<template>
  <div class="main">
    <!-- 热门动态 -->
    <div class="hot-state bra">
      <div class="hot-state-title flex">
        <text class="f28 fw5 c0">热门动态</text>
        <text class="f24 c153 fw4 pl20 pt10 pb10" @click="hotAllEvent">全部</text>
      </div>
      <scroller class="state-scroller-item flex-dr hot-state-content" scroll-direction="horizontal"
        show-scrollbar="false">
        <div class="hot-state-item bra mr20" v-for="(item,index) in imageArr" :key='index'>
          <div class="item-title flex">
            <div class="flex-dr flex-ac">
              <bui-image src="/image/test.jpg" radius='20px' width="40px" height="40px"></bui-image>
              <text class="f24 pl20 fw4 c51">Amy3053</text>
            </div>
            <text class="f20 fw4 c153">2小时前</text>
          </div>
          <div class="item-content">
            <text class="f20 c102 lines2 fw4">个性需求接受or拒绝，揭秘产品组和项目组不为人知的秘密！</text>
            <scroller class="state-scroller-image mt20 mb20" scroll-direction="horizontal" show-scrollbar="false">
              <div class="flex-dr">
                <div class="item-image pr10" v-for="(item ,index) in imageArr" :key='index'>
                  <bui-image src="/image/test.jpg" width="72px" height="72px"></bui-image>
                </div>
              </div>
            </scroller>
          </div>
          <div class="item-comment">
            <div class="flex-dr flex-je">
              <div class="flex-dr flex-ac mr30">
                <bui-image src="/image/test.jpg" width="20px" height="20px"></bui-image>
                <text class="f22 fw4 pl10 c85">230</text>
              </div>
              <div class="flex-dr flex-ac">
                <bui-image src="/image/test.jpg" width="20px" height="20px"></bui-image>
                <text class="f22 fw4 pl10 c85">230</text>
              </div>
            </div>
          </div>
        </div>
      </scroller>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        imageArr: ['','','','','','','']
      }
    },
    methods: {
      // 更多
      hotAllEvent() {
        this.$alert()
      }
    }
  }
</script>

<style lang="css" src="../css/common.css"></style>
<style>
  .main {
    flex: 1;
    padding-top: 100px;
    background-color: #666;
  }

  .hot-state {
    height: 397px;
    background-color: #fff;
  }

  .hot-state-title {
    height: 40px;
    margin: 18px 23px 0 25px;
  }

  .hot-state-content {
    padding-left: 24px;
  }

  .hot-state-item {
    width: 553px;
    height: 294px;
    box-shadow: 0px 0px 23px 3px rgba(229, 232, 237, 0.53);
    background-color: #fff;
  }

  .item-title {
    margin: 24px 24px 16px 18px;
  }

  .item-content {
    width: 451px;
    margin: 0 24px 0 78px;
  }

  .state-scroller-image {
    width: 451px;
    height: 75px;
  }

  .state-scroller-item {
    height: 294px;
    padding-top: 20px;
  }

  .item-comment {
    padding-right: 24px;
  }
</style>