<template>
  <!--下一步按钮-->
  <div class="button-container">
    <div class="button" @click="balancePay">
      <text class="title" style="height: 80px ;padding: 20px;color: #FFFFFF">余额支付</text>
    </div>
  </div>
</template>

<style>
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

<script>
    import { WxcButton,WxcLoading } from 'weex-ui';
    var navigator = weex.requireModule('navigator')
    var modal = weex.requireModule('modal');
    var stream = weex.requireModule('stream');
    var storage = weex.requireModule('storage');
  export default {
    data () {
      return {
          orderId:""
      }
    },
      created:function () {
          storage.getItem('authToken', event => {
              this.authToken = event.data;
              storage.getItem('token', event => {
                  this.token = event.data;
              });
          });
          storage.getItem('orderId',event =>{
              this.orderId = event.data;
          })
      },
    methods: {
        balancePay:function () {
                var me = this;
                stream.fetch({
                    method: 'GET',
                    headers:{'Content-Type':'application/json',
                        'token': me.token,
                        'x-auth-token': me.authToken,},
                    url:"url" + me.orderId+"/type=8/orderType=1?hkcoin=0",
                    type:'json'
                }, function(ret) {
                    if(!ret.ok){
                        me.postResult = "request failed";
                    }else{
                        me.toSuccess();
                    }
                },function(response){
                    console.log('get in progress:'+response.length);
                });
        },
        toSuccess:function () {
            navigator.push({
                url: 'http://192.168.100.104:8085/dist/components/success.js',
                animated: "true",
            }, event => {
                modal.toast({ message: 'callback: ' + event })
            })
        },
    }
  }
</script>
