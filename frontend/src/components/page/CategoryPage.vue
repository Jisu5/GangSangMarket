<template>
<div id="app">
    <v-row style="border-bottom:1px solid; padding:30px 0px 20px;height:78px; " id="subMenu">
        <div style="font-size:14px; height:28px; font-weight:550;z-index:5; ">
            <a @click="gohome()" class="top_a"><img style="width:16px; height:16px; margin-right:5px" :src="require(`@/assets/post/home.png`)">홈</a>
            <img style="width:6px; height:10px; margin:0px 10px" :src="require(`@/assets/post/next.png`)">
            <a @click="goBigCategory()" class="top_a">  {{categoryBig }} </a>
            <img style="width:6px; height:10px; margin:0px 10px" :src="require(`@/assets/post/next.png`)">
            <a class="top_a"> {{categoryMid }} </a>
            <!-- <img style="width:15px; height:15px; margin-right:5px" :src="require(`@/assets/post/home.png`)"> 홈
            <img style="width:6px; height:10px; margin:0px 10px" :src="require(`@/assets/post/next.png`)">  {{ categoryBig }} 
            <img style="width:6px; height:10px; margin:0px 10px" :src="require(`@/assets/post/next.png`)"> {{ categoryMid }}  -->
        </div>
    </v-row> 
    <template v-if="noData" >
            <v-col cols="12" style="text-align:center;margin-top:100px;font-weight:bold;font-size:20px">해당 카테고리에 등록된 상품이 없습니다.</v-col>
    </template>
    <div id="postList" style="margin-top: 30px;"> 
            <PostList :posts="posts"/>
    </div>
    <infinite-loading @infinite="infiniteHandler" spinner="waveDots" ref="InfiniteLoading">
    </infinite-loading>
</div>
</template>

<script>
import httpPost from "@/util/http-post"
import PostList from "@/components/post/PostList.vue"
import InfiniteLoading from 'vue-infinite-loading'

export default {
    components:{
        PostList,
        InfiniteLoading,
    },
    data(){
        return{
            posts:[],
            start: 0,
            categoryBig: '',
            categoryMid: '',
            noData: false,
            categoryBigId: '',
            categoryMidId: '',
        }
    },
    watch: {
        '$route.params.categoryNum': 'fetchData'
    },
    methods: {
        gohome(){
          this.$router.push("/")
        },
        goBigCategory(){
            this.$router.push({name:'BigCategoryPage', params:{bigCategoryNum: this.categoryBigId}})
        },
        goMidCategory(){
            this.$router.push({name:'CategoryPage',params:{categoryNum:this.categoryMidId}})
        },
        fetchData() {
            this.posts = []
            this.start = 0
            this.categoryMid = ''
            this.categoryBig = ''
            this.noData = false
            this.$refs.InfiniteLoading.stateChanger.reset();
            this.getCategory(this.$route.params.categoryNum)
        },
        infiniteHandler($state) {
            httpPost.get(`/api/post/category/${this.$route.params.categoryNum}?sno=` + this.start)
            .then(res => {
                const data = res.data.slice(1,res.data.length)
                if (data.length) {
                    this.start += 12;
                    this.posts.push(...data);
                    $state.loaded();
                } 
                else {
                    $state.complete();
                }
            });      
        },
        getCategory(categoryId) {
            // 카테고리 저장
            httpPost.get(`/api/post/category/${categoryId}?sno=0`)
            .then(res => {
                this.categoryMid = res.data[0].cate_mid_name
                this.categoryBig = res.data[0].cate_big_name
                this.categoryBigId = res.data[0].cate_big_id
                this.categoryMidId = res.data[0].cate_mid_id
                if (res.data.length == 1) {
                    this.noData = true
                }
            });      
           
        }

    },
    created() {
        this.getCategory(this.$route.params.categoryNum)
             
    },
}
</script>

<style scoped>
#app {
    width: 80%;
    margin: 0 auto 0 auto;
}
.top_a{
  color:#3d1860;
  font-weight:bold;
}
</style>
