<template>
  <div class="login" id="login">

    <div class="login_form">

      <div class="company_logo">
        <img v-bind:src=src>
      </div>

      <div class="content_box">
        <div class="field">
          <i class="iconfont icon-sign_phone color_text"></i>
          <input type="number" pattern="\d*" @blur="lostPointFn" v-model="mobile" oninput="if(value.length > 11)value = value.slice(0, 11)" placeholder="请输入手机号" class="tel" >
        </div>
        <div class="field">
          <i class="iconfont icon-sign_password color_text"></i>
          <input type="password" @blur="lostPointFn" v-model="password" placeholder="请输入密码" class="password" maxlength="20"  v-on:keyup.enter="signin">
        </div>

      </div>

    </div>

    <ul class="bg-bubbles">
      <li class="bubble_color" v-for="(item,index) in bubbles" :key="index"></li>
    </ul>


    <orangeBtn v-on:clickEvent="signin" :name="btnName"></orangeBtn>
    <div class="login_button">
      <div class="another_page">
        <div @click="$router.push('/forgetPsw')">忘记密码?</div>
        <div @click="$router.push('/Register')">注册</div>
      </div>
    </div>

  </div>
</template>
<script>
  import orangeBtn from '../../../components/orange_btn/orange_btn'

  export default {
    name: 'login',
    components: {
      orangeBtn: orangeBtn,
    },

    data(){
        return{

          bubbles:[],

          mobile: '',

          password:'',

          btnName:'登录',//按钮名称

          Authorization:'',

          code:'',

          src: "./static/images/logo.png",

          channel:''

        }

    },


    created() {
      this.bubbles.length = 5;
    },

    mounted(){


      var str = this.getCookie('anotherCompany')|| 'orange';

      str=='orange'?this.channel='jiaxin':this.channel = str



      switch (str) {

        case 'orange':

          this.src = "./static/images/logo.png";

          break;

        case 'blue':

          this.src = "./static/images/paiyun/logo.png";

          break;

        case 'lexiang':

          this.src = "./static/images/lexiangyibao/logo.png";

          break;

        case 'rongyada':

          this.src = "./static/images/rongyada/logo.png";

          break;

        case 'yibangsheng':

          this.src = "./static/images/yibangsheng/logo.png";

          break;

        case 'xinqidian':

          this.src = "./static/images/xinqidian/logo.png";

          break;

        case 'zuirenli':

          this.src = "./static/images/zuirenli/logo.png";

          break;

        case 'jianzhile':

          this.src = "./static/images/jianzhile/logo.png";

          break;

        case 'facaile':

          this.src = "./static/images/facaile/logo.png";

          break;

      }

    // console.log(this.getStorage('thisKey'))

    },
    methods:{

      signin:function () {

            let _this=this;

            let str = location.href;

            if(str.includes('code')){

              var thisUserCode = str.split('?')[1].split('&')[0].split('=')[1];

            }

            else {

              var thisUserCode = 0
            }


          if(_this.mobile==''){

            this.$toast({
              message: '请输入手机号',
              position: 'bottom',
              duration: 1500


            });


          }

          else if(_this.password ==''){

            this.$toast({
              message: '请输入密码',
              position: 'bottom',
              duration: 1500


            });



          }

          else {

            /**
             * 接口：登录
             * 请求方式：POST
             * 接口：login
             * 入参：mobile，password,code
             **/

            this.$http({

              method: 'post',

              url:process.env.API_ROOT+'login',

              params: {

                mobile:_this.mobile,

                password:hexMD5(_this.password),

                code: thisUserCode,

                device: 'platform',

                channel:_this.channel

              }


            }).then(function (res) {

              console.log(res.data);

              if(res.data.code=='-1'){

                this.$toast({
                  message: res.data.msg,
                  position: 'bottom',
                  duration: 1500


                });

              }

              else if(res.data.code=='0000'){

                  this.setStorage('userId',res.data.data.userId);

                  this.$router.go(-1);
              }



            }.bind(this)).catch((res)=>{})

          }

        },

      lostPointFn:function () {

       document.body.scrollTop =document.documentElement.scrollTop = window.pageYOffset = 100;

      }

    }

  }
</script>
<style lang="less" scoped>
  @import "login.less";

</style>
