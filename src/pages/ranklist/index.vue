<template>
    <div class="ranklist">
        <div class="top_list" 
            v-for="(item,index) of topList" 
            :key="item.id" 
            @click="enterSongList(item.id)"
            :data-id="item.id"
            >
            <div class="top_list_img">
                <img class="picUrl" :src="item.picUrl" alt="item.topTitle">
                <div class="top_list_count">
                    <img src="./../../../static/images/radio.png" alt="">
                    {{item.listenCount}} 万
                </div>
            </div>
            <div class="top_list_info">
                <div class="info_title">{{item.topTitle}}</div>
                <div class="song_list" 
                    v-for="(ite,ind) of item.songList"
                    :key="ind"
                    >
                    {{ind+1}} 
                    <span class="song_singername"> {{ite.songname}}</span>
                    - {{ite.singername}}
                </div>
            </div>
            <img class="song_next" src="./../../../static/images/next.png" alt="">
        </div> 
    </div>
</template>

<script>
export default {
    data(){
        return {
            topList:[],
        }
    },
    methods:{
        enterSongList(id){
             wx.navigateTo({url:'/pages/songlist/main?id='+id});
        },
    },
    onShow(){
       wx.request({
         url: 'https://c.y.qq.com/v8/fcg-bin/fcg_myqq_toplist.fcg', //开发者服务器接口地址",
         data:{
            "_":'1554704775937',
            "g_tk":"5381",
            "uin":0,
            "format":'json',
            "inCharset":"utf-8",
            "outCharset":"utf-8",
            "notice":0,
            "platform":'h5',
            "needNewCode":1
         },
         success: res => {
            this.topList = res.data.data.topList;
         },
       });
    }
}
</script>


<style scoped>
.ranklist{
    background: #f4f4f2;
}
.top_list{
    position: relative;
    margin: 8px auto;
    padding-right: 20px;
    width: 96%;
    height: 100px;
    background: #fff;
    display: flex;
}
.song_next{
    position: absolute;
    right: 5px;
    top: 40px;
    width: 20px;
    height: 20px;
}
.top_list .top_list_img{
    position: relative;
    width: 100px;
    height: 100px;
}
.top_list_img .picUrl{
    width: 100px;
    height: 100px;
}
.top_list_count{
    position: absolute;
    bottom: 5px;
    height: 20px;
    width: 100px;
    line-height: 20px;
    font-size: 10px;
    color: white;
}
.top_list_count img{
    width: 16px;
    height: 16px;
}

.top_list_info{
    margin-left: 10px;
}
.info_title{
    margin-bottom: 5px;
    font-weight: bolder;
}
.song_singername{
    margin-left: 5px;
    color: #000;
}
.song_list{
  box-sizing: border-box;
  height: 20px;
  font-size: 16px;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
  word-break: break-word;
  color: #232326;
  line-height: 20px;
  margin-bottom: 3px;
  padding: 0 4px;
 /* 文字颜色  */
 color: rgba(0,0,0,.5);
}

</style>

