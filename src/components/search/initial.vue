<template>
  <div class="search-wrap">
    <!--<p>{{searchVal.value}}</p>-->
    <!--<input autofocus class="val-search" placeholder="搜索歌曲、歌手、专辑" ref="searchVal" @click="search_val()" type="text">-->
    <input autofocus class="val-search" placeholder="搜索歌曲、歌手、专辑" v-model.trim="songVal" v-on:input="monitorVal"  type="text">
    <div v-if="hot_list" class="hot-search-wrap">
      <p>热门搜索</p>
      <a class="hotsearch" v-for="(item,index) in random_search_list" :href="item.songUrl">{{item.song}}</a>
    </div>
    <ul class="cur_search_list" v-if="cur_search">
      <li v-for="(item,index) in search_end_list">
        {{item.songname}}-{{item.singer[0].name}}
      </li>
    </ul>
  </div>
</template>
<script>
  const API_PROXY = 'https://bird.ioliu.cn/v1/?url=';
  const throttle = (function(){
    let timer = null;
    return function(callback,ms){
      clearTimeout(timer);
      timer = setTimeout(callback,ms);
    }
  })();
  export default {
    props:{
      songName:String
    },
    data() {
      return {
        search_list: '',//默认热门搜索数据
        random_search_list:'',//随机10首热门搜索
        search_end_list:'',
        searchVal:'',
        songVal:'',
        hot_list:true,//是否显示热门搜索
        cur_search:true//当前说搜索结果
      }
    },
    methods: {
      // sendVal:function(){
      //   if(this.$refs.searchVal.value){
      //     this.$emit('addListenToChild',this.$refs.searchVal.value)
      //   }else{
      //     return
      //   }
      // },
      monitorVal:function(){
        this.$store.state.search_song = this.songVal;
      },
      // throttle:function(){
      //   let timer =  null;
      //   return function (callback,ms) {
      //     clearTimeout(timer);
      //     timer = setTimeout(callback,ms);
      //   }
      // },
      getRandomArrayElements: function (arr, count) {
        var shuffled = arr.slice(0), i = arr.length, min = i - count, temp, index;
        while (i-- > min) {
          index = Math.floor((i + 1) * Math.random());
          temp = shuffled[index];
          shuffled[index] = shuffled[i];
          shuffled[i] = temp;
        }
        return shuffled.slice(min);
      },
      fetchData:function(song_val){
        this.axios({
          method:'get',
          url:'/qq?format=json&w='+song_val,
          data:{
            g_tk: 5381,
            uin: 0,
            inCharset: 'utf-8',
            outCharset: 'utf-8',
            notice: 0,
            platform: 'h5',
            needNewCode: 1,
            zhidaqu: 1,
            catZhida: 1,
            t: 0,
            flag: 1,
            ie: 'utf-8',
            sem: 1,
            aggr: 0,
            perpage: 20,
            n: 20,
            p: 1,
            remoteplace: 'txt.mqq.all',
          }
        }).then(res=>{
          this.search_end_list = res.data.data.song.list;
          console.log(res.data.data.song.list);
          if(this.search_end_list.length>0){
            this.hot_list = false;
          }
        })
      }
    },
    watch:{
      songVal(){
        if(!this.songVal){
          this.hot_list = true;
        }else{
          window.localStorage.setItem('songName',this.songVal);
        }
        throttle(()=>{
          this.fetchData(this.songVal);
        },300)
      }
    },
    mounted: function () {
      this.axios({
        method: 'post',
        url: API_PROXY + 'https://c.y.qq.com/splcloud/fcgi-bin/gethotkey.fcg',
        data: {
          g_tk: '5381',
          uin: '0',
          format: 'json',
          inCharset: 'utf-8',
          outCharset: 'utf-8',
          notice: 0,
          platform: 'h5',
          needNewCode: 1
        }
      }).then(res => {
        this.search_list = res.data.data.hotkey;
        this.search_list = this.$options.methods.getRandomArrayElements(this.search_list,9);
        let new_list = [];
        for(var k=0;k<this.search_list.length;k++){
          let tempObj = new Object;
          tempObj.song = this.search_list[k].k;
          tempObj.songUrl = this.search_list[k].n;
          new_list.push(tempObj);
        }
        new_list.unshift({song:res.data.data.special_key,songUrl:res.data.data.special_url});
        this.random_search_list = new_list;
      });
    }
  }
</script>

<style lang="less" scoped>
  .search-wrap{
    padding-top: 15px;
    .val-search{
      display: block;
      width: 90%;
      height: 30px;
      margin: 15px auto;
      padding-left: 10px;
      line-height: 30px;
      background-color: #ebecec;
      color: #333;
      border: none;
      border-radius: 15px;
      font-size: 14px;
      outline: none;
    }
  }
  .hot-search-wrap{
    padding-left: 15px;
    font-size: 14px;
    >p{
      padding: 10px 0 10px 5px;
      font-weight: 500;
    }
    .hotsearch{
      display: inline-block;
      font-size: 14px;
      padding: 0 10px;
      height: 30px;
      line-height: 30px;
      color: #000;
      border: 1px solid rgba(0,0,0,.6);
      border-radius: 99px;
      word-break: keep-all;
      margin-bottom: 10px;
      margin-right: 14px;
      text-decoration: none;
    }
  }
  .cur_search_list{
    li{
      padding:15px 0 15px 20px;
      border-bottom: 1px solid rgba(0,0,0,.1);
    }
  }

</style>
