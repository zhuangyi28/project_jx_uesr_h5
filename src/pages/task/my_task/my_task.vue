<template>
  <div class="my_task">
    <!-- 订单 -->
    <div class="my_task_order">
      <div class="tab_box">
        <div v-for="item in tabList" @click="selectSort(item)"  v-bind:data-num="item.state" v-bind:class="{'selected color_text':item.show===true}">{{item.sort}}</div>
      </div>
      <div class="tab_box_content"></div>

    </div>
    <!-- 列表 -->
    <div class="my_task_list" v-infinite-scroll="loadMore" infinite-scroll-disabled="moreLoading" infinite-scroll-distance="20" infinite-scroll-immediate-check="false">
      <!-- list -->
      <div class="my_task_one" v-for="item in taskList" v-bind:data-id="item.taskId" @click="lookTaskDetailFn(item.taskId,item.taskState)">
        <div class="my_task_information">
          <div class="task_left">
            <div class="task_title color_background">
              <img src="../../../../static/images/jx_bag.png">
            </div>
            <div class="task_content">
              <div class="contract_is_read" v-if="item.isRead && item.isRead=='1'"><div class="title">NEW!</div></div>
              <div class="name">{{item.taskName}}</div>
              <div class="task_money color_text">
                <span v-if="item.taskMaxUnit==item.taskMinUnit">￥{{item.taskMaxUnit}}</span>
                <span v-else>￥{{item.taskMinUnit}}-￥{{item.taskMaxUnit}}</span>
              </div>
            </div>
          </div>

          <div class="task_status">
            <span class="color_text" v-if="item.taskState=='1'">已报名</span>
            <span class="color_text" v-else-if="item.taskState=='2'">工作中</span>
            <span class="color_text" v-else-if="item.taskState=='3'">待验收</span>
            <span class="color_text" v-else-if="item.taskState=='4'">待收款</span>
            <span v-else-if="item.taskState=='5'">已收款</span>
            <span v-else-if="item.taskState=='6'">已取消</span>
            <span v-else-if="item.taskState=='7'">未被录用</span>
          <!--  <span v-if="item.taskState=='1'">未被录用</span>
            <span v-else-if="item.taskState=='2'">已取消</span>
            <span class="orange" v-else-if="item.taskState=='3'">进行中</span>
            <span class="orange" v-else-if="item.taskState=='4'">已完成</span>-->
            <!--<span v-else-if="item.taskState=='8'">已关闭</span>-->
            <span class="next_btn"></span>
          </div>
        </div>



      </div>
      <div class="loadmore" v-show="!noData">
        <!-- 暂无账单 -->
        <div class="bill_nodata_img" v-if="taskList.length == 0">
          <img src="../../../../static/images/nodetail_img.png">
          <div>暂无相关任务</div>
        </div>
        <!-- 加载完毕-->
        <div class="loadmore_tips" v-else><span class="data">没有更多数据啦~</span></div>
      </div>
      <!-- 显示加载中-->
      <div class="loadmore" v-show="noData">
        <mt-spinner class="loadmore_icon" type="double-bounce" color="#ababab" :size="16"></mt-spinner>
        <div class="loadmore_tips">正在加载</div>
      </div>
    </div>
  </div>
