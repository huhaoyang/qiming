<template>
  <div class="container">
    <x-header class="header" :title="title" :left-options="leftOptions" @on-click-back="close"></x-header>
    <div class="block">
      <group label-width="5em">
        <x-input title="收货人" v-model="form.consignee_name"></x-input>
      </group>
      <group label-width="5em">
        <x-input title="手机号码" v-model="form.consignee_mobile"></x-input>
      </group>
      <group label-width="5em">
        <selector placeholder="请选择省份" v-model="form.province_id" title="省份" :options="province" @on-change="chPro"></selector>
      </group>
      <group label-width="5em">
        <selector placeholder="请选择城市" v-model="form.city_id" title="城市" :options="city" @on-change="chCity"></selector>
      </group>
      <group label-width="5em">
        <selector placeholder="请选择区" v-model="form.district_id" title="区" :options="district"></selector>
      </group>
      <group label-width="5em">
        <x-input title="详细地址" v-model="form.address"></x-input>
      </group>
      <x-button class="btn" type="primary" @click.native="addData()">添加</x-button>
    </div>
    <toast v-model="error.code" type="warn">{{error.message}}</toast>
  </div>
</template>

<script>
  import { Group,XHeader,Toast,Selector,XInput,XButton } from 'vux'
  import {getProvince,getCity,getCounty,addConsignee} from '@/service/getData'

  export default {
    name: "addressAdd",
    components: {
      XHeader,
      Toast,
      Group,
      Selector,
      XInput,
      XButton
    },
    data () {
      return {
        title: "添加收货地址",
        loading: false,
        form: {
          consignee_name: "",
          consignee_mobile: "",
          address: "",
          province_id: "",
          city_id: "",
          district_id: "",
          town_id:0
        },
        leftOptions: {
          preventGoBack: true
        },
        province:[],
        city:[],
        district:[],
        error: {
          code: false,
          message: ""
        }, //错误信息
      }
    },
    mounted(){
      this.initData();
    },
    methods: {
      close(){
        this.$emit("close");
      },
      async addData(){
        if(this.form.consignee_name===""){
          this.error.code = true;
          this.error.message = "请填写收件人姓名";

          return false;
        }
        if(this.form.consignee_mobile===""){
          this.error.code = true;
          this.error.message = "请填写收件人手机号码";

          return false;
        }
        if(this.form.province_id===""){
          this.error.code = true;
          this.error.message = "请选择省份";

          return false;
        }
        if(this.form.city_id===""){
          this.error.code = true;
          this.error.message = "请选择城市";

          return false;
        }
        if(this.form.district_id===""){
          this.error.code = true;
          this.error.message = "请选择地区";

          return false;
        }
        if(this.form.address===""){
          this.error.code = true;
          this.error.message = "请填写详细地址";

          return false;
        }

        let res = await addConsignee(this.form);
        if(res.code === 0){
          let that = this;
          this.$vux.toast.show({
            text: '添加成功',
            type: "success"
          });
          setTimeout(function(){
            that.close();
          },1000);
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
      async chPro(val){
        let res = await getCity(val);
        if(res.code === 0){
          this.city = [];
          this.district = [];
          for(let p in res.resultData){//遍历json对象的每个key/value对,p为key
            this.city.push({key: p ,value:res.resultData[p]});
          }
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
      async chCity(val){
        let res = await getCounty(val);
        if(res.code === 0){
          this.district = [];
          for(let p in res.resultData){//遍历json对象的每个key/value对,p为key
            this.district.push({key: p ,value:res.resultData[p]});
          }
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

        let res = await getProvince();
        if(res.code === 0){
          this.province = [];
          for(let p in res.resultData){//遍历json对象的每个key/value对,p为key
            this.province.push({key: p ,value:res.resultData[p]});
          }
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
    }
  }
</script>

<style lang="scss" scoped>
  * {
    box-sizing: border-box;
  }

  .container {
    width: 100%;
    min-height: 100%;
    background: #EFEFF4;
    padding-bottom: 20px;

    .block {
      width:100%;
      background: #ffffff;
      padding: 20px 0;

      .line {
        width:100%;
        position: relative;
        padding: 1rem 0;
        line-height: 1.5rem;
        font-size: 1rem;
        border-bottom: 1px dotted #c8c7cc;
      }
      .btn {
        background: #eb0c0c;
        width: 90%;
        margin: 5% 5%;
      }
    }
  }

</style>
