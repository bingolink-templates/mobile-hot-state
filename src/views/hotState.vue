<template>
  <div ref="wrap">
    <!-- 热门动态 -->
    <div class="hot-state">
      <div class="hot-state-title flex">
        <text class="f28 fw5 c0">{{i18n.HotBlog}}</text>
        <text class="f24 c153 fw4 pl20 pt10 pb10" @click="hotAllEvent">{{i18n.All}}</text>
      </div>
      <div v-if="isShow">
        <scroller v-if='hotStateArr.length!=0' class="state-scroller-item flex-dr hot-state-content"
          scroll-direction="horizontal" show-scrollbar="false">
          <div class="hot-state-item bra mr20" v-for="(item,index) in hotStateArr" :key='index'
            @click="hotEvent(item.id)">
            <div class="item-title flex">
              <div class="flex-dr flex-ac">
                <bui-image :src="item.headImage" radius='20px' width="40px" height="40px" v-if="item.headImage"
                  @click="hotEvent(item.id)">
                </bui-image>
                <div class="avatar-image flex-ac" v-if="!item.headImage">
                  <text class="cf f28">{{item.accountNameLastWord}}</text>
                </div>
                <text class="f24 pl20 fw4 c51">{{item.accountName}}</text>
              </div>
              <text class="f20 fw4 c153">{{item.time}}</text>
            </div>
            <div class="item-content">
              <text class="f20 c102 lines2 fw4">{{item.content}}</text>
              <scroller class="state-scroller-image mt20 mb20" scroll-direction="horizontal" show-scrollbar="false">
                <div class="flex-dr">
                  <div class="item-image pr10" v-for="(image ,index) in item.imageArr" :key='index'>
                    <bui-image :src='image' width="72px" height="72px" @click="hotEvent(item.id)"></bui-image>
                  </div>
                </div>
              </scroller>
            </div>
            <div class="item-comment">
              <div class="flex-dr flex-je">
                <div class="flex-dr flex-ac mr30">
                  <bui-image src="/image/comment.png" width="22px" height="22px"></bui-image>
                  <text class="f22 fw4 pl10 c85">{{item.commentCount}}</text>
                </div>
                <div class="flex-dr flex-ac">
                  <bui-image src="/image/yes.png" width="22px" height="22px"></bui-image>
                  <text class="f22 fw4 pl10 c85">{{item.praiseCount}}</text>
                </div>
              </div>
            </div>
          </div>
        </scroller>
        <div class="no-content flex-ac flex-jc" v-if='hotStateArr.length==0'>
          <div class="flex-dr">
            <bui-image src="/image/sleep.png" width="42px" height="39px"></bui-image>
            <text class="f26 c51 fw4 pl15 center-height">{{isError?i18n.NoneData:i18n.ErrorLoadData}}</text>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  const link = weex.requireModule("LinkModule");
  const linkapi = require('linkapi');
  const dom = weex.requireModule('dom');
  export default {
    data() {
      return {
        hotStateArr: [],
        isShow: false,
        isError: true,
        channel: new BroadcastChannel('WidgetsMessage'),
        i18n: ''
      }
    },
    methods: {
      hotAllEvent(id) {
        link.launchLinkService(['[OpenBuiltIn] \n key=BlogMain'], (res) => {}, (err) => {});
      },
      hotEvent(id) {
        link.launchLinkService(['[OpenBuiltIn] \n key=BlogDetail \n blogId=' + id], (res) => {}, (err) => {});
      },
      timeFormat(m) {
        return m < 10 ? '0' + m : m
      },
      getNowFormatDate(type, dat) {
        let date = new Date()
        if (dat) {
          date = new Date(dat)
        }
        let year = date.getFullYear(),
          month = date.getMonth() + 1,
          strDate = date.getDate(),
          currentdate = ''
        if (month >= 1 && month <= 9) {
          month = "0" + month;
        }
        if (strDate >= 0 && strDate <= 9) {
          strDate = "0" + strDate;
        }
        if (type == 1) {
          currentdate = year + month + strDate;
        } else {
          currentdate = year + '/' + month + '/' + strDate;
        }
        let hour = date.getHours(),
          min = date.getMinutes();
        if (dat) {

          if (new Date(dat).toDateString() === new Date().toDateString()) {
            currentdate = '今天 ' + this.timeFormat(hour) + ':' + this.timeFormat(min)
          } else {
            currentdate = month + '-' + strDate + ' ' + this.timeFormat(hour) + ':' + this.timeFormat(min)
          }
        }
        return currentdate;
      },
      getToken(success, error) {
        return new Promise((resolve, reject) => {
          link.getToken([], res => {
            resolve(res);
            success && success(res);
          }, err => {
            reject(err);
            error && error(err);
          });
        });
      },
      getHotState(start, end) {
        this.getToken((token) => {
          link.getServerConfigs([], (params) => {
            let objData = {
              labelId: 'BrowsingBlogs',
              cursor: '',
              limit: 5,
              commentNum: 5,
              forwardNum: 5,
              praiseNum: 10
            }
            linkapi.fetch('GET', {
              url: params.blogUri + '/v1/blog/list/label',
              data: objData,
            }).then((res) => {
              this.isError = true
              this.isShow = true
              this.broadcastWidgetHeight()
              if (res.code == 200) {
                let hotArr = []
                for (let index = 0; index < res.data.length; index++) {
                  let element = res.data[index];
                  let hotObj = {}
                  hotObj['commentCount'] = element.commentCount
                  hotObj['praiseCount'] = element.praiseCount
                  let newDate = element.blogInfo.publishTime
                  let getMonthWeek = this.getNowFormatDate(1, newDate)
                  hotObj['time'] = getMonthWeek
                  hotObj['accountName'] = element.blogInfo.accountName
                  hotObj['content'] = element.blogInfo.content
                  hotObj['id'] = element.blogInfo.blogId
                  hotObj['accountNameLastWord'] = element.blogInfo.accountName.charAt(element.blogInfo
                    .accountName.length - 1)
                  hotObj['headImage'] = ''
                  if (element.blogInfo.accountImage) {
                    hotObj['headImage'] = params.uamUri + "/api/uam/getAvatarById?id=" + element.blogInfo
                      .accountId + '&type=1&width=40&height=40&&access_token=' + token.accessToken
                  }
                  hotObj['imageArr'] = []
                  for (let jindex = 0; jindex < element.resourceList.length; jindex++) {
                    let resourceItem = element.resourceList[jindex];
                    let imageId = resourceItem.resourceUrl.split('//')[1]
                    let imageItem =
                      params.storeUri + "/store/getFile?fileId=" + imageId + '&size=0x71&access_token=' +
                      token.accessToken
                    hotObj['imageArr'].push(imageItem)
                  }
                  hotArr.push(hotObj)
                }
                this.hotStateArr = hotArr
              } else {
                this.broadcastWidgetHeight()
              }
            }, (err) => {
              this.error()
            })
          }, (err) => {
            this.error()
          });
        }, (err) => {
          this.error()
        })
      },
      error() {
        this.isError = false
        this.isShow = true
        this.broadcastWidgetHeight()
      },
      broadcastWidgetHeight() {
        let _params = this.$getPageParams();
        setTimeout(() => {
          dom.getComponentRect(this.$refs.wrap, (ret) => {
            this.channel.postMessage({
              widgetHeight: ret.size.height,
              id: _params.id
            });
            channel.close();
          });
        }, 100)
      }
    },
    created() {
      linkapi.getLanguage((res) => {
        this.i18n = this.$window[res]
      })
    },
    mounted() {
      this.channel.onmessage = (event) => {
        if (event.data.action === 'RefreshData') {
          this.getHotState()
        }
      }
      this.getHotState()
    }
  }
</script>

<style lang="css" src="../css/common.css"></style>
<style>
  .hot-state {
    background-color: #fff;
  }

  .hot-state-title {
    height: 40px;
    margin: 18px 23px 10px 25px;
  }

  .hot-state-content {
    padding-left: 24px;
    background-color: #f8f9fa;
  }

  .hot-state-item {
    width: 553px;
    height: 294px;
    border: 1px solid #f8f9fa;
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
    height: 330px;
    padding-top: 15px;
  }

  .item-comment {
    padding-right: 24px;
  }

  .avatar-image {
    width: 40px;
    height: 40px;
    border-radius: 20px;
    background: #4ca4fe;
  }

  .no-content {
    height: 166px;
    width: 750px
  }

  .center-height {
    line-height: 40px;
  }
</style>