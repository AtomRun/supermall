<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"/>
    <scroll class="content" ref="scroll" :probe-type="3" @scroll="contentScroll">
      <detail-swiper :top-images="topImages"></detail-swiper>
      <detail-base-info :goods="goods"/>
      <detail-shop-info :shop="shop"/>
      <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"/>
      <detail-param-info ref="params" :param-info="paramInfo"/>
      <detail-comment-info ref="comment" :comment-info="commentInfo"/>
      <goods-list ref="recommend" :goods="recommends"/>
    </scroll>
    <detail-bottom-bar @addCart="addToCart"/>
    <!--组件不能被直接监听，需要使用native修饰符-->
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
  </div>
</template>

<script>

  import DetailNavBar from "./childComps/DetailNavBar";
  import DetailSwiper from "./childComps/DetailSwiper";
  import DetailBaseInfo from "./childComps/DetailBaseInfo";
  import DetailShopInfo from "./childComps/DetailShopInfo";
  import DetailGoodsInfo from "./childComps/DetailGoodsInfo";
  import DetailParamInfo from "./childComps/DetailParamInfo";
  import DetailCommentInfo from "./childComps/DetailCommentInfo";
  import DetailBottomBar from "./childComps/DetailBottomBar";

  import GoodsList from "components/content/goods/GoodsList";
  import BackTop from "components/content/backTop/BackTop";

  import {getDetail,Goods,Shop,GoodsParam,getRecommend} from "network/detail";

  import Scroll from "components/common/scroll/Scroll";
  import {debounce} from "../../common/utils";
  import {BACKTOP_DISTANCE} from "common/const";
  import {mapActions} from 'vuex'

  export default {
    name: "Detail",
    components:{
      DetailNavBar,
      DetailSwiper,
      DetailBaseInfo,
      DetailShopInfo,
      Scroll,
      DetailGoodsInfo,
      DetailParamInfo,
      DetailCommentInfo,
      DetailBottomBar,
      GoodsList,
      BackTop,
      // Toast
    },
    data(){
      return{
        iid:null,
        topImages:[],
        goods:{},
        shop:{},
        detailInfo:{},
        paramInfo:{},
        commentInfo:{},
        recommends:[],
        themeTopYs:[],
        getThemeTopY:null,
        currentIndex:0,
        isShowBackTop:false,
      }
    },
    created() {
      //1. 保存传入的iid
      this.iid = this.$route.params.iid

    //2.  根据iid请求详情数据,做一层封装
      getDetail(this.iid).then(res =>{
        //1. 获取顶部的轮播图数据
        // console.log(res);
        const data = res.result;
        this.topImages = data.itemInfo.topImages;

        //2. 获取商品信息
        this.goods = new Goods(data.itemInfo,data.columns,data.shopInfo.services)

      //  3. 创建店铺信息对象
        this.shop = new Shop(data.shopInfo)

      //  4. 保存商品的详情数据
        this.detailInfo = data.detailInfo;

      //  5. 获取参数信息
        this.paramInfo = new GoodsParam(data.itemParams.info,data.itemParams.rule)

      //  6. 取出评论信息
        if (data.rate.cRate !==0 ){
          this.commentInfo = data.rate.list[0]
        }
      })

    //  3. 获取推荐数据
      getRecommend().then(res=>{
        // console.log(res);
        this.recommends = res.data.list
      })

      //4. 给getThemeY赋值
      this.getThemeTopY = debounce(()=>{
        this.themeTopYs = []

        this.themeTopYs.push(0)
        this.themeTopYs.push(this.$refs.params.$el.offsetTop)
        this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
        this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
        // console.log(this.themeTopYs);
      },100)
    },
    methods:{
      ...mapActions({
        add:'addCart'
      }),
      imageLoad(){
        this.$refs.scroll.refresh()
        this.getThemeTopY()
      },
      titleClick(index){
        // console.log(index);
        this.$refs.scroll.scrollTo(0,-this.themeTopYs[index],200)
      },
      contentScroll(position){
        //1. 获取y值
        const positionY = -position.y
        let length = this.themeTopYs.length;
      //  2. 和主题中的值做对比
        for (let i=0;i< length;i++){
          if (this.currentIndex!=i && (i<length - 1 && positionY > this.themeTopYs[i] && positionY < this.themeTopYs[i+1])
            || (i===length-1 && positionY>this.themeTopYs[i])){
            this.currentIndex = i;
            this.$refs.nav.currentIndex = this.currentIndex
          }
          this.isShowBackTop = (-position.y) > BACKTOP_DISTANCE
        }
      },
      backClick(){
        //通过组件访问组件里的属性，方法
        this.$refs.scroll.scrollTo(0, 0,300)
      },
      addToCart(){
        //1. 获取购物车需要的信息
        const product = {}
        product.image = this.topImages[0];
        product.title = this.goods.title;
        product.desc = this.goods.desc;
        product.price = this.goods.newPrice;
        product.iid = this.iid;

      //  2. 将商品添加到购物车中
        this.add(product).then(res=>{
          console.log(this.$toast);
          this.$toast.show(res);
        })
      }
    },
  }
</script>

<style scoped>
  #detail{
    position: relative;
    z-index: 9;
    background-color: #fff;
    height: 100vh;
  }
  .content{
    height: calc(100% - 43px - 0px);
  }
  .detail-nav{
    position: relative;
    z-index: 9;
    background-color: #fff;
  }
</style>
