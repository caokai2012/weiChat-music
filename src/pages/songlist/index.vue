<template>
    <div class="songlist">
        <div class="frontPicUrl">
            <img class="typePic" :src="songInfoType.frontPicUrl" alt="">
            <p>{{songInfoType.title}}</p>
            <p>{{songInfoType.titleDetail}}</p>
            <p>更新时间：{{songInfoType.updateTime}}</p>
            <button class="song_play" @click="startPlay">
                <img src="./../../../static/images/startPlay.png" alt="">
            </button>
        </div>
        <div class="song_list">
            <div class="list_title">
                排行榜 共 {{ songInfoType.totalNum}} 首
            </div>
            <div class="song_list_item" v-for="(item,index) of songInfoList"
                :key="item.id"
                :data-mid="item.mid"
                :data-name="item.album.name"
                :data-albumid="item.album.mid"
                :data-songerid="item.singer[0].mid"
                :data-songer="item.singer[0].name"
                @click="playSong"
            >
                <div class="song_name"> <span :class="{active: index<3}">{{index+1}}  </span> <span>{{item.title}}</span> </div>
                <div class="song_rank"> 
                    <img class="song_dkw" src="./../../../static/images/dkw.png" alt="">
                    {{item.interval}} {{item.singer[0].name}} {{item.album.subtitle}}
                </div>
                <img class="song_download" src="./../../../static/images/download1.png" alt="">
            </div>
        </div>
    </div>    
</template>


<script>
const audio = wx.createInnerAudioContext();

export default {
    data(){
        return{
            id:'',
            songInfoType:{},
            songInfoList:[],
            musicList:[],
        }
    },
    methods:{
        startPlay(){
            this.musicList = this.songInfoList.map((item,index,key)=>{
               return item.mid;
            });
            let index = 0;
            wx.playBackgroundAudio({
                dataUrl:`https://v1.itooi.cn/tencent/url?id=${this.songInfoList[index].mid}&quality=128`,
                title:this.songname,
            });

            wx.getBackgroundAudioManager().onTimeUpdate((res)=>{
            wx.getBackgroundAudioPlayerState({
                success:(res)=>{
                    let status = res.status;
                    if(status == 2){
                        index++;
                        index = index > this.songInfoList.length-1 ? 0 : index; 
                        // wx.getBackgroundAudioManager().stop();
                       wx.playBackgroundAudio({
                           dataUrl:`https://v1.itooi.cn/tencent/url?id=${this.songInfoList[index].mid}&quality=128`,
                          title:this.songname,
                        });
                    }
                }
            });
        });

        },
        playSong(e){
            // 跳转到详情的页面开始进行播放 
            let songName = e.currentTarget.dataset.name;
            let songer = e.currentTarget.dataset.songer;
            let mid = e.currentTarget.dataset.mid;
            let songerid  = e.currentTarget.dataset.songerid ;
            wx.navigateTo({ 
                url: `/pages/songplay/main?mid=${mid}&songname=${songName}&songer=${songer}&songerid=${songerid}`
             });
        },
    },
    onLoad(options){
        this.id = options.id;
    },
    onShow(){
        wx.request({
          url: 'https://u.y.qq.com/cgi-bin/musicu.fcg?'+
          '-=getUCGI6688677494328337&g_tk=1789291141&loginUin=&hostUin=0&'+
          'format=json&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq.json'+
          '&needNewCode=0&data={%22detail%22:'+
          '{%22module%22:%22musicToplist.ToplistInfoServer%22,%22method%22:%22GetDetail%22,%22param%22:'+
          '{%22topId%22:'+ this.id+',%22offset%22:0,%22num%22:20,%22period%22:%22%22}},'+
          '%22comm%22:{%22ct%22:'+ this.id+',%22cv%22:0}}', //开发者服务器接口地址",
          data: {}, 
          method: 'GET',
          dataType: 'json', //如果设为json，会尝试对返回的数据做一次 JSON.parse
          success: res => {
            this.songInfoType = res.data.detail.data.data;
            this.songInfoList = res.data.detail.data.songInfoList
          },
        });
    },
    onUnload(){
       wx.getBackgroundAudioManager().stop();
    }
}
</script>


<style scoped>
.songlist{
    width: 100%;
    background: #f8f8f8;
}

.frontPicUrl{
    margin: 10px auto;
    width: 200px;
    height: 350px;
    text-align: center;
    font-size: 16px;
}
.frontPicUrl .typePic{
    width: 200px;
    height: 200px;
    border-radius: 20px;
    box-shadow: 0 6px 3px rgba(24,24,24,0.5);
}
.frontPicUrl p{
    height: 30px;
    line-height: 30px;
}
.song_play{
    margin-top: 10px;
    width: 80%;
    display: block;
    height: 40px;
    border-radius: .4rem;
    box-shadow: 0 6px 32px rgba(24,213,156,.5);
    background-color: #22d59c;
    display: -webkit-box;
    -webkit-box-align: center;
    -webkit-box-pack: center;
}
.song_play img{
    margin-top: -10px;
    width: 32px;
    height: 32px;
}
/*  循环列表 */
.song_list{
    margin: 5px auto;
    width: 92%;
}
.list_title{
    padding: 10px;
    font-size: 18px;
    color: grey;
}
.song_list_item{
    position: relative;
    margin-bottom: 20px;
    font-size: 15px;
    color: rgba(0,0,0,.4);
    box-shadow:0 6px 3px #f8f8f8;
}
.song_name{
    padding-left: 10px;
    margin: 5px 0;
}
.song_name span:nth-child(1){
    color: rgba(0,0,0,.4);
}
.song_name span:nth-child(2){
    margin-left: 5px;
    color: #000;
    text-overflow:ellipsis; 
    white-space:nowrap; 
    overflow:hidden;
}
.song_name .active{
    color: #f00 !important;

}
.song_rank{
    padding-right: 10px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space:nowrap; 
}
.song_rank .song_dkw{
    width: 8px;
    height: 8px;
}
.song_download{
    position: absolute;
    right: 10px;
    top: 10px;
    width: 16px;
    height: 16px;
}
</style>
