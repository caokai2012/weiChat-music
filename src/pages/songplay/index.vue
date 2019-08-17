<template>
    <div class="song_play">
         <div class="frontPicUrl">
            <img class="typePic" src="./../../../static/images/playbg.jpg" alt="">
             <ul class="show_words" ref="scrollTop">
                 <li v-for="(item,index) of res" :key="index"
                    :class="{activeWords:(index == 2)}"
                 >
                     {{item}}
                </li>
             </ul>
            <p>歌曲： {{songname}}</p>
            <p>演唱：{{songer}}</p>
            <div class="play_option" @click='btnClick'>
                <button data-type='stop' class="play_option_btn">
                    <img src="./../../../static/images/stopplay.png" alt="">
                </button>
                <button data-type='play' class="play_option_btn">
                    <img src="./../../../static/images/startPlay.png" alt="">
                </button>
                <button data-type='speed' class="play_option_btn">
                    <img src="./../../../static/images/speed.png" alt="">
                </button>
            </div>
        </div>
        <div class="other_song_info">
            <div class="other_song_title">{{songer}}的歌曲</div>
            <div class="other_song_list">
                <div class="other_song_item" 
                    @click="playSong"
                    :data-mid="item.musicData.songmid"
                    :data-name="item.musicData.albumname"
                    :data-albumid="item.musicData.albummid"
                    :data-songerid="item.musicData.singer[0].mid"
                    :data-songer="item.musicData.singer[0].name"
                    v-for="(item,index) of songList" :key="index">
                    <img class="song_logo" src="./../../../static/images/playlogo.jpg" alt="">
                    <p>{{item.musicData.songname}} -- {{item.musicData.albumname}}</p>
                    <img class="song_next" src="./../../../static/images/next.png" alt=""> 
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data(){
        return {
            mid:'',
            songname:'',
            songer:'',
            imgUrl:'',
            songWords:[],
            showWords:{},
            songList:[],
            words:[],
            res:[]
        }
    },
    methods:{
        parseLyric(lrc) {
            var lyrics = lrc.split("\n");
            var lrcObj = {};
            for(var i=0;i<lyrics.length;i++){
                var lyric = decodeURIComponent(lyrics[i]);
                var timeReg = /\[\d*:\d*((\.|\:)\d*)*\]/g;
                var timeRegExpArr = lyric.match(timeReg);
                if(!timeRegExpArr)continue;
                var clause = lyric.replace(timeReg,'');
                for(var k = 0,h = timeRegExpArr.length;k < h;k++) {
                    var t = timeRegExpArr[k];
                    var min = Number(String(t.match(/\[\d*/i)).slice(1)),
                        sec = Number(String(t.match(/\:\d*/i)).slice(1));
                    var time = min * 60 + sec;
                    lrcObj[time] = clause;
                }
            }
            return this.delNull(lrcObj);
        },
        delNull(lrcObj){
            for(let key in  lrcObj){
                if(lrcObj[key] == ''){
                    delete lrcObj[key];
                }
            }
            return lrcObj;
        },
        // 点击播放事件
        btnClick(e){
           let type = e.target.dataset.type;
           let audioManager = wx.getBackgroundAudioManager();
            switch(type){
                case 'stop':
                    audioManager.stop();  
                    wx.seekBackgroundAudio({
                        position: 0
                    });  
                    break;
                case 'play':
                    wx.playBackgroundAudio({
                        dataUrl:`https://v1.itooi.cn/tencent/url?id=${this.mid}&quality=128`,
                        title:this.songname,
                    });
                    this.setSongWords();
                    break;
                case 'speed':
                    let time = audioManager.currentTime;
                    time += 30;
                    audioManager.seek(time);
                    break;
                default:break;
            }
        },
        setSongWords(time){
            let showWords = Object.keys(this.songWords);
            this.words = Object.values(this.songWords);
            let len = Object.keys(this.songWords).length;
           wx.getBackgroundAudioManager().onTimeUpdate((res)=>{
            wx.getBackgroundAudioPlayerState({
                success:(res)=>{
                    let currentPosition = res.currentPosition
                    let duration = res.duration
                    let downloadPercent = res.downloadPercent
                    // 
                    let indx =  showWords.findIndex((item)=>{
                        return item == currentPosition;
                    });
                    // 开始处理歌词的显示与否
                    if(indx > 0){
                        if(indx < 4){
                            this.res =  this.words.slice(0,5);
                        }else{
                            this.res =  this.words.slice(indx-2,indx+3);
                        }
                    }   
                }
            });
        });
        },
        playSong(e){
            let songName = e.currentTarget.dataset.name;
            let songer = e.currentTarget.dataset.songer;
            let mid = e.currentTarget.dataset.mid;
            let songerid  = e.currentTarget.dataset.songerid ;
            wx.getBackgroundAudioManager().stop();
            wx.navigateTo({ 
                url: `/pages/songplay/main?mid=${mid}&songname=${songName}&songer=${songer}&songerid=${songerid}`
             });
        },
        
    },
    computed:{
        
    },
    onLoad(options){
        this.mid = options.mid,
        this.songname = options.songname,
        this.songer = options.songer;
        this.songerid = options.songerid;
        this.res = [];
        //先获取歌词
           wx.request({
              url: 'https://v1.itooi.cn/tencent/lrc', //开发者服务器接口地址",
              data: {
                  id:this.mid
              }, 
              method: 'GET',
              dataType: 'json', //如果设为json，会尝试对返回的数据做一次 JSON.parse
              success: res => {
                this.songWords = res.data;
                this.songWords = this.parseLyric(this.songWords); 
              }
            });    
            // 获取歌手的音乐
            wx.request({
              url: 'https://v1.itooi.cn/tencent/song/artist', 
              data: {
                id:this.songerid,
                pageSize:8,
              }, 
              method: 'GET',
              dataType: 'json', 
              success: res => {
                  this.songList = res.data.data;
              }
            });
    },
   onUnload(){
       wx.getBackgroundAudioManager().stop();
    }
}
</script>


