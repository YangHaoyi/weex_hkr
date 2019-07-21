<template>
  <div class="item-content">

    <div class="car-info-content">

      <div class="vno-content">
        <image class="vno" src="../../images/vno.png"></image>
        <div class="vno-text">
          <text class="vno-text" style="width: 140px;height: 60px ;padding: 5px;color: #ffffff">{{carInfo.vno}}</text>
        </div>
      </div>

      <div class="car-info">
        <div>{{carInfo.vehicleMode.vehicleModelName}}</div>
        <div>{{carInfo.vehicleMode.vehicleModelStr}}</div>
      </div>

      <image class="car" :src="carInfo.vehicleMode.pictureUrl"></image>

    </div>


    <div class="order-line"> </div>

    <div style="display: flex;flex-direction: row;justify-content:space-between;">
      <text class="order-detail-title">
        行驶里程
      </text>
      <text class="order-detail-title">
        租用时长
      </text>
      <text class="order-detail-title">
        消费金额
      </text>
    </div>

    <div style="display: flex;flex-direction: row;justify-content:space-between;">
      <text class="order-detail-value">
        {{realTimeInfo.vehicleMileage}}公里
      </text>
      <text class="order-detail-value">
        {{realTimeInfo.spendsTime}}分钟
      </text>
      <text class="order-detail-value">
         ￥{{realTimeInfo.rentalCost}}
        <!--<block wx:if="{{realTimeInfo.rentalCost>0}}">-->
        <!--￥{{filters.toFix(realTimeInfo.rentalCost)}} -->
        <!--</block>-->

      </text>
    </div>
    <div class="order-line"> </div>



    <div class="button-container">
      <div class="button"  @click="controlCar()">
          <text class="title" style="height: 80px ;padding: 20px;color: #FFFFFF">{{controlButtonText}}</text>
      </div>
    </div>

  </div>
</template>

<style>

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

  .order-detail-title{
    font-size: 25px;
    display: flex;
    flex: 1;
    text-align: center;
    justify-content:center;
    color: #999999;
  }
  .order-detail-value{
    font-size: 30px;
    display: flex;
    flex: 1;
    text-align: center;
    justify-content:center;
    color: #333333;
  }
  .order-line{
    display: flex;
    height: 1px;
    margin-left: 25px;
    margin-right: 25px;
    margin-top: 10px;
    margin-bottom: 10px;
    background: #f2f2f2;
  }

</style>

