<template>
  <div class="container">
    <x-header class="header" :title="title"></x-header>
    <div v-if="detail!={}" class="item">
      <swiper :aspect-ratio="800/800" dots-position="center" class="banner" :loop="true" :interval="5000">
        <swiper-item v-for="(item, index) in detail.jdImages" :key="index">
          <img class="banner-img" :src="item.path">
        </swiper-item>
      </swiper>
      <div class="block">
        <p class="model">{{detail.displayModel}}</p>
        <ul>
          <li class="price">￥{{detail.price}}</li>
          <!--<li class="s-price">会员价 | {{sPrice}}</li>-->
          <li class="count">已售{{detail.salesCount}}件</li>
        </ul>
        <group class="number">
          <x-number title="购买" v-model="form.amount" :min="detail.quantityMin" :fillable="true"></x-number>
        </group>
      </div>
      <div class="block B2B">
        <p class="title">适配车型</p>
        <p class="name">{{detail.note}}</p>
      </div>
      <div class="block B2B" v-if="detail.merchantId===0">
        <p class="title">销售商家信息</p>
        <p class="name">{{detail.B2BNameCN}}</p>
        <p class="introduce">{{detail.B2BIntroduce}}</p>
      </div>
      <div class="block B2B" v-else>
        <p class="title">销售商家信息</p>
        <p class="name">{{detail.placeName}}</p>
        <p class="introduce">商家简介</p>
      </div>

      <div class="block">
        <p class="title">宝贝参数</p>
        <span v-if="detail.partLevelId===0" class="level">{{detail.levelNote}}</span>
        <span v-if="detail.merchantId===0" class="type">品位高级优质<span>B</span></span>
        <div class="label">
          <span class="label-name">品牌</span>
          <span class="label-value">{{detail.brandName}}</span>
        </div>
        <div class="label" v-for="(item,index) in detail.keyParameterMap" :key="index">
          <span class="label-name">{{item.attrName}}</span>
          <span class="label-value">{{item.attrValue}}</span>
        </div>
      </div>
      <p v-if="more===false" class="more" @click="showMore()">点击查看详情</p>
      <div v-else class="block">
        <p class="title">商品详情</p>
        <div class="more-content" v-html="goodsIntroduce">
          {{goodsIntroduce}}
        </div>
        <div class="more-content" v-html="merchantIntroduce">
          {{merchantIntroduce}}
        </div>
        <p @click="more = false" class="more">点击收起详情</p>
      </div>
    </div>
    <p class="bottom">
      <router-link :to="{name:'cart'}">
        <div class="cart">
          <img src="~@/images/cart-2.png" /><badge v-if="itemCount!==0" class="counts" :text="itemCount"></badge>
        </div>
      </router-link>
      <span class="add-cart" @click="addCart()">加购物车</span>
    </p>
    <toast v-model="error.code" type="warn">{{error.message}}</toast>
  </div>
</template>

<script>
  import { XHeader,Toast, Swiper, SwiperItem,Group,XNumber,Badge } from 'vux'
  import btnArea from '@/components/orders/btnArea'
  import {goodsDetail,goodsIntroduce,addMercahnt,getConfigs} from '@/service/getData'

  export default {
    components: {
      XHeader,
      Toast,
      Swiper,
      SwiperItem,
      Group,
      XNumber,
      Badge,
      btnArea
    },
    data () {
      return {
        title: "商品详情",
        detail: {},
        more: false,
        itemCount: 0,
        goodsIntroduce: "",
        merchantIntroduce: "",
        configs:"",
        form: {
        	amount:0
        },
        error: {
          code: false,
          message: ""
        }, //错误信息
      }
    },
    mounted(){
      this.initData();
      this.getRate();
    },
    methods: {
      async getRate(){
        let res = await getConfigs();
        if(res.code === 0){
          this.configs = res.resultData;
          console.log(res.resultData)
        }else{
          this.error.code = true;
          this.error.message = res.message;
        }
      },
      async addCart(){
      	if(this.form.amount===0){
          this.$vux.toast.show({
            text: '购买数量不能为零',
            type: "warn"
          })

          return false;
        }

        let id = this.$route.params.id;
        let res = await addMercahnt({goods_price_id:id,count:this.form.amount});
        if(res.code === 0){
          this.itemCount = res.resultData.itemCount;
        }else if(res.code===90002){
          this.error.code = true;
          this.error.message = res.message;
          this.$router.push({
            name: "login",
            query: { redirect: this.$route.path }
          })
        }else{
          this.error.code = true;
          this.error.message = res.message;
        }
      },
    	async showMore(){
        let id = this.$route.params.id;
        let res = await goodsIntroduce(id);
        if(res.code === 0){
          this.goodsIntroduce = res.resultData.goodsIntroduce;
          this.merchantIntroduce = res.resultData.merchantIntroduce;
          this.more = true;
        }else if(res.code===90002){
          this.error.code = true;
          this.error.message = res.message;
          this.$router.push({
            name: "login",
            query: { redirect: this.$route.path }
          })
        }else{
          this.error.code = true;
          this.error.message = res.message;
        }
      },
      async initData(){
        let id = this.$route.params.id;
        let res = await goodsDetail(id);
        if(res.code === 0){
          if(res.resultData.merchantId===0){
            res.resultData.jdImages.unshift({path:res.resultData.partPhoto});
          }else{
            res.resultData.jdImages = [{path:res.resultData.partPhoto}];
          }
          this.form.amount = res.resultData.quantityMin;
          this.detail = res.resultData;
        }else if(res.code===90002){
          this.error.code = true;
          this.error.message = res.message;
          this.$router.push({
            name: "login",
            query: { redirect: this.$route.path }
          })
        }else{
          this.error.code = true;
          this.error.message = res.message;
        }

      },
    },
    computed:{
    	sPrice(){
    		return (this.detail.price*(1-this.configs.max_point_pay)).toFixed(2);
      }
    }
  }
