<template>
  <div class="warp">
    <div class="header">
      <div class="cloaeBox" @click="close"><image src="/app/images/pictures/return42.png" class="close"></image></div>
      <!--小编-->
      <div style="flex-direction: row;">
        <image class="avatar" :src="user.avator"></image>
        <text class="editor">{{user.username}}</text>
      </div>
      <div @click="share">
        <image class="share" src="/app/images/pictures/share42.png"></image>
      </div>
    </div>
    <slider class="slider" :style="{height:sheight+'px'}" @change="changes">
      <div class="frame" v-for="(img,index) in article.pictures" :style="{height:sheight+'px'}">
        <image class="image" :src="img.src" :style="{width:img.width+'px',height:img.height+'px'}"></image>
      </div>
    </slider>
    <div class="describeArea">
      <text class="text">{{n}} /{{article.pictures_count}}  {{first_view}}</text>
    </div>
    <!--底部页脚-->
    <div class="footer">
      <!--写评论-->
      <div class="input" @click="getCommentTxt">
        <image class="write" src="/app/images/pictures/write36.png"></image>
        <text class="write-text">写评论...</text>
      </div>
      <div class="box" @click="toComment">
        <image class="icon" src="/app/images/pictures/comment52.png"></image>
        <text class="num">{{article.comment_count}}</text>
      </div>
      <div class="box" @click="voteUp(2)">
        <text class="addOne" ref="addUp2">+1</text>
        <image v-if="upShow" class="icon" src="/app/images/pictures/voteup52.png"></image>
        <image v-else class="icon" src="/app/images/pictures/voteup_s52.png"></image>
        <text class="num">{{article.vote_up}}</text>
      </div>
      <div class="box" @click="voteDown(2)">
        <text class="addOne" ref="addDown2">+1</text>
        <image v-if="downShow" class="icon" src="/app/images/pictures/votedown52.png"></image>
        <image v-else class="icon" src="/app/images/pictures/votedown_s52.png"></image>
        <text class="num">{{article.vote_down}}</text>
      </div>
      <div class="box" style="width: 90px">
        <image v-if="collection" class="icon" @click="Collection" src="/app/images/pictures/collection52.png"></image>
        <image v-else class="icon" @click="cancelCollection" src="/app/images/pictures/collection_s52.png"></image>
      </div>
    </div>
  </div>
