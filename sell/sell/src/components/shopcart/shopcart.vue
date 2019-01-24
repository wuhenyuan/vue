<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList"> 
          <div class="content-left">
               <div class="logo-wrapper" >
                    <div class="logo" :class="{'hightlight':totalCount>0}">
                      <i class="icon-shopping_cart" :class="{'hightlight':totalCount>0}"></i>
                    </div>
                    <div class="num">
                      {{totalCount}}
                    </div>
               </div>
               <div class="price" :class="{'hightlight':totalPrice>0}">
                {{totalPrice}}元
               </div>
               <div class="description">
                另需配送费￥{{deliveryPrice}}元
               </div>
          </div>        
          <div class="content-right" @click.stop.prevent="pay">
             <div class="pay" :class="payClass ">
                {{paydesc}}
             </div>
          </div>
      </div>
      <div class="ball-container">
          <div v-for="ball in balls" :key="ball.id">
              <transition name="drop" @before-enter="beforeEnter" @enter="enter" @after-enter="afterEnter">
                <div v-show="ball.show" class="ball">
                  <div class="inner inner-hook">
                  </div>
                </div>
              </transition>
          </div> 
      </div>
      <transition name="fade">
          <div class="shopcart-list" v-show="listShow">
            <div class="list-header">
              <h1 class="title">购物车</h1>
              <span class="empty" @click="empty">
                清空
              </span>
            </div>
            <div class="list-content" ref="listContent">
              <ul>
                <li class="food" v-for="food in selectFoods" :key="food.id">
                  <span class="name">{{food.name}}</span>
                  <div class="price">
                    <span> ￥{{food.price * food.count}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <cartcontrol :food="food"></cartcontrol>
                  </div>
                </li>
              </ul>
            </div>
          </div>
      </transition>
  </div>
  <transition name="fade">
    <div class="list-mask" v-show="listShow" @click="hiddList()"></div>
  </transition>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from "better-scroll"
  import cartcontrol from "../cartcontrol/cartcontrol.vue"
   export default{
   	props: {
   		selectFoods:{
            type: Array,
            default() {
            	return [
            	{
            		price: 10,
            		count: 1 
            	}       	
            	];
            }
   		},
   		deliveryPrice:{
   			type: Number,
   			default: 0
   		},
   		minPrice:{
   			type: Number,
   			default: 0
   		}
   	},
   	data() {
   			return {
               balls: [{
        show: false
      }, {
        show: false
      }, {
        show: false
      }, {
        show: false
      }, {
        show: false
      }], 
          dropBalls:[],
          fold:true
   			};
   		},
   	computed:{
   		totalPrice() {
   			let total = 0;
   			this.selectFoods.forEach((food)=>{
   				total += food.price * food.count;
   			});
   			return total;
   		},
   		totalCount() {
   			let count = 0;
   			this.selectFoods.forEach((food)=>{
   				count  += food.count;
   			});
   			return count;
   		},
   		paydesc() {
   			if(this.totalPrice === 0){
   				return `￥${this.minPrice}元起送`;
   			}else if(this.totalPrice<this.minPrice){
                let diff  = this.minPrice  - this.totalPrice;
   				return `￥还差${diff}元起送`;
   			}else{
   				return '去结算';
   			}
   		},
   		payClass(){
   			if(this.totalPrice<this.minPrice){
   				return "not-enough"; 
   			}else{
   				return  "enough";
   			}
   		},
      listShow(){
        if(!this.totalCount){
          this.fold = true;
          return false; 
        }
          let show  = !this.fold;
          if(show){
            this.$nextTick(()=>{
              if(!this.scroll){
                this.scroll = new BScroll(this.$refs.listContent,{
                  click:true
                });
              }else{
                this.scroll.refresh();
              }
            });
          }
          return show;
      }
   	},
    methods: {
            drop(el) {
                //遍历balls，拿到第一个show为false的球，做一个动画
                console.log(el);
                for (let i = 0; i < this.balls.length; i++) {
                    let ball = this.balls[i];
                    if (!ball.show) { //show为false的球
                        ball.show = true; //小球下落
                        ball.el = el;//保留当前的DOM对象，用来计算位置
                        this.dropBalls.push(ball); //dropBalls存的是已经下落的小球,后续要对已经下落的小球进行处理
                        return;
                    }
                }
            },
       //定义三个钩子函数实现动画
            beforeEnter(el) { //el为当前执行transition动画的DOM对象
            //先找到所有为true的小球（连续点击的情况）
                console.log(el);
                let count = this.balls.length;
                while (count--) {
                    let ball = this.balls[count];
                    if (ball.show) { //这个是要运动的小球true
                        let rect = ball.el.getBoundingClientRect();//获得元素相当于视口的位置
                        let x = rect.left - 32;
                        let y = -(window.innerHeight - rect.top - 22);
                        el.style.display = ''; //v-show默认display：none，设置为空，让它显示
                        //外层元素是纵向的动画，内层元素是横向的动画
                        el.style.webkitTransform = `translate3d(0,${y}px,0)`;
                        el.style.transform = `translate3d(0,${y}px,0)`;
                        let inner = el.getElementsByClassName('inner-hook')[0];
                        inner.style.webkitTransform = `translate3d(${x}px, 0, 0)`;
                        inner.style.transform = `translate3d(${x}px, 0, 0)`;
                    }
                }
            },           
            enter(el) {
                /* 触发浏览器重绘，重绘之后才可以设置transform*/
                /* eslint-disable no-unused-vars */
                console.log(el);
                let rf = el.offsetHeight;
                this.$nextTick(() => { //样式重置
                    el.style.webKitTransform = 'translate3d(0,0,0)';//没有变量时只能用单引，不能用反引
                    el.style.transform = 'translate3d(0,0,0)';
                    let inner = el.getElementsByClassName('inner-hook')[0];
                    inner.style.webkitTransform = 'translate3d(0,0,0)';
                    inner.style.transform = 'translate3d(0,0,0)';
                });
            },
            afterEnter(el) { //动画完成
              console.log(el);
                let ball = this.dropBalls.shift();//删除并返回第一个ball
                if (ball) {
                    ball.show = false; //重置ball.show的状态
                    el.style.display = 'none';
                }
            },
           toggleList(){
            if(!this.totalCount){
              return ;
            }
            this.fold = !this.fold;
           },
           empty(){
            this.selectFoods.forEach((food) => {
              food.count = 0;
            });
           },
           hiddList(){
            this.fold = true;
           },
           pay(){
            if(this.totalPrice < this.minPrice){
              return ;
            }
            window.alert(`支付${this.totalPrice}元`);
           }
    },
    components:{
      cartcontrol
    }
   }
</script>

<style lang="stylus" rel="stylesheet/stylus">
    .shopcart
        position: fixed
        left: 0
        bottom: 0
        z-index: 50
        width: 100%
        height: 48px
        .content
            display: flex
            background: #141d27
            .content-left
                flex: 1
                .logo-wrapper
                    display: inline-block
                    vertical-align: top
                    position: relative
                    top: -10px
                    margin: 0 12px 
                    padding: 6px
                    width: 56px
                    height: 56px
                    box-sizing: border-box                    
                    border-radius: 50%
                    background:  #141d27
                    .logo
                        width: 100%
                        height: 100%
                        border-radius: 50%
                        background: #2b343c
                        text-align: center
                        &.hightlight
                            background:rgb(0,160,220)
                        .icon-shopping_cart
                            line-height: 44px
                            font-size: 24px
                            color: #80858a
                            &.heightlight
                                color: #fff
                    .num
                        position: absolute
                        top: 0
                        right: 0
                        width: 24px
                        height: 16px 
                        line-height: 16px
                        text-align: center
                        border-radius: 16px
                        font-size: 9px
                        font-weight: 700
                        color: #fff
                        background: rgb(240,20,20)
                        box-shadow: 0 4px 8px 0 rgba(0,0,0,0.4)
                .price
                    display: inline-block
                    vertical-align: top
                    margin-top: 12px
                    line-height: 24px
                    box-sizing: border-box
                    padding-right: 12px
                    border-right: 1px solid rgba(255,255,255,0.1)
                    font-size: 16px
                    font-weight: 700
                    color: rgba(255,255,255,0.4)
                    &.hightlight
                        color: #fff
                .description
                    display: inline-block
                    vertical-align: top
                    line-height: 24px
                    margin: 12px 0 0 12px 
                    color: rgba(255,255,255,0.4)
                    font-size: 10px
            .content-right
                flex: 0 0 105px
                width: 105px
                .pay
                    height: 48px
                    line-height: 48px
                    text-align: center
                    font-size: 12px
                    color: rgba(255,255,255,0.4)
                    background:#2b333c
                    &.not-enough
                        background: #2b333c
                    &.enough
                        background: #00b43c
                        color: #fff
        .ball-container
            .ball
                position fixed //相对于视口做布局
                left 32px
                bottom 22px
                z-index 200
                transition: all 0.6s cubic-bezier(0.49, -0.29, 0.75, 0.41)
                .inner
                    width 16px
                    height 16px
                    border-radius 50%
                    background rgb(0,160,220)
                    transition all 0.4s linear  
         .shopcart-list
             position: absolute
             top: 0
             left: 0
             z-index: -1
             width: 100%
             transform: translate3d(0,-100%,0)
             &.fade-enter.active,&.fade.leave.active
                 transition: all 0.5s linear
                 transform: translate3d(0,-100%,0)
             &.fade-enter,&.fade-leave-active
                 transform: translate3d(0,0,0)
             .list-header
                 height: 40px
                 line-height: 40px
                 padding:0 18px
                 background: #f3f5f7
                 border-bottom: 1px solid rgba(7,17,27,0.1)
                 .title
                     float: left
                     font-size: 14px
                     color:rgb(7,17,27)
                 .empty
                     float: right
                     font-size: 12px
                     color: rgb(0,160,220)
             .list-content
                 padding: 0 18px
                 max-height: 217px
                 overflow: hidden
                 background:#ffffff
                 .food
                     position: relative
                     padding: 12px 0
                     box-sizing: border-box
                     border-bottom:1px solid rgba(7,17,27,0.1)
                     .name
                         line-height: 24px
                         font-size: 14px
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
                         right:0
                         bottom: 6px
    .list-mask
      position fixed
      top 0
      bottom 0
      left 0
      right 0
      background rgba(7,17,27,0.6)
      backdrop-filter blur(10px)
      z-index 40
      &.fade-enter-active,&.fade-leave-active
        transition opacity 0.5s
      &.fade-enter,&.fade-leave-active
        opacity 0
</style>
