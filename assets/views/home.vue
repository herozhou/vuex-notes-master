<template>
    <div >
        <home-header v-on:inputEvent="inputSearch"></home-header>
        <top-channel v-on:changeEvent="changeTab"></top-channel>
        <!-- <top-channel></top-channel> -->
        <div class="content">
        <list class="list" >
            <cell class="cell" v-for="job_item in show_job_list">
              <div class="" @click="jumpTo(job_item.id)">
                <text class="text panel"><span style="color:#0f88eb">{{job_item.title}}</span> {{'['+job_item.website+']'}}</text>
              </div>
            </cell>
       </list>
       <div  class="button" @click="loadmore" v-if="show_job_list.length!==all_job_list.length">
      <text style="color:#0f88eb;text-align:center">{{loading_text}}</text>
         
       </div>
    </div>
    </div>
</template>
<style scoped>
     .panel {
        width: 700px;
    height: 130px;
    margin: 15px;
    flex-direction: column;
    justify-content: center;

    border-width: 2px;
    border-style: solid;
    border-color: #0f88eb;
    background-color: white;
  }
  .text {
    white-space:pre-line;
    text-align: left;
    color: #0f88eb;
  }


  .list{
    justify-content: center;
  

  }
  .cell{
    justify-content: center;
    align-items: center;

  }


.content{
    margin-top: 200px;
    margin-bottom: 150px;
}
  .button {
    width: 700px;
      margin: 15px auto;
    text-align: center;
    background-color: white;
      border-width: 2px;
    border-style: solid;
    border-color: #0f88eb;
    padding-top: 15px;
    padding-bottom: 15px;
    margin-bottom: 30px;
    color: #0f88eb;

    border-radius: 10px;
  }
  
</style>

<script>
 import Header from '../components/Header.vue';
    import topChannel from '../components/topChannel.vue';

     const modal = weex.requireModule('modal')
     const stream = weex.requireModule('stream')
  const LOADMORE_COUNT = 5

  export default {
      components: {
            'home-header': Header,
            'top-channel': topChannel,


        },
    data () {
      return {
        all_job_list: [],
        show_job_list:[],
        loading_text:"加载更多",
        tag: '全部', 
        now_index: -1 ,
      }
    },
    methods: {
      loadmore (mes) {
     this.loading_text = "正在加载"

          //可以用同步代码块优化，加载时点击界面会跳，用户体验不好
          if(mes && mes==='changeTab') {
              this.show_job_list=[];
              this.now_index = -1;
            } 
          const now_length = this.show_job_list.length;

          const total_length = this.all_job_list.length;

          const tag = this.tag;
          for (let i = this.now_index + 1,j = 0; j < LOADMORE_COUNT  && i < total_length ; ++i) {
            let list_item = this.all_job_list[i]
            if(list_item.job_tag.indexOf(tag)!==-1 || tag==="全部") {
                j++;
                this.show_job_list.push(list_item);
            }
            this.now_index = i;
          }

          if(now_length===this.show_job_list.length)   modal.toast({ message: '没有更多数据了', duration: 1 })

         this.loading_text = "加载更多"

       
      },

       getJobList (api,callback) {
        return stream.fetch({
          method: 'GET',
          type: 'json',
          url: 'http://192.168.43.75:8880'+api
        }, callback)
      },
      jumpTo(id){
        this.$router.push('job_detail/'+id)
      },
      changeTab(tag){
        console.log(tag)
        this.tag=tag;
        this.loadmore('changeTab');
      },
      inputSearch(tag){
         console.log(tag)
         this.tag=tag;
         this.loadmore('changeTab');
      }
    },
     created () {
      this.getJobList('/job_list', res => {
        this.all_job_list = res.ok ? res.data : '(network error)'

          this.$store.dispatch('setJobList', this.all_job_list)
          console.log(this.$store.getters.getJobList)
        this.loadmore();
      })
    }

  }
</script>