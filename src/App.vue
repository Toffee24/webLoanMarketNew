<template>

  <div class="mainView" ref="main">
    <div ref="top">
      <div class="banner">差钱吗贷款超市</div>
      <div class="tips" ref="tips">技巧：想提高借款成功率？可多申请几款速贷产品哦～</div>
      <div class="swiper-pagination" slot="pagination"></div>
    </div>
    <swiper :options="swiperOption" :not-next-tick=false ref="mySwiper">
      <swiper-slide v-for="(item,idx) in TAB_NAME" :key="idx">
        <pull-to :is-bottom-bounce=true :top-load-method="refresh">
          <ul class="scroll-ul">
            <li v-for="(item,idx) in pageData" :key="idx">
              <a :data="item.id" @click="static" :clickUrl="item.url">
                <div class="smallLoanItem">
                  <div class="leftLogo"><img v-lazy="item.avatar"></div>
                  <div class="rightInfo">
                    <div class="infoTop">
                      <div class="infoLeft">
                        <p class="name">{{item.name}}<span class="hot" v-if="item.isHot == 1">热</span></p>
                        <p class="borrowAmount">借款额度：{{item.minBorrowAmount}}~{{item.maxBorrowAmount}}元</p>
                        <p class="ratio">参考{{item.ratio == 1 ? '日' : '月'}}利率：{{item.ratio}}%</p>
                      </div>
                      <div class="infoRight">
                        <div class="borrowNumber">{{item.applyNum}}人下款成功</div>
                        <div class="time">最快<span>{{item.sendMoneyTime}}</span>下款</div>
                      </div>
                    </div>
                    <div class="infoBottom">
                      <div class="need">需</div>
                      {{item.materialState || '暂无'}}
                    </div>
                  </div>
                </div>
              </a>
            </li>
          </ul>
        </pull-to>
      </swiper-slide>
    </swiper>
  </div>
</template>


<script>
  import axios from 'axios'
  import PullTo from 'vue-pull-to'
  //  const SERVERPATH = 'http://116.62.64.190:8080' //线上
  const SERVERPATH = 'http://116.62.146.57:8080/mobile'; //借款端测试
  //  const SERVERPATH = 'http://192.168.199.166:8099' //胡磊
  let TAB_NAME_FORK = []
  export default {
    name: 'v-recommend',
    created(){
      this.$nextTick(() => {
        this.computedHeight()
        this.$refs.tips.addEventListener('click', (ev) => {
          this.$refs.tips.style.display = 'none'
          this.computedHeight()
        })
      })
      //链接统计接口
      let channel = this.GetQueryString('channel')
      axios.post(SERVERPATH + '/api/app/next/common/participate/wap/source/link/statistics', {
        from: channel
      }, {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      })
        .then(function (response) {
          // if (response.data.code == 626) {
          console.log(response);
          // }
        }).catch(function (error) {
        console.log(error);
      })

    },
    data() {
      return {
        TAB_NAME: [],
        pageData: [],
        pageNumId: 0,
        swiperOption: {
          pagination: '.swiper-pagination',
          direction: 'horizontal',
          onlyExternal: true,
          paginationClickable: true,
          paginationType: 'bullets',
          paginationBulletRender: (swiper, index, className) => {
            if (TAB_NAME_FORK[index]) {
              return `<div class="${className}">${TAB_NAME_FORK[index]}</div>`;
            } else {
              return `<div class="${className}"></div>`;
            }
          },
          onSlideChangeStart: (swiper) => {
            this.pageNumId = this.TAB_NAME[swiper.activeIndex].id
            this.getSmallLoanData(this.pageNumId)
          },
          onInit: (swiper) => {
            //应经在坑里出不来了
            axios({
              method: 'get',
              url: SERVERPATH + '/api/app/next/common/wap/get/amount',
              params: {type: 1},
              headers: {
                'Content-type': 'application/x-www-form-urlencoded'
              }
            }).then((res) => {
              this.TAB_NAME = res.data.resultModel
              this.pageNumId = res.data.resultModel[0].id
              res.data.resultModel.forEach((el, idx) => {
                TAB_NAME_FORK.push(el.tagName)
              })
              //请求相应列表的数据
              let id = this.TAB_NAME[swiper.activeIndex].id
              this.getSmallLoanData(id)
            }).catch((err) => {
              console.log(err)
            })
          }
        }
      }
    },
    computed: {
      swiper() {
        return this.$refs.mySwiper.swiper
      }
    },
    components: {
      PullTo
    },
    methods: {
      getAmount(){
        axios({
          method: 'get',
          url: SERVERPATH + '/api/app/next/common/wap/get/amount',
          params: {type: 1},
          headers: {
            'Content-type': 'application/x-www-form-urlencoded'
          }
        }).then((res) => {
          this.TAB_NAME = res.data.resultModel
          res.data.resultModel.forEach((el, idx) => {
            TAB_NAME_FORK.push(el.tagName)
          })
        }).catch((err) => {
          console.log(err)
        })
      },
      getSmallLoanData(id, loaded){
        axios({
          method: 'post',
          url: SERVERPATH + '/api/app/next/borrow/order/wap/recommend/smallloan',
          params: {type: id},
          headers: {
            'Content-type': 'application/x-www-form-urlencoded'
          }
        }).then((res) => {
          this.pageData = res.data.resultModel
          if (loaded) {
            loaded('done')
          }
        }).catch((err) => {
          console.log(err)
          if (loaded) {
            loaded('fail')
          }
        })
      },
      refresh(loaded) {
        this.getSmallLoanData(this.pageNumId, loaded)
      },
      computedHeight(){
        let documentHeight = document.body.clientHeight
        let topHeight = this.$refs.top.clientHeight
        let height = documentHeight - topHeight
        document.getElementsByClassName('swiper-container')[0].style.height = height + 'px'
      },
      static(el){
        let id = el.currentTarget.getAttribute('data')
        let channel = this.GetQueryString('channel')
        let url = el.currentTarget.getAttribute('clickUrl')
        axios({
          method: 'post',
          url: SERVERPATH + '/api/app/next/common/participate/wap/loanstatistics',
          params: {
            clickId: id,
            from: channel
          },
          headers: {
            'Content-type': 'application/x-www-form-urlencoded'
          }
        }).then((res) => {
          window.location.href = url
        })
      },
      GetQueryString(name){
        var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)");
        var r = window.location.search.substr(1).match(reg);
        if (r != null)return unescape(r[2]);
        return null;
      }
    }
  }