<style scoped>
.song_play{
    width: 100%;
    height: 100%;
    background: #f8f8f8;
    border: 1px transparent solid;
}
.frontPicUrl{
    position: relative;
    margin: 10px auto;
    width: 80%;
    height: 60%;
    text-align: center;
    font-size: 16px;
}
.frontPicUrl .typePic{
    width: 100%;
    height: 300px;
    border-radius: 20px;
    box-shadow: 0 6px 3px rgba(24,24,24,0.5);
}
.frontPicUrl p{
    line-height: 30px;
    font-weight: bolder;
    color: #22d59c;
}
.play_option{
    padding: 10px;
    width: 100%;
    height: 50px;
    display: flex;
    justify-content: space-around;
    align-items: center;
}
.play_option_btn{
    width: 80px;
    display: block;
    height: 50px;
    border-radius: .4rem;
    box-shadow: 0 6px 32px rgba(24,213,156,.5);
    background-color: #22d59c;
    display: -webkit-box;
    -webkit-box-align: center;
    -webkit-box-pack: center;
}
.play_option_btn img{
    margin: 8px 0 0 0px;
    width: 32px;
    height: 32px;
}
/*  其他的信息 */
.other_song_info{
    margin-bottom: 10px;
}
.other_song_title{
    margin: 30px 20px;
    text-align: left;
    font-weight: bolder;
    color: #22d59c;
}
.other_song_list{
    padding: 0 10px;
}
.other_song_item{
    margin: 10px auto;
    height: 80px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: #fff;
}
.other_song_item p{
    padding-left: 20px;
    width: 250px;
    font-size: 18px;
    text-overflow:ellipsis; 
    white-space:nowrap; 
    overflow:hidden;
    /* color: skyblue; */
    font-weight: bolder;
}
.song_logo{
    width: 80px;
    height: 80px;
}
.song_next{
    width: 32px;
    height: 32px;
}
.show_words{
    margin-top: 20px;
    padding: 20px 0;
    position: absolute;
    top: 0px;
    left: 0px;
    width: 100%;
    height: 300px;
    overflow: hidden;
    transition: 1s;
}
.show_words ul{
    margin-top: 30px;
    height: 200px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-around;
    overflow: hidden;
}
.show_words li {
    margin: 5px auto;
    height: 40px;
    width: 90%;
    /* line-height: 50px; */
    color: aquamarine;
}
.show_words .activeWords{
    color: red;
}
</style>
