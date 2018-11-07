<template>
	<div class="setting">
		<ul class="setting-nav">
			<li :class="{current:this.currentRoute=='/setting/info'}"><router-link to="/setting/info">基础设置</router-link></li>
			<li :class="{current:this.currentRoute=='/setting/auth'}"><router-link to="/setting/auth">权限设置</router-link></li>
			<li :class="{current:this.currentRoute=='/setting/adv'}"><router-link to="/setting/adv">其他设置</router-link></li>
		</ul>
		<div class="setting-content">
			<router-view></router-view>
		</div>
	</div>
</template>
<script>
	export default {
		data(){
			return {
				currentRoute:'/setting/info',
			}
		},
		watch:{
			'$route':function(to,from){
				this.currentRoute = to.path;
			}
		},
		mounted(){
			this.$router.push(this.currentRoute);
		}
	}
</script>
<style>
	.setting-nav {
		overflow: hidden;
	}
	.setting-nav > li {
		float: left;
		width: 120px;
		text-align: center;
	}
	.setting-nav > li.current {
		background-color: #ccc;
	}
	.setting-content {
		padding: .5em 0;
	}
</style>