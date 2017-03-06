---
title: 仿豆瓣电影webapp单页应用SPA实战总结
date: 2017-03-03 10:57:27
tags:
  - vue.js
---
16年年前的几个月时间，工作之余，一直在致力于重构联航的H5，写来写去，都是在重复的写一些东西，因为没有后端的支持，便没有将`vue-resourse`整合进来。业界都说，没有上线的项目，项目用到的技术栈就不会落地的很成熟。为了尽量模拟真实的开发环境，几经搜索，找到了豆瓣电影开放的API，于是从中联航项目源码抽丝剥茧，结合豆瓣电影的API，计划实现一个完整的SPA应用。于是，在春节返京，作为新年的开始的第一次技术探索，用了大约一周工作之余的时间，完成了这样一个简单但很全面的小型DEMO.
<!-- more -->

### 项目地址
源码地址：`git@github.com:daqd/douban-movie-webapp.git`
在线预览地址：http://movie.mife.io/

### 本地预览步骤

下载源码
```
git clone git@github.com:daqd/douban-movie-webapp.git
```
进入到当前项目主目录
```
cd douban-movie-webapp
```
安装依赖包
```
npm install
```
开发模式运行
```
npm run dev
```
项目打包
```
npm run build
```

### 所用到的技术栈
- webpack
- vue.js
- vue-router
- vue-resource
- vuex
- mint-ui
- less

### 项目结构
#### 主目录概览
![list](/images/2017/03/0303-1.png)

#### 主目录介绍
目录或文件 | 注解
----|----
build | webpack的配置目录，可配置一些入口文件，别名，输出目录，插件，文件模块打包等等
config | VUE项目的基本配置
src | 基本所有的开发都在此目录下进行的，下面详细介绍
static | 存放静态资源的目录，如一些图片、文件字体等
node_modules | 项目的依赖包
.gitignore | github过滤文件，过滤掉我们不需要提交的文件/或文件夹
.babelrc | bable编译文件
.editorconfig | 制定的一些代码规范，在一些编辑器可结合使用
index.html | 项目的唯一html文件，所有的视图切换都是在此文件上展现的，`webpack`会在打包时动态引入相关css,js文件
package.json | 项目的一些基本信息，以及开发所用到的依赖包，都会在安装时通过`--save`/ `--dev-save`写入到文件中
README.md | 项目的介绍markdown文件

#### src目录概览
![list](/images/2017/03/0303-2.png)

#### src目录介绍

目录或文件 | 注解
----|----
assets/images | 资源文件夹，放置一些项目用到的图片资源
components | 手动实现的一些`vue组件`
css        | 公共的css样式文件，也可理解为整个单页应用的一些基础样式
js         | 项目中用到的一些公共的js方法定义在这里，之后我们可在需要的组件文件中手动引入某个方法调用
views      |所有的视图存放的位置，可以理解为每个网页，但他们却不是网页，因为SPA应用只有一个html文件，姑且称呼其为视图
vuex       |vuex状态管理目录，里面存放了状态管理所需的action，state，mutation,getter等
App.vue    | APP组件，引入了header,container,footer组件
main.js    | 项目入口，路由，`vuex`状态管理，挂载节点等都定义在这里，`webpack`打包也会从这里开始
routes.js  | SPA应用路由配置文件，用来指向视图间的跳转

### 项目开发思路
上面对于整个项目的目录结构，进行了一个基本的介绍，通过测试豆瓣电影的api，发现有几个api已经不通了，最终从里面筛选出四个API：最热电影列表，最新电影列表，top250的排行榜列表，已经电影详情的api，所以，整个SPA应用的构建之后，应该有三个列表页，三个列表页可通过底部的`tabbar`点击响应到，点击每个列表中的电影条目，进入到电影的详细页，最后的一个tabbar是一个个人中心视图，里面主要引入了mint-ui组件拼凑的一些页面。

