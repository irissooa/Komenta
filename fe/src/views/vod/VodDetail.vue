<template>
  <div>
    <div id="appBody">
      <div class="video">
        <video @loadstart="goLastVod" class="video__height" ref="video" id="videotag" controls="controls" @timeupdate="onTimeUpdate" width="100%" height="100%">
            <source :src="getVideo()" id="player" type='video/mp4'/>
        </video>    
        <div id="comment_div">
          <div class="comment__scroll" id="comment__scroll">
            <div v-for="(comment,index) in commentsList" :key="index" class="comment__text"> <!--  @mousewheel="stopScroll"  -->
              <div v-show="comment.c_playtime <= nowTime(videoCurrentTime)" class="comment__lineheight" >
                <span class="comment__time" @click="goCommentTime(timeToSec(comment.c_playtime))"> {{comment.c_playtime}}</span>
                <span @click="goFeed(comment.u_id)" class="comment__nickname" :class=" {comment__highlight:userFollowing(comment.u_id)}">{{comment.u_nickname}} </span> 
                <template v-if="itsMe(comment.u_id)">
                  <span>{{ comment.c_contents}} </span>
                  <span @click="commentLike(index,comment)">
                    <font-awesome-icon :icon="['fas', 'thumbs-up']" :id="`like-btn-${comment.c_id}`" class="comment_like_btn" :class="[comment.is_like_comment ? 'commet__like' :' comment__unlike' ]" style="cursor:pointer"/>
                    <span :id="`like-cnt-${comment.c_id}`"  class="comment__block margin_right" :class="[comment.is_like_comment ? 'commet__like' :' comment__unlike' ]">{{ comment.comment_good_count }}</span>
                  </span>    
                  <span class="comment__block" @click="DeleteComment(comment.c_id)">삭제</span>
                </template>
                <template v-else-if="userBlocking(comment.u_id)">
                  <span class="comment__block__content">차단한 댓글입니다. </span>
                  <span @click="blockUser(comment.u_id)" class="comment__block margin_right">차단취소</span>
                </template>
                <template v-else>
                  <span>{{ comment.c_contents}} </span> 
                  <span @click="commentLike(index,comment)">
                    <font-awesome-icon :icon="['fas', 'thumbs-up']" :id="`like-btn-${comment.c_id}`" class="comment_like_btn" :class="[comment.is_like_comment ? 'commet__like' :' comment__unlike' ]" style="cursor:pointer"/>
                    <span :id="`like-cnt-${comment.c_id}`"  class="comment__block margin_right" :class="[comment.is_like_comment ? 'commet__like' :' comment__unlike' ]">{{ comment.comment_good_count }}</span>
                  </span>    
                  <span @click="blockUser(comment.u_id)" class="comment__block margin_right">차단하기</span>
                </template>
              </div>
            </div>
          </div>
          <div class="video__comment">
            <div class="video__intro">
              <span @mouseover="showIntro" @mouseleave="closeIntro">
                <font-awesome-icon :icon="['far','question-circle']"  @mouseover="showIntro" @mouseleave="closeIntro"/>
              </span>
              <div class="intro-drag-and-drop vod-detail" :style="{display:isShowIntro}">
                <div class="triangle"></div>
                <div class="img-box">
                  <img :src="getIntro()" alt="">
                </div>
              </div>
            </div>
            <div class="video__comment__box">
              <img :src="getPoster()" alt="" class="video__comment__profile">
            </div>
            <div class="video__comment__inner">
              <span class="video__comment__inner__nickname">{{userInfo.u_nickname}}</span> <br>
              <div>
                <input type='text' class="video__comment__input" id=msg v-model.trim ="userComment" :placeholder="getUserBlockedInfo()" @keydown.enter="createComment()" :class="{'blockuser__placeholder':userInfo.is_blocked}" :disabled="userInfo.is_blocked"/>
                <span @click="createComment()"><i class="far fa-paper-plane"></i></span>
              </div>
            </div>
          </div>
        </div>
      </div>
      <hr>
      <div class="vodepi">
        <div class="vodepi__img">
          <img :src="getVodPoster(vodEpiInfo.v_poster)" alt="" width="200px">
        </div>
        <div class="vodepi__detail">
          <h3>
            <strong> {{vodEpiInfo.v_title}} {{vodEpiInfo.ve_episode_num}}회</strong>
          </h3>
          <h5> {{vodEpiInfo.ve_contents}}</h5>
          {{vodEpiInfo.v_summary}} <br>
          <p></p>
          <strong>개요</strong> {{vodEpiInfo.g_name}}/{{vodEpiInfo.gd_name}} <br>
          <strong>출연</strong> {{vodEpiInfo.v_actors}} <br>
          <strong>감독</strong> {{vodEpiInfo.v_director}}
        </div>
      </div>
      <br>
      <div class="vodEpiAll">
        <div v-show="!showVodDetail" @click="showVod()" class="vodepi__button"> 
          <span class="vodepi__button__title"> 
            <strong>전체 회차 더보기</strong> 
            <i class="fas fa-sort-down"></i>
          </span> 
        </div>
        <div v-show="showVodDetail">
          <div>
            <h4><strong>전체회차</strong></h4>
          </div>
          <div class="btn-cover">
            <button :disabled="pageNum === 0" @click="prevPage" class="page-btn">
              <font-awesome-icon :icon="['fas', 'angle-left']"/>
            </button>
            <span class="page-count">{{pageNum+1}}/{{pageCount}} </span>
            <button :disabled="pageNum >= pageCount-1" @click="nextPage" class="page-btn">
              <font-awesome-icon :icon="['fas', 'angle-right']"/>
            </button>
          </div>
          <div class="vod__detail">
            <div v-for="(vod,index) in paginatedData" :key="index" class="vod__detail__column">
              <div class="at-user vod__detail__img" @click="goVodEpi(vod.ve_id)"><img :src="getVodPoster(vodEpiInfo.v_poster)" alt="" height="150px">
                <div class="vod__detail__num">{{vod.ve_episode_num}}회 </div>
              </div>
            </div>
          </div>
          <br>
          <div @click="showVod()" class="vodepi__button">
            <strong>닫기</strong><i class="fas fa-sort-up"></i>
          </div>
        </div>
      </div>
      <br>
      <div class="comments__container"> 
        <router-link :to="{name:'BestComments'}" active-class="comments__menu">
        <i class="fas fa-check"></i> BEST </router-link>
        <router-link :to="{name:'AllComments'}" active-class="comments__menu">
        <i class="fas fa-check"></i> 전체 댓글 </router-link>
        <router-view  @goCommentTime="goCommentTime" @commentLike="commentLikeEmit" ></router-view>
      </div>
    </div>
  </div>
