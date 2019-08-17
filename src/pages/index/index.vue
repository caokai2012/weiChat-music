<template>
    <div class="index">
        <swiper indicator-dots autoplay circular>
            <block v-for="(item,i) of swiperList" :key="item.id">
                <swiper-item>
                    <image
                      :src="item.picUrl"
                      mode="scaleToFill"
                      lazy-load="false">
                    </image>
                </swiper-item>
            </block>
        </swiper>
        <div class="radio">
          <div class="radio_title">{{radioTitle}}</div>
            <div class="radio_list">
              <div class="radio_list_item" 
                v-for="(item,index) of radioList" :key="item.radioid">
                  <img :src="item.picUrl" alt="item.Ftitle" title="item.Ftitle">
                  <p class="radio_list_desc">{{item.Ftitle}}</p>
                  <!-- <img class="radio_play" src="./../../../static/images/play.png" alt="">  -->
                </div>
            </div>
        </div>
        <div class="song">
          <div class="song_title">{{songListTitle}}</div>
          <div class="song_list">
            <div class="song_list_item" 
                v-for="(item,index) of songList" :key="item.id">
                  <img :src="item.picUrl" alt="item.songListAuthor" title="item.songListAuthor">
                  <p class="song_list_desc">{{item.songListDesc}}</p>
                  <p class="song_list_desc">{{item.songListAuthor}}</p>
                  <!-- <img id="erji" src="./../../../static/images/radio.png" alt="">
                  <p class="song_accessnum">{{item.accessnum}} 万</p> -->
                </div>
          </div>
        </div>
    </div>
</template>

<script>
export default {
  data () {
    return {
      swiperList:[],
      radioTitle:'电台',
      radioList:[],
      songListTitle:'热门歌单',
      songList:[],
    }
  },
  onShow(){
    console.log(this.swiperList);
    wx.showLoading({
      title: '正在加载...', //提示的内容,
    });
    wx.request({
      url: 'https://c.y.qq.com/musichall/fcgi-bin/fcg_yqqhomepagerecommend.fcg', 
      data: {
        _:1554690676597,
        g_tk:5381,
        uin:0,
        format:'json',
        inCharset:"utf-8",
        outCharset:"utf-8",
        notice:0,
        platform:'h5',
        needNewCode:1
      }, //请求的参数",
      method: 'GET',
      dataType: 'json', //如果设为json，会尝试对返回的数据做一次 JSON.parse
      success: res => {
          this.swiperList = res.data.data.slider;
          this.radioList = res.data.data.radioList;
          this.songList = res.data.data.songList;
      },
      fail: () => {},
      complete: () => {
        wx.hideLoading();
      }
    });

  }

  
}
</script>

<style scoped>
*{
  box-sizing: border-box;
}
.index{
  padding-bottom: 30px;
  background: #ebe2e3;
}
.swiper{
  width: 100%;
  height: 300px;
  border: 1px red solid;
}
/*  电台 */
.radio_title{
  padding-left: 20px;
  line-height: 35px;
  font-size: 20px;
  font-weight: bolder;
}
.radio_list{
  margin: 5px auto 0;
  width: 96%;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}
.radio_list_item{
  position: relative;
  width: 48%;
  height: 330px;
  background: #fff;
}
.radio_list_item .radio_play{
  position: absolute;
  top: 70%;
  right: 1%;
  width: 48px !important;
  height: 48px;
}
.radio_list_item img{
  width: 100%;
}
.radio_list_desc{
  padding-left: 20px;
  font-weight: bolder;
}
/* 热门歌单 */
.song_title{
  padding-left: 20px;
  line-height: 35px;
  font-size: 20px;
  font-weight: bolder;
}
.song_list{
  margin: 5px auto 0;
  width: 96%;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}
.song_list_item{
  position: relative;
  margin-bottom: 10px;
  width: 48%;
  height: 330px;
  background: #fff;
}
.song_accessnum {
  position: absolute;
  top: 75%;
  left:24%;
  color: white;
  font-size: 16px;
  line-height: 28px;
}
#erji{
  position: absolute;
  top: 75%;
  left:7%;
  width: 28px !important;
  height: 28px;
}
.song_list_item img{
  width: 100%;
}
.song_list_desc{
  box-sizing: border-box;
  height: 16px;
  font-size: 14px;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
  word-break: break-word;
  color: #232326;
  line-height: 16px;
  margin-bottom: 3px;
  padding: 0 4px;
  font-weight: bolder;
}

</style>