在这四个主要的视图中，三个列表页除了数据，样式应该都是基本一致的，所有我们定义一个电影列表组件（movieList.vue），在定义三个视图页调用这个电影列表组件，传入不同的请求地址即可。所有的电影详情页面除了数据不同，结构也是一致的,我们在定义一个电影详情单文件组件（movDetails.vue）,点击每个电影条目传入不同的请求地址，以及一些参数如电影id等即可，下面主要介绍两个组件：movieList.vue和movDetails.vue。

#### movieList.vue
```
<template lang="html">
  <div class="hot-movive-list">
    <div class="onePxLineWrap" v-for="item in movData">
      <div class="movie-item onePxLine" @click="toShowDetails(item.id)">
        <!-- 电影海报 -->
        <div class="movie-item-pic">
          <img :src="item.images.large" alt="">
        </div>
        <!-- 电影基本信息 -->
        <div class="movie-item-info">
            <!-- 电影名称 -->
            <div class="movie-item-info-title">
              {{item.title}}
            </div>
            <!-- 电影评分 -->
            <div class="movie-item-info-star fontSize-13">
              观众评分：<span class="rating">{{item.rating.average}}</span> 分
            </div>
            <!-- 电影类型 -->
            <div class="movie-item-info-toppic fontSize-13">
              {{item.genres.toString().split(',').join('/')}}
            </div>
            <!-- 演员 -->
            <div class="movie-item-info-superStar fontSize-13">
              <span href="#" v-for="childItem in item.casts">{{childItem.name}} </span>
            </div>
        </div>
        <!-- 电影条目详细信息 -->
        <div class="movie-item-toShowDetails">
            <span class="toShowDetails">详细</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import { Indicator } from 'mint-ui';

export default {
  data(){
    return{
      movData:[]
    }
  },
  props:['type'],

  created(){
    this.getCustomers();
  },
  methods:{
    //获取数据
    getCustomers(){
      Indicator.open('加载中...');
      let apiUrl = this.getApiUrl(this.type);
      console.log(apiUrl);
      this.$http.jsonp(apiUrl).then(
                  function (res) {
                    console.log(res.body.subjects);
                      this.$data.movData = res.body.subjects;
                      Indicator.close();
                  },function (res) {
                      // 处理失败的结果
                      Indicator.close();
                  }
              );
    },
    // 显示详细信息
    toShowDetails(id){
      this.$router.push({path:'details',query:{type:this.type,mvId: id}});
    },
    //获取url
    getApiUrl(type){
      switch(type){
        case 'hot':
           return 'https://api.douban.com/v2/movie/in_theaters';
        case 'coming':
           return 'https://api.douban.com/v2/movie/coming_soon';
        default:
          return 'https://api.douban.com/v2/movie/top250';
      }
    }
  }
}
</script>

<style lang="less" scoped>
.movie-item{
    width: 100%;
    height: 140px;
    // border-bottom:1px solid #c9c9c9;
    display: flex;
    ....
</style>
```
在组件的生命周期函数created中，首先去读取调用该组件时传入的type,`props:['type']`在这里的作用的是定义从父组件传的到数据字段名称为`type`，子组件来使用，例如，定义了组件A,在组件A中定义了`props:['type']`，在组件B中可通过`<A :type="值"></A>`调用组件A,在A组件可直接通过传入的type值进行处理。详情可参考vue.js文档说明。

通过type值来决定去请求哪个URl,请求完数据之后，将数据集合存至data中，因为data中的数据跟视图是相互绑定的，所有的数据都是响应式的，我们只需要在`template`中解析data的数据集合即可。

#### movDetails.vue

