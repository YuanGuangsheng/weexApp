<template>
    <div class="details-view">
        <list class="list" show-scrollbar=false>
            <!--文章详情-->
            <cell class="article">
                <!--视频-->
                <div class="video-info">
                    <text class="title">{{videoInfo.title}}</text>
                    <!-- 标签 -->
                    <div class="tag-box m-bottom" v-if="videoInfo.tags!=''">
                        <image class="tag-img" src="/app/imgs/tag.png"></image>
                        <div class="tag-box" v-for="(i,n) in videoInfo.tags.split(',')">
                            <text class="tag">{{i}}</text>
                            <image class="tag-line" src="/app/imgs/tag_line.png" v-if="n < (videoInfo.tags.split(',').length-1)"></image>
                        </div>
                    </div>
                    <!--时间+播放量-->
                    <div class="play-info">
                        <text class="info-text">{{getDateArticle(videoInfo.created_at)}}</text>
                        <text class="info-text" style="margin-left:30px;">{{videoInfo.view_count}}次播放</text>
                        <text class="point">·</text>
                        <text class="info-text" @click="report('article',videoInfo.id,videoInfo.user.data,videoInfo.title)">举报</text>
                    </div>
                    <div class="click-info">
                        <!--赞-->
                        <div @click="Vote(1)" style="position: relative;">
                            <text class="addOne" ref="addUp2" style="left:50px;">+1</text>
                            <div style="flex-direction: row" v-if="voteState.vote_up == 1">
                                <image class="info-icon" src="/app/imgs/vote_up36_on.png"></image>
                                <text class="info-text vote-colorful">{{videoInfo.vote_up}}</text>
                            </div>
                            <div style=" flex-direction: row" v-else>
                                <image class="info-icon" src="/app/imgs/vote_up36.png"></image>
                                <text class="info-text vote-gray">{{videoInfo.vote_up}}</text>
                            </div>
                        </div>

                        <!--踩-->
                        <div @click="Vote(2)" style="position: relative;">
                            <text class="addOne" ref="addDown2" style="left:50px;">+1</text>
                            <div style="flex-direction: row" v-if="voteState.vote_down == 1">
                                <image class="info-icon" src="/app/imgs/vote_down36_on.png"></image>
                                <text class="info-text vote-colorful">{{videoInfo.vote_down}}</text>
                            </div>
                            <div style=" flex-direction: row" v-else>
                                <image class="info-icon" src="/app/imgs/vote_down36.png"></image>
                                <text class="info-text vote-gray">{{videoInfo.vote_down}}</text>
                            </div>
                        </div>
                    </div>
                </div>
            </cell>

            <!--自媒体入口-->
            <cell v-if="isMedia">
                <!--选集-->
                <div class="media">
                    <div class="title-t"><text class="xuanji">选集</text></div>
                    <scroller class="Anthology" scroll-direction="horizontal"  show-scrollbar="false">
                        <div class="Anthology-li" v-for="(item,index) in AnthologyList">
                            <!--当前正在播放的视频-->
                            <div v-if="item.id == id">
                                <div class="cover-box">
                                    <image class="cover-img" :src="item.cover"></image>
                                    <image class="cover-bg" src="/app/images/cover.png"></image>
                                    <text class="media-num">{{item.zmt_period}}</text>
                                </div>
                                <div class="media-title-box">
                                    <text class="media-title active">{{stringCut(item.description,12)}}</text>
                                </div>
                            </div>
                            <!--不是当前播放的视频-->
                            <div @click="changeVideo(`${item.id}`,`${item.video_sid}`)" v-else>
                                <div class="cover-box">
                                    <image class="cover-img" :src="item.cover"></image>
                                    <image class="cover-bg" src="/app/images/cover.png"></image>
                                    <text class="media-num">{{item.zmt_period}}</text>
                                </div>
                                <div class="media-title-box">
                                    <text class="media-title">{{stringCut(item.description,12)}}</text>
                                </div>
                            </div>
                        </div>
                    </scroller>
                </div>
            </cell>
            <!--给去评论按钮标识-->
            <cell><text style="opacity: 0;" ref="commentFlag">评论区</text></cell>
            <!--评论-->
            <cell>
                <div v-for="(item,index) in commentsArea">
                    <div class="comment">
                        <div class="user">
                            <image class="avatar" :src="item.user.data.avator"></image><!--有头像-->
                            <text class="u-name">{{stringCut(item.user.data.username,17)}}</text>
                            <!--赞过-->
                            <div class="zan" @click="commentVoteUp(`${item.id}`,index)" v-if="item.vote==10">
                                <image class="vote-img" src="/app/imgs/vote_up36_on.png"></image>
                                <text class="zan-on">{{item.vote_up}}</text>
                            </div>
                            <!--没赞过-->
                            <div class="zan" @click="commentVoteUp(`${item.id}`,index)" v-else>
                                <image class="vote-img" src="/app/imgs/vote_up36.png"></image>
                                <text class="zan-text" v-if="item.vote_up == 0">赞</text>
                                <text class="zan-text" v-else>{{item.vote_up}}</text>
                            </div>
                        </div>
                        <!--评论的脚标-->
                        <div class="comment-text">
                            <div @click="replyComment(`${item.id}`,`${item.id}`,index,`${item.username}`)">
                                <text class="comment-content">{{item.content}}</text><!--评论的内容-->
                                <div class="comment-info">
                                    <text class="foot-item">{{getDateDiff(item.created_at)}}</text>
                                    <text class="foot-point">·</text>
                                    <text class="foot-item">{{item.reply_count}}个回复</text>
                                    <text class="foot-point">·</text>
                                    <text class="report" @click="report('comment',item.id,item.user.data,item.content)">举报</text>
                                </div>
                            </div>
                        </div>

                        <!--评论的回复-->
                        <div class="reply-comment">
                            <div class="reply-comment-con" v-for="(reply,index2) in item.reply">
                                <div class="reply-explain">
                                    <text class="reply-explain-foot reply-name">{{stringCut(reply.username,6)}}</text> <!--回复人:有时没有user.data字段-->
                                    <text class="reply-explain-foot black huifu">回复</text>
                                    <text class="reply-explain-foot reply-name" v-if="reply.reply_from_user == null">{{stringCut(item.user.data.username,6)}}</text><!--被回复人-->
                                    <text class="reply-explain-foot reply-name" v-else>{{stringCut(reply.reply_from_user,6)}}</text><!--被回复人-->
                                    <!--赞过-->
                                    <div class="reply-zan" @click="replyVoteUp(`${reply.id}`,index,index2)" v-if="reply.vote == 10">
                                        <image class="vote-img" src="/app/imgs/vote_up36_on.png"></image>
                                        <text class="zan-on">{{reply.vote_up}}</text>
                                    </div>
                                    <!--没赞过-->
                                    <div class="reply-zan" @click="replyVoteUp(`${reply.id}`,index,index2)" v-else>
                                        <image class="vote-img" src="/app/imgs/vote_up36.png"></image>
                                        <text class="zan-text" v-if="reply.vote_up == 0">赞</text>
                                        <text class="zan-text" v-else>{{reply.vote_up}}</text>
                                    </div>
                                </div>
                                <div @click="replyComment(`${item.id}`,`${reply.id}`,index,`${reply.username}`)">
                                    <text class="reply-content black">{{reply.content}}</text>
                                    <text class="reply-create-at">{{getDateDiff(reply.created_at)}}</text>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div ref="preventjump"><text text style="opacity: 0;">防止滚动跳转</text></div>
            </cell>
            <loading class="loading" @loading="onloading" :display="showLoading" v-if="loading">
                <text class="indicator">努力加载中...</text>
            </loading>
        </list>

        <!--底部页脚-->
        <div class="footer">
            <!--写评论-->
            <div class="input" @click="getCommentTxt">
                <image class="write" src="/app/imgs/write36.png"></image>
                <text class="write-text">写评论...</text>
            </div>
            <div class="box" @click="goComment">
                <image class="icon" src="/app/imgs/comment52.png"></image>
                <text class="num">{{videoInfo.comment_count}}</text>
            </div>
            <!--赞-->
            <div class="box" @click="Vote(1)">
                <text class="addOne" ref="addUp" style="left: 20px;">+1</text>
                <div v-if="voteState.vote_up == 1">
                    <image class="icon" src="/app/imgs/vote_up52_on.png"></image>
                    <text class="num">{{videoInfo.vote_up}}</text>
                </div>
                <div v-else>
                    <image class="icon" src="/app/imgs/vote_up52.png"></image>
                    <text class="num">{{videoInfo.vote_up}}</text>
                </div>
            </div>

            <!--踩-->
            <div class="box" @click="Vote(2)">
                <text class="addOne" ref="addDown" style="left: 20px;">+1</text>
                <div v-if="voteState.vote_down == 1">
                    <image class="icon" src="/app/imgs/vote_down52_on.png"></image>
                    <text class="num">{{videoInfo.vote_down}}</text>
                </div>
                <div v-else>
                    <image class="icon" src="/app/imgs/vote_down52.png"></image>
                    <text class="num">{{videoInfo.vote_down}}</text>
                </div>
            </div>
            <!--收藏-->
            <div class="box" style="width: 90px">
                <image v-if="collection == 0" class="icon" @click="Collection" src="/app/imgs/collection52.png"></image>
                <image v-else class="icon" @click="cancelCollection" src="/app/imgs/collection52_on.png"></image>
            </div>
        </div>
    </div>
