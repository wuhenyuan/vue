<template>
  <transition name="move">
  	<div v-show="showFlag" class="food" ref="food">
       <div class="food-content ">
       	 <div class="image-header">
       	 	<img :src="food.image">
       	 	<div class="back" @click="hide">
       	 	<i class="icon-arrow_lift"></i>
       	 	</div>
       	 </div>
       </div>
       <div class="content">
       	 <h1 class="title">{{food.name}}</h1>
       	 <div class="detial">
       	 	<span class="sell-count">月售{{food.sellCount}}份</span>
       	 	<span class="rating">好评率{{food.rating}}</span>
       	 </div>
       	 <div class="price">
       	 	<span class="now">${{food.price}}</span>
       	 	<span class="oldPrice" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
       	 </div>
       	 <div class="cartcontrol-wrapper"> <!-- 加减号组件cartcontrol -->
            <cartcontrol :food="food"></cartcontrol>
         </div>
         <transition name="fade"> 
            <div @click.stop.prevent="addFirst" class="buy" v-show="food.count === 0 || !food.count">
                 加入购物车    
             </div> <!-- 这两种情况下加入购物车都会显示,否则就会隐藏-->
         </transition>
       </div>
       <split v-show="food.info"></split>
        <div class="info" v-show="food.info"> <!-- 并不是所有的商品都有info的 -->
            <h1 class="title">商品信息</h1>
            <p class="text">{{food.info}}</p>
        </div>
        <split></split>
        <ratingselect @increment="incrementTotal" :select-type="selectType" :only-content="onlyContent" :desc="desc" :rating="food.ratings"></ratingselect>
        <div class="rating-wrapper"> <!-- 评价列表-->
            <ul v-show="food.ratings && food.ratings.length">
                <li v-show="needShow(rating.rateType, rating.text)" v-for="rating in food.ratings" :key="rating.id" class="rating-item border-1px">
                    <div class="user">
                        <span class="name">{{rating.username}}</span>
                        <img class="avatar" height="12" width="12"  :src="rating.avatar" alt="">
                    </div>
                    <div class="time">{{rating.rateTime }}</div>
                    <p class="text">
                        <span :class="{'icon-thumb_up':rating.rateType === 0, 'icon-thumb_down':rating.rateType === 1}"></span>{{rating.text}}
                    </p>
                </li>
            </ul>  
            <div class="no-ratings" v-show="!food.ratings || !food.ratings.length">暂无评价</div>
        </div>

    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
  import BScroll from "better-scroll"
  import cartcontrol from "../cartcontrol/cartcontrol.vue"
  import split from "../split/split.vue"
  import ratingselect from"../ratingselect/ratingselect.vue"
  import Vue from "vue"

 const ALL = 2;

   export default{
      props: {
      	food:{
      		type: Object
      	}
      },
      data() {
      	return {
      		showFlag:false,
      		selectType: ALL,
      		onlyContent: true,
      		desc:{
      			all:"全部",
      			positive: "推荐",
      			negative: "吐槽"
      		}
      	};
      },
      methods: {
      	show(){
      		this.showFlag=true;
      		this.$nextTick(() => {
      			if(!this.scroll){
      				this.scroll = new BScroll(this.$refs.food,{
      					click: true
      				});	
      			}else{
      				this.scroll.refresh();
      			}

      		});
      	},
      	addFirst(event){
            if(!event._constructed) return;
      		this.$emit('cart-add',event.target);
      		Vue.set(this.food,'count',1);
      	},
      	hide(){
      		this.showFlag=false;
      	},
  	    incrementTotal(type, data) {
          this[type] = data;
          this.$nextTick(() => {
          this.scroll.refresh();
        });
        },
        needShow(type, text) {
             // console.log('this.selectType: ' + this.selectType + '  type: ' + type + ' out  ' + text);
            if (this.onlyContent && !text) {
                return false;
            }
            if (this.selectType === ALL) {
                return true;
            } else {
                //console.log('this.selectType: ' + this.selectType + 'type: ' + type + ' in ' + text);
                return type === this.selectType;
            }
        }
      },
      components:{
      	cartcontrol,
      	split,
      	ratingselect
      }
   };
</script>

<style lang="stylus" rel="stylesheet/stylus">
    .food
    	position: fixed
    	left: 0
    	top: 0
    	bottom: 48px
    	z-index: 30
    	width: 100%
    	background: #fff
    	transform: translate3d(0,0,0)
    &.move-enter-active,&.move-leave-active
    	transition: all 0.2s linear
    	transform: translate3d(0,0,0)
    &.move-enter,&.move-leave-active
    	transform: translate3d(100%,0,0)

	.image-header
    	position:relative
    	width: 100%
    	height: 0
    	padding-top: 100%
    	img
    	    position: absolute
    	    top: 0
    	    left: 0
    	    width: 100%
    	    height: 100% 
    	.back
    	    position: absolute
    	    top: 10px
    	    left: 0
    	    .icon-arrow_lift
    	        display: block
    	        padding: 10px
    	        font-size: 20px
    	        color: #fff

    .content
        padding: 18px
        position: relative
        .title
            line-height: 14px
            margin-bottom: 8px
            font-size: 14px
            color: rgb(7,17,27)
            font-weight: 700
        .detial
            margin-bottom: 18px
            height: 10px
            line-height: 10px
            font-size: 0
            .sell-count,.rating
                font-size: 10px
                color:rgb(147,153,159)
            .sell-count
                margin-right: 12px
        .price
            height: 24px
            line-height:24px
            font-weight: 700
            .now
                margin-right: 8px
                font-size: 14px
                color: rgb(240,20,20)
            .oldPrice
                font-size: 10px
                font-weight: 700
                color: rgb(147,153,159)
                line-height: 24px
                text-decoration:line-through
        .cartcontrol-wrapper
            position: absolute
            right: 12px
            bottom :12px
        .buy
            position: absolute
            right: 18px
            bottom: 18px
            z-index: 10
            height: 24px
            line-height: 24px
            padding: 0 12px
            border-radius: 12px
            font-size: 10px
            color: #fff
            background: rgb(0,160,220)
            &.fade-enter-active,&.fade-leave-active
                transition: all 0.2s
                opacity: 1
            &.fade-enter,&fade-leave-active
                opacity: 0

    .info
    padding 18px
    .title
        line-height 14px
        margin-bottom 16px
        font-size 14px
        color rgb(7,17,27)
    .text
        line-height 24px
        padding 0 8px
        font-size 12px
        color rgb(77,85,93)

</style>
