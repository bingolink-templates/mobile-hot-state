<template>
    <div ref="wrap">
        <!-- 热门动态 -->
        <div class="hot-state">
            <div class="hot-state-title flex" v-bind:style="{'height': $isIPad ? '44wx': '88px'}">
                <div class="title flex">
                    <span class="line" v-bind:style="{'background-color': themeColor}"></span>
                    <text class="c0 f30">{{i18n.HotBlog}}</text>
                </div>
                <text class="f24 c153 fw4 pl20 pt10 pb10" @click="hotAllEvent">{{i18n.All}}</text>
            </div>
            <div v-if="isShow">
                <scroller v-if='hotStateArr.length!=0' class="flex-dr hot-state-content" scroll-direction="horizontal" show-scrollbar="false" v-bind:style="{'height': $isIPad ? '171wx': '342px', 'padding-top': $isIPad ? '10wx': '20px'}">
                    <div class="hot-state-item bra mr20" v-for="(item,index) in hotStateArr" :key='index' @click="hotEvent(item.id)" v-bind:style="{'height': $isIPad ? '160wx': '320px'}">
                        <div class="item-title flex">
                            <div class="flex-dr flex-ac">
                                <bui-image placeholder='/image/ellipsis.png' :src="item.headImage" radius='16wx' width="32wx" height="32wx" v-if="item.headImage" @click="hotEvent(item.id)">
                                </bui-image>
                                <div class="avatar-image flex-ac flex-jc" v-if="!item.headImage" v-bind:style="{'height': $isIPad ? '32wx': '64px'}">
                                    <text class="cf f30">{{item.accountNameLastWord}}</text>
                                </div>
                                <text class="f30 pl20 fw4 c51">{{item.accountName}}</text>
                            </div>
                            <text class="f24 fw4 c153">{{item.time}}</text>
                        </div>
                        <div class="item-content">
                            <text style="padding-bottom: 5px;" v-if='!item.contentFace && !item.produce' class="f28 c102 fw4" :class="[item.isExisImage || item.isExisDoc ? 'lines2' : 'lines3']">{{item.content}}</text>
                            <text v-if='!item.contentFace && item.produce' style="padding-bottom: 3px;" class="f28 c102 fw4 lines5">{{item.content}}</text>
                            <div v-if='item.contentFace' class="content-face-main flex-dr" v-bind:style="{'height': $isIPad ? '48wx': '96px'}">
                                <div class="content-face" v-for="face in item.contentFace">
                                    <text class="content-text lines1 f28 c102 fw4" v-if='!face.img' v-bind:style="{'height': $isIPad ? '15wx': '30px', 'line-height': $isIPad ? '15wx': '30px'}">{{face.con}}</text>
                                    <bui-image class="content-image" v-else placeholder='/image/ellipsis.png' :src='face.con' width="13wx" height="13wx">
                                    </bui-image>
                                </div>
                            </div>
                            <div v-if='item.isExisDoc' class="flex-dr doc-list mt20 mb20 flex-ac" v-bind:style="{'height': $isIPad ? '30wx': '60px'}">
                                <bui-image placeholder='/image/ellipsis.png' :src='item.docImage' width="25wx" height="25wx" @click="hotEvent(item.id)">
                                </bui-image>
                                <text class="doc-name f24 c128 lines1">{{item.docName}}</text>
                            </div>
                            <div v-else>
                                <scroller class="state-scroller-image mt20 mb20" scroll-direction="horizontal" show-scrollbar="false" v-bind:style="{'height': $isIPad ? '48px': '96px'}">
                                    <div class="flex-dr">
                                        <div class="item-image pr10" v-for="(image ,index) in item.imageArr" :key='index'>
                                            <div v-if="index<=4" class='posi-re'>
                                                <bui-image placeholder='/image/ellipsis.png' :src='image.item' width="30wx" height="30wx" @click="hotEvent(item.id)"></bui-image>
                                                <div v-if='image.type == 1' class="posi-ab">
                                                    <bui-image src='/image/play.png' width="12wx" height="12wx" @click="hotEvent(item.id)"></bui-image>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </scroller>
                            </div>
                        </div>
                        <div class="item-comment">
                            <div class="flex-dr flex-je">
                                <div class="flex-dr flex-ac mr30">
                                    <bui-image src="/image/comment.png" width="12wx" height="12wx"></bui-image>
                                    <text class="f24 fw4 pl10 c85">{{item.commentCount}}</text>
                                </div>
                                <div class="flex-dr flex-ac flex-jc">
                                    <bui-image src="/image/yes.png" width="12wx" height="12wx"></bui-image>
                                    <text class="f24 fw4 pl10 c85 mt4">{{item.praiseCount}}</text>
                                </div>
                            </div>
                        </div>
                    </div>
                </scroller>
                <div class="no-content flex-ac flex-jc" v-if='hotStateArr.length==0' v-bind:style="{'height': $isIPad ? '83wx': '166px'}">
                    <div class="flex-dr">
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
const storage = weex.requireModule('storage');
const globalEvent = weex.requireModule('globalEvent');
const navigator = weex.requireModule('navigator');
export default {
    data() {
        return {
            hotStateArr: [],
            isShow: false,
            isError: true,
            channel: new BroadcastChannel('WidgetsMessage'),
            i18n: '',
            faceList: {},
            faceArr: [],
            AT_PATTERN: new RegExp("@{[^}]*}", "g"),
            themeColor: '',
            $isIPad: false,
            urlParams: {}
        }
    },
    created() {
        this.getFaceImg()
        this.$fixViewport();
        // 语言
        linkapi.getLanguage((res) => {
            this.i18n = this.$window[res]
        })
        linkapi.getThemeColor(res => {
            this.themeColor = res.background_color;
        })
        this.$isIPad = this.$isIPad()
        this.urlParams = this.resolveUrlParams(weex.config.bundleUrl)
    },
    mounted() {
        var that = this
        // 首页刷新
        this.channel.onmessage = (event) => {
            if (event.data.action === 'RefreshData') {
                this.getHotState()
            }
        }
        this.getStorage(function () {
            that.getHotState()
        })
        globalEvent.addEventListener("androidback", function (e) {
            navigator.close()
        });
    },
    methods: {
        getStorage(callback) {
            let pageId = this.urlParams.userId ? this.urlParams.userId : ''
            let ecode = this.urlParams.ecode ? this.urlParams.ecode : 'localhost'
            storage.getItem('hotblog2020612' + ecode + pageId, res => {
                if (res.result == 'success') {
                    var data = JSON.parse(res.data)
                    this.isError = true
                    this.isShow = true
                    this.hotStateArr = data
                    this.broadcastWidgetHeight()
                    setTimeout(() => {
                        this.getHotState()
                    }, 2000)
                } else {
                    callback()
                }
            })
        },
        hotAllEvent(id) {
            link.launchLinkService(['[OpenBuiltIn] \n key=BlogMain'], (res) => { }, (err) => { });
        },
        hotEvent(id) {
            if (id == 1) {
                link.launchLinkService(['[OpenBuiltIn] \n key=BlogMain'], (res) => { }, (err) => { });
                return
            }
            link.launchLinkService(['[OpenBuiltIn] \n key=BlogDetail \n blogId=' + id], (res) => { }, (err) => { });
        },
        timeFormat(m) {
            return m < 10 ? '0' + m : m
        },
        getFileImages(ext) {
            let fileImageTypes = {
                'excel': ['xls', 'xlsx'],
                'music': ['mp3', 'wma', 'wav', 'mod', 'ogg', 'm4a'],
                'pdf': ['pdf'],
                'photo': ['bmp', 'gif', 'jpeg', 'jpg', 'svg', 'png', 'psd'],
                'ppt': ['ppt', 'pptx'],
                'txt': ['txt', 'key'],
                'video': ['rm', 'rmvb', 'wmv', 'avi', 'mp4', '3gp', 'mkv', 'flv', 'mov', 'mpg'],
                'word': ['doc', 'docx', 'wps'],
                'zip': ['zip', 'rar', '7z'],
                'unknow': ['file'],
                'folder2': ['folder']
            }
            let fileImages = {};
            let fileTypeImages = {};
            for (let fext in fileImageTypes) {
                fileImages[fext] = '/image/' + fext + '.png';
                let arr = fileImageTypes[fext];
                if (arr.length > 0) {
                    for (let i = 0; i < arr.length; i++) {
                        fileTypeImages[arr[i]] = fext;
                    }
                }
            }
            let type = fileTypeImages[ext];
            if (type) {
                return fileImages[type];
            } else {
                return fileImages['unknow'];
            }
        },
        getFaceImg() {
            var faceStr = "微笑,呲牙,害羞,色,苦笑,酷,发怒,飞吻,大舌头,抠鼻,偷笑,嘘,得意,享受,生病,顽皮,鄙视,难过,怒,惊呆了,委屈,流泪,哭泣,馋嘴,疑问,嘟嘴,眨眼,努力,吐舌,糟糕,不爽,傲慢,困,打瞌睡,不开心,好吧,呆,咧嘴,恶魔,晕,坏笑,打哈欠,可怜,吐,勾引,ok,握手,抱拳,点赞,胜利,奋斗,鼓掌,祈祷,握拳,手掌,no,爱心,心碎,拥抱,礼花,玫瑰,礼物,奖杯,红旗,高铁,地铁,单车,步行,飞机,互联网,公告,电脑,电话,公文包,钢笔,满分,对,错,感叹,警告,问号,top,结束,向上,向下,向左,向右,拳头,不要,不要不要,不赞,吻,睡觉,太阳,便便,爆筋,白天,夜晚,多云,彩虹,下雨,闪电,中国,蛋糕,炸弹,邮箱,刀,高跟鞋,唱歌,钓鱼,画画,酒杯,咖啡,麻将,射箭,足球,放大镜,录像,南瓜,圣诞老人,沙漏,手表,西瓜,回形针,枪,小狗,小猪,音乐,上,下,雪人,会议";
            var faceArr = faceStr.split(',');
            var faceList = {};
            for (var i = 0; i < faceArr.length; i++) {
                var imgUrl = '/image/face/' + i + '.png',
                    imgWord = "[" + faceArr[i] + "]";
                faceList[imgUrl] = imgWord;
            }
            this.faceList = faceList
        },
        resolveUrlParams(url) {
            // let url = weex.config.bundleUrl;
            if (!url) return {};
            url = url + "";
            var index = url.indexOf("?");
            if (index > -1) {
                url = url.substring(index + 1, url.length);
            }
            var pairs = url.split("&"),
                params = {};
            for (var i = 0; i < pairs.length; i++) {
                var pair = pairs[i];
                var indexEq = pair.indexOf("="),
                    key = pair,
                    value = null;
                if (indexEq > 0) {
                    key = pair.substring(0, indexEq);
                    value = pair.substring(indexEq + 1, pair.length);
                }
                params[key] = value;
            }
            return params;
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
        formatMsgContent(msgContent) {
            if (!msgContent) {
                return false;
            }
            var pattern2 = /\[[\u4e00-\u9fa5]+\]/g;
            var content = msgContent.match(pattern2);

            if (!content)
                return null

            var faceArr = []
            for (let index = 0; index < content.length; index++) {
                const element = content[index];
                for (var attr in this.faceList) {
                    if (this.faceList[attr] == element) {
                        msgContent = msgContent.replace(element, '`|_|`_ _' + attr + '`|_|`');
                    }
                }
            }
            msgContent = msgContent.split('`|_|`')
            var contentFaceArr = []
            for (let Jndex = 0; Jndex < msgContent.length; Jndex++) {
                var contentFaceObj = {}
                const elementI = msgContent[Jndex];
                if (elementI.indexOf('_ _') > -1) {
                    contentFaceObj['img'] = true
                    contentFaceObj['con'] = elementI.replace('_ _', '')
                } else {
                    contentFaceObj['img'] = false
                    contentFaceObj['con'] = elementI
                }
                contentFaceArr.push(contentFaceObj)
            }
            //表情
            return contentFaceArr
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
                    linkapi.get({
                        url: params.blogUri + '/v1/blog/list/label',
                        data: objData,
                    }).then((res) => {
                        this.isError = true
                        this.isShow = true
                        if (res.code == 200) {
                            try {
                                if (res.data.length <= 2) {
                                    linkapi.getLoginInfo((info) => {
                                        this.hotStateData(params, token, res, info.userName)
                                    })
                                } else {
                                    this.hotStateData(params, token, res, null)
                                }
                            } catch (error) {
                                this.error()
                            }
                        } else {
                            this.error()
                        }
                        this.broadcastWidgetHeight()
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
        hotStateData(params, token, res, isUserName) {
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
                var content = element.blogInfo.content
                if (content && content.indexOf('@{"id"') != -1) {
                    // var users = element.blogInfo.content.match(this.AT_PATTERN);
                    // var userName = JSON.parse(users[0].replace('@', ''))
                    // var co = content.substring(0, content.indexOf('@{"id"'))
                    // var cc = content.replace(co, '').replace(users, '')
                    // hotObj['content'] = userName.name ? (co + "@" + userName.name + cc) : co + cc
                    var a = this.strCharPosition(content, '@{"id"')
                    for (let index = 0; index < a; index++) {
                        var users = element.blogInfo.content.match(this.AT_PATTERN);
                        var userName = JSON.parse(users[index].replace('@', ''))
                        var co = content.substring(0, content.indexOf('@{"id"'))
                        var cc = content.replace(co, '').replace(users[index], '')
                        content = userName.name ? (co + "@" + userName.name + cc) : co + cc
                    }
                    element.blogInfo.content = content
                    hotObj['content'] = content
                } else {
                    hotObj['content'] = content
                }
                hotObj['contentFace'] = this.formatMsgContent(element.blogInfo.content)
                hotObj['id'] = element.blogInfo.blogId
                hotObj['accountNameLastWord'] = element.blogInfo.accountName.charAt(element.blogInfo.accountName.length - 1)
                hotObj['headImage'] = ''
                if (element.blogInfo.accountImage) {
                    hotObj['headImage'] = params.uamUri + "/api/uam/getAvatarById?id=" + element.blogInfo.accountId + '&type=1&width=40&height=40&access_token=' + token.accessToken
                }
                hotObj['imageArr'] = []
                hotObj['isExisDoc'] = false
                hotObj['isExisImage'] = false
                for (let jindex = 0; jindex < element.resourceList.length; jindex++) {
                    let resourceItem = element.resourceList[jindex];
                    // 视频 默认和图片一起
                    if (resourceItem.resourceType == 1 || resourceItem.resourceType == 0) {
                        let id = resourceItem.resourceType == 0 ? resourceItem.resourceUrl.split('//')[1] : resourceItem.resourceThumb.split('//')[1]
                        let videoItem = {}
                        hotObj['isExisImage'] = true
                        videoItem['type'] = resourceItem.resourceType
                        videoItem['item'] = params.storeUri + "/store/getFile?fileId=" + id + '&size=0x71&access_token=' + token.accessToken
                        hotObj['imageArr'].push(videoItem)
                        // 文件
                    } else if (resourceItem.resourceType == 3) {
                        // 默认展示一个文档
                        hotObj['isExisDoc'] = true
                        let docImage = resourceItem.resourceDescription.split('.')[resourceItem.resourceDescription.split('.').length - 1]
                        hotObj['docImage'] = this.getFileImages(docImage)
                        hotObj['docName'] = resourceItem.resourceDescription
                        break
                        // 分享链接
                    } else if (resourceItem.resourceType == 4) {
                        hotObj['isExisDoc'] = true
                        hotObj['docImage'] = '/image/url.png'
                        hotObj['docName'] = resourceItem.resourceDescription
                    }
                }
                if (hotObj['imageArr'].length != 0) {
                    hotObj['imageArr'].sort(this.compare("type"));
                }
                hotArr.push(hotObj)
            }
            if (isUserName) {
                hotArr.unshift({
                    commentCount: 0,
                    praiseCount: 0,
                    time: '',
                    headImage: '/image/system.png',
                    accountName: '聆客在线',
                    content: 'hi，' + isUserName + '。欢迎您使用聆客。在这里可以跟同事一起聊天、事务审批、任务管理、日程安排、文档分享，另外还有CRM、项目协作等好多功能，赶紧邀请小伙伴一起体验吧。',
                    contentFace: null,
                    id: 1,
                    accountNameLastWord: '聆',
                    imageArr: [],
                    isExisDoc: false,
                    isExisImage: false,
                    produce: true
                })
            }
            this.hotStateArr = hotArr
            let pageId = this.urlParams.userId ? this.urlParams.userId : ''
            let ecode = this.urlParams.ecode ? this.urlParams.ecode : 'localhost'
            storage.setItem('hotblog2020612' + ecode + pageId, JSON.stringify(hotArr))
        },
        strCharPosition(str, char) {
            var pos;
            var arr = [];
            pos = str.indexOf(char);
            while (pos > -1) {
                arr.push(pos);
                pos = str.indexOf(char, pos + 1);
            }
            return arr.length;
        },
        compare(pro) {
            return function (obj1, obj2) {
                var val1 = obj1[pro];
                var val2 = obj2[pro];
                if (val1 < val2) { //正序
                    return 1;
                } else if (val1 > val2) {
                    return -1;
                } else {
                    return 0;
                }
            }
        },
        error() {
            this.isError = false
            this.isShow = true
            this.broadcastWidgetHeight()
        },
        // 获取高度,通知首页,需延时,防止失败,间隔发送多次
        getComponentRect(_params) {
            dom.getComponentRect(this.$refs.wrap, (ret) => {
                this.channel.postMessage({
                    widgetHeight: ret.size.height,
                    id: _params.id
                });
            });
        },
        broadcastWidgetHeight() {
            let _params = this.$getPageParams();
            // 防止高度通知失败
            setTimeout(() => {
                this.getComponentRect(_params)
            }, 200)
            setTimeout(() => {
                this.getComponentRect(_params)
            }, 1200)
        }
    }
}
</script>

<style lang="css" src="../css/common.css"></style>
<style>
.hot-state {
    background-color: #fff;
    position: relative;
}

.hot-state-title {
    padding: 0 12wx;
}

.line {
    width: 5px;
    height: 36px;
    margin-right: 12px;
}

.hot-state-content {
    padding-left: 24px;
    background-color: #f3f5f8;
}

.hot-state-item {
    width: 553px;
    border: 1px solid #f8f9fa;
    background-color: #fff;
}

.item-title {
    margin: 12wx 12wx 0 9wx;
}

.item-content {
    width: 451px;
    margin: 0 12wx 0 50wx;
}

.content-face-main {
    width: 451px;
    overflow: hidden;
    align-items: center;
    flex-direction: row;
    flex-wrap: wrap;
}

.content-text {
    max-width: 401px;
}

.content-image {
    margin: 0 1px;
}

.state-scroller-image {
    width: 451px;
}

.item-comment {
    position: absolute;
    bottom: 5wx;
    right: 0;
    padding-right: 24px;
}

.avatar-image {
    width: 32wx;
    border-radius: 16wx;
    background: #4ca4fe;
}

.no-content {
    width: 750px;
}

.center-height {
    line-height: 20wx;
}

.doc-list {
    width: 451px;
    background-color: #f2f2f2;
    padding: 4wx;
}

.doc-name {
    padding-left: 11px;
    width: 320px;
}
.posi-re {
    position: relative;
}
.posi-ab {
    position: absolute;
    left: 9wx;
    top: 10wx;
}
</style>