```
<template lang="html">
  <div class="" v-if="movData">
    <div class="movie-info">
      <div class="movie-info-content">
        <!-- 电影宣传片 -->
        <div class="movie-info-video">
            <img :src="movData.images.large" alt="">
        </div>
        <div class="movie-info-list">
          <div class="movie-info-list-item">
            {{movData.title}}
          </div>
          <div class="movie-info-list-item  fontSize-13" v-if="movData.aka.length>0">
            {{movData.aka[0].toString()}}
          </div>
          <div class="movie-info-list-item fontSize-13">
            <span class="rating">{{movData.rating.average}}</span> <span class="ratings_count">({{movData.ratings_count}}人评)</span>
          </div>
          <div class="movie-info-list-item fontSize-13">
            {{movData.genres.toString()}}
          </div>
          <div class="movie-info-list-item fontSize-13">
            {{movData.countries.toString()}}
          </div>
          <div class="movie-info-list-item fontSize-13">
            {{movData.year}}年上映
          </div>

        </div>
      </div>
      <!-- 观看人数信息 -->
      <div class="movie-info-ratings_count">
        <!-- 幽灵按钮 -->
        <a href="javascript:void(0)">({{movData.wish_count}}人)想看</a>
        <a href="javascript:void(0)">({{movData.collect_count}}人)看过</a>
      </div>
    </div>
    <!-- 电影介绍 -->
    <div class="movie-summary" :class="{setAuto:isAuto}">
      <div class="content" :class="{setAuto:isAuto}">
        {{movData.summary}}
      </div>
      <div v-if="!isAuto" class="more" @click="readMore">
        展开全部
      </div>
    </div>
    <!-- 明星 -->
    <div class="movie-casts-wrap">
      <div class="movie-casts-content">
        <!-- 单个明星 -->
        <div class="movie-casts-item" v-for="item in movData.casts">
          <div class="movie-casts-item-img">
            <img :src="item.avatars.medium" alt="">
          </div>
          <div class="movie-casts-item-name">
            {{item.name}}
          </div>
        </div>
      </div>
    </div>
    <!-- 影评信息 -->
      <!-- 长短影评切换按钮 -->
    <!-- <div class="commentsBtn-wrap">
      <a href="javascript:void(0)" class="shortComment">短评</a>
      <a href="javascript:void(0)" class="longComment">长评</a>
    </div>
    <commentsList :movId="movData.id"></commentsList> -->
  </div>
</template>

<script>
import { Indicator } from 'mint-ui';
import commentsList from '../components/commentsList.vue';
export default {
  data(){
    return{
      movData:null,
      comments:null,
      isAuto:false
    }
  },
  created(){
    // this.$store.dispatch('setPath',this.$route.query.type); //设置底部tabbar
    this.checkUrl();
    console.log(this)
  },
  components:{
    commentsList
  },
  methods:{
    //检查是否缺少参数
    checkUrl(){
      //console.log(this.$route);
      if(!this.$route.query.mvId){
        this.$router.replace({path:'hot'});
      }else{
        this.$store.dispatch('setMovieSrc',this.$route.query.type);
        this.getCustomers(this.$route.query.mvId);
      }
    },
    //获取数据
    getCustomers(mvId){
      Indicator.open('加载中...');
      const apiUrl = 'https://api.douban.com/v2/movie/subject/'+mvId;
      // console.log(apiUrl);
      this.$http.jsonp(apiUrl).then(
                  function (res) {
                    // console.log(res.body);
                      this.$data.movData = res.body;
                      //判断电影介绍内容长度,大约显示110个字符
                      if(this.$data.movData.summary.length<110){
                        this.$data.isAuto = true;
                      }else{
                        this.$data.isAuto = false;
                      }
                      Indicator.close();
                  },function (res) {
                      // 处理失败的结果
                      Indicator.close();
                  }
              );
    },
    readMore(){
      this.$data.isAuto = true;
    }
  }
}
</script>

<style lang="less" scoped>
  .movie-info{
    width: 100%;
    height: 300px;
    ...
</style>

```
该组件大致的逻辑跟列表的组件还是极其相似的，不同点在于需要判断是否是非法访问，判断是否直接通过路径进入，缺少movie Id参数，如果非法访问，我们需要将其重置到列表页或者首页，重置路由不是将当前路径跳转到列表页，而是将当前路径替换到列表页，否则返回会出现错误。