</script>

<style lang="scss" scoped>
  @import "~@/styles/layout.scss";
  .container {
    width: 100%;
    min-height: 100vh;
    background: #EFEFF4;
    padding-bottom: 20px;

    .item {
      width:100%;
      padding-bottom: 50px;

      .banner-img {
        width: 100%;
      }

      .block {
        position: relative;
        width: 100%;
        margin: 5% 0;
        padding: 10px 0;
        background: #ffffff;

        .model {
          font-size: 1.2rem;
          line-height: 2rem;
          padding: 0 10px;
        }

        ul{
          list-style: none;
          padding: 0 10px;

          li {
            display: inline-block;
            line-height: 1.3rem;
          }

          .price {
            color: #eb0c0c;
            font-size: 1.3rem;
          }

          .s-price {
            margin-left: 30px;
            border-radius: 5px;
            background: url("~@/images/sp-bg.png") no-repeat;
            background-size: 100% 100%;
            color: #fff;
            padding: 5px 10px;
          }

          .count {
            font-size: 1rem;

            color: #c8c7cc;
            float: right;
          }
        }

        .title {
          color: #000;
          font-size: 16px;
          font-weight: bold;
          padding: 0 10px;
          border-bottom: 1px dotted #c8c7cc;
        }

        .label {
          position: relative;
          padding: 0 10px;
          display: flex;
          flex-direction: row;
          justify-content: flex-start;
          flex-flow: wrap;

          .label-name {
            color: #8e8e93;
            width:35%;
          }

          .label-value {
            color: #8e8e93;
            padding-left: 20px;
          }
        }

        .level {
          position: absolute;
          right: 15%;
          top: 10%;
        }

        .type {
          position: absolute;
          right: 5%;
          top: 5%;
          font-size: 1rem;
          line-height: 2rem;

          span {
            padding: 0 5px;
            background: #007aff;
            color: #ffffff;
            margin-left: 10px;
          }
        }

      }

      .more {
        text-align: center;
        background: #f1f2f4;
        color: #606060;
        font-size: 1rem;
      }

      .B2B {
        .name,.introduce {
          padding: 0 10px;
          font-size: 1rem;
          line-height: 2rem;
          color: #8d8d8d;
        }
      }

      p {
        line-height: 30px;
        font-size: 14px;
      }

    }

    .bottom {
      position: fixed;
      bottom:0;
      background: #ffffff;
      width:100%;
      height:50px;

      a {
        padding: 10px 0;
        display: inline-block;
        vertical-align: middle;
        margin-left: 10px;
        height:50px;
      }

      .cart {
        display: inline-block;
        img {
          vertical-align: middle;
          width:30px;
          height:30px;
        }

        .counts {
          vertical-align: top;
          margin-left: -0.5rem;
        }
      }

      .add-cart {
        vertical-align: middle;
        float: right;
        font-size: 16px;
        line-height: 50px;
        padding: 0 10px;
        background: #eb0c0c;
        color: #ffffff;
      }
    }

  }
</style>

<style lang="scss">
  * {
    box-sizing: content-box;
  }

  .more-content {
    width:100%;
    margin-top: 20px;

    div {
      width: 100% !important;
    }

    img {
      width: 100% !important;
      height:auto;
    }
  }
</style>
