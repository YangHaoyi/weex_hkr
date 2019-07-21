<template>


  <div class="wrapper">
    <image class="success" src="../../images/pic_pay_success.png"></image>


    <div class="button-container">
    <text class="title" style="height: 80px ;padding: 20px;color: #1fa67b">支付成功</text>
    </div>
  </div>



</template>

<style>
  .wrapper { align-items: center; margin-top: 120px; }
  .success{
    width: 750px;
    height: 400px;
  }
  .button-container{
    display: flex;
    align-items: center;
    justify-content: center;
    width: 750px;
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
        logoUrl: 'http://img1.vued.vanthink.cn/vued08aa73a9ab65dcbd360ec54659ada97c.png',
        target: 'World'
      }
    },
    methods: {
    }
  }
</script>