</template>

<style scoped>
    .details-view {
        width: 750px;
        background-color: #fff;
    }
    /*视频详情*/
    .article {
        background-color:#f3f6f8;
        width: 750px;
    }
    .video-info {
        width: 710px;
        margin-left:20px;
        margin-right:20px;
    }
    .title {
        font-size: 38px;
        font-weight: bold;
        color: #363636;
        width: 710px;
        margin-top: 20px;
        margin-bottom: 20px;
        line-height: 48px;
        text-align: left;
    }
    .addOne {
        position: absolute;
        top: -30px;
        color:#FFC847;
        opacity: 0;
    }
    /*标签*/
    .tag-box {
        flex-direction: row;
        align-items: center;
        justify-content: flex-start;
    }
    .m-bottom {
        margin-bottom: 20px;
    }
    .tag-img{
        width: 28px;
        height:28px;
        margin-right: 10px;
    }
    .tag {
        color: #ffa829;
        font-size: 26px;
    }
    .tag-line{
        width: 8px;
        height: 16px;
        margin-right: 10px;
        margin-left: 10px;
    }
    /*时间+播放量*/
    .play-info {
        width: 710px;
        flex-direction: row;
        line-height: 30px;
    }
    .click-info {
        width: 750px;
        flex-direction: row;
        margin-top: 30px;
        padding-bottom:30px;
    }
    .info-text {
        font-size: 25px;
        color:#999999;
    }
    .point{
        color: #999999;
        font-weight: bold;
        margin-left:5px;
        margin-right: 5px;
    }
    .info-icon {
        width: 36px;
        height: 36px;
    }
    .vote-colorful {
        margin-right:30px;
        margin-left: 10px;
        margin-top:10px;
        color: #ffbb13;
    }
    .vote-gray{
        margin-right:30px;
        margin-left: 10px;
        margin-top:10px;
        color: #999;
    }
    /*评论*/
    .comment {
        width: 710px;
        margin-left: 20px;
        margin-top:30px;
    }
    .user {
        position: relative;
        flex-direction: row;
        align-items: center;
    }
    .avatar {
        width: 78px;
        height: 78px;
        border-radius: 40px;
    }
    .u-name {
        font-size: 28px;
        color: #3f6599;
        margin-left: 10px;
    }
    .vote-img {
        width: 34px;
        height: 34px;
    }
    .zan-text {
        font-size: 24px;
        margin-top: 10px;
        margin-left: 5px;
        color: #6b6b6b;
    }
    .zan-on{
        font-size: 24px;
        margin-top: 10px;
        margin-left: 5px;
        color: #ffbb13;
    }
    .comment-text {
        position: relative;
        width: 620px;
        margin-left: 90px;
        color: #999999;
    }
    .comment-content {
        font-size: 30px;
        line-height: 48px;
        color: #353535;
        padding-bottom: 20px;
    }
    .comment-info {
        flex-direction: row;
        align-items: center;
        margin-bottom:10px;
    }
    .zan {
        position: absolute;
        right: 5px;
        top: 9px;
        flex-direction: row;
    }
    .foot-item {
        font-size: 24px;
        color: #999999;
        margin-right:10px;
    }
    .report {
        font-size: 24px;
        color: #999999;
    }
    .foot-point {
        font-size: 38px;
        color: #999999;
        margin-right:10px;
    }
    /*评论的回复*/
    .reply-comment {
        width: 620px;
        background-color: #F3F6F8;
        margin-left: 90px;
    }
    .reply-comment-con {
        width: 620px;
        padding-left: 10px;
        padding-right: 10px;
        padding-top: 10px;
        padding-bottom: 10px;
    }
    .reply-explain {
        position: relative;
        flex-direction: row;
        height: 36px;
    }
    .reply-explain-foot {
        font-size: 28px;
    }
    .reply-name {
        color: #3f6599;
    }
    .black {
        color: #353535;
    }
    .huifu{
        padding-left: 15px;
        padding-right: 15px;
    }
    .reply-content {
        padding-top:20px;
        padding-bottom: 10px;
        line-height: 40px;
        font-size: 28px;;
    }
    .reply-create-at {
        font-size: 24px;
        color: #999999;
    }
    .reply-zan{
        position: absolute;
        right: 0;
        flex-direction: row;
    }
    /*页脚*/
    .footer {
        width: 750px;
        padding-top: 10px;
        padding-bottom: 10px;
        border-top-style:solid;
        border-top-width:1px;
        border-top-color:#e9e9e9;
        background-color: #fff;
        flex-direction: row;
        justify-content: center;
        align-items: center;
    }
    .input {
        width: 340px;
        height: 58px;
        border-style:solid;
        border-width:1px;
        border-color:#e9e9e9;
        background-color: #f3f6f8;
        border-radius: 50px;
        font-size: 25px;
        padding-left: 20px;
        flex-direction: row;
        justify-content: flex-start;
        align-items: center;
        margin-right:40px;
    }
    .write {
        width: 36px;
        height: 36px;
    }
    .write-text {
        font-size: 28px;
        color: #6c6c6c;
        margin-left:10px;
    }
    .icon {
        width: 52px;
        height: 52px;
        margin-top: 16px;
    }
    .box {
        position: relative;
        width: 60px;
        height: 80px;
        margin-left: 20px;
    }
    .num {
        position: absolute;
        font-size: 14px;
        background-color: #ff4444;
        color: #fff;
        padding-top:3px;
        padding-bottom:3px;
        padding-left: 8px;
        padding-right: 8px;
        right: 5px;
        top: 5px;
        border-radius: 10px;
    }
    .loading {
        width: 750px;
        padding-top:10px;
        padding-bottom: 10px;
        justify-content:center;
    }
    .indicator {
        font-size: 24px;
        color: #999;
        text-align: center;
    }

    /*自媒体*/
    .media {
        width: 710px;
        margin-left: 20px;
        height: 270px;
        margin-top: 30px;
        border-bottom-style: solid;
        border-bottom-width: 1px;
        border-bottom-color: #e9e9e9;
    }
    /*选集*/
    .title-t {
        margin-bottom: 20px;
    }
    .xuanji{
        font-size: 30px;
        color: #353535;
    }
    .more-box {
        flex-direction: row;
        width: 130px;
        height:55px;
    }
    .lookMore {
        color: #6b6b6b;
        font-size: 24px;
    }
    .lookMore-bg {
        width: 28px;
        height: 26px;
    }
    .Anthology {
        flex-direction: row;
        justify-content: flex-start;
    }
    .Anthology-li{
        width: 190px;
        margin-right:10px;
        flex-direction: column;
    }
    .cover-box {
        position: relative;
    }
    .cover-img {
        width: 190px;
        height: 110px;
    }
    .cover-bg {
        width: 190px;
        height: 50px;
        position: absolute;
        left:0;
        bottom:0;
    }
    .media-num {
        position: absolute;
        left:0;
        bottom:0;
        width: 190px;
        padding-left:10px;
        line-height: 40px;
        font-size: 20px;
        color: #fff;
    }
    .media-title-box {
        margin-top:10px;
    }
    .media-title{
        font-size: 24px;
        line-height: 30px;
        color: #353535;
    }
    .active{
        color: #ffa829;
    }
    .list {
        width: 750px;
    }
    .cover-box {
        position: relative;
    }
    .cover-img {
        width: 190px;
        height: 110px;
    }
    .cover-bg {
        width: 190px;
        height: 50px;
        position: absolute;
        left:0;
        bottom:0;
    }
    .media-num {
        position: absolute;
        left:0;
        bottom:0;
        width: 190px;
        padding-left:10px;
        line-height: 40px;
        font-size: 20px;
        color: #fff;
    }
