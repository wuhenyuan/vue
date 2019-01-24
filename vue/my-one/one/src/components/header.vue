<template>
      <div>
           <header class="header clearfix">
           <div v-show="!searchShow" style = "display:flex;width:100%;">
           	     <i class="menu"></i>
           	     <h3 class="one-title">one</h3>
                 <i class="search" @click="searchFn()"></i>
           </div>
           <div v-show="searchWhow" class="search-input">
                <input type="text" v-model="searchString" placeholder="请输入关键字">
                <i class="cancle" @click="cancleFn()"></i>
                <i class="search-act" @click="sendSearch()">
                </i>           	
           </div>
           </header>
           <div v-show="searchShow" class="search-result">
		
	       </div>
      </div>
</template>

<script type="text/javascript">
	export default{
		return {
			searchShow:false,
			searchString:""
		},
		methods:{
			searchFn:function(){
				this.searchShow = !this.searchShow;
			},
			cancleFn:function(){
				this.searchShow = !this.searchShow;
			}
			sendSearch:function(){
				var searString = this.searString;
				this.$http.get("search?searchString="+searString).then(function(){

				})
			}
		}
	}
</script>
<style type="text/css">
	.search-input{
		width:100%;
		padding:0 .3rem;
	}

	.search-input input[type=text]{
		height: .3rem;
		width: 4.5rem;
        padding:1rem,.2rem;
        font-size: .24rem;
        border:none;
        outline: none;
	}
	.search-act,.cancle{
		float:right;
	}
	.search-result{
		height: calc(100vh - .8rem);
		width:100%;
		position: :absolute;
		z-index:5;
		top:.8rem;
		background-color: rgba(0,0,0,.3);
	}
</style>