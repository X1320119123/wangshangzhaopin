<template>
	<div class="app">
		<!-- 头 -->
		<div class="header">
			<div class="title">
				<i class="fa fa-tv">&emsp;网上招聘</i>
			</div>
			<div class="info">
				<div class="user">用户：{{user.username}}&emsp;<el-button size="mini" @click="logout">退出</el-button></div>
				<!-- <img class="photo" :src="user.userface">
				<div class="user">
					<el-dropdown>
						<span class="el-dropdown-link">
							{{user.nickname}}
							<i class="el-icon-arrow-down el-icon--right"></i>
						</span>
						<el-dropdown-menu slot="dropdown">
							<el-dropdown-item>退出</el-dropdown-item>
						</el-dropdown-menu>
					</el-dropdown>
				</div> -->
			</div>
		</div>
		<!-- 体 -->
		<div class="center">
		 	<!-- 左导航 -->
		 	<div class="left-nav">
		 		<ul>
		 			<li :class="{current:this.currentRoute=='/'}">
		 				<i class="fa fa-home left-fu"></i>
				 		<router-link to="/">招聘首页</router-link>
		 				<i class="fa fa-angle-right right-fu"></i>
		 			</li>
		 			<li :class="{current:this.currentRoute=='/category'}">
		 				<i class="fa fa-navicon left-fu"></i>
				 		<router-link to="/category">招聘管理</router-link>
		 				<i class="fa fa-angle-right right-fu"></i>
		 			</li>
		 			<li :class="{current:this.currentRoute=='/article'}">
		 				<i class="fa fa-file left-fu"></i>
				 		<router-link to="/article">招聘信息</router-link>
		 				<i class="fa fa-angle-right right-fu"></i>
		 			</li>
		 			<li :class="{current:this.currentRoute=='/user'}">
		 				<i class="fa fa-user left-fu"></i>
				 		<router-link to="/user">用户管理</router-link>
		 				<i class="fa fa-angle-right right-fu"></i>
		 			</li>
		 			<li :class="{current:currentRoute.indexOf('/setting')>=0}">
		 				<i class="fa fa-gear left-fu"></i>
		 				<router-link to="/setting/info">系统设置</router-link>
		 				<i class="fa fa-angle-right right-fu"></i>
		 			</li>
		 		</ul>
		 	</div>
		 	<!-- 右侧内容区 -->
		 	<div class="content">
		 		<div class="wrapper">
		 			<router-view></router-view>
		 		</div>
		 	</div>
		</div>
	</div>
</template>
<script>
	export default {
		data() {
			return{
				currentRoute:'/',
				user:{},
			}
		},
		watch:{
			'$route':function(to,from){
				this.currentRoute = to.path;
			}
		},
		created(){
			this.currentRoute = this.$route.path;
		    let user = JSON.parse(localStorage.getItem('user'));
		    if(user && user.username){
		    	this.user = user;
		    } else {
		    	window.vm.currentComponent = 'Login';
		    }
		},
		methods:{
			logout(){
				localStorage.removeItem('user');
		    	window.vm.currentComponent = 'Login';
			}
		}
	}
</script>
<style>
	html {
		font: normal normal 12px '微软雅黑','Microsoft YaHei';
		color: #666;
	}
	body, ul, ol, dl, p, h1, h2, h3{
		margin: 0;
		padding: 0;
	}
	ul, ol {
		list-style: none;
	}
	a {
		color: #666;
		text-decoration: none;
	}
	div {
		box-sizing: border-box;
	}
	.header {
		position: absolute;
		width: 100%;
		height: 60px;
		top: 0;
		background-color: teal;
		padding: 0 1em;
	}
	.header .title {
		line-height: 60px;
		color: #fff;
		font-size: 24px;
		float: left;
	}
	.header .info {
		float: right;
		margin-top: 1.5em;
	}
	.header .info .user {
		font-size: 18px;
		color: #fff;		
	}
	.header .photo {
		width: 50px;
		border-radius: 50%;
	}
	.center {
		position: absolute;
		top: 60px;
		bottom: 0;
		width: 100%;
	}
	.center > .left-nav {
		width: 180px;
		height: 100%;
		background-color: ;
		float: left;
	}
	.center > .left-nav > ul > li{
		position: relative;
		line-height: 2.4em;
		text-align: center;
		border-bottom: 1px solid #f0f0f0;
	}
	.center > .left-nav > ul > li.current {
		background-color: #f0f0f0;
	}
	.center > .left-nav > ul > li i.left-fu {
		position: absolute;
		left: 3em;
		top: 30%;
	}
	.center > .left-nav > ul > li i.right-fu {
		position: absolute;
		right: 2em;
		top: 30%;
	}
	.center > .content {
		margin-left: 180px;
		height: 100%;
		background-color: #f0f0f0;
		padding: 1em 1em 0 1em;
	}
	.center > .content > .wrapper {
		overflow: auto;
		width: 100%;
		height: 100%;
		background-color: #fff;
		border-radius: 5px;
		padding: .5em;
	}
</style>