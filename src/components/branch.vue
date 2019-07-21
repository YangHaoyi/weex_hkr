<template>
  <list class="list" @loadmore="fetch" loadmoreoffset="10">
    <cell class="cell" v-for="num in carList">

      <div class="item-content">

        <div class="car-info-content">

          <div class="vno-content">
            <image class="vno" src="../../images/vno.png"></image>
            <div class="vno-text">
              <text class="vno-text" style="width: 140px;height: 60px ;padding: 5px;color: #ffffff">{{num.vno}}</text>
            </div>
          </div>

          <div class="car-info">
            <div>{{num.vehicleMode.vehicleModelName}}</div>
            <div>{{num.vehicleMode.vehicleModelStr}}</div>
          </div>

          <image class="car" :src="num.vehicleMode.pictureUrl"></image>

        </div>


        <div class="button-container">
          <div class="button"  @click="bookCar(num.vehicleId)">
            <text class="title" style="height: 80px ;padding: 20px;color: #FFFFFF">立即约车</text>
          </div>
        </div>

      </div>
      <!--加载进度框-->
      <wxc-loading :show="isShow"
                   :type="type"
                   :loading-text="loadingText"
                   :interval="interval"></wxc-loading>


    </cell>
  </list>


</template>

<script>
    import { WxcButton,WxcLoading } from 'weex-ui';
    var navigator = weex.requireModule('navigator')
    var modal = weex.requireModule('modal');
    var stream = weex.requireModule('stream');
    var storage = weex.requireModule('storage');
    const LOADMORE_COUNT = 4

    export default {
        data () {
            return {
                lists: [1, 2, 3, 4, 5],
                carList:[],
                authToken:"",
                token:"",
                isShow: false,
                type: 'default',
                interval:0,
                loadingText: '',
                phoneNo:'',
                flag:0
            }
        },
        created:function () {
            storage.getItem('phoneNo', event => {
                this.phoneNo = event.data;
            });
            storage.getItem('authToken', event => {
                this.authToken = event.data;
                storage.getItem('token', event => {
                    this.token = event.data;
                    this.loadCar();
                });
            });

        },
        methods: {
            fetch (event) {
                modal.toast({ message: 'loadmore', duration: 1 })

                setTimeout(() => {
                    const length = this.lists.length
                    for (let i = length; i < length + LOADMORE_COUNT; ++i) {
                        this.lists.push(i + 1)
                    }
                }, 800)
            },
            //登录
            loadCar:function (event) {
                this.interval = 1000;
                var me = this;
                me.isShow = true;
                stream.fetch({
                    method: 'GET',
                    headers:{'Content-Type':'application/json',
                        'token': me.token,
                        'x-auth-token': me.authToken,},
                    url: 'url',
                    type:'json'
                }, function(ret) {
                    if(!ret.ok){
                        me.postResult = "request failed";
                    }else{
                        me.isShow = false;
                        console.log('get:'+JSON.stringify(ret.data));
                        me.carList = ret.data.payload.list;

                    }
                },function(response){
                    console.log('get in progress:'+response.length);
                });
            },
            bookCar:function (event) {
                this.interval = 1000;
                var me = this;
                me.isShow = true;

                console.log('get:'+ event);
                console.log('phoneNo:'+ me.phoneNo);
                stream.fetch({
                    method: 'POST',
                    headers:{'Content-Type':'application/json',
                        'token': me.token,
                        'x-auth-token': me.authToken,},
                    url: 'url',
                    type:'json',
                    body:JSON.stringify({
                        phoneNo: me.phoneNo,
                        source:'2',
                        vehicleId: event,
                    })
                }, function(ret) {
                    if(!ret.ok){
                        me.postResult = "request failed";
                    }else{
                        me.isShow = false;
                        console.log('get:'+JSON.stringify(ret.data));
                        me.toOrder();
                    }
                },function(response){
                    console.log('get in progress:'+response.length);
                });
            },
            toOrder: function (event) {

                navigator.push({
                    url: 'http://192.168.100.104:8085/dist/components/order.js',
                    animated: "true",
                }, event => {
                    modal.toast({ message: 'callback: ' + event })
                })
            },
        }
    }
</script>

<style scoped>
  .vno-content{
    display: flex;
    justify-content: center;
    position: relative;
    width: 140px;
    height: 60px;
    margin-top: 60px;
    margin-left: 20px;
  }

  .vno{
    width: 140px;
    height: 60px;
  }
  .vno-text{
    width: 140px;
    height: 60px;
    line-height: 60px;
    position:absolute;
    top:0;
    left:0;
    display: flex;
    text-align: center;
    font-size: 25px;
    justify-content: center;
    color: #ffffff;
  }

  .item-content{
    display: flex;
    width: 750px;
    flex-direction: column;
  }

  .car-info-content{
    display: flex;
    width: 750px;
    flex-direction: row;
  }

  .car-info{
    display: flex;
    width: 750px;
    flex-direction: column;
    flex: 1;
    margin-top: 30px;
    align-items: center;
    text-align: center;
    justify-content: center;
  }

  .car{
    width: 200px;
    height: 130px;
    margin-right: 6%;
  }
  .button-container{
    display: flex;
    align-items: center;
    justify-content: center;
    width: 750px;
  }
  .button{
    background-color: #1fa67b;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 700px;
    margin-top: 20px;
    border-radius: 10px;
  }







</style>