<template>
  <div class="course_detail">
    <y-watch-video  v-if="courseInfo.isPay" :playerOptions="playerOptions" :courseInfo="courseInfo" @playfunc="videoPlay" :nowNo="nowPeriodNo" ref="watchVideo"></y-watch-video>
    <y-display v-else :courseInfo="courseInfo" ref="watchVideo"></y-display>
    <div class=" detail_info detail_box clearfix">
      <div class="layout_left">
        <ul class="course_tab clearfix">
          <li :class="{on: tab == 'info'}"><a href="javascript:" @click="tab = 'info'">课程介绍</a></li>
          <li :class="{on: tab == 'big'}"><a href="javascript:" @click="tab = 'big'">录播课程</a></li>
        </ul>
        <div class="content_info"  v-if="tab == 'info'">
          <div class="introduce" v-html="courseInfo.introduce"></div>
          <y-syllabus @playfunc="videoPlay" :list="courseInfo.chapterList" :nowNo="nowPeriodNo"></y-syllabus>
        </div>
        <div class="content_info"  v-if="tab == 'big'">
          <y-syllabus @playfunc="videoPlay" :list="courseInfo.chapterList" :nowNo="nowPeriodNo"></y-syllabus>
        </div>
      </div>
      <div class="layout_right">
        <div class="teacher_info clearfix">
          <span class="head">讲师简介</span>
          <!-- <VueVideo :playerOptions="playerOptions" ></VueVideo> -->
          <div class="teacher_msg">
            <div class="teacher_msg_right">
              <img class="teacher_phone" v-if="teacherInfo.headImgUrl" :src="teacherInfo.headImgUrl" alt="">
              <img class="teacher_phone" v-else src="~/assets/image/friend.jpg" alt="">
              <div class="teacher_name">
                {{teacherInfo.lecturerName}}
              </div>
              <div v-html="teacherInfo.introduce" class="info_box"></div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <y-footer></y-footer>
  </div>
</template>
<script>
import YDisplay from '~/components/course/Display'
import YFooter from '~/components/common/Footer'
import YSyllabus from '~/components/course/Syllabus'
import YWatchVideo from '~/components/course/WatchVideo'
import {courseDetail, userCourseDetail, chapterSign} from '~/api/course.js'

 import VueVideo from '~/components/VueVideo'
 require('video.js/dist/video-js.css')
 require('vue-video-player/src/custom-theme.css')