### Vuex
vuex是用来存储状态的，也可理解为是用来存储数据的，通俗的讲，可以理解为在整个应用之上会有一个仓库，仓库里存储着一些变量（状态），可以在应用的各个视图中修改，读取，假如状态得到修改，因为vue的双向绑定，相应的视图也会得到更新。

#### state(定义的一些状态值)
```
const state = {
  path:'',  //返回路径
  nextPath:'',  //被中断的路由路径
  headerTit:'', //当前路由的title
  loginStatus:false,  //是否已登录
  srcType:'', //电影详细页来源类型
  userName:'', //用户名
  pageChangeStatus:'' //当前路由的切换状态 go/back
}
```
#### getter（获取的状态的方法）
```
//base
export const getPath = state => state.path //获取当前路径
export const getNextPath = state => state.nextPath //获取被中断的路径
export const getPageChangeStatus = state => state.pageChangeStatus //获取被中断的路径
export const getHeaderTit = state => state.headerTit //获取返title
export const getUserName = state => state.userName //获取title
export const getLoginStatus = state => state.loginStatus //获取登录状态
export const getSrcType= state => state.srcType //获取电影详细来源
```
#### mutations (改变状态的方法)
```
import {
  SET_PATH,
  NEXT_PATH,
  PAGE_CHANGE_STATUS,
  HEADER_TIT,
  SET_USERNAME,
  SET_LOGINSTATUS,
  SET_MOVIE_SRC

} from './mutation-types'

const mutations = {
  //设置当前路径
  [SET_PATH] (state, path) {
    state.path = path
  },
  //设置将要跳转到的路径
  [NEXT_PATH] (state, path) {
    state.nextPath = path
  },
  //设置路由切换时的状态
  [PAGE_CHANGE_STATUS] (state, status) {
    state.pageChangeStatus = status
  },
  //设置顶部tit
  [HEADER_TIT] (state, tit) {
    state.headerTit = tit
  },
  //设置用户名
  [SET_USERNAME] (state, userName) {
    state.userName = userName
  },
  //设置登录状态
  [SET_LOGINSTATUS] (state, loginStatus) {
    state.loginStatus = loginStatus
  },
  //设置token
  [SET_MOVIE_SRC] (state, srcType) {
    state.srcType = srcType
  },

}
export default mutations;

```
### vue-router
传统的web开发，页面的跳转都是通过form提交，或者js的location对象来执行跳转，SPA应用的区别在于整个应用的视图切换都是在一个页面下进行的，所以需要路由来指向视图的切换跳转。
```
export default [
  //重定向
  {
    path: '/', redirect: '/hot'
  },
  //首页
  {
    path: '/hot',
    component: resolve => {
            require(['./views/home.vue'], resolve)
        },
    name:'豆瓣电影',
  },
  //即将上映
  {
    path: '/coming_soon',
    component: resolve => {
            require(['./views/coming_soon.vue'], resolve)
        },
    name:'即将上映'
  },
  //top250
  {
    path: '/top250',
    component: resolve => {
            require(['./views/top250.vue'], resolve)
        },
    name:'TOP250'
  },
  //我的
  {
    path: '/user',
    component: resolve => {
            require(['./views/user.vue'], resolve)
        },
    name:'关于我'
  },
  //电影详细信息
  {
    path: '/details',
    component: resolve => {
            require(['./views/movDetails.vue'], resolve)
        },
    name:'电影详情'
  }
];
```
## 总结
整个SPA应用看似简单，但所用到的技术栈很全面，很多技术细节未提到，详细可参考vue.js官方文档，当前还有一个实现不太友好，title的标题目前存储在了router的name上，路由的name相当于路径的别名，也可用来执行跳转，显然，将title存储在name上是不合理的。有更好的实现方式，再来补充！
