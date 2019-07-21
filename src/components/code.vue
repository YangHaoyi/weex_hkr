<template>
  <div class="wrapper">

    <!--顶部分割线-->
    <div class="line"/>

    <!--验证码提示-->
    <div class="code-notice">
      <text class="notice-text">已发送验证码短信到</text>
    </div>

    <!--电话号码-->
    <div class="phone-no">
      <text class="input-notice">{{phoneNo}}</text>
    </div>

    <!--验证码输入框-->
    <div class="input-container">
      <text class="input-notice">验证码</text>
      <input type="number" placeholder="输入验证码" class="input" :autofocus=true value="" @input="oninput" maxlength="4"/>
    </div>


    <!--登录按钮-->
    <div class="button-container">
      <div class="button" @click="login">
        <text class="title" style="height: 80px ;padding: 20px;color: #FFFFFF">登录</text>
      </div>
    </div>


    <!--加载进度框-->
    <wxc-loading :show="isShow"
                 :type="type"
                 :loading-text="loadingText"
                 :interval="interval"></wxc-loading>



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

  .input {
    font-size: 30px;
    height: 40px;
    width: 750px;
  }
  .input-container{
    display: flex;
    width: 100%;
    margin-top: 20px;
    flex-direction: row;
  }

  .code-notice{
    width: 750px;
    display: flex;
    text-align: center;
    align-items: center;
    justify-content: center;
  }

  .notice-text{
    color: #999999;
    font-size: 30px;
  }


  .phone-no{
    display: flex;
    width: 750px;
    justify-content: center;
    text-align: center;
    align-items: center;
    color: #1fa67b;
    font-size: 30px;
    margin-top: 30px;
  }

  .input-notice{
    display: flex;
    color: #1fa67b;
    font-size: 30px;
    align-items: center;
    justify-content: center;
    margin-left: 25px;
    margin-right: 15px;
  }
  .line{
    background: #f2f2f2;
    width: 750px;
    height: 1px;
  }
</style>

<script>
    var navigator = weex.requireModule('navigator');
    var storage = weex.requireModule('storage');
    var stream = weex.requireModule('stream');
    import { WxcButton,WxcLoading } from 'weex-ui';
    var modal = weex.requireModule('modal');

    export default {
        components: { WxcButton,WxcLoading },
        data () {
            return {
                txtInput: '',
                phoneNo:'',
                isShow: false,
                type: 'default',
                interval:0,
                loadingText: ''
            }
        },
        created:function () {
            storage.getItem('phoneNo',event =>{
                this.phoneNo = event.data;
            })
        },
        methods: {
//            跳转网点
            toBranch: function (event) {

                navigator.push({
                    url: 'http://192.168.100.104:8085/dist/components/branch.js',
                    animated: "true"
                }, event => {
                    modal.toast({ message: 'callback: ' + event })
                })
            },

            //监听输入
            oninput: function (event) {
                this.txtInput = event.value;
            },

            //登录
            login:function (event) {
                this.interval = 1000;
                var me = this;
                me.isShow = true;
                console.log('phone'+me.phoneNo);
                console.log('code'+me.txtInput);
                stream.fetch({
                    method: 'POST',
                    headers:{'Content-Type':'application/json'},
                    url: 'url',
                    type:'json',
                    body:JSON.stringify({
                        phoneNo: me.phoneNo,
                        deviceToken:'Aj3QS7A_8AtitAf5fPhDu5sWDoXv1U8XTTmZ74UTuH99',
                        identifyingCode: me.txtInput,
                        openId:'',
                        source:'2',
                        cityID: '1758',
                    })
                }, function(ret) {
                    if(!ret.ok){
                        me.postResult = "request failed";
                    }else{
                        me.isShow = false;
                        console.log('get:'+JSON.stringify(ret));
                        storage.setItem('phoneNo', me.phoneNo, event => {
                            console.log('success');
                        });
                        storage.setItem('login', "login", event => {
                            console.log('success');
                        });
                        storage.setItem('authToken', ret.data.payload.authToken, event => {
                            console.log('success'+ret.data.payload.authToken);
                        });
                        storage.setItem('token', ret.data.payload.token, event => {
                            console.log('success'+ret.data.payload.token);
                        });
                        me.toBranch();
                    }
                },function(response){
                    console.log('get in progress:'+response.length);
                });
            },

            //保存token
            saveToken:function (event) {
                storage.setItem('phoneNo', this.phoneNo, event => {
                    console.log('success');
                });
                storage.setItem('login', "login", event => {
                    console.log('success');
                });
                storage.setItem('authToken', event.data.payload.authToken, event => {
                    console.log('success');
                });
                storage.setItem('token', event.data.payload.token, event => {
                    console.log('success');
                });
            }
        }
    }
</script>