</template>
<script>
  export default {
    name: 'myTask',
    data(){

      return{

        //tab标签 sort-文案 show-是否高亮 state-任务状态
        tabList: [
/*          {
            sort: '全部',
            show: true,
            state:''
          },
          {
            sort: '未被录用',
            show: false,
            state:'7'
          },
          {
            sort: '已取消',
            show: false,
            state:'6'
          },
          {
            sort: '已报名',
            show: false,
            state:'1'
          },
          {
            sort:'工作中',
            show:false,
            state:'2'
          },
          {
            sort:'待验收',
            show:false,
            state:'3'
          },
          {
            sort:'待收款',
            show:false,
            state:'4'
          },
          {
            sort:'已收款',
            show:false,
            state:'5'
          },*/

       {
            sort: '全部',
            show: true,
            state:''
          },
          {
            sort: '未被录用',
            show: false,
            state:'7'
          },
          {
            sort: '已取消',
            show: false,
            state:'6'
          },
          {
            sort: '进行中',
            show: false,
            state:'4'
          },
          {
            sort:'已完成',
            show:false,
            state:'5'
          }
        ],

        taskList:[],//任务列表

        thisState:'',//任务状态

        pageNum: 1,//分页

        pageSize: 10,//一页的数量

        hasMoreData: true,//是否可以加载更多

        noData: true,//是否显示暂无数据 false为隐藏 true为显示

        moreLoading: false,//上拉加载

        scrollTop:''


      }


    },
    mounted(){



      if(this.getStorage(this.getStorage('mobile')+'myTaskItemState')){

        var state = this.getStorage(this.getStorage('mobile')+'myTaskItemState');

        var item = this.findItem(state);

        this.removeStorage(this.getStorage('mobile')+'myTaskItemState');

        this.selectSort(item);

        for(var tab of this.tabList){

          if(tab.state == state){

            tab.show = true;

          }

        }

      }else{

        //重新调用data方法
        //Object.assign(this.$data, this.$options.data());

        this.paging();

      }

    },
    methods:{

      selectSort:function(item) {

        //点击高亮
        for (let i =0 ;i<this.tabList.length;i++) {

          this.tabList[i].show=false;

        }

        item.show=true;

        //存储此时状态
        this.thisState = item.state;

        this.pageNum = 1;//初始值为2

        this.hasMoreData = true;//是否可以加载更多

        this.noData = true;//是否显示暂无数据 true为隐藏 false为显示

        this.taskList = [];//发薪企业列表

        console.log('此时的状态'+this.thisState);

        this.paging();



      },

      paging:function () {

        console.log('嫩否加载'+this.hasMoreData)

        if (this.hasMoreData) {

          this.hasMoreData = false;

          //如果值为空 选择全部
          if (!this.thisState) {

            //全部不需要传
            this.loadList()
          }
          else {

            //传入对应的状态
            this.loadList(this.thisState)
          }
        }


      },

      loadList:function (num) {

        let thisData = {};

        let _this= this


        if (num) {

          thisData = {

            taskState: num,

            pageNum: this.pageNum,

            pageSize: this.pageSize

          }


        }

        else {


          thisData = {

            pageNum: this.pageNum,

            pageSize: this.pageSize
          }

        }


        /**
         * 接口：我的众包任务
         * 请求方式：POST
         * 接口：user/task/getapplytask
         * 入参：taskState(null-全部 1-已报名 2-工作中 3-待验收 4-待收款 5-已收款 6-已取消 7-未被录用 8-已关闭),pageNum，pageSize
         **/

        this.$http({

          method: 'post',

          url: process.env.API_ROOT + 'user/task/getapplytask',

          params: thisData,

        }).then((res) => {

          console.log(res.data)


          let thisList = res.data.data.list;


          if(thisList){


            if (thisList.length < 10) {

              if(this.pageNum=='1'){

                this.taskList = res.data.data.list;

              }


              else {


                let lastList = this.taskList;

                //把获取到的list合并成一个数组
                let nowList = lastList.concat(thisList);

                this.taskList = nowList;

              }


              //不加载并且显示没有更多数据

              this.hasMoreData = false;

              this.noData = false


            }

            else {

              console.log('有分页')

              let lastList = this.taskList;

              //把获取到的list合并成一个数组
              let nowList = lastList.concat(thisList);

              this.taskList = nowList;

              //页数加1

              this.pageNum = this.pageNum + 1;
              //可以加载
              setTimeout(function () {

                _this.hasMoreData = true

              }, 500)


            }



          }

          else if(!thisList&&this.pageNum==1){

            console.log('没有数据')

            this.taskList = []

            this.hasMoreData = false;

            this.noData = false;


          }



          /*        if(res.data.data.list){

                    for(var i=0; i<res.data.data.list.length;i++){

                      this.taskList.push(res.data.data.list[i])

                    }


                    /!*f(thisList.length<this.pageSize){

                        //如果是第一页的话，直接显示list，否则拼接list

                        if(this.pageNum==1){

                          this.taskList = thisList;

                        }

                        else {


                          //上一次获取到的list

                          let lastList = this.taskList;

                          //把获取到的list合并成一个数组
                          let nowList = lastList.concat(thisList);

                          this.taskList = nowList;


                        }

                        this.hasMoreData = false;

                        this.noData = false


                    }*!/
                    if(res.data.list.length<10){

                      this.hasMoreData = false;

                      this.noData = false

                    }else {

                      this.hasMoreData = false;

                      this.noData = true;
                    }


                  }
                  else {


                    this.taskList = [];

                    this.hasMoreData = false;

                    this.noData = false;

                  }*/


        }).catch((res) => {})



      },

      loadMore:function () {

        console.log('加载更多');

        this.paging()

      },

      lookTaskDetailFn:function (taskId,thisState) {

        //存下任务id 在任务详情中取
        this.setStorage('taskId',taskId);

        (!!this.thisState) && this.setStorage(this.getStorage('mobile')+'myTaskItemState',this.thisState);

        this.$router.push({path: '/taskDetail',query: {taskId: taskId, thisState: thisState}});



      },

      findItem: function (state) {

        var item = {};

        switch (state) {

          case '':

            item.sort = '全部';

            break;

          case '1':

            item.sort = '已报名';

            break;

          case '2':

            item.sort = '工作中';

            break;

          case '3':

            item.sort = '待验收';

            break;

          case '4':

            item.sort = '待收款';

            break;

          case '5':

            item.sort = '已收款';

            break;

          case '6':

            item.sort = '已取消';

            break;

          case '7':

            item.sort = '未被录用';

            break;

        }

        item.state = state;

        item.show = false;

        return item;

      }

    },


  }
</script>
<style lang="less" scoped>
  @import "my_task.less";
</style>