export default {
  components: {
    YFooter,
    YDisplay,
    YSyllabus,
    VueVideo,
    YWatchVideo
  },
  head () {
    return {
      title: '课程详情'
    }
  },
  data () {
    return {
      tab: 'info',
      nowPeriodNo: '' , //当前播放章节
      playerOptions:{}
    }
  },
  validate ({ params }) {
    // 必须是number类型
    return /^\d+$/.test(params.id)
  },
  async asyncData(context) {
    let tk = context.store.state.tokenInfo;
    try {
      let result = new Object();
      if (tk) {

        let {data} = await userCourseDetail({courseId: context.params.id}, tk);
        if (data.code == 200) {
          result.courseInfo = data.data;
          result.teacherInfo = data.data.lecturer;
          result.isbuy = false;
          result.isLogin = false;
        }else if (data.code >= 300 && data.code <= 400){
          // context.redirect('login');
          context.store.dispatch('REDIRECT_LOGIN')
        }else{
          result.courseInfo = null;
        }
      } else{

        let {data} = await courseDetail({courseId: context.params.id});
        if (data.code == 200) {
          result.courseInfo = data.data;
          result.teacherInfo = data.data.lecturer;
          result.isbuy = false;
          result.isLogin = false;
        }else{
          result.courseInfo = null;
        }
      }
      return result
    } catch (e) {
      context.error({ message: 'User not found', statusCode: 404 })
    }

  },
  methods: {
    videoPlay (data) {
      if (this.courseInfo.isPay || data.isFree) {
      window.scrollTo(0, 0)
      this.nowPeriodNo = data.id
      chapterSign({
        ip: 'string',
        periodId: data.id,
        videoVid: data.videoVid
      }).then(res => {
        res = res.data
        this.isResetVideo = false
        if (res.code === 200) {
          this.play(Object.assign({vid: data.videoVid}, res.data));
        } else if (res.code === 402) {
          this.$msgBox({
            content: '购买后才可以观看',
            isShowCancelBtn: false
          })
        }
      }).catch(() => {
        this.isResetVideo = false
      })
      } else {
        this.$msgBox({
          content: '购买后才可以播放',
          isShowCancelBtn: false
        })
        return false;
      }
    },
    play (data) {
      let box = this.$refs.watchVideo.$refs.videobox;
      if (this.player) {
        this.player.changeVid({
          vid:data.vid,
          playsafe: data.token,
          ts: data.ts,
          sign: data.sign,
          autoplay: true
        });
      } else {
        this.player = polyvObject('#player').videoPlayer({
            'width': box.offsetWidth,
            'height': box.offsetHeight,
            'forceH5': true,
            'code': data.code,
            'playsafe': data.token,
            'ts': data.ts,
            'sign': data.sign,
            'vid': data.vid
        });
      }
    }
  },
  mounted () {
  this.playerOptions = {
                                      //播放速度
                                      playbackRates: [0.5, 1.0, 1.5, 2.0],
                                      //如果true,浏览器准备好时开始播放。
                                      autoplay: true,
                                      // 默认情况下将会消除任何音频。
                                      muted: false,
                                      // 导致视频一结束就重新开始。
                                      loop: true,
                                      // 建议浏览器在<video>加载元素后是否应该开始下载视频数据。auto浏览器选择最佳行为,立即开始加载视频（如果浏览器支持）
                                      preload: 'auto',
                                      language: 'zh-CN',
                                       sourceOrder: true,
                                       // 将播放器置于流畅模式，并在计算播放器的动态大小时使用该值。值应该代表一个比例 - 用冒号分隔的两个数字（例如"16:9"或"4:3"）
                                      aspectRatio: '16:9',

                                       // 当true时，Video.js player将拥有流体大小。换句话说，它将按比例缩放以适应其容器。
                                      fluid: true,

                                      sources: [
                                        {
                                          type: "video/mp4",
                  					              src: "http://localhost:5710/course/auth/course/play?filename=1407942667639193602.mp4" //你的m3u8地址（必填
                                        }
                                      ],
                                      //你的封面地址
                                      poster: '',
                                       //允许覆盖Video.js无法播放媒体源时显示的默认信息。
                                      notSupportedMessage: '此视频暂无法播放，请稍后再试',
                                      controlBar : {
                                                                timeDivider : true,//当前时间和持续时间的分隔符
                                                                durationDisplay : true,//显示持续时间
                                                                remainingTimeDisplay : false,//是否显示剩余时间功能
                                                                fullscreenToggle : true  //全屏按钮
                                                            }
                                  };
  }
}
</script>
<style lang="scss" rel="stylesheet/scss">
.course_detail{
  .detail_info {
    margin: 20px auto 60px;
    width: 1200px;
  }
  .layout_left {
    width: 920px;
    float: left;
    .info_body {
      margin-bottom: 30px;
    }
  }
  .course_tab{
    padding-left: 30px;
    background: #fff;
    border-radius: 8px;
    margin-bottom: 20px;
    li {
      float: left;
      height: 66px;
      line-height: 66px;
      margin-right: 80px;
      &.on {
        border-bottom: 2px solid #D51423;
        a {
          color: #D51423;
        }
      }
      a {
        color: #000;
        font-size: 14px;
        font-weight: 700;
        &:hover {
          text-decoration: none;
          color: #D51423;
        }
      }
    }
  }
  .content_info{
    padding: 30px;
    background: #fff;
    border-radius: 8px;
    min-height: 400px;
    .title {
      border-left: 3px solid #000;
      padding-left: 12px;
      font-size: 16px;
      color: #000;
      font-weight: 700;
      margin-bottom: 25px;
    }
    .introduce{
      font-size: 14px;
      line-height: 30px;
      color: #333;
      padding-left: 8px;
    }
  }
  .layout_right{
    width: 260px;
    float: right;
  }
  .teacher_info {
    background: #fff;
    border-radius: 8px;
    position: relative;
    min-height: 180px;
    .head {
      display: block;
      line-height: 60px;
      height: 60px;
      padding-left: 20px;
      font-size: 14px;
      color: #333;
      border-bottom: 1px solid rgb(228, 228, 228);
    }
    .teacher_phone {
      width: 46px;
      height: 46px;
      border-radius: 50%;
      background: rgb(228, 228, 228);
      text-align: center;
      line-height: 46px;
      font-size: 13px;
      color: #999;
      float:left;
      margin: 0px 10px 0 10px;
      img {
        width: 46px;
        height: 46px;
        border-radius: 50%;
      }
    }
    .teacher_msg_right {
      width: 235px;
      float: right;
      margin: 12px 15px 12px 12px;
      line-height: 20px;
    }
    .teacher_name {
      font-size: 14px;
      font-weight: 700;
      color: #333;
      margin-bottom: 10px;
    }
  }
}
</style>
