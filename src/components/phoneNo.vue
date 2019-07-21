<!--<template>-->
<!--<div>-->
<!--<wxc-button text="Open Popup"-->
<!--@wxcButtonClicked="buttonClicked">-->
<!--</wxc-button>-->


<!--<wxc-loading :show="isShow"-->
<!--:type="type"-->
<!--:loading-text="loadingText"-->
<!--:interval="interval"></wxc-loading>-->

<!--</div>-->
<!--</template>-->

<!--<script>-->
<!--import { WxcButton,WxcLoading } from 'weex-ui';-->

<!--module.exports = {-->
<!--components: { WxcButton,WxcLoading },-->
<!--data: () => ({-->
<!--isShow: false,-->
<!--type: 'default',-->
<!--loadingText: ''-->
<!--}),-->
<!--methods: {-->
<!--buttonClicked () {-->
<!--this.isShow = true;-->
<!--setTimeout(() => {-->
<!--// 正常使用时候直接设置即可，不需setTimeout-->
<!--this.interval = 1000;-->
<!--this.isShow = true;-->
<!--}, 10)-->
<!--},-->
<!--overlayClicked () {-->
<!--this.isShow = false;-->
<!--}-->
<!--}-->
<!--};-->
<!--</script>-->
<template>

  <div>

    <!--手机号输入框-->
    <div class="input-container">
      <text class="input-notice">手机号</text>
      <input type="number" placeholder="请输入手机号" class="input" :autofocus=true value="" @change="onchange" @input="oninput" maxlength="11"/>
    </div>

    <!--下一步按钮-->
    <div class="button-container">
      <div class="button" @click="clickTypePost">
        <text class="title" style="height: 80px ;padding: 20px;color: #FFFFFF">下一步</text>
      </div>
    </div>

    <!--加载进度框-->
    <wxc-loading :show="isShow"
                 :type="type"
                 :loading-text="loadingText"
                 :interval="interval"></wxc-loading>

  </div>



</template>

<style scoped>

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

  .input-notice{
    display: flex;
    color: #1fa67b;
    font-size: 30px;
    align-items: center;
    justify-content: center;
    margin-left: 25px;
    margin-right: 15px;
  }


</style>

<script>

    import { WxcButton,WxcLoading } from 'weex-ui';
    var navigator = weex.requireModule('navigator')
    var modal = weex.requireModule('modal');
    var stream = weex.requireModule('stream');
    var storage = weex.requireModule('storage');
    //    var config = require('./config.js')

    module.exports = {
        components: { WxcButton,WxcLoading },
        data: function () {
            return {
                txtInput: '',
                txtChange: '',
                txtReturnType: '',
                txtSelection:'',
                inputPhone:false,
                autofocus: false,
                isShow: false,
                type: 'default',
                interval:0,
                loadingText: ''
            };
        },
        methods: {
            ready: function () {
                var self = this;
                setTimeout(function () {
                    self.autofocus = true;
                }, 1000);
            },
            showLoading:function () {
                this.isShow = true;
            },
            clickTypePost:function(){
                this.interval = 1000;
                var me = this;
                me.isShow = true;
                stream.fetch({
                    method: 'POST',
                    headers:{'Content-Type':'application/json'},
                    url: 'url',
                    type:'json',
                    body:JSON.stringify({ phoneNo: me.txtInput})
                }, function(ret) {
                    if(!ret.ok){
                        me.postResult = "request failed";
                    }else{
                        me.isShow = false;
                        console.log('get:'+JSON.stringify(ret.data));
                        console.log('get:'+JSON.stringify(ret.data.payload.timeout));
//                        me.postResult = JSON.stringify(ret.data);
                        me.toCode();
                    }
                },function(response){
                    console.log('get in progress:'+response.length);
//                    me.postResult = "bytes received:"+response.length;
                });
            },

            toCode: function (event) {

                storage.setItem('phoneNo', this.txtInput, event => {
                    console.log('success');
                });

                navigator.push({
                    url: 'http://192.168.100.104:8082/dist/components/code.js',
                    animated: "true",
                }, event => {
                    modal.toast({ message: 'callback: ' + event })
                })
            },

            onchange: function (event) {
                this.txtChange = event.value;
                console.log('onchange', event.value);
            },
            onreturn: function (event) {
                this.txtReturnType = event.returnKeyType;
                console.log('onreturn', event.type);
            },
            oninput: function (event) {
                this.txtInput = event.value;
                console.log('oninput', event.value);
            },
            focus: function () {
                this.$refs['input1'].focus();
            },
            blur: function () {
                this.$refs['input1'].blur();
            },
            setRange: function() {
                console.log(this.$refs["inputselection"]);
                this.$refs["inputselection"].setSelectionRange(2, 6);
            },
            getSelectionRange: function() {
                console.log(this.$refs["inputselection"]);
                var self = this;
                this.$refs["inputselection"].getSelectionRange(function(e) {
                    self.txtSelection = e.selectionStart +'-' + e.selectionEnd;
                });
            }
        }
    };
</script>
