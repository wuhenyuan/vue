<template>
 <div class="ratingselect">
    <div class="rating-type" border-1px>
	    <span class="block positive" @click="select(2,$event)" :class="{'active':sType === 2}">{{desc.all}}<span class="count">47</span> </span>
	    <span class="block positive" @click="select(0,$event)" :class="{'active':sType === 0}">{{desc.positive}}<span class="count">40</span></span>
	    <span class="block negative" @click="select(1,$event)" :class="{'active':sType === 1}">{{desc.negative}}<span class="count">47</span></span>
    </div>
    <div @click="toggleContent($event)"  class="switch" :class="{'on':oContent}">
        <span class="icon-check_circle"></span>
        <span class="text">只看有内容的评价</span>
    </div>
  </div>

</template>

<script type="text/ecmascript-6">
 const ALL  = 2;
  export default{
  	    props: {
           ratings: {
               type: Array,
               default() {
                   return [];
               }
           },
            selectType: { //全部，满意，不满意
                type: Number,
                default: ALL //默认情况时ALL,值等于2
            },
            onlyContent: { //只看有内容的评价还是所有的评价
                type: Boolean,
                default: false //设置为可以看到所有的评价
            },
            desc: { //描述
                type: Object,
                default() { //默认desc是这三种，在商品详情页的时候传入推荐或者吐槽
                    return {
                        all: '全部',
                        positive: '满意',
                        negative: '不满意'
                    };
                }
            }
        },
        data() {
            return {
                    sType: this.selectType,
                    oContent: this.onlyContent
                }
            },
        methods: {
            select (type, event) { //点击的时候外层是有一个BScroll的，所以要传递event阻止默认点击
                if (!event._constructed) { //浏览器直接return掉,去掉自带click事件的点击
                    return;
                }
                //将this.selectType设置成传入的参数,而不是food传过来的初始化的值，之后样式就可以随着点击改变了
                this.sType = type;
                //派发事件通知父组件food.vue selectType的改变,将type值传出去
                 console.log('ratingselect.vue ' + type);
                //this.$emit('se-type', type); 
                this.$emit('increment', 'selectType', this.sType); //selectType要跟food.vue中data定义的同名
              },
            toggleContent (event) {
                if (!event._constructed) { //浏览器直接return掉,去掉自带click事件的点击
                        return;
                }
                this.oContent = !this.oContent;
                console.log('ratingselect.vue ' + this.oContent);
               // this.$emit('toggle-content', this.oContent);
                this.$emit('increment', 'onlyContent', this.oContent);//onlyContent要跟food.vue中data定义的同名
            }
        },  

  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
</style>
