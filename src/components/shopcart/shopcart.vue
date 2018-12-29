<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
        <!--购物车内容-->
        <div class="content-left">
          <!--购物车图片-->
          <div class="logo-wrapper">
            <div class="logo" :class="{'highlight':totalCount>0}">
              <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
            </div>
            <div class="num" v-show="totalCount>0">{{totalCount}}</div>
          </div>
          <!--购买总额-->
          <div class="price" :class="{'highlight':totalPrice>0}">
            ¥{{totalPrice}}
          </div>
          <!--描述-->
          <div class="desc">另需配送费¥{{deliveryPrice}}元</div>
        </div>
        <!--配送条件-->
        <div class="content-right" @click.stop.prevent="pay">
          <div class="pay" :class="payClass">
            {{payDesc}}
          </div>
        </div>
      </div>
      <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click="setEmpty">清空</span>
          </div>
          <div class="list-content" ref="list-content">
            <ul>
              <li class="food" v-for="(food,key) in selectFoods" :key="key">
                <span class="name">{{food.name}}</span>
                <div class="price">
                  <span>¥{{food.price*food.count}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <v-cartcontrol :food="food"></v-cartcontrol>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <transition name="fade">
      <div class="list-mask" @click="hideList" v-show="listShow"></div>
    </transition>

  </div>
</template>

<script type="text/ecmascript-6">
  import cartcontrol from '../cartcontrol/cartcontrol';
  import BSrcoll from 'better-scroll';
  export default {
    props: {
      selectFoods: {
        type: Array,
        default() {
          return [];
        }
      },
      deliveryPrice: {
        type: Number,
        default: 0
      },
      minPrice: {
        type: Number,
        default: 0
      }
    },
    methods: {
      toggleList() { // 设置点击可以列表联动
        if (!this.totalCount) {
          return;
        }
        this.fold = !this.fold;
        if (!this.fold) {
          this.$nextTick(function () {
            if (!this.listContent) {
              this.listContent = new BSrcoll(this.$refs['list-content'], {
                click: true
              });
            } else {
              this.listContent.refresh();
            }
          });
        }
      },
      hideList() {
        this.fold = true;
      },
      setEmpty() {
        this.selectFoods.forEach((food) => {
          food.count = 0;
        });
        this.fold = true;
      },
      pay() {
        if (this.totalPrice < this.minPrice) {
          return;
        }
        window.alert(`支付${this.totalPrice}元`);
      }
    },
    data() {
      return {
        fold: true
      };
    },
    computed: {
      totalPrice() {
        let total = 0;
        this.selectFoods.forEach((food) => {
          total += food.price * food.count;
        });
        return total;
      },
      totalCount() {
        let count = 0;
        this.selectFoods.forEach((food) => {
          count += food.count;
        });
        return count;
      },
      payDesc() {
        if (this.totalPrice === 0) {
          return `¥${this.minPrice}元起送`;
        } else if (this.totalPrice < this.minPrice) {
          let diff = this.minPrice - this.totalPrice;
          return `还差¥${diff}元起送`;
        } else {
          return '去结算';
        }
      },
      payClass() {
        if (this.totalPrice < this.minPrice) {
          return 'not-enough';
        } else {
          return 'enough';
        }
      },
      listShow() { // 设置列表的显示
        if (!this.totalCount) {
          return false;
        }
        if (!this.fold) { // this.fold为false时显示
          return true;
        }
        return false;
      }
    },
    watch: {
      selectFoods(newFoods, oldFoods) {
        if (newFoods.length === 0) {
          this.fold = true;
        }
      }
    },
    components: {
      'v-cartcontrol': cartcontrol
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixinn.styl";
  .shopcart
    position: fixed
    left:0
    bottom: 0
    z-index: 50
    height:48px
    width:100%
    .content
      display: flex
      vertical-align: top
      background: #141d27
      color: rgba(255,255,255,0.4)
      font-size:0
      .content-left
        flex:1
        .logo-wrapper
          display: inline-block
          position: relative
          top: -10px
          margin:0 12px
          padding:6px
          width:56px
          height:56px
          box-sizing: border-box
          vertical-align: top
          border-radius: 50%
          background:#141d27
          .logo
            width: 100%
            height: 100%
            border-radius: 50%
            text-align: center
            background: #2b343c
            &.highlight
              background: rgb(0,160,220)
            .icon-shopping_cart
              line-height: 44px
              font-size: 24px
              color: #80858a
              &.highlight
                color: #fff
          .num
            position: absolute
            top:0
            right:0px
            width:24px
            height:16px
            line-height:16px
            text-align: center
            border-radius:16px
            font-size:9px
            font-weight:700
            color: #fff
            background: rgb(240, 20, 20)
            box-shadow : 0 4px 8px 0 rgba(0, 0, 0, 0.4)
        .price
          display: inline-block
          vertical-align: top
          margin-top: 12px
          line-height:24px
          padding-right:12px
          box-sizing:border-box
          border-right:1px solid rgba(255,255,255,0.1)
          font-size:16px
          font-weight:700
          &.highlight
            color:#fff
        .desc
          display: inline-block
          vertical-align: top
          margin:12px 0 0 12px
          line-height:24px
          font-size:10px
      .content-right
        flex:0 0 95px
        width:95px
        .pay
          height:48px
          line-height:48px
          text-align:center
          font-size:12px
          font-weight:700
          background:#2b333b
          &.not-enough
            background:#2b333b
          &.enough
            background:#00b43c
            color: #fff
    .shopcart-list
      position : absolute
      left:0
      top:0
      z-index: -1
      width: 100%
      transition: all 0.5s
      transform: translate3d(0,-100%,0)
      &.fold-enter,&.fold-leave-active
        transform: translate3d(0,0,0)
      .list-header
        height: 40px
        line-height: 40px
        padding: 0 18px
        border-bottom: 1px solid rgba(7,17,27,0.1)
        background: #f3f5f7
        .title
          float: left
          font-size: 14px
          color: rgb(7,17,27)
        .empty
          float: right
          font-size: 12px
          color: rgb(0,160,220)

      .list-content
        padding:0 18px
        max-height :217px
        overflow : hidden
        background : #fff
        .food
          position: relative
          padding :12px 0
          box-sizing :border-box
          border-1px(rgba(7,17,27,0.1))
          .name
            line-height: 24px
            font-size: 12px
            color: rgb(7,17,27)
          .price
            position: absolute
            right: 90px
            bottom: 12px
            line-height: 24px
            font-size: 14px
            font-weight: 700
            color: rgb(240,20,20)
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 6px
  .list-mask
    position: fixed
    top:0
    left:0
    width: 100%
    height: 100%
    z-index:40px
    backdrop-filter : blur(10px)
    background: rgba(7,17,27,0.6)
    transition:all 0.5s
    &.fade-transition
      opacity :1
      background: rgba(7,17,27,0.6)
    &.fade-enter, &.fade-leave-to
      opacity: 0
      background: rgba(7, 17, 27, 0)
</style>
