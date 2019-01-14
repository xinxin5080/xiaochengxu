# 微信小程序开发-第2天

## 今天知识点
1.什么是mpvue
2.mpvue5分钟上手教程
3.mpvue基本结构介绍
4.开始搭建商城
5.请求首页接口数据
6.封装请求模块
7.小程序的本地存储

## 1.什么是mpvue
介绍：**一个使用 Vue.js 开发小程序的前端框架,即把vuejs文件编译成微信小程序文件**，当然会存在一定的平台性差异
官网：http://mpvue.com/
源码：https://github.com/Meituan-Dianping/mpvue

## 2.mpvue5分钟上手教程

1.安装:

`npm install --global vue-cli@2.9`

`vue init mpvue/mpvue-quickstart shop`

**注意：appid需要输入个人的appid,Vuex（y/n）和eslint（y/n）都输入 n**

2.切换到项目目录
`cd shop` shop是项目目录

3.安装依赖
`npm install`

4.启动服务
`npm run dev`
注意：运行成功后，可以看到本地多了个 dist 目录，这个目录里就是生成的小程序相关代码。

5.使用微信开发工具运行小程序，
新建小程序需要的信息：
- 项目目录：就是刚刚创建的项目目录（非 dist 目录）
- AppID，个人的appid
- 项目名称

## 3.mpvue基本结构介绍

    build - 把mpvue构建成小程序的配置文件(忽略)
    config - 基础的配置文件(忽略)
    dist - 构建后的小程序项目(忽略)
    src - 项目开发文件
        components - 项目公共组件
        pages - 小程序页面
        utils - 工具文件夹 (忽略)
        app.json - 小程序项目配置文件
        App.vue - 项目VUE入口文件
        main.js - 项目入口文件
    static - 静态资源文件
    -....
    project.config.json - 同原生小程序的配置文件

## 4.开始搭建商城

1.新建tabBar需要的所有页面和配置tabBar

2.商城首页基本布局

    // src/pages/index/index.vue
    <template>
    <div class="container">
        <div class="search">
        <div class="search-input">
            <icon size="14" type="search"></icon>
            搜索
        </div>
        </div>
        <swiper indicator-dots>
        <swiper-item v-for="(item, key) in imgUrls" :key="key">
            <img :src="item.image_src" class="slide-image" mode="aspectFill"/>
        </swiper-item>
        </swiper>
        <div class="menus">
        <div class="menu-item" v-for="(item, key) in menus" :key="key">
            <img :src="item.image_src" mode="aspectFill"/>
        </div>
        </div>
        <div class="floors">
        <div class="floor" v-for="(item, key) in floors" :key="key">
            <div class="floor-title">
            <img :src="item.floor_title.image_src" alt="" mode="aspectFill">
            </div>
            <div class="floor-content">
            <div class="floor-left">
                <img :src="item.product_list[0].image_src" mode="aspectFill"/>
            </div>
            <div class="floor-right">
                <div class="floor-right-item" 
                v-for="(product, pkey) in item.product_list" 
                :key="pkey"
                v-if="pkey !== 0">
                <img :src="product.image_src" alt="" mode="aspectFill">
                </div>
            </div>
            </div>
        </div>
        </div>
    </div>
    </template>

3.scss样式的支持
需要安装包：`npm install sass-loader node-sass --save-dev`
使用导入拆分样式

    <style scoped lang=scss>
    @import "./style.scss"
    </style>

## 5.请求首页接口数据

`wx.request`方法的使用
文档地址： https://developers.weixin.qq.com/miniprogram/dev/api/network/request/wx.request.html

    // src/pages/index/index.vue
    export default {
        data () {
            return {
            imgUrls: [],
            menus: [],
            floors: []
            }
        },
        mounted(){
            // 请求轮播图数据接口
            wx.request({
            url: "https://itjustfun.cn/api/public/v1/home/swiperdata",
            success: (res) => {
                this.imgUrls = res.data.data;
            }
            });
    
            // 请求分类数据接口
            wx.request({
            url: "https://itjustfun.cn/api/public/v1/home/catitems",
            success: (res) => {
                this.menus = res.data.data;
            }
            })
    
            // 请求楼层数据接口
            wx.request({
            url: "https://itjustfun.cn/api/public/v1/home/floordata",
            success: (res) => {
                this.floors = res.data.data;
            }
            })
        }
    }
​    
## 6.封装请求模块

### Promise封装请求模块

    // utils/request.js
    function request(url, method){
        return new Promise((resolve, reject) => {
            wx.request({
                url: url,
                method:  method,
                success: function(res){
                    resolve(res);
                },
                fail: function(error){
                    console.log(error)
                    reject(error);
                }
            })
        })
    }
    
    // 封装get请求
    request.get = (url) => {
        return request(url, "GET");
    }

### async await的使用
async函数执行的时候，一旦遇到await就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。

```
注意：
1.await后面接的是Promise对象
2.await的返回值是Promise对象then方法的参数
```

最后代码：

    //src/pages/index/index.vue
    export default {
        data () {
            return {
            imgUrls: [],
            menus: [],
            floors: []
            }
        },
        mounted(){
            this.getData();
        }
        methods: {
            async getData(){
            const imgUrls = await request.get("https://itjustfun.cn/api/public/v1/home/swiperdata");
            this.imgUrls = imgUrls.data.data;
    
            const menus = await request.get("https://itjustfun.cn/api/public/v1/home/catitems");
            this.menus =  menus.data.data;
            
            const floors = await request.get("https://itjustfun.cn/api/public/v1/home/floordata")
            this.floors = floors.data.data;
            }
        }
    }


## 今天总结