</template>
<style scoped>
  .warp {
    background-color: #1b1b1b;
  }
  .header {
    width: 715px;
    margin-top:20px;
    flex-direction: row;
    align-items:center;
    justify-content:space-between;
  }
  .cloaeBox {
    padding-top:10px;
    padding-left:30px;
    padding-right:10px;
    padding-bottom:10px;
  }
  .avatar {
    width: 46px;
    height:46px;
    border-radius:46px;
  }
  .editor {
    font-size: 28px;
    color: #babcc5;
    margin-left:10px;
    line-height: 46px;
  }
  .close {
    width: 42px;
    height:42px;
  }
  .share {
    margin-left:10px;
    width: 42px;
    height: 42px;
  }
  .image {
    width: 750px;
    height: 900px;
  }
  .slider {
    width: 750px;
    height: 1000px;
    justify-content:center;
    align-items:center;
    background-color: #1B1B1B;
  }
  .frame {
    width: 750px;
    height: 1000px;
    justify-content:center;
    align-items:center;
    position: relative;
  }
  .describeArea {
    position: fixed;
    bottom:90px;
    padding-top:40px;
    padding-left:20px;
    padding-right:20px;
    padding-bottom:20px;
    width: 750px;
    justify-content:flex-start;
    background-color: #1B1B1B;
    opacity: 0.7;
  }
  .text {
    font-size: 30px;
    color: #babcc5;
    line-height: 45px;
  }
  .big-num{
    line-height: 50px;
    color:#ff0000;
    font-size: 48px;
    display: inline;
  }
  /*页脚*/
  .footer {
    position: fixed;
    bottom:0;
    width: 750px;
    height: 90px;
    padding-top: 10px;
    padding-bottom: 10px;
    background-color: #1B1B1B;
    flex-direction: row;
    justify-content: center;
    align-items: center;
  }
  .placeholder {
    font-size: 24px;
    line-height: 58px;
    color: #84887f;
  }
  .input {
    width: 340px;
    height: 60px;
    background-color: #272727;
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
    color: #babcc5;
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
  .addOne {
    font-size: 30px;
    position: absolute;
    top: -35px;
    left: 10px;
    color:#ff4444;
    opacity: 0;
  }
</style>
<script>
  var stream = weex.requireModule('stream')
  var modal = weex.requireModule('modal')
  var animation = weex.requireModule('animation')
  // 调用android
  var app_module = weex.requireModule('app_module')
  export default {
    data () {
      return {
        baseUrl: '',
        ID:'',
        user:"",
        sheight:0,
        src:'',
        imgLen:0,
        article:"",
        cate:'',
        img:'',
        title:'',
        content:'',
        imgUrl:'',
        url:'',
        upShow: true,
        downShow:true,
        token:'',
        n:1,
        first_view:'',
        collection:true,
        collection_state:"",
        ifReply:false,
        vote:"00",
        ifFavorite:0,
        empty:"  "
      }
    },
    created(){
      this.sheight = 1110;
      var that = this;
      app_module.getVideoID(function(objId) {
        // 获取文章内容
        that.ID = objId.postsId;
        if(objId.isOpenInput=="Y"){
          setTimeout(function(){
            that.getCommentTxt();
          }.bind(this),500)
        }
        that.getImages(objId.postsId,res => {
          if(res.ok){
            that.article = res.data.data;
            that.user = that.article.user.data;
            console.log(that.user)
          }else{
            that.article = "(network error)";
            that.user = "(network error)";
            //*调原生弹出刷新**********************************************************************
            app_module.noGetNewInfoData();
          }

          // 判断图片高度
          for(var i in that.article.pictures){
            if(that.article.pictures[i].height>that.sheight){
              that.article.pictures[i].height = that.sheight;
            }
            // 判断第一张是否有描述
            if(that.article.pictures[0].desc == ""){
              that.first_view = that.article.title;
            }else{
              that.first_view = that.article.pictures[0].desc;
            }
          }
          that.title = that.article.title;
          that.content = that.article.title;
          that.imgUrl = that.article.cover;
          that.url = that.article.share_url;

          if(that.article.app_category_id == 1064){
            that.cate = "videoPage"
          }else{
            that.cate = "newsPage"
          }
          // 调用原生，加载图标
          app_module.hiddenPicsLoading();
          // *********请求数据:当前用户做过操作*****************************
          // 请求数据:当前用户是否点赞或者点踩
          app_module.getToken(function(obj){
            if(obj.token==""){
              return;
            }else{
              stream.fetch({
                method: 'GET',
                url: that.baseUrl + objId.postsId+'/vote?Authorization=Bearer%20'+obj.token,
                type: 'json'
              },function(res){
                if(res.ok){
                  if(res.data.vote_up==1){ // 该用户点过赞
                    that.upShow = !that.upShow;
                    that.vote = "10";
                  }else if(res.data.vote_down==1){ // 该用户点过踩
                    that.downShow = !that.downShow;
                    that.vote = "01";
                  }else{
                    that.vote = "00";
                  }
                }
              })
            }
          });
          // 判断用户是否收藏过该文章
          app_module.getToken(function(obj){
            if(obj.token == ""){
              return;
            }else{
              stream.fetch({
                method: 'GET',
                url: that.baseUrl + objId.postsId+"/favorite?Authorization=Bearer%20"+obj.token,
                type: 'json'
              },function(res){
                if(res.ok){
                  if(res.data.data==1){
                    // 该用户点收藏过
                    that.collection = !that.collection;
                    that.ifFavorite = 1;
                  }else{
                    return;
                  }
                }
              })
            }
          })
        })
      })
    },
    methods:{
      // 切换图片 改变文字
      changes(event){
        this.n = event.index+1;
        for(var i in this.article.pictures){
          if(this.article.pictures[event.index].desc == ""){
            this.first_view = this.article.title;
          }else{
            this.first_view = this.article.pictures[event.index].desc;
          }
        }
        if(this.n>=10){
          this.empty = "     ";
        }else{
          this.empty = "  ";
        }
      },
      getImages (repo,callback) {
        return stream.fetch({
          method: 'GET',
          type: 'json',
          url: this.baseUrl + repo
        }, callback)
      },
      // 点击分享新闻，调用原生shareLink
      share(){
        app_module.shareLink(this.title,this.content,this.imgUrl,this.url)
      },
      // 请求数据：点赞
      voteUp(flag){
        // 调用原生获取token
        var that = this;
        app_module.getToken(function(obj){
          if(obj.token==""){ // token为空
            app_module.showToast(false,"请登录后操作");
            app_module.goLogin(); // 跳转到登录
            return;
          }else{
            that.getVote('vote_up',obj.token,res => {
              if(res.ok){
                  if(res.data.status_code==0){
                      that.upShow = !that.upShow;
                      that.article.vote_up++;
                      that.vote="10";
                      app_module.showToast(true,res.data.message);
                      app_module.returnCommentCount(that.article.comment_count,that.article.vote_up,that.article.vote_down,that.vote,that.ifFavorite,that.cate);
                      // +1效果
                      if(flag==1){ var addOne = that.$refs.addUp;
                      }else{ var addOne = that.$refs.addUp2;}
                      that.addOne(addOne);
                  }else if(res.data.status_code == 1){
                      app_module.showToast(false,res.data.message);
                  }else if(res.status==401){
                      app_module.showToast(false,"请重新登录")
                      app_module.goLogin();
                  }
              }else{
                  app_module.showToast(false,"网络出错");
              }
            })
          }
        })
      },
      // 请求数据
      voteDown(flag){
        var that = this;
        app_module.getToken(function(obj) {
          if (obj.token == "") {
            app_module.showToast(false,"请登录后操作");
            app_module.goLogin(); // 跳转到登录
          } else {
            that.getVote('vote_down',obj.token, res => {
              if(res.ok){
                if(res.data.status_code == 0){ // 点踩成功
                    that.downShow = !that.downShow;
                    that.article.vote_down++;
                    that.vote = "01";
                    app_module.showToast(true,res.data.message);
                    // +1效果
                    if(flag==1){ var addOne = that.$refs.addDown;
                    }else{ var addOne = that.$refs.addDown2;}
                    that.addOne(addOne);

                    app_module.returnCommentCount(that.article.comment_count,that.article.vote_up,that.article.vote_down,that.vote,that.ifFavorite,that.cate);
                }else if(res.data.status_code == 1){
                    app_module.showToast(false,res.data.message);
                }else if(res.status==401){
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
      },
      getVote(action,token,callback){
        // 请求数据：赞和踩API
        return stream.fetch({
          method: 'POST',
          url: this.baseUrl + this.ID+"/vote",
          type: 'json',
          body:JSON.stringify({
            authorization:'Bearer '+token
          })
        },callback)
      },
      close(){
        // 关闭图集页面
        app_module.atlasReturn();
      },
      // 调用原生，弹出评论框
      getCommentTxt(){
        var that = this;
        // 调用原生获取token
        app_module.getToken(function(obj){
          if(obj.token==""){
            app_module.showToast(false,"请登录后操作");
            app_module.goLogin();
          }else{
            app_module.comment('PICS_COMMENT',that.ifReply,'',function(commentTxt){
              stream.fetch({
                method: 'POST',
                url: that.baseUrl + that.ID+"/comments",
                body:JSON.stringify({
                  authorization:'Bearer '+obj.token
                }),
                type: 'json'
              }, function (res) {
                if(res.ok){ // 请求数据:
                    if(res.data.status_code == 0) {
                        app_module.showToast(true,res.data.message);
                        that.article.comment_count++;
                        app_module.returnCommentCount(that.article.comment_count,that.article.vote_up,that.article.vote_down,that.vote,that.ifFavorite,that.cate);
                        app_module.goAtlasCommentList(that.ID+'');
                    }else{
                        app_module.showToast(false,res.data.message);
                    }
                }else if(res.status==401){
                    app_module.goLogin();
                    app_module.showToast(false,"请重新登录");
                }else{
                    app_module.showToast(false,"网络出错");
                }
              });
            })
          }
        })
      },
      toComment(){
        // 跳到评论页面
        app_module.goAtlasCommentList(this.ID+'');
      },
      // 收藏
      Collection(){
        // 调用原生获取token
        var that = this;
        app_module.getToken(function(obj){
          if(obj.token==""){
            app_module.showToast(false,"请登录后操作");
            app_module.goLogin(); // 跳转到登录
          }else{
            stream.fetch({
              method:'POST',
              type:'json',
              url: that.baseUrl + that.ID+'/favorite',
              body:JSON.stringify({
                authorization:'Bearer '+obj.token
              })
            },function(res){
              if(res.ok){
                if(res.data.status_code==0){
                  that.collection = !that.collection;
                  that.ifFavorite = 1;
                  app_module.showToast(true, res.data.message);
                  app_module.returnCommentCount(that.article.comment_count,that.article.vote_up,that.article.vote_down,that.vote,that.ifFavorite,that.cate);
                }else if(res.data.status_code==1){
                    app_module.showToast(false,res.data.message);
                }else if(res.status==401){
                  app_module.goLogin();
                  app_module.showToast(false,"请重新登录");
                }
              }else{
                  app_module.showToast(false,"网络出错");
              }
            })
          }
        })
      },
      cancelCollection(){
        var that = this;
        app_module.getToken(function(obj){
          if(obj.token==""){
            app_module.showToast(false,"请登录后操作");
            app_module.goLogin();
          }else{
            stream.fetch({
              method:'POST',
              type:'json',
              url: that.baseUrl,
              body:JSON.stringify({
                authorization:'Bearer '+obj.token
              })
            },function(res){
              if(res.ok){
                if(res.data.status_code==0){
                  that.collection = !that.collection;
                  that.ifFavorite = 0;
                  app_module.showToast(true,res.data.message);
                  app_module.returnCommentCount(that.article.comment_count,that.article.vote_up,that.article.vote_down,that.vote,that.ifFavorite,that.cate);
                }else if(res.data.status_code==1){
                    app_module.showToast(false,res.data.message);
                }else if(res.status==401){
                  app_module.goLogin();
                  app_module.showToast(false,"请重新登录");
                }
              }else{
                  app_module.showToast(false,"网络出错");
              }
            })
          }
        })
      }
    }
  }
</script>