<template>

  <div class="authentication">

    <div class="banner_box">
      <div class="iconfont icon-new_signing color_text"></div>
      <div class="icon_ball_big color_background"></div>
      <div class="icon_ball_middle color_background">
        <img src="../../../../static/images/jx_right_icon.png">
      </div>
      <div class="three border_color"></div>
      <div class="little_point color_background"></div>
      <div class="ball color_background"></div>
      <div class="border_ball border_color"></div>
      <div class="shadow"></div>
    </div>

    <div class="text_box">
      <p>首次查看需要进行身份验证，验证通过后即可接收该企业发放的个人收入金额或收入明细</p>
      <p class="color_text">输入错误3次将被锁定，次日自动解锁</p>
    </div>


    <div class="content_box">

      <div class="field">
        <input disabled v-model="userName">
      </div>

      <div class="field">
        <input type="text" maxlength="6" v-model="idCard" autofocus="autofocus" :placeholder="placeholderValue"/>
      </div>

    </div>

    <orangeBtn :name="btnName" v-on:clickEvent="identityFn"></orangeBtn>

    <serviceArea :type1="serviceLeft" :type2="serviceRight" :iconName1="iconName1" :iconName2="iconName2" v-on:clickEventRight="customerFn" v-on:clickEventLeft="$router.push('/helpCenter')" :spanShow="true"></serviceArea>

  </div>


</template>

<script>

  //按钮
  import orangeBtn from '../../../components/orange_btn/orange_btn'

  //帮助
  import serviceArea from '../../../components/service/service'

  import { customerInit, customerClick } from "../../../../static/js/basic"

  export default {

    name: 'authentication',

    components: {

      orangeBtn: orangeBtn,

      serviceArea: serviceArea,

    },


    data(){

      return {

        idCard: '',

        userName: '',

        mobile: '',

        goFrozenValue: '',

        placeholderValue: '',

        btnName: '确定',//按钮名称

        serviceLeft: '查看帮助',

        serviceRight: '联系客服',

        iconName1:'icon-help_question',

        iconName2:'icon-withdraw_custom',


      }

    },

    mounted(){

      //美恰初始化
      customerInit({
        name:this.getStorage('userName'),// 名字
        tel:this.getStorage('mobile'),// 电话
      });

      /**
       * 接口：用户中心
       * 请求方式：POST
       * 接口：/user/center/usercenter
       * 入参：mobile
       **/


      this.$http({

        method: 'post',

        url: process.env.API_ROOT + 'user/center/usercenter',

      }).then((res) => {

        console.log(res.data);

        this.userName = res.data.data.userName

        this.mobile = res.data.data.mobile


        if (res.data.data.idType == '1') {


          this.placeholderValue = '请输入身份证后六位'


        }
        else if (res.data.data.idType == '2') {

          this.placeholderValue = '请输入护照后六位'


        }

        else if (res.data.data.idType == '3') {


          this.placeholderValue = '请输入回乡证后六位'


        }

        else if (res.data.data.idType == '4') {


          this.placeholderValue = '请输入台胞证后六位'


        }


      })


    },

    methods: {

      identityFn: function () {


        var thisEntId = this.getStorage('entId');



        /**
         * 接口：身份验证
         * 请求方式：GET
         * 接口：/salary/home/selectidnumber
         * 入参：idCard
         * */

        this.$http({

          method: 'get',

          url: process.env.API_ROOT + 'salary/home/selectidnumber',

          params: {

            idCard : this.idCard,

            entId:thisEntId,

          },


      }).then((res) => {

            console.log(res.data)


          var thisCode = res.data.code;

          //判断验证码
          if (thisCode == '-1') {

            this.$toast({

              message: res.data.msg,
              position: 'bottom',
              duration: 1500

            });

          }

          else if (thisCode == '-2') {

            this.$router.replace('/locked')

          }

          //验证成功后显示工资
          else if (thisCode == '0000') {

            //var goFrozen = (localStorage.getItem('goFrozen') && localStorage.removeItem('goFrozen')) || false;

            var goFrozen = localStorage.getItem('goFrozen')

            console.log('数值'+goFrozen)

            if(goFrozen=='1'){

              this.$router.go(-2)

            }else if(goFrozen=='5'){

              this.$router.go(-1)

            }else if(goFrozen == '3'){

              this.$router.replace('/salary');

            }else if(goFrozen == '2'){

              this.$router.replace('/balance');

            }
            else {

              this.$router.go(-1);

            }

            //this.$router.go(-1)



  /*          if(goFrozenValue=='1'){

              console.log('解冻回工资余额')

           /!*   wx.navigateBack({
                delta: 2
              })*!/

            }
            else if(goFrozenValue=='2'){


              this.$router.go(-1)

            }

            else {

              this.$router.go(-1)

            }*/


          }



        })



      },

      customerFn:function () {
        customerClick()
      },



    }


  }


</script>
<style lang="less">

  @import "authentication.less";

</style>