</script>

<style lang="scss">
  .mainView {
    -webkit-tap-highlight-color: rgba(255, 255, 255, 0);
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    .banner {
      background-color: blue;
      height: 50px;
      line-height: 50px;
      text-align: center;
      font-size: 16px;
      color: #ffffff;
      background-image: linear-gradient(-90deg, #36d2ff 0%, #0076ff 100%);
    }
    .tips {
      height: 30px;
      line-height: 30px;
      padding-left: 15px;
      background-color: #ffedd2;
      font-size: 13px;
      color: #ff9600;
      letter-spacing: 0;
      background-image: url("./assets/close.png");
      background-repeat: no-repeat;
      background-size: 14px 14px;
      background-position: 96% center;
    }
    .swiper-container {

    }
    .swiper-pagination {
      position: relative;
      height: 44px;
      display: flex;
      flex-direction: row;
      justify-content: space-around;
      align-items: center;
      width: 100%;
      border-bottom: 5px solid #f5f5f5;
      .swiper-pagination-bullet {
        opacity: 1;
        flex: 1;
        height: 44px;
        line-height: 44px;
        background-color: #fff;
        font-size: 14px;
        color: #333333;
        border-radius: 0;
        box-sizing: border-box;
        &.swiper-pagination-bullet-active {
          color: #0095ff;
          border-bottom: 2px solid #0095ff;
        }
      }
    }
    .scroll-ul {
      padding: 0;
      margin: 0;
      list-style: none;
      padding-left: 15px;
      li {
        border-bottom: 1px solid #edf0f3;
        margin-top: 15px;
      }
      .smallLoanItem {
        display: flex;
        align-items: center;
        .leftLogo {
          width: 75px;
          align-self: baseline;
          img {
            display: inline-block;
            border-radius: 5px;
            width: 52px;
            height: 52px;
          }
        }
        .rightInfo {
          flex: 1;
          .infoTop {
            display: flex;
            border-bottom: 1px solid #edf0f3;
            .infoLeft {
              flex: 1;
              p {
                margin: 0;
              }
              .name {
                font-size: 15px;
                color: #333333;
                font-weight: bold;
                .hot {
                  position: relative;
                  top: -2px;
                  margin-left: 5px;
                  display: inline-block;
                  padding-left: 5px;
                  color: #fff;
                  font-size: 11px;
                  width: 27px;
                  height: 15px;
                  padding-top: 2px;
                  background: url("./assets/hot.png") no-repeat;
                  background-size: 27px 15px;
                }
              }
              .borrowAmount {
                margin-top: 5px;
              }
              .ratio {
                margin-top: 5px;
                margin-bottom: 7px;
              }
              .borrowAmount, .ratio {
                font-size: 12px;
                color: #666666;
                text-align: left;
              }
            }
            .infoRight {
              width: 105px;
              .borrowNumber {
                border-radius: 4px;
                height: 17px;
                line-height: 17px;
                text-align: center;
                width: 90px;
                border: 1px solid #0076ff;
                font-size: 11px;
                color: #0076ff;
              }
              .time {
                margin-top: 7px;
                font-size: 13px;
                color: #666666;
                span {
                  color: #FF4949;
                }
              }
            }
          }
          .infoBottom {
            height: 32px;
            line-height: 32px;
            font-size: 12px;
            color: #666666;
            width: 220px;
            overflow: hidden;
            text-overflow: ellipsis;
            display: -webkit-box;
            -webkit-line-clamp: 1;
            -webkit-box-orient: vertical;
            word-break: break-all;
            .need {
              margin-right: 5px;
              line-height: 14px;
              font-size: 12px;
              display: inline-block;
              text-align: center;
              height: 14px;
              width: 14px;
              color: #fff;
              background-color: #ff9600;
            }
          }
        }
      }
    }
  }
</style>