</template>

<script>
import { startVodWatch, fetchVodEpiDetail, fetchVodDetail, endVodWatch } from '@/api/vod'
import { modifyunfollow } from '@/api/user'
import { fetchEpiComment, userlikeComment, commentInsert ,removeComment } from '@/api/comment'
import { mapState } from 'vuex';

export default {
  name: 'VodDetail',
  data(){
    return {
      pageNum:0,
      // vod epi 세부정보
      vodEpiInfo :{},
      // vod 세부정보
      vodInfo : [],
      commentsList:[],
      selectedId:0,
      videoCurrentTime: 0,
      userComment:'',
      now: "00:00:00",
      pre_diffHeight :0,
      bottom_flag : true,
      showEpiDetail:true,
      showVodDetail:true,
      isShowIntro: 'none',
    }
  },
  created(){
    this.getVodEpi();
    this.getEpiComment();
    this.getUserBlockedInfo();
    this.$store.dispatch('FETCH_FOLLOWING',this.userInfo.u_id)
    this.$store.dispatch('FETCH_UNFOLLOWING',this.userInfo.u_id)
    this.startWatchTime();
  },
  methods : {
    showIntro(){
      this.isShowIntro = 'block';
    },
    closeIntro(){
      this.isShowIntro = 'none';
    },
    getIntro(){
      return require('@/assets/images/vod-detail.gif');
    },
    DeleteComment(cId){
    removeComment(cId).then(()=>{
      this.getEpiComment();
    })
    },
    itsMe(uId) {
      if (this.userInfo.u_id == uId) {
        return true
      }
      return false
    },
    async blockUser(uId) {
      const blockInfo = {
        u_id : this.userInfo.u_id,
        uf_id : uId
      }
      await modifyunfollow(blockInfo)
      this.$store.dispatch('FETCH_UNFOLLOWING',this.userInfo.u_id)
    },
    goVodEpi(veId){
      this.$router.push(`/voddetail/${veId}`).catch(error => {
        if(error.name === "NavigationDuplicated" ){
          location.reload();
        }
      });
    },
    nextPage() {
      this.pageNum += 1;
    },
    prevPage() {
      this.pageNum -= 1;
    },
    showVod(){
      if (this.showVodDetail) {
        this.showVodDetail = false
      } else {
        this.showVodDetail = true
      }
    },
    showEpi(){
      if (this.showEpiDetail) {
        this.showEpiDetail = false
      } else {
        this.showEpiDetail = true
      }
    },
    getPoster(){
      const profile = this.userInfo.u_profile_pic.split('.')
      return `${process.env.VUE_APP_PICTURE}profile/${profile[0]}`
    },
    startWatchTime(){
      const edID = Number(this.$route.params.id)
      startVodWatch(edID);
    },
    goLastVod(){
      const endtime = document.getElementById("videotag");
      if (this.vodEpiInfo.vh_watching_time) {
        endtime.currentTime = this.timeToSec(this.vodEpiInfo.vh_watching_time)
      }
    },
    goFeed(uId){
      this.$router.push(`/feed/${uId}`)
    },
    getVodPoster(poster){
      return `${process.env.VUE_APP_PICTURE}poster/${poster}`
    },
    async getVodEpi() {
      try {
        const epiId = Number(this.$route.params.id);
        const res = await fetchVodEpiDetail(epiId)
        this.vodEpiInfo = res.data
        this.getVodDetail();
      } catch {
        console.log('vod epi detail에러')
      }
    },
    async getVodDetail() {
      try {
        const res = await fetchVodDetail(this.vodEpiInfo.v_id)
        this.vodInfo = res.data
      } catch {
        console.log('vod episode 에러')
      }
    },
    async getEpiComment() {
      try {
        const epiId = this.$route.params.id; 
        const res = await fetchEpiComment(epiId)
        this.commentsList = res.data
        this.commentsList.sort(function (a,b) {
          return a.c_playtime < b.c_playtime ? -1 :a.c_playtime > b.c_playtime ? 1:0;
        })
      } catch {
        console.log('epicomment 에러!!')
      }
    },
    onscroll() {
      const objDiv = document.getElementById("comment_div");
      if((objDiv.scrollTop + objDiv.clientHeight) == objDiv.scrollHeight){
        // 채팅창 전체높이 + 스크롤높이가 스크롤 전체높이와 같다면
        // 이는 스크롤이 바닥을 향해있다는것이므로
        // 스크롤 바닥을 유지하도록 플래그 설정
        this.bottom_flag = true;
      }
      if(this.pre_diffHeight > objDiv.scrollTop + objDiv.clientHeight ){
          // 스크롤이 한번이라도 바닥이 아닌 위로 상승하는 액션이 발생할 경우
          // 스크롤 바닥유지 플래그 해제
          this.bottom_flag = false;  
      }
    },
    nowTime(num){
      let myNum = parseInt(num, 10);
      let hours   = Math.floor(myNum / 3600);
      let minutes = Math.floor((myNum - (hours * 3600)) / 60);
      let seconds = myNum - (hours * 3600) - (minutes * 60);

      if (hours   < 10) {hours   = "0"+hours;}
      if (minutes < 10) {minutes = "0"+minutes;}
      if (seconds < 10) {seconds = "0"+seconds;}
      return hours+':'+minutes+':'+seconds;
    },
    timeToSec(time){
      let splitTime = time.split(':')
      let changeTime = Number(splitTime[splitTime.length-1])
      for (let i = splitTime.length-2; i >= 0; i--) {
          let element = Number(splitTime[i]);
          changeTime += element*(60**(splitTime.length-i-1))
      }
      return changeTime
    },
    getCurTime() { 
      const vod = document.getElementById("videotag");
      alert(vod.currentTime,'현재시간?');
    },
    // 비디오 불러오기
    getVideo() {
      const path =`${process.env.VUE_APP_VIDEO}${this.vodEpiInfo.gd_id}_${this.vodEpiInfo.v_title.replace(/(\s*)/g, "")}_${this.vodEpiInfo.ve_episode_num}화`
      return path
    },
    // 해당시간으로 댓글 이동
    goCommentTime(time){
      const vod = document.getElementById("videotag");
      vod.currentTime = time;
    },
    // 비디오 시간과 currentTIme 일치시킴
    onTimeUpdate(){
      const vod = document.getElementById("videotag");
      this.videoCurrentTime = vod.currentTime;
    },
    async createComment() {
      if (this.userComment.trim()=='') {
        return
      }
      try {
        const vod = document.getElementById("videotag");
        const commentInfo = {
          c_contents : this.userComment,
          c_playtime : this.nowTime(vod.currentTime),
          u_id : this.userInfo.u_id,
          ve_id : this.$route.params.id
        }
        await commentInsert(commentInfo)
        this.getEpiComment();
        this.userComment=""
      } catch {
          console.log('댓글썼는데 실패함') 
      }
    },
    scrollEvent (){
      let _documentY = document.documentElement.scrollTop;
      // let _direction = _documentY - window.__scrollPosition >= 0 ? 1 : -1;

      window.__scrollPosition = _documentY; // Update scrollY
    },
    // 팔로잉들 댓글 강조
    userFollowing(uId){
      for (let i = 0; i < this.myFollowingList.length; i++) {
        const following = this.myFollowingList[i];
        if (following.f_id == uId) {
          return true
        }
      }
      return false
          
    },
    // 차단한 댓글 강조
    userBlocking(uId){
      for (let i = 0; i < this.myUnFollowingList.length; i++) {
        const blocking = this.myUnFollowingList[i];
        if (blocking.f_id == uId) {      
          return true
          
        }
      }
      return false    
    },
    commentLikeEmit(cId){
      const likeBtn = document.querySelector(`#like-btn-${cId}`)
      likeBtn.classList.toggle('comment__like')
      likeBtn.classList.toggle('comment__unlike')
    },
    //유저 댓글 좋아요 class추가/제거
    commentLike(index,comment){
      const likeBtn = document.querySelector(`#like-btn-${comment.c_id}`)
      this.commentsList[index].is_like_comment = !this.commentsList[index].is_like_comment
      if (this.commentsList[index].is_like_comment){
        this.commentsList[index].comment_good_count += 1
        likeBtn.style.color = '#fc3c44'
      } else {
        this.commentsList[index].comment_good_count -=1
        likeBtn.style.color ='grey'
      }
      const commentInfo = {
        c_id : comment.c_id,
        u_id : this.userInfo.u_id
      }
      userlikeComment(commentInfo)
    },
    getUserBlockedInfo(){
      if(this.userInfo.is_blocked) {
        return '제한회원은 댓글 기능을 이용하실 수 없습니다'
      }else {
        return '댓글을 입력하세요'
      }
    }
  },
  watch : {
    // 비디오 시간을 보며 스크롤 자동으로 내리기
    videoCurrentTime :function (){
        const scrollDiv = document.getElementById('comment__scroll');
        scrollDiv.scrollTop = scrollDiv.scrollHeight;
    },    
  },
  props: {
    pageSize: {
      type: Number,
      required: false,
      default: 4
    }
  },
  computed:{
    pageCount() {
      let listLeng = this.vodInfo.length,
          listSize = this.pageSize,
          page = Math.floor(listLeng / listSize);

      if(listLeng % listSize > 0) page += 1;
      return page;
    },
    paginatedData() {
      const start = this.pageNum * this.pageSize,
            end = start + this.pageSize;

      return this.vodInfo.slice(start, end);
    },
    ...mapState({
      userInfo: state => state.user.userInfo,
      myFollowingList: state => state.user.myFollowingList,
      myUnFollowingList: state => state.user.myUnFollowingList,
    }),
  },
  beforeDestroy(){
    const watching = {
      ve_id: this.vodEpiInfo.ve_id,
      vh_watching_time: this.nowTime(this.videoCurrentTime)
    }
    endVodWatch(watching);
  }
};
</script>

<style scoped>
</style>