</style>

<script>
    var webview = weex.requireModule('webview')
    var stream = weex.requireModule('stream')
    var dom = weex.requireModule('dom')
    var animation = weex.requireModule('animation')
    var modal = weex.requireModule('modal')
    // 调用android
    var app_module = weex.requireModule('app_module')
    export default {
        data() {
            return {
                baseUrl: "",
                videoInfo: {
                    "tags":"",
                    "created_at":""
                }, // 视频详情
                comments:'', // 评论内容
                voteState:{ "vote_up": 0, "vote_down" : 0 }, // 视频点赞、踩的状态
                vote: '00', // 10:赞， 01：踩， 00：没操作
                commentsArea:[], // 评论内容，填充到页面的
                commentText:"",
                page:1, // 评论页数
                id:'', // 文章ID
                commentId:'', // 评论id
                loading: true, // 是否显示加载
                showLoading: 'hide', // 努力加载中……
                cate:"videoPage", // 类型：视频、文章
                favorite:"",
                collection:0, // 是否收藏
                ifReply:false,
                isMedia:'false', // 是否是自媒体
                AnthologyList:[], // 选集列表
                videoCategory:'', // 视频类型
                shareMain:'' // 分享的内容
            }
        },
        created(){
            var that = this;
            app_module.getVideoID(function(objId){
                that.curentId = objId.postsId;
                that.id=objId.postsId;
                // 请求数据：详情
                that.getArticles(that.id, res => {
                    if(res.ok){
                        that.videoInfo = res.data.data;
                    }else{
                        that.videoInfo = "";
                        app_module.noGetNewInfoData(); // 获取失败，调用原生刷新机制
                    }
                });
                // 是否在列表页点击评论
                if(objId.isOpenInput=="Y"){
                    setTimeout(function(){
                        that.getCommentTxt();
                    }.bind(this),500)
                }
                // 判断视频入口
                if(objId.category == 0){
                    that.getAnthology('daodao', res => {
                        if(res.ok){
                            that.AnthologyList = res.data.data;
                            that.isMedia = true;
                        }else{
                            that.AnthologyList = '没有数据';
                            app_module.noGetNewInfoData(); // 调原生弹出刷新
                        }
                    });
                }else if(objId.category == 1){
                    that.getAnthology('laile', res => {
                        if(res.ok){
                            that.AnthologyList = res.data.data;
                            that.isMedia = true; // 显示选集
                        }else{
                            that.AnthologyList = '没有数据';
                            app_module.noGetNewInfoData(); //*调原生弹出刷新****
                        }
                    });
                }else{
                    that.isMedia = false; // 隐藏选集
                }

                // 请求数据
                app_module.getToken(function(obj){
                    // 获取：文章评论
                    that.getComments(that.page, that.id, obj.token);
                    // 获取：收藏，点赞/踩 状态
                    if(obj.token==""){
                        return;
                    }else{
                        // 当前用户是否点赞或者点踩
                        that.getVote(that.id, obj.token);
                        // 判断用户是否收藏过该文章
                        that.getFavorite(that.id, obj.token, res=> {
                            that.collection = res.ok ? res.data.data : 0; // 收藏状态1/0
                        });
                    }
                })
            })
        },
        methods: {
            /*!
            * GET请求操作
            */
            // GET请求数据: 视频详情API
            getArticles (videoId,callback) {
                return stream.fetch({
                    method: 'GET',
                    type: 'json',
                    url: this.baseUrl + videoId
                }, callback)
            },
            // GET请求数据: 文章评论(首次请求)
            getComments (page,repo,token) {
                var _self = this;
                return stream.fetch({
                    method: 'GET',
                    type: 'json',
                    url: _self.baseUrl + repo+'/comments?page='+page+'&Authorization=Bearer%20'+token
                }, function(res){
                    if(res.ok){
                        _self.comments = res.data.data;
                        for (let item in _self.comments) { _self.commentsArea.push(_self.comments[item]);}
                        if(_self.comments.length < 10){ // 小于一页，则不加载loading：false
                            _self.loading = false;
                        }
                    }else{
                        _self.comments = "没有数据";
                    }
                })
            },
            // GET请求数据：文章评论加载内容
            getLoadComments(page, repo, token) {
                var _self = this;
                return stream.fetch({
                    method: 'GET',
                    type: 'json',
                    url: _self.baseUrl + repo+'/comments?page='+page+'&Authorization=Bearer%20'+token
                },function(res){
                    if(res.ok){
                        _self.comments = res.data.data;
                        if(_self.comments.length == 0){ // 没有数据，则不加载loading：false
                            _self.loading = false;
                        }
                        setTimeout(() => {
                            for (let item in _self.comments) {
                                _self.commentsArea.push(_self.comments[item]);
                            }
                            if(_self.$getConfig().env.platform == 'iOS'){ // ios跳转，安卓无操作
                                dom.scrollToElement(_self.$refs.preventjump,{animated:false, offset: 0});
                            }
                            _self.showLoading = 'hide'; // 隐藏加载提示
                        }, 500)
                    }else{
                        _self.comments = "没有数据";
                    }
                })
            },
            // GET请求数据：点赞/踩
            getVote(id, token){
                var _self = this;
                return stream.fetch({
                    method: 'GET',
                    url: _self.baseUrl + id+'/vote?Authorization=Bearer%20'+token,
                    type: 'json'
                }, function(res){
                    if(res.ok){
                        _self.voteState.vote_up = res.ok ? res.data.vote_up : 0; // 点赞状态1/0
                        _self.voteState.vote_down = res.ok ? res.data.vote_down : 0; // 点踩状态1/0
                        if(_self.voteState.vote_up == 1){ _self.vote = '10'; }; // 返回给原生赞
                        if(_self.voteState.vote_down == 1){ _self.vote = '01'; }; // 返回给原生踩
                    }
                })
            },
            // GET请求数据：收藏文章
            getFavorite(id, token, callback){
                return stream.fetch({
                    method: 'GET',
                    url: this.baseUrl + id+'/favorite?Authorization=Bearer%20'+token,
                    type: 'json'
                }, callback)
            },
            // GET请求数据
            getAnthology (typeFlag, callback) {
                return stream.fetch({
                    method: 'GET',
                    type: 'json',
                    url: this.baseUrl + typeFlag
                }, callback)
            },

            /*!
            * 功能事件
            */
            // 事件：
            Vote(flag){
                var that = this;
                app_module.getToken(function(obj){
                    if(obj.token==""){
                        app_module.showToast(false,"请登录后操作");
                        app_module.goLogin();// 跳转到登录
                        return;
                    }else if(flag == 1){ // 点赞操作
                        that.postVote(that.id, 'vote_up', obj.token,res => {
                            if(res.ok){
                                if(res.data.status_code==0){ // 点赞成功
                                    that.videoInfo.vote_up++; // 数量+1
                                    that.voteState.vote_up = 1; // 改变点赞状态
                                    that.vote = "10"; // 返回给原生赞状态10
                                    app_module.showToast(true,res.data.message); // 弹框提示
                                    // +1效果
                                    var addOne = that.$refs.addUp;
                                    var addTwo = that.$refs.addUp2;
                                    that.addOne(addOne);
                                    that.addOne(addTwo);
                                    // 点赞成功返回给原生评论数和点赞数
                                    app_module.returnCommentCount(that.videoInfo.comment_count,that.videoInfo.vote_up,that.videoInfo.vote_down,that.vote,that.collection,that.cate);
                                }else if(res.data.status_code == 1){
                                    app_module.showToast(false,res.data.message);
                                }else if(res.status==401){ // token过期 跳转到登录
                                    app_module.showToast(false,"请重新登录");
                                    app_module.goLogin();
                                }
                            }else{
                                app_module.showToast(false,"网络出错");
                            }
                        })
                    }else if(flag == 2){
                        that.postVote(that.id, 'vote_down', obj.token,res => {
                            if(res.ok){
                                if(res.data.status_code==0){ // 点踩成功
                                    that.videoInfo.vote_down++; // 数量+1
                                    that.voteState.vote_down = 1; // 改变点赞状态
                                    that.vote = '01'; // 返回给原生踩状态01
                                    app_module.showToast(true,res.data.message); // 弹框提示
                                    // +1效果
                                    var addOne = that.$refs.addDown;
                                    var addTwo = that.$refs.addDown2;
                                    that.addOne(addOne);
                                    that.addOne(addTwo);
                                    // 点赞成功返回给原生评论数和点赞数
                                    app_module.returnCommentCount(that.videoInfo.comment_count,that.videoInfo.vote_up,that.videoInfo.vote_down,that.vote,that.collection,that.cate);
                                }else if(res.data.status_code == 1){
                                    app_module.showToast(false,res.data.message);
                                }else if(res.status==401){// token过期 跳转到登录
                                    app_module.showToast(false,"请重新登录");
                                    app_module.goLogin();
                                }
                            }else{
                                app_module.showToast(false,"网络出错");
                            }
                        })
                    }
                })
            },
            // 事件：下拉加载
            onloading(){
                this.page++;
                var that = this;
                this.showLoading = 'show'
                // 请求数据:
                app_module.getToken(function(obj){
                    that.getLoadComments(that.page,that.id,obj.token);
                })
            },
            // 收藏
            Collection(){
                // 调用原生获取token
                var that = this;
                app_module.getToken(function(obj){
                    if(obj.token==""){
                        app_module.showToast(false,"请登录后操作");
                        app_module.goLogin();// 跳转到登录
                        return;
                    }else{
                        that.postFavorite(that.id, obj.token, res=>{
                            if(res.ok){
                                if(res.data.status_code==0){
                                    that.collection = 1;
                                    app_module.showToast(true, res.data.message);
                                    // 收藏成功返回给原生评论数和点赞数
                                    app_module.returnCommentCount(that.videoInfo.comment_count,that.videoInfo.vote_up,that.videoInfo.vote_down,that.vote,that.collection,that.cate);
                                }else if(res.data.status_code == 1){
                                    app_module.showToast(false,res.data.message);
                                }else if(res.status==401){
                                    app_module.goLogin();// token过期 跳转到登录
                                    app_module.showToast(false,"请重新登录");
                                }
                            }else {
                                app_module.showToast(false,"网络出错");
                            }
                        })
                    }
                })
            },
            // 取消收藏
            cancelCollection(){
                // 调用原生获取token
                var that = this;
                app_module.getToken(function(obj){
                    if(obj.token==""){
                        app_module.showToast(false,"请登录后操作");
                        app_module.goLogin();// 跳转到登录
                        return;
                    }else{
                        that.postCancelFavorite(that.id, obj.token, res=> {
                            if(res.ok){
                                if(res.data.status_code==0){
                                    that.collection = 0;
                                    app_module.showToast(true,res.data.message);
                                    app_module.returnCommentCount(that.videoInfo.comment_count,that.videoInfo.vote_up,that.videoInfo.vote_down,that.vote,that.collection,that.cate);
                                }else if(res.data.status_code==1){
                                    app_module.showToast(false,res.data.message);
                                }else if(res.status==401){
                                    app_module.goLogin();// token过期 跳转到登录
                                    app_module.showToast(false,"请重新登录");
                                }
                            }else{
                                app_module.showToast(false,"网络出错");
                            }
                        })
                    }
                })
            },
            // 评论文章: 弹出原生评论框
            getCommentTxt(){
                var that = this;
                this.ifReply = false; // 告诉原生 不是回复
                // 调用原生获取token
                app_module.getToken(function(obj){
                    if(obj.token==""){
                        app_module.showToast(false,"请登录后操作");
                        app_module.goLogin(); // 跳转到登录
                        return;
                    }else{
                      app_module.comment('VIDEOS_COMMENT',that.ifReply,'',function(commentTxt){
                        that.postComment(that.id, obj.token, commentTxt, res=> {
                            if(res.ok){
                                if(res.data.status_code == 0){ // 评论成功
                                    var txt = {
                                        "id":res.data.data.id,
                                        "username": obj.username,
                                        "vote_down":"/app/imgs/vote_up36.png",
                                        "content": commentTxt,
                                        "vote_up": 0,
                                        "created_at": "1分钟前",
                                        "reply_count": 0,
                                        "reply": [],
                                        "vote":"00",
                                        "user": {
                                            "data": {
                                                "uid": obj.uid,
                                                "avator": obj.avatar,
                                                "username": obj.username
                                            }
                                        }
                                    };
                                    that.commentsArea.unshift(txt);
                                    that.videoInfo.comment_count++;
                                    app_module.showToast(true,res.data.message);
                                }else{ // 评论失败
                                    app_module.showToast(false,res.data.message);
                                }
                            }else if(res.status==401){
                                app_module.goLogin();// token过期 跳转到登录
                                app_module.showToast(false,"请重新登录");
                            }else{
                                app_module.showToast(false,"网络出错");
                            }
                        })
                      })
                    }
                })
            },
            // 回复评论：弹出原生评论框
            replyComment(comment_id,parent_id,index,replyName){
                var that = this;
                this.ifReply = true; // 告诉原生 是回复
                // 调用原生获取token
                app_module.getToken(function(obj){
                    if(obj.token==""){
                        app_module.showToast(false,"请登录后操作");
                        app_module.goLogin();// 跳转到登录
                    }else{
                        app_module.comment('VIDEOS_COMMENT',that.ifReply,replyName,function(commentTxt){
                            that.postReplyComment(that.id, comment_id, obj.token, commentTxt, parent_id, res=> {
                                if(res.ok){
                                    if(res.data.status_code == 0){ // 回复成功
                                        var txt = {
                                            "id": res.data.data.id,
                                            "content": commentTxt,
                                            "reply_from_user": replyName,
                                            "vote_up": "赞",
                                            "created_at": "1分钟前",
                                            "vote":"00",
                                            "username":obj.username
                                        };
                                        that.commentsArea[index].reply.splice(0,0,txt);
                                        that.videoInfo.comment_count++;
                                        that.commentsArea[index].reply_count++;
                                        app_module.showToast(true,res.data.message);
                                    }else{ // 回复失败
                                        app_module.showToast(false,res.data.message);
                                    }
                                }else if(res.status==401){ // token过期 跳转到登录
                                    app_module.goLogin();
                                    app_module.showToast(false,"请重新登录")
                                }else{
                                    app_module.showToast(false,"网络出错")
                                }
                            });
                        })
                    }
                })
            },
            // 评论点赞
            commentVoteUp(commentId,index){
                var that = this;
                app_module.getToken(function(obj) {
                    if (obj.token == "") {
                        app_module.showToast(false,"请登录后操作");
                        app_module.goLogin(); // 跳转到登录
                    }else {
                        that.postCommentVote(commentId, obj.token, res=> {
                            if(res.ok){
                                if(res.data.status_code==0){
                                    if(that.commentsArea[index].vote_up == "赞"){
                                        that.commentsArea[index].vote_up = 1;
                                    }else{
                                        that.commentsArea[index].vote_up++;
                                    }
                                    that.commentsArea[index].vote = 10;
                                    app_module.showToast(true,res.data.message);
                                }else if(res.status==401){ // token过期 跳转到登录
                                    app_module.showToast(false,"请重新登录");
                                    app_module.goLogin();
                                }else{
                                    app_module.showToast(false,res.data.message)
                                }
                            }
                        });
                    }
                })
            },
            //回复评论点赞
            replyVoteUp(replyId,ParentIndex,index){
                // 调用原生获取token
                var that = this;
                app_module.getToken(function(obj) {
                    if (obj.token == "") {
                        app_module.showToast(false,"请登录后操作");
                        app_module.goLogin();// 跳转到登录
                    }else {
                        that.postCommentVote(replyId, obj.token, res=> {
                            if(res.ok){
                                if(res.data.status_code==0){
                                    if(that.commentsArea[ParentIndex].reply[index].vote_up == "赞"){
                                        that.commentsArea[ParentIndex].reply[index].vote_up = 1;
                                    }else{
                                        that.commentsArea[ParentIndex].reply[index].vote_up++;
                                    }
                                    that.commentsArea[ParentIndex].reply[index].vote = 10;
                                    app_module.showToast(true,res.data.message)
                                }else if(res.status==401){ // token过期 跳转到登录
                                    app_module.showToast(false,"请重新登录");
                                    app_module.goLogin();
                                }else{
                                    app_module.showToast(false,res.data.message)
                                }
                            }
                        });
                    }
                })
            },
            // 举报
            report(types,id,user,content){
                var that = this;
                app_module.getToken(function(obj) {
                    if (obj.token == "") {
                        app_module.showToast(false,"请登录后操作");
                        app_module.goLogin(); // 跳转到登录
                    } else {
                        app_module.userReport("videoReport",types,that.id,id,user,content); // 去举报
                    }
                })
            },
            // 去评论区
            goComment(){
                if(this.commentsArea.length < 3){
                    return;
                }else{
                    var el = this.$refs.commentFlag;
                    dom.scrollToElement(el, { animated : false });
                }
            },
            // 自媒体：切换数据
            changeVideo(id,video_sid){
                var that = this;
                app_module.checkNetwork(function(networkState){
                    if(networkState == 'YES'){
                        // 清空之前的数据
                        that.videoInfo = {
                            "tags":"",
                            "created_at":""
                        }, // 视频详情
                        that.commentsArea = []; // 清空评论列表
                        that.page = 1; // 重置当前页数
                        that.id = id; // 重置当前id
                        that.voteState = { "vote_up": 0, "vote_down" : 0}; // 视频点赞、踩的状态
                        that.vote = '00'; // 10:赞， 01：踩， 00：没操作
                        that.favorite = "";
                        that.collection = 0; // 是否收藏
                        that.ifReply = false;
                        that.loading =  true; // 是否显示加载
                        that.showLoading = 'hide'; // 努力加载中……
                        that.isMedia = true; // 显示选集
                        // 重新获取数据
                        that.getArticles(id, res => {
                            if(res.ok){
                                that.videoInfo = res.data.data;
                                that.shareMain = {
                                    "title":that.videoInfo.title,
                                    "content": that.videoInfo.description,
                                    "imgUrl": that.videoInfo.cover,
                                    "url": that.videoInfo.share_url
                                }; // 分享的内容
                                app_module.postVideoId(video_sid, that.shareMain);
                            }else{
                                that.videoInfo = "";
                                app_module.noGetNewInfoData(); // 获取失败，调用原生刷新机制
                            }
                        }); // 视频内容
                        // 请求数据
                        app_module.getToken(function(obj){
                            that.getComments(that.page, id, obj.token)
                            if(obj.token==""){
                                return;
                            }else{
                                // 当前用户是否点赞或者点踩
                                that.getVote(id, obj.token);
                                // 判断用户是否收藏过该文章
                                that.getFavorite(id, obj.token, res=> {
                                    that.collection = res.ok ? res.data.data : 0; // 收藏状态1/0
                                });
                            }
                        })
                    }else{
                        app_module.showToast(false,"请检查网络");
                        return;
                    }
                });
            },

            /*!
             * POST请求操作
             */
            // POST请求数据
            postVote(id, action ,token ,callback){
                // 请求数据：
                return stream.fetch({
                    method: 'POST',
                    url: this.baseUrl + id+"/vote",
                    type: 'json',
                    body:JSON.stringify({
                        authorization:'Bearer '+token
                    })
                },callback)
            },
            // POST请求数据：文章收藏
            postFavorite(id, token, callback){
                return stream.fetch({
                    method:'POST',
                    type:'json',
                    url: this.baseUrl + id+'/favorite',
                    body:JSON.stringify({
                        authorization:'Bearer '+token
                    })
                }, callback)
            },
            // POST请求数据：取消文章收藏
            postCancelFavorite(id, token, callback){
                return stream.fetch({
                    method:'POST',
                    type:'json',
                    url: this.baseUrl,
                    body:JSON.stringify({
                        authorization:'Bearer '+token
                    })
                }, callback)
            },
            // POST请求数据：评论文章
            postComment(id, token, commentTxt, callback){
                return stream.fetch({
                    method: 'POST',
                    url: this.baseUrl + id+"/comments",
                    body:JSON.stringify({
                        authorization:'Bearer '+token
                    }),
                    type: 'json'
                }, callback)
            },
            // POST请求数据：回复评论
            postReplyComment(id, comment_id, token, commentTxt, parent_id, callback){
                return stream.fetch({
                    method: 'POST',
                    url: this.baseUrl + id,
                    body:JSON.stringify({
                        authorization:'Bearer '+token
                    }),
                    type: 'json'
                },callback)
            },
            // POST请求数据： 评论点赞
            postCommentVote(id, token, callback){
                return stream.fetch({
                    method:'POST',
                    type:'json',
                    url: this.baseUrl + id+"/vote",
                    body:JSON.stringify({
                        authorization:'Bearer '+token
                    })
                }, callback)
            },
            /*!
             * 函数
             */
            //文章时间格式化
            getDateArticle(timer){
                var dateTimeStamp = Date.parse(timer.replace(/-/gi,"/"));
                var result = timer;
                var minute = 1000 * 60;
                var hour = minute * 60;
                var now = new Date().getTime();
                var diffValue = now - dateTimeStamp;
                if(diffValue < 0){return;}
                var hourC =diffValue/hour;
                var minC =diffValue/minute;
                if(hourC>=1){
                    result=timer;
                }
                else if(minC>=1){
                    result=parseInt(minC) +"分钟前";
                }else{
                    result="1分钟前";
                }
                return result;
            },
            //评论时间格式化
            getDateDiff(timer){
                var dateTimeStamp = Date.parse(timer.replace(/-/gi,"/"));
                var result = timer;
                var year = new Date().getFullYear();
                var minute = 1000 * 60;
                var hour = minute * 60;
                var now = new Date().getTime();
                var diffValue = now - dateTimeStamp;
                if(diffValue < 0){return;}
                var hourC =diffValue/hour;
                var minC =diffValue/minute;
                if(hourC>=1){
                    if(timer.substr(0,4) == year){
                        result=timer.substr(5,11);
                    }else{
                        result=timer;
                    }
                }
                else if(minC>=1){
                    result=parseInt(minC) +"分钟前";
                }else{
                    result="1分钟前";
                }
                return result;
            },
            // 字符串截取
            stringCut(txt,cutNum){
                var cutStr = ''
                if(txt.length>cutNum){
                    cutStr = txt.substring(0,cutNum)+'...';
                }else{
                    cutStr = txt;
                }
                return cutStr;
            },
            // 点赞+1动画
            addOne(addOne){
                animation.transition(addOne, {
                    styles: {opacity:1,transform:'scale(1)'},
                    duration: 300, //ms
                    timingFunction: 'ease'
                }, function () {
                    animation.transition(addOne,{
                        styles: {opacity:0, transform:'scale(0.5)'},
                        duration: 300, //ms
                        timingFunction: 'ease'
                    },function(){})
                });
            }
        }
    }
</script>