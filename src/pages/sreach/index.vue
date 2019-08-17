<template>
    <div class="sreach">
        <div class="sreach_demo">
            <template v-if="isShow">
                <div class="sreach_inp">
                    <icon
                    type="search"
                    size="16">
                    </icon>
                    <input type="search" 
                        @focus="hideTags"
                        autocomplete="off"
                        autocorrect="off"
                        placeholder="搜索歌曲、歌单、专辑">
                </div>
            </template>
            <template v-else>
                <div class="sreach_focue">
                    <icon
                    type="search"
                    size="16">
                    </icon>
                    <input type="text" 
                        autocomplete="on"
                        autocorrect="off"
                        confirm-type="搜索"
                        @confirm="startSreach"
                        @focus="showStorage"
                        v-model="sreachName"
                        placeholder="搜索歌曲、歌单、专辑" />
                    <span @click="tagsQuxiao">取消</span>
                    <!-- <span @click="startSreach">取消</span> -->
                    <!-- @confirm="startSreach" -->
                </div>
            </template>
        </div>
        <div class="search_result" v-show="isShow">
            <div class="result_tit">热门搜索</div>
            <div class="result_tags">
                <div class="tag_s"
                    @click="setSreachVal"
                    :data-val='item.dissname'
                    :class="{active:index==0}"
                    v-for="(item,index) of tagslist" :key="index"
                >
                    {{item.dissname}}
                </div>
            </div>
        </div>
        <div class="sreach_result" v-show="isResult">
            <div class="sreach_result_list"
            >
                <div class="sreach_result_item"
                    v-for="(item,index) of showList" :key='index'
                        :data-mid="item.mid"
                        :data-name="item.name"
                        :data-singer="item.singer"
                        @click="goToPlay"
                    >
                    <icon type="waiting" size="16" color="#999"/>
                    <p>{{item.name}}--{{item.singer}}</p>
                    <icon :data-index="index" @click.stop="delBtn" type="clear" size="16" color="#ccc"/>
                </div>
            </div>
        </div>
        <div class="show_stroage" v-show="isStorage">
            <div class="storage_result_list">
                <div class="storage_result_item" 
                    @click="getSreachClick"
                    :data-val="item"                
                    v-for="(item,index) of sreachList" :key="index">
                    <icon type="waiting" size="16" color="#999"/>
                    <p>{{item}}</p>
                    <icon :data-index="index" @click.stop="delSreachBtn" type="clear" size="16" color="#ccc"/>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data(){
        return {
            tagslist:[],
            isShow:true,
            isStorage:false,
            isResult:false,
            sreachName:'',
            sreachList:[],
            StroageList:[],
            showList:[],
        }
    },
    methods:{
        hideTags(){
            this.isShow = !this.isShow;
            this.isStorage = true;
        },
        tagsQuxiao(){
            this.isShow = true;
            this.isStorage = false;
            this.isResult = false;
            this.sreachName = '';
        },
        showStorage(){
            this.isStorage = true;
            this.isResult = false;
        },
        startSreach(e){
            if(!this.sreachName){
                return;
            }
            this.isStorage = false;
            this.isResult = true;
            console.log(this.sreachList);
            this.sreachList.push(this.sreachName);
            // 本地存储
            this.sreachList = Array.from(new Set(this.sreachList))
            wx.setStorageSync('sreachList',this.sreachList);
            // console.log(this.sreachList,wx.getStorageInfoSync('sreachList'),'获取本地的事件');
            // 开始搜索
            let url = `https://c.y.qq.com/splcloud/fcgi-bin/smartbox_new.fcg?
            is_xml=%200&format=jsonp&key=${this.sreachName}&g_tk=5381&loginUin=0&hostUin=0&format=jsonp&inCharset=utf-8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0`;
            wx.request({
              url: url, //开发者服务器接口地址",
              data: {}, //请求的参数",
              method: 'GET',
              dataType: 'json', //如果设为json，会尝试对返回的数据做一次 JSON.parse
              success: res => {
                 this.showList = res.data.data.song.itemlist;
              },
            });
            wx.setStorageSync('sreachList',this.sreachList);
        },
        goToPlay(e){
          let mid = e.currentTarget.dataset.mid;
          let songer = e.currentTarget.dataset.singer;
          let songname = e.currentTarget.dataset.name;
          let songerid = '';
          this.sreachName = '';
        wx.request({
          url: 'https://v1.itooi.cn/tencent/song', //开发者服务器接口地址",
          data: {
              id:e.currentTarget.dataset.mid
          }, //请求的参数",
          method: 'GET',
          dataType: 'json', //如果设为json，会尝试对返回的数据做一次 JSON.parse
          success: res => {
              songerid = res.data.data[0].singer[0].mid;
              console.log(songerid);
          },
        });
        wx.navigateTo({ 
            url: `/pages/songplay/main?mid=${mid}&songname=${songname}&songer=${songer}&songerid=${songerid}`
        });
        },
        delBtn(e){
            let index = e.currentTarget.dataset.index;
            this.showList.splice(index,1);
        },
        delSreachBtn(e){
            let index = e.currentTarget.dataset.index;
            this.sreachList.splice(index,1);
            wx.setStorageSync('sreachList',this.sreachList);
        },
        getSreachBtn(e){
            this.sreachName = e.currentTarget.dataset.index;
        },
        setSreachVal(e){
            console.log(e.currentTarget.dataset.val);
            this.sreachName = e.currentTarget.dataset.val;
            this.isShow = false;
            this.isStorage = true;
        }
       
    },
    onLoad(){
        wx.request({
          url: 'https://v1.itooi.cn/tencent/songList/hot', //开发者服务器接口地址",
          data: {
            pageSize:8
          }, //请求的参数",
          method: 'GET',
          dataType: 'json', //如果设为json，会尝试对返回的数据做一次 JSON.parse
          success: res => {
              this.tagslist = res.data.data.list;
          },
       
        });
    },
    onUnload(){
        this.isShow = true;
        this.isStorage = false;
        this.isResult = false;
    }
    
}
</script>

