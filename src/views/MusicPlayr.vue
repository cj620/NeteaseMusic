<template>
  <div class="music-playr">
    <!-- 背景 -->
    <div
      ref="song-bg"
      class="song-bg"
      :style="{backgroundImage:`url(${songInfo.picUrl})`}"
    />
    <!-- 头部 -->
    <div class="song-title">
      <div class="back" @click="playerBack"> <img src="../assets/back.svg" alt=""></div>
      {{songInfo.name}}
    </div>
<!-- 中部 -->
    <div @click="getLyric" class="content">
      <Lyric :lyric="lyric" :currentTime="currentTime"  v-show="lyricShow"/>
      <demoPic :img="songInfo.picUrl" v-show="!lyricShow"  :stopPop="stopPop"/>
    </div>
    <!-- 进度条 -->
    <div class="time-line">
      <!-- <TimeLine :currentTime="currentTime" /> -->
      <van-slider v-model="value" @change="onChange" active-color="#443e8f9d" button-size="10px" />
    </div>
    <!-- 底部控制条 -->
 <div class="bottom">
   <controllBtn @stop="change"  :isStop='isStop' @next="nextSong" @prev="preSong" />
 </div>
  <audio :src="musicUrl" autoplay ref='audio' v-show="false"></audio>
    </div>
</template>
<script>
import { getMusicDetail ,getMusicUrl,getMusicLyric} from '@/api/api'
import Lyric from '@/components/Player/Lyric'
import controllBtn from '@/components/Player/ControllBtn'
import demoPic from '@/components/Player/demoPic'
import { Slider , Toast} from 'vant'
// import TimeLine from '@/components/Player/TimeLine'
  export default {
    components:{
      Lyric,
      controllBtn,
      demoPic,
      // TimeLine
    },
    data() {
      return {
        musicId: this.$route.query.id,  //当前点击的歌曲id
        lyric:'',
        musicUrl:'',
        isStop:true,
        currentTime:0,
        timer:null ,
        isStop:false,
        idIndex:0,
        lyricShow:false,
        stopPop:false,
        value:1,
        playCount:0,
        songInfo: {
        picUrl: '',
        name: '',
       }
      }
    },
    created(){
      this.init()
      this.$nextTick(
        ()=>{
          this.updateTime()  
          this.$refs.audio.play() 
        }
      )
    },

    beforeDestroy(){
      clearInterval(this.timer)
    },
    methods:{
      async init(){
      await getMusicUrl(this.musicId).then(res=>this.musicUrl=res.data[0].url)  //得到歌曲地址
      await getMusicDetail(this.musicId).then(res => {
        this.songInfo = res.songs[0].al;
        // console.log(this.songInfo);   
      })
      await getMusicLyric(this.musicId).then(res =>{
        this.lyric = res.lrc.lyric        
      })
      },
    change(state){      
      this.isStop = state;
      if(!state){
        this.$refs.audio.play()
        this.stopPop = false
        this.updateTime();
      }else{
        this.$refs.audio.pause();
        this.stopPop =true
        clearInterval(this.timer)
      }
    },
    updateTime(){
      this.timer = setInterval(() => {
          this.currentTime = this.$refs.audio.currentTime;  
          this.value = parseInt(this.$refs.audio.currentTime/this.$refs.audio.duration*100)+1   
              if(this.$refs.audio.currentTime == this.$refs.audio.duration){
                    this.change()       //循环播放
              }
      }, 10);
    },
    getLyric(){
      this.lyricShow = !this.lyricShow
      // console.log("-----");
    },
    onChange(value) {
      Toast('当前进度：' + value+'%');
      // console.log(parseInt(this.$refs.audio.currentTime/this.$refs.audio.duration*100)+1);  //
      this.$refs.audio.currentTime = value*this.$refs.audio.duration/100
      
    },
    playerBack(){
      this.$router.go(-1)
    },
    nextSong(){
      // console.log(this.$store.state.trackIds[this.playCount].id);
      // getMusicUrl(this.$store.state.trackIds[0].id).then(res =>console.log(res.data[0].url))
      if(this.$store.state.trackIds.length != 0){
        this.playCount++
      this.musicId = this.$store.state.trackIds[this.playCount].id
      clearInterval(this.timer)
      this.init()
      this.updateTime();               //重启进度条
      }
    },
    preSong(){  
      // console.log("=====");
      if(this.$store.state.trackIds.length != 0){
        if(this.playCount<=0){
        this.playCount =0
      }else{
        this.playCount--
      this.musicId = this.$store.state.trackIds[this.playCount].id
      clearInterval(this.timer)
      this.init()
      this.updateTime();
      }
      }
      
    }
    
    }
  }
</script>

<style lang="scss" scoped>
.music-playr{
  font-size: 20px;
  .song-bg {
    position: fixed;
    left: 0;
    right: 0;
    top: -50px;
    height: 100%;
    background-color: #161824;
    background-position: 50%;
    background-repeat: no-repeat;
    background-size: auto 100%;
    transform-origin: center top;
    transform: scale(1.5);
    transition: opacity 0.3s linear;
    z-index: -1;
    filter: blur(16px) brightness(50%);
  }
  .song-title{
    position: fixed;
    top: 0;
    height: 10vh;
    width: 100vw;
    padding: 0 5vh;
    text-align: center;
    line-height: 10vh;
    color: #ffffff9d;
    font-size: 7vw;
    overflow: hidden;
	  text-overflow:ellipsis;
    white-space:nowrap;
    z-index: 5;
    .back{
      position: fixed;
      left: 0;
      >img{
        width: 10vw;
        margin-top: 4px;
      }
    }
  }
  .content{
    position: fixed;
    top: 10vh;
    height: 50vh;
    width: 100vw;
    display: flex;
    justify-content: center;
  }
  .bottom{
    position: fixed;
    top: 80vh;
    z-index: 5;
  }
  .time-line{
    position: fixed;
    top: 75vh;
    height: 10vh;
    width: 100vw;
    // background-color: pink;
    display: flex;
    align-items: center;
    z-index: 6;
  }
}
 
</style>