<script>
    import { WxcButton,WxcLoading } from 'weex-ui';
    var navigator = weex.requireModule('navigator')
    var modal = weex.requireModule('modal');
    var stream = weex.requireModule('stream');
    var storage = weex.requireModule('storage');

  export default {

    data () {
      return {
          orderCurrent:{},
          carInfo:{},
          appStyle:{},
          startBranch:{},
          endBranch:{},
          realTimeInfo:{},
          controlButtonText:"",

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
                  this.requestOrderCurrent();
              });
          });

      },
    methods: {

        //请求订单基础信息
        requestOrderCurrent:function(){
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
                    me.orderCurrent = ret.data.payload;
                    storage.setItem('orderId', me.orderCurrent.orderId, event => {
                        console.log('success');
                    });
                    me.requestCarInfo(ret.data.payload.vehicleId);
                    me.requestAppStyle(ret.data.payload.billingRuleId);
                    me.requestStartBranch(ret.data.payload.startRentalShopId);
                    if (ret.data.payload.endRentalShopId!=0){
                        me.requestEndBranch(ret.data.payload.endRentalShopId);
                    }
                    me.requestRealTime();
//                    if (res.data.payload.billingStartTime!=0){
//                        that.setData({
//                            billingStartTime: that.formateTime(res.data.payload.billingStartTime)
//                        });
//                    }
//                    if (res.data.payload.billingEndTime != 0) {
//                        that.setData({
//                            billingEndTime: that.formateTime(res.data.payload.billingEndTime)
//                        });
//                    }
                    if(ret.data.payload.orderStatus==1){
                        me.controlButtonText = "取车";
                    }else if(ret.data.payload.orderStatus==2){
                        me.controlButtonText = "还车";
                    }else {
                        me.controlButtonText = "立即支付";
                    }
                }
            },function(response){
                console.log('get in progress:'+response.length);
            });
        },

        //请求车辆信息
        requestCarInfo:function(vehicleId){
            var me = this;
            stream.fetch({
                method: 'GET',
                headers:{'Content-Type':'application/json',
                    'token': me.token,
                    'x-auth-token': me.authToken,},
                url: 'url'+vehicleId,
                type:'json'
            }, function(ret) {
                if(!ret.ok){
                    me.postResult = "request failed";
                }else{
                    me.carInfo = ret.data.payload;
                }
            },function(response){
                console.log('get in progress:'+response.length);
            });
        },

        //请求计价规则
        requestAppStyle:function(billingRuleId){
            var me = this;
            stream.fetch({
                method: 'GET',
                headers:{'Content-Type':'application/json',
                    'token': me.token,
                    'x-auth-token': me.authToken,},
                url: 'url'+billingRuleId,
                type:'json'
            }, function(ret) {
                if(!ret.ok){
                    me.postResult = "request failed";
                }else{
                    me.appStyle=ret.data.payload
                }
            },function(response){
                console.log('get in progress:'+response.length);
            });
        },

        //请求计价规则
        requestAppStyle:function(billingRuleId){
            var me = this;
            stream.fetch({
                method: 'GET',
                headers:{'Content-Type':'application/json',
                    'token': me.token,
                    'x-auth-token': me.authToken,},
                url: 'url'+billingRuleId,
                type:'json'
            }, function(ret) {
                if(!ret.ok){
                    me.postResult = "request failed";
                }else{
                    me.appStyle=ret.data.payload
                }
            },function(response){
                console.log('get in progress:'+response.length);
            });
        },

        //请求取车网点信息
        requestStartBranch:function(startRentalShopId){
            var me = this;
            stream.fetch({
                method: 'GET',
                headers:{'Content-Type':'application/json',
                    'token': me.token,
                    'x-auth-token': me.authToken,},
                url: 'url'+startRentalShopId,
                type:'json'
            }, function(ret) {
                if(!ret.ok){
                    me.postResult = "request failed";
                }else{
                    me.startBranch=ret.data.payload
                }
            },function(response){
                console.log('get in progress:'+response.length);
            });
        },

        //请求还车网点信息
        requestEndBranch:function(endRentalShopId){
            var me = this;
            stream.fetch({
                method: 'GET',
                headers:{'Content-Type':'application/json',
                    'token': me.token,
                    'x-auth-token': me.authToken,},
                url: 'url'+endRentalShopId,
                type:'json'
            }, function(ret) {
                if(!ret.ok){
                    me.postResult = "request failed";
                }else{
                    me.endBranch=ret.data.payload
                }
            },function(response){
                console.log('get in progress:'+response.length);
            });
        },

        //请求订单实时信息
        requestRealTime:function(){
            var me = this;
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
                    me.realTimeInfo=ret.data.payload
                }
            },function(response){
                console.log('get in progress:'+response.length);
            });
        },

        //控车指令
        controlCar:function(){
            var that = this;
            if (that.orderCurrent.orderStatus==1){
                that.takeCar();
            } else if (that.orderCurrent.orderStatus == 2){
                that.requestReturn();
            } else if (that.orderCurrent.orderStatus == 4){
                navigator.push({
                    url: 'http://192.168.100.104:8085/dist/components/pay.js',
                    animated: "true",
                }, event => {
                    modal.toast({ message: 'callback: ' + event })
                })
            }

        },

        //请求取车
        takeCar:function(){
            var me = this;
            stream.fetch({
                method: 'PUT',
                headers:{'Content-Type':'application/json',
                    'token': me.token,
                    'x-auth-token': me.authToken,},
                url: "url" + me.orderCurrent.orderId+"/start",
                type:'json'
            }, function(ret) {
                if(!ret.ok){
                    me.postResult = "request failed";
                }else{
                    modal.toast({message: "取车成功"});
                    me.requestOrderCurrent();
                }
            },function(response){
                console.log('get in progress:'+response.length);
            });
        },

        //请求还车
        requestReturn:function(){
            var me = this;
            stream.fetch({
                method: 'PUT',
                headers:{'Content-Type':'application/json',
                    'token': me.token,
                    'x-auth-token': me.authToken,},
                url: "url" + me.orderCurrent.orderId + "/return",
                type:'json'
            }, function(ret) {
                if(!ret.ok){
                    me.postResult = "request failed";
                }else{
                    if(ret.data.code==0){
                        modal.toast({message: "还车成功"});
                    }else {
                        modal.toast({message: "还车关系校验失败"});
                    }

                    me.requestOrderCurrent();
                }
            },function(response){
                console.log('get in progress:'+response.length);
            });
        },

    }
  }
</script>
