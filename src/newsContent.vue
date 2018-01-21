<template>
    <div class="details-view">
        <!--海报新闻-->
        <list show-scrollbar=false v-if="article.ishaibao">
            <cell>
                <div class="ishaibao" v-for="item in articleCon">
                    <div v-if="item.type=='img'" class="haibao-img">
                        <image :style="{width:item.width+'px',height:item.height+'px'}" :src="item.src"></image>
                    </div>
                </div>
            </cell>
        </list>
        <list show-scrollbar=false v-else>
            <!--文章详情-->
            <cell class="article">
                <!--图文文章-->
                <div class="article-main" v-if="article.type<=3">
                    <text class="title" ref="top">{{article.title}}</text>
                    <!-- 标签 -->
                    <div class="tag-box m-bottom" v-if="article.tags!=''">
                        <image class="tag-img" src="/app/imgs/tag.png"></image>
                        <div class="tag-box" v-for="(i,n) in article.tags.split(',')">
                            <text class="tag">{{i}}</text>
                            <image class="tag-line" src="/app/imgs/tag_line.png" v-if="n < (article.tags.split(',').length-1)"></image>
                        </div>
                    </div>
                    <!--有小编-->
                    <div class="author" v-if="article.user!=''">
                        <image class="author-avatar" :src="article.user.data.avator"></image>
                        <div class="editor-card">
                            <text class="editor-name">{{article.user.data.username}}</text>
                            <text class="created-at">{{getDateArticle(article.created_at)}}</text>
                        </div>
                    </div>
                    <!--无小编-->
                    <div class="author" v-else>
                        <image class="author-avatar" src="/app/imgs/logo.png"></image>
                        <div class="editor-card">
                            <text class="editor-name">游宝</text>
                            <text class="created-at">{{getDateArticle(article.created_at)}}</text>
                        </div>
                    </div>
                    <div class="con-text">
                        <div v-for="item in articleCon">
                            <div v-if="item.type=='text'">
                                <text class="text">{{item.value}}</text>
                            </div>
                            <div v-else-if="item.type=='img'" class="article-img">
                                <image class="image" :style="{width:item.width+'px',height:item.height+'px'}" :src="item.src"></image>
                            </div>
                            <div v-else-if="item.type=='embed' && item.src.match('youku.com')">
                                <web class="video" :src="basiVideoSrc + item.src.match(/(?!youku.com)sid\/([\S]+)==/g)[0].replace('sid/', '')"></web>
                            </div>
                        </div>
                        <!--举报-->
                        <div><text class="report-btn" @click="report('article',article.id,article.user.data,article.title)">举报</text></div>
                        <!--点赞，点踩-->
                        <div class="action">
                            <!--点赞-->
                            <div class="circular c-colorful" @click="Vote(1)" v-if="voteState.vote_up == 1">
                                <image class="icon_img" src="/app/imgs/vote_up42_on.png"></image>
                                <text class="vote-colorful">{{article.vote_up}}</text>
                            </div>
                            <div class="circular c-gray" @click="Vote(1)" v-else>
                                <text class="addOne" ref="addUp2" style="left:50px;">+1</text>
                                <image class="icon_img" src="/app/imgs/vote_up42.png"></image>
                                <text class="vote_txt">{{article.vote_up}}</text>
                            </div>
                            <!--点踩-->
                            <div class="circular c-colorful" @click="Vote(2)" v-if="voteState.vote_down == 1">
                                <image class="icon_img" src="/app/imgs/vote_down42_on.png"></image>
                                <text class="vote-colorful">{{article.vote_down}}</text>
                            </div>
                            <div class="circular c-gray" @click="Vote(2)" v-else>
                                <text class="addOne" ref="addDown2" style="left:50px;">+1</text>
                                <image class="icon_img" src="/app/imgs/vote_down42.png"></image>
                                <text class="vote_txt">{{article.vote_down}}</text>
                            </div>
                        </div>
                    </div>
                </div>
                <!--话题-->
                <div class="article-main" v-else-if="article.type==6">
                    <div class="ht-iframe"><text class="ht-content">{{article.description}}</text></div>
                    <div class="time">
                        <text class="text-time">{{article.created_at}}</text>
                        <text class="ht-point">·</text>
                        <text class="text-time" @click="report('article',article.id,article.user.data,article.title)">举报</text>
                    </div>
                    <div class="attitude">
                        <div>
                            <image v-if="article.enjoy_type==2" src="/app/images/ht-b1.png" class="att-dog"></image>
                            <image v-if="article.enjoy_type==1" src="/app/images/ht-y1.png" class="att-dog"></image>
                            <div class="text4-iframe">
                                <text class="text-num">{{article.vote_up}}</text>
                                <text class="piao">票</text>
                            </div>
                        </div>
                        <div class="att-vs-box"><image src="/app/imgs/ht-vs98.png" class="att-vs"></image></div>
                        <div>
                            <image v-if="article.enjoy_type==2" src="/app/images/ht-b2.png" class="att-dog"></image>
                            <image v-if="article.enjoy_type==1" src="/app/images/ht-y2.png" class="att-dog"></image>
                            <div class="text4-iframe">
                                <text class="text-num">{{article.vote_down}}</text>
                                <text class="piao">票</text>
                            </div>
                        </div>
                    </div>
                    <div class="operation">
                        <!--支持-->
                        <div class="btn ht-selected" style="margin-left: 45px;" @click="Vote(1)" v-if="voteState.vote_up == 1">
                            <image src="/app/images/w-zan.png" class="ht-vote-img"></image>
                            <text style="color:#fff;font-size: 28px;margin-left: 10px;">支持</text>
                        </div>
                        <div class="btn ht-gray" style="margin-left: 45px;" @click="Vote(1)" v-else>
                            <text class="addOne" ref="addUp2" style="left:50px;">+1</text>
                            <image src="/app/images/w-zan.png" class="ht-vote-img"></image>
                            <text style="color:#fff;font-size: 28px;margin-left: 10px;">支持</text>
                        </div>
                        <!--反对-->
                        <div class="btn ht-selected" style="margin-left: 210px;" @click="Vote(2)" v-if="voteState.vote_down == 1">
                            <image src="/app/images/w-cai.png" class="ht-vote-img"></image>
                            <text style="color:#fff;font-size: 28px;margin-left: 10px;">反对</text>
                        </div>
                        <div class="btn ht-gray" style="margin-left: 210px;" @click="Vote(2)" v-else>
                            <text class="addOne" ref="addDown2" style="left:50px;">+1</text>
                            <image src="/app/images/w-cai.png" class="ht-vote-img"></image>
                            <text style="color:#fff;font-size: 28px;margin-left: 10px;">反对</text>
                        </div>
                    </div>
                </div>
                <!--段子-->
                <div class="article-main" v-else-if="article.type==7">
                        <div class="con-text">
                            <text class="duanzi">{{article.content}}</text>
                            <!--举报-->
                            <div><text class="report-btn" @click="report('article',article.id,article.user.data,article.title)">举报</text></div>
                            <!--点赞，点踩-->
                            <div class="action">
                                <!--点赞-->
                                <div class="circular c-colorful" @click="Vote(1)" v-if="voteState.vote_up == 1">
                                    <image class="icon_img" src="/app/imgs/vote_up42_on.png"></image>
                                    <text class="vote-colorful">{{article.vote_up}}</text>
                                </div>
                                <div class="circular c-gray" @click="Vote(1)" v-else>
                                    <text class="addOne" ref="addUp2" style="left:50px;">+1</text>
                                    <image class="icon_img" src="/app/imgs/vote_up42.png"></image>
                                    <text class="vote_txt">{{article.vote_up}}</text>
                                </div>
                                <!--点踩-->
                                <div class="circular c-colorful" @click="Vote(2)" v-if="voteState.vote_down == 1">
                                    <image class="icon_img" src="/app/imgs/vote_down42_on.png"></image>
                                    <text class="vote-colorful">{{article.vote_down}}</text>
                                </div>
                                <div class="circular c-gray" @click="Vote(2)" v-else>
                                    <text class="addOne" ref="addDown2" style="left:50px;">+1</text>
                                    <image class="icon_img" src="/app/imgs/vote_down42.png"></image>
                                    <text class="vote_txt">{{article.vote_down}}</text>
                                </div>
                            </div>
                        </div>
                    </div>
            </cell>
            <!--给去评论按钮标识-->
            <cell style="background-color: #f3f6f8;"><text style="opacity: 0;height: 0;" ref="commentFlag">评论区</text></cell>
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
        <div v-if="!article.ishaibao" class="footer">
            <!--写评论-->
            <div class="input" @click="getCommentTxt">
                <image class="write" src="/app/imgs/write36.png"></image>
                <text class="write-text">写评论...</text>
            </div>
            <div class="box" @click="goComment">
                <image class="icon" src="/app/imgs/comment52.png"></image>
                <text class="num">{{article.comment_count}}</text>
            </div>
            <!--赞-->
            <div class="box" @click="Vote(1)">
                <text class="addOne" ref="addUp" style="left: 20px;">+1</text>
                <div v-if="voteState.vote_up == 1">
                    <image class="icon" src="/app/imgs/vote_up52_on.png"></image>
                    <text class="num">{{article.vote_up}}</text>
                </div>
                <div v-else>
                    <image class="icon" src="/app/imgs/vote_up52.png"></image>
                    <text class="num">{{article.vote_up}}</text>
                </div>
            </div>

            <!--踩-->
            <div class="box" @click="Vote(2)">
                <text class="addOne" ref="addDown" style="left: 20px;">+1</text>
                <div v-if="voteState.vote_down == 1">
                    <image class="icon" src="/app/imgs/vote_down52_on.png"></image>
                    <text class="num">{{article.vote_down}}</text>
                </div>
                <div v-else>
                    <image class="icon" src="/app/imgs/vote_down52.png"></image>
                    <text class="num">{{article.vote_down}}</text>
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
    /*海报*/
    .ishaibao {
        width: 750px;
        margin-bottom: -1px;
    }
    /*文章详情*/
    .article {
        width: 750px;
        background-color: #f3f6f8;
    }
    .article-main {
        width: 710px;
        margin-left: 20px;
        margin-right: 20px;
    }
    .title {
        font-size: 38px;
        font-weight: bold;
        line-height: 50px;
        margin-top:45px;
        margin-bottom:20px;
        color: #353633;
        text-align: left;
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
    /*小编*/
    .author {
        flex-direction: row;
    }
    .author-avatar {
        width: 78px;
        height: 78px;
        border-radius: 40px;
    }
    .editor-card {
        flex-direction: column;
        margin-left: 10px;
    }
    .editor-name {
        font-size: 28px;
        color:#3f6599;
        margin-top: 8px;
    }
    .created-at {
        font-size: 24px;
        color:#999999;
        margin-top:5px;
    }
    /*文章内容*/
    .con-text {
        margin-top:30px;
    }
    .duanzi {
        font-size: 32px;
        line-height:60px;
        margin-top: 20px;
    }
    .text {
        font-size: 32px;
        color: #353535;
        line-height: 60px;
    }
    .article-img {
        justify-content: center;
        align-items: center;
    }
    .image {
        margin-top:20px;
        margin-bottom:20px;
    }
    .video {
        width: 710px;
        height: 400px;
        margin-top:20px;
        margin-bottom:20px;
    }
    /*举报*/
    .report-btn {
        font-size:20px;
        color:#aeabab;
        text-align: right;
        padding-top:20px;
    }
    /*点赞，点踩按钮*/
    .action {
        flex-direction: row;
        height: 240px;
        justify-content: center;
        align-items:center;
    }
    .circular {
        width: 125px;
        height: 125px;
        border-style: solid;
        border-width: 1px;
        border-radius: 62px;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        background-color: #fff;
        position: relative;
        margin-left:60px;
        margin-right:60px;
    }
    .c-gray {
        border-color: #d5d5d5;
    }
    .c-colorful{
        border-color: #ffbb13;
    }
    .addOne {
        font-size: 30px;
        position: absolute;
        top: -35px;
        color:#ffbb13;
        opacity: 0;
    }
    .icon_img {
        width: 42px;
        height: 42px;
    }
    .vote_txt {
        font-size: 22px;
        margin-top: 10px;
        color: #6B6B6B;
    }
    .vote-colorful {
        font-size: 22px;
        margin-top: 10px;
        color: #ffbb13;
    }
    .play-info {
        width: 710px;
        flex-direction: row;
        margin-left: 20px;
        margin-top: 25px;
        padding-bottom: 20px;
        border-bottom-color: #e5e5e5;
        border-bottom-style: solid;
        border-bottom-width: 1px;
    }
    .info-text {
        font-size: 25px;
        color:#919191;
        margin-top: 3px;
        margin-left: 5px;
    }
    .info-icon {
        width: 32px;
        height: 32px;
        margin-left: 25px;
    }
    /*话题*/
    .ht-iframe{
        margin-top: 50px;
    }
    .ht-content{
        font-size: 35px;
        color: #222;
        line-height: 50px;
        text-align: left;
    }
    .text-time{
        font-size: 25px;
        color: #aaa;
    }
    .ht-point {
        color: #aaa;
        margin-left:10px;
        margin-right:10px;
    }
    .time{
        margin-top:20px;
        flex-direction: row;
    }
    .operation{
        width:710px;
        margin-left: 20px;
        flex-direction: row;
        padding-bottom: 80px;
    }
    .ht-vote-img {
        width:38px;
        height:39px;
    }
    .btn{
        flex-direction: row;
        align-items: center;
        justify-content: center;
        margin-top: 20px;
        width:185px;
        height:59px;
        border-radius: 12px;
    }
    .ht-gray{
        background-color: #919191;
    }
    .ht-selected{
        background-color: #ffbb13;
    }
    .attitude{
        width:710px;
        flex-direction: row;
        justify-content: center;
        margin-top:50px;
    }
    .att-dog{
        width:142px;
        height:140px;
    }
    .att-vs-box{
        height:140px;
        width:255px;
        justify-content: center;
        align-items: center;
    }
    .att-vs{
        width:97px;
        height:49px;
    }
    .text-num{
        font-size: 28px;
        color:#FE4643;
    }
    .piao {
        font-size: 24px;
        color:#353535;
        margin-left:5px;
        margin-top:5px;
    }
    .text4-iframe{
        flex-direction: row;
        justify-content: center;
        width:142px;
        margin-top:10px;
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
                baseUrl: '',
                article: '', // 新闻详情
                articleCon: {}, // 新闻主要内容
                basiVideoSrc:'',
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
                cate:"newsPage", // 类型：视频、文章
                favorite:"",
                collection:0, // 是否收藏
                ifReply:false,
                shareMain:'' // 分享的内容
            }
        },
        created(){
            var that = this;
            app_module.getVideoID(function(objId){
                that.id=objId.postsId;
                // 是否在列表页点击评论
                if(objId.isOpenInput=="Y"){
                    setTimeout(function(){
                        that.getCommentTxt();
                    }.bind(this),500)
                }
                // 请求数据：详情
                that.getArticles(that.id, res => {
                    if(res.ok){
                        that.article = res.data.data;
                        that.articleCon = JSON.parse(that.article.parse_content); // 新闻详情
                        // 详情请求成功，请求评论
                        app_module.getToken(function(obj){
                            // 获取：文章评论
                            that.getComments(that.page, that.id, obj.token);
                        })
                    }else{
                        that.article = "";
                        that.articleCon = "";
                        app_module.noGetNewInfoData(); // 获取失败，调用原生刷新机制
                    }
                    // 判断该新闻是资讯板块还是视频板块(用于gif判断)
                    if (that.article.app_category_id == 1064) { // 视频中的GIF
                        that.cate = "newsPage";//videoPage下的，但是要再新闻中显示
                    }
                 })

                // 请求数据
                app_module.getToken(function(obj){
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
            // GET请求数据: 新闻情API
            getArticles (videoId,callback) {
                return stream.fetch({
                    method: 'GET',
                    type: 'json',
                    url: this.baseUrl+videoId
                }, callback)
            },
            // GET请求数据：新闻内容
            getArticleCon(type, id){
                var _self = this;
                if(type != 6){ // 不是话题，则请求文章详情
                    return stream.fetch({
                        method: 'GET',
                        type: 'json',
                        url: _self.baseUrl + id+'/view'
                    }, function(res){
                        if(res.ok){
                            _self.articleCon = res.data.data;
                            // 判断视频来源，只显示优酷视频
                            for (let item in _self.articleCon) {
                                if (_self.articleCon[item].type == "embed") {
                                    if(_self.articleCon[item].src.match("youku.com")) { // 优酷视频
                                        _self.articleCon[item].src = ''+_self.articleCon[item].src.match(/(?!youku.com)sid\/([\S]+)==/g)[0].replace("sid/", "");
                                    }else{ // 不是优酷，则为空
                                        _self.articleCon[item].src = "";
                                    }
                                }
                            }
                        }else{
                            _self.articleCon = "没有数据";
                            app_module.noGetNewInfoData(); // 获取失败，调用原生刷新机制
                        }
                    })
                }
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
                        },500)
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

            /*!
            * 功能事件
            */
            // 事件：文章点赞1/踩2
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
                                    that.article.vote_up++; // 数量+1
                                    that.voteState.vote_up = 1; // 改变点赞状态
                                    that.vote = "10"; // 返回给原生赞状态10
                                    app_module.showToast(true,res.data.message); // 弹框提示
                                    // +1效果
                                    var addOne = that.$refs.addUp;
                                    var addTwo = that.$refs.addUp2;
                                    that.addOne(addOne);
                                    that.addOne(addTwo);
                                    // 点赞成功返回给原生评论数和点赞数
                                    app_module.returnCommentCount(that.article.comment_count,that.article.vote_up,that.article.vote_down,that.vote,that.collection,that.cate);
                                }else if(res.data.status_code == 1){
                                    app_module.showToast(false,res.data.message); // 重复投票提示
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
                                if(res.data.status_code==0){ // 点赞成功
                                    that.article.vote_down++; // 数量+1
                                    that.voteState.vote_down = 1; // 改变点赞状态
                                    that.vote = '01'; // 返回给原生踩状态01
                                    app_module.showToast(true,res.data.message); // 弹框提示
                                    // +1效果
                                    var addOne = that.$refs.addDown;
                                    var addTwo = that.$refs.addDown2;
                                    that.addOne(addOne);
                                    that.addOne(addTwo);
                                    // 点赞成功返回给原生评论数和点赞数
                                    app_module.returnCommentCount(that.article.comment_count,that.article.vote_up,that.article.vote_down,that.vote,that.collection,that.cate);
                                }else if(res.data.status_code == 1){
                                    app_module.showToast(false,res.data.message); // 重复投票提示
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
                this.showLoading = 'show';
                // 请求数据: 文章评论加载更多
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
                                if(res.data.status_code==0){ // 操作成功
                                    that.collection = 1; // 改变收藏状态
                                    app_module.showToast(true, res.data.message);
                                    // 收藏成功返回给原生评论数和点赞数
                                    app_module.returnCommentCount(that.article.comment_count,that.article.vote_up,that.article.vote_down,that.vote,that.collection,that.cate);
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
                                    that.collection = 0; // 更改收藏状态
                                    app_module.showToast(true,res.data.message);
                                    // 点赞成功返回给原生评论数和点赞数
                                    app_module.returnCommentCount(that.article.comment_count,that.article.vote_up,that.article.vote_down,that.vote,that.collection,that.cate);
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
                this.ifReply = false;
                // 调用原生获取token
                app_module.getToken(function(obj){
                    if(obj.token==""){
                        app_module.showToast(false,"请登录后操作");
                        app_module.goLogin(); // 跳转到登录
                        return;
                    }else{
                        app_module.comment('NEWS_COMMENT',that.ifReply,'',function(commentTxt){
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
                                        that.article.comment_count++;
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
                this.ifReply = true;
                // 调用原生获取token
                app_module.getToken(function(obj){
                    if(obj.token==""){
                        app_module.showToast(false,"请登录后操作");
                        app_module.goLogin();// 跳转到登录
                    }else{
                        app_module.comment('NEWS_COMMENT',that.ifReply,replyName,function(commentTxt){
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
                                        that.article.comment_count++;
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
                        app_module.userReport("newsReport",types,that.id,id,user,content); // 去举报
                    }
                })
            },
            // 去评论区
            goComment(){
                var el = this.$refs.commentFlag;
                dom.scrollToElement(el, { animated : false});
            },

            /*!
             * POST请求操作
             */
            // POST请求数据：文章点赞/踩
            postVote(id, action ,token ,callback){
                // 请求数据：赞和踩API
                return stream.fetch({
                    method: 'POST',
                    url: this.baseUrl + id+"/vote",
                    type: 'json',
                    body:JSON.stringify({
                        authorization:'Bearer '+token,
                        action:action
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
                    url: '',
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