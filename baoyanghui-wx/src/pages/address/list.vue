<template>
  <div class="container">
    <x-header class="header" :title="title" :left-options="leftOptions" @on-click-back="close"></x-header>
    <div class="block">
      <div class="line" v-for="item in addresses" :key="item.consigneeId" @click="changeAddress(item)">
        <p>
          <span>
            {{item.consigneeName}}
            <span v-if="item.isDefault" class="def-flag">默认地址</span>
          </span>
          <span class="right">{{item.consigneeMobile}}</span>
        </p>
        <p>{{item.consigneeAddress}}</p>

        <img class="icon edit" @click.stop.prevent="edit(item.consigneeId)" src="~@/images/edit.png">
      </div>
      <group class="add-btn">
        <x-button class="btn" type="primary" @click.native="addShow=true">
          <img class="icon" src="~@/images/add.png"/>
          新建收货地址
        </x-button>
      </group>
    </div>
    <popup v-model="addShow" position="right" width="100%">
      <address-add v-on:close="closeAdd"></address-add>
    </popup>
    <popup v-model="editShow" position="right" width="100%">
      <address-edit v-if="consigneeId!=0" :id="consigneeId" v-on:close="closeEdit"></address-edit>
    </popup>
    <toast v-model="error.code" type="warn">{{error.message}}</toast>
  </div>
</template>

<script>
  import { XHeader,Toast,TransferDom,Popup,Group,XButton } from 'vux'
  import addressAdd from '@/pages/address/add'
  import addressEdit from '@/pages/address/edit'
  import {getConsignees} from '@/service/getData'

  export default {
    name: "address",
    components: {
      XHeader,
      Toast,
      TransferDom,
      Popup,
      addressAdd,
      addressEdit,
      Group,
      XButton
    },
    data () {
      return {
        title: "选择收货地址",
        loading: false,
        addShow: false,
        editShow: false,
        consigneeId: 0,
        leftOptions: {
          preventGoBack: true
        },
        addresses: [],
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
      changeAddress(item){
        this.$emit("change",item);
      },
      close(){
        this.$emit("close");
      },
      closeAdd(){
        this.addShow = false;
        this.initData();
      },
      closeEdit(){
        this.editShow = false;
        this.initData();
      },
      edit(id){
        this.consigneeId = id;
        this.editShow = true;
      },
      async initData(){

        let res = await getConsignees();
        if(res.code === 0){
          this.addresses = res.resultData.items;
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
      background: #ffffff;
      padding: 20px 0;

      .add-btn {
        width:80%;
        margin: 20px auto;

        .btn {
          background: #eb0c0c;
          border-radius: 0;

          .icon {
            height: 20px;
            width:auto;
            vertical-align: middle;
          }
        }
      }

      .line {
        width:100%;
        position: relative;
        padding: 1rem 40px 1rem 20px;
        line-height: 1.5rem;
        font-size: 1rem;
        border-bottom: 1px dotted #c8c7cc;

        .def-flag {
          color: #eb0c0c;
        }

        .icon {
          position: absolute;
          height: 30%;
          top:35%;
          right:2%;
        }

        .right {
          float: right;
        }

        .add {
          height: 60%;
          top:20%;
        }
      }
    }
  }

</style>