<style scoped>
    .sreach_demo{
        height: 56px;
        line-height: 56px;
        background: #f4f4f4;
        border: 1px transparent solid;
    }
    .sreach_inp{
        position: relative;
        margin: 8px auto;
        padding-top: 8px;
        width: 96%;
        height: 36px;
        background: #fff;
        border-radius: 8px;
    }

    .sreach_inp icon{
        position: absolute;
        left: 10px;
        top: 4px;
    }
    .sreach_inp input{
        margin: 0px auto;
        padding-left: 35px;
        width: 100%;
        height: 20px;
        line-height: 20px;
        border: 1px blue solid;
        outline-style: none;
        font-size: 16px;
        color: rgba(0,0,0,.3);
        border: none;
    }
    /*  获取焦点的时候 */
    .sreach_focue{
        position: relative;
        margin: 8px 0 8px 8px;
        padding-top: 8px;
        width: 80%;
        height: 36px;
        background: #fff;
        border-radius: 8px;
    }
    .sreach_focue icon{
        position: absolute;
        left: 10px;
        top: 4px;
    }
    
    .sreach_focue input{
        margin: 0px auto;
        padding-left: 35px;
        width: 100%;
        height: 20px;
        line-height: 20px;
        border: 1px blue solid;
        outline-style: none;
        font-size: 16px;
        color: rgba(0,0,0,.3);
        border: none;
    }
    .sreach_focue span{
        position: absolute;
        right: -60px;
        top: 6px;
        height: 36px;
        width: 50px;
        line-height: 36px;
        text-align: center;
        font-size: 14px;
        border: 1px #fff solid;
    }
    /* 热门搜索的列表 */
    .result_tit{
        display: block;
        font-size: 20px;
        margin-block-start: 1em;
        margin-block-end: 1em;
        margin-inline-start: 0px;
        margin-inline-end: 0px;
        font-weight: bold;
        color: rgba(0,0,0,.6);
        margin: 8px 0px 10px 12px;
    }
    .tag_s{
        margin:0 5px 10px;
        padding: 5px 12px;
        height: 30px;
        display: inline-block;
        line-height: 30px;
        color: #000;
        border: 1px solid rgba(0,0,0,.6);
        border-radius: 99px;
        word-break: keep-all;
        font-size: 14px;
    }
    .result_tags .active{
        color: #fc4524;
        border-color: #fc4524;
    }

    .sreach_result{
        margin: 10px auto;
        min-height: 300px;
        
    }
    .sreach_result_list{
       width: 100%;
       min-height: 300px;
       display: flex;
       flex-direction: column;

    }
    .sreach_result_item{
        margin: 0 auto;
        padding: 0 10px;
        height: 50px;
        width: 90%;
        border-bottom: 10px #f8f8f8 solid;
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        text-align: left !important;
    }
    .sreach_result_item p{
        width: 100%;
        padding-left: 10px;
        line-height: 50px;
        font-size: 16px;
        text-overflow:ellipsis; 
        white-space:nowrap; 
        overflow:hidden;
    }
    .show_stroage{
        margin: 10px auto;
        min-height: 300px;
    }
    .storage_result_list{
       width: 100%;
       height: 50px !important;
       display: flex;
       flex-direction: column;
    }
    .storage_result_item{
        padding: 0 10px;
        height: 80px !important;
        border-bottom: 10px #f8f8f8 solid;
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        text-align: left !important;
    }
</style>
