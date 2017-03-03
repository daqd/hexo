---
title: 仿豆瓣电影webapp单页应用SPA实战总结
date: 2017-03-03 10:57:27
tags:
  - vue.js
---
去年年前的几个月时间，工作之余，一直在致力于重构M网的H5，写来写去，都是在重复的写一些页面，因为没有后端的支持，便没有将`vue-resourse`整合进来，俗话讲，没有上线的项目，技术栈就没有很成熟。为了尽量模拟真实的开发环境，几经搜索，找到了豆瓣电影开放的API，于是从中联航项目源码抽丝剥茧，结合豆瓣电影的API，实现一个完整的SPA应用。于是，在春节返京，作为新年的开始的第一次技术探索，用了大约一周工作之余的时间，完成了这样一个简单但很全面的小型DEMO.
<!-- more -->

## 一些抱(mei)怨(yong)的话
`github`这个demo的上一次更新时间是15天前，至于为什么拖到现在才来总结：
忙！忙！忙！有多忙？年前6天半欠公司的调休假,在没有白天和黑夜的加班中，在一个星期内基本还完了。高投入低产出的工作内容：后台项目兼容IE,切图静态页等等。

## 正(che)文(dan)开(jie)始(shu)

### 项目地址
源码地址：`git@github.com:daqd/douban-movie-webapp.git`
在线预览地址：http://movie.mife.io/

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
components | 手动实现的一些`vue组件``
css | 公共的css样式文件，也可理解为整个单页应用的一些基础样式
js  | 项目中用到的一些公共的js方法定义在这里，之后我们可在需要的组件文件中手动引入某个方法调用
views | 所有的视图存放的位置，可以理解为每个网页，但他们却不是网页，因为SPA应用只有一个html文件，姑且称呼其为视图
vuex  | vuex状态管理目录，里面存放了状态管理所需的`action`，`state`，`mutation`,`getter`等
App.vue  | APP组件，引入了`header`,`container`,`footer`组件
main.js | 项目入口，路由，`vuex`状态管理，挂载节点等都定义在这里，`webpack`打包也会从这里开始
routes.js | SPA应用路由配置文件，用来指向视图间的跳转

### 项目开发思路
上面对于整个项目的目录结构，进行了一个基本的介绍，通过测试豆瓣电影的api，发现有几个api已经不通了，最终能用到的api只有四个：最热电影列表，最新电影列表，top250的排行榜列表，已经电影详情的api，所以，整个SPA应用的构建之后，应该有三个列表页，三个列表页可通过底部的`tabbar`点击响应到，点击每个列表中的电影条目，进入到电影的详细页，最后的一个tabbar是一个个人中心视图，里面主要引入了mint-ui组件拼凑的一些页面。

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
    .movie-item-pic{
      width: 180px;
      height: 120px;
      margin-left: 10px;
      margin-top: 10px;
      overflow: hidden;
    }
    .movie-item-pic img{
      width: 100%;
    }

    .movie-item-info{
      width: 100%;
      height: 140px;
      margin-top: 10px;
      margin-left: 7px;
      .movie-item-info-title{
        width: 100%;
        height: 30px;
        line-height:30px;
        font-size: 18px;
      }
      .movie-item-info-star,.movie-item-info-toppic,.movie-item-info-superStar{
        width: 100%;
        height: 30px;
        line-height: 30px;
        overflow: hidden;
        .rating{
          color: #ffb400;
          font-size: 14px;
        }
      }
    }
    .movie-item-toShowDetails{
      width: 120px;
      height: 140px;
      display: flex;
      align-items: center;
      justify-content: center;
      .toShowDetails{
        display: block;
        width: 60px;
        height: 30px;
        border:1px solid #df2d2d;
        color: #df2d2d;
        border-radius: 5px;
        line-height: 30px;
        text-align: center;
        margin-right: 10px;
      }
    }
}
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
    background: url("../assets/images/background.jpg") no-repeat;
    background-size:cover;
    background-position: 0px -260px;
    .movie-info-content{
      width: 100%;
      height: 240px;
      display: flex;
    .movie-info-video{
      width: 130px;
      height: 200px;
      margin: 30px 0px 0px 10px;
    }
    .movie-info-video img{
      width: 130px;
    }
    .movie-info-list{
      width: 100%;
      height: 200px;
      margin: 30px 0px 0px 10px;
      .movie-info-list-item{
        width: 100%;
        height: 30px;
        line-height: 30px;
        color: #FFF;
        .rating{
          font-size: 14px;
          color: #ffb400;
        }
        .ratings_count{
          font-size: 10px;
        }
      }
    }
  }

    .movie-info-ratings_count{
      width: 100%;
      height: 50px;
      display: flex;
      justify-content: space-around;
    }
    .movie-info-ratings_count a{
      width: 45%;
      height: 45px;
      display: block;
      line-height: 45px;
      text-align: center;
      background: #595e57;
      color: #FFF;
      border-radius: 5px;
      text-decoration: none;
    }
  }

  /** 电影介绍 **/
  .movie-summary{
      width: 100%;
      height: 165px;
      margin-top: 8px;
      border-bottom: 5px solid #f5f5f5;
      .content{
        width: 95%;
        height: 120px;
        overflow: hidden;
        margin: 0 auto;
        line-height: 25px;
        text-align: justify;
        color: #505050;
      }
      .more{
          width: 100%;
          height: 30px;
          margin-top: 5px;
          color:#ff3535;
          display: flex;
          justify-content: center;
          align-items:center;
      }
  }

  /** 明星 **/
  .movie-casts-wrap{
    &:extend(.movie-summary);
    height: 150px;
    display: flex;
    justify-content: center;
    .movie-casts-content{
      width: 95%;
      height: 150px;
      display: flex;
      .movie-casts-item{
        width: 25%;
        height: 120px;
        margin-right: 5px;
        .movie-casts-item-img{
          width: 100%;
          height: auto;
        }
        .movie-casts-item-img img{
          width: 100%;
        }
        .movie-casts-item-name{
          width: 100%;
          height: 30px;
          line-height: 30px;
          text-align: center;
          overflow: hidden;
        }
      }
    }
  }


/** 长短影评切换 **/
  .commentsBtn-wrap{
    width: 95%;
    height: 40px;
    margin: 0 auto;
    margin-top: 15px;
    display: flex;
    &>a{
      display: block;
      width: 50%;
      height: 40px;
      line-height: 40px;
      text-align: center;
      border:1px solid #df2d2d;
      text-decoration: none;

    }
    .shortComment{
      border-right-width: 0;
      border-top-left-radius: 5px;
      border-bottom-left-radius: 5px;
      background: #df2d2d;
      color: #FFF;
    }
    .longComment{
      border-top-right-radius: 5px;
      border-bottom-right-radius: 5px;
    }
  }

  /** setAuto **/
  .setAuto{
    height: auto !important;
  }
</style>

```
该组件大致的逻辑跟列表的组件还是极其相似的，不同点在于需要判断是否是非法访问，判断是否直接通过路径进入，缺少movie Id参数，如果非法访问，我们需要将其重置到列表页或者首页，重置路由不是将当前路径跳转到列表页，而是将当前路径替换到列表页，否则返回会出现错误。

### Vuex
文章开始提到，这个SPA是一个简单而又全面的demo，实际上，在这次的demo中，没有必要引入vuex，只有在构建中大型相对复杂的单页应用时才适合引入，简单的项目使用vuex可能会带来困扰。那么，vuex到底是一个东西？

vuex是用来存储状态的，也可理解为是用来存储数据的，那么，在每个vue组件中提供了data属性，data属性是一个函数，他返回一个数据集合，我们完全可以将数据状态存在data中，vuex不是多此一举么？所有新技术的出现肯定是为了解决某种问题而问世的，那么它是为了解决什么问题？他跟data又有什么区别？
