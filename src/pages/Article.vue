<template>
	<div class="article">
		<!-- 按钮区 -->
		<div class="btns">
			<el-select v-model="params.categoryId" size="mini" clearable placeholder="请选择">
			    <el-option v-for="c in categories" :label="c.name" :value="c.id" :key='c.id'></el-option>
			 </el-select>
			<el-input style="width: 120px" size="mini" placeholder="请输入关键字" v-model="params.keywords" clearable></el-input>
			<el-button size="mini" @click="toAddArticle()">添加</el-button>
			<el-button size="mini" @click="batchDeleteArticle">批量删除</el-button>
		</div>
		<!-- 按钮区结束 -->
		<!-- 数据区 -->
		<div class="article_table" v-loading="loading">
			<el-table size="small" :data="articles" :border="true" stripe style="width: 100%" @selection-change="handleSelectionChange"> 
				<el-table-column type="selection" width="35"></el-table-column>
				<el-table-column prop="title" label="招聘标题" width="180"></el-table-column> 
				<el-table-column prop="category.name" label="所属方向" width="180"></el-table-column>
				<el-table-column prop="liststyle" label="发布样式"></el-table-column>
				<el-table-column prop="author" label="作者"></el-table-column>
				<el-table-column prop="publishtime" label="发布时间"></el-table-column>
				<el-table-column prop="readtimes" label="阅读次数"></el-table-column>
				<el-table-column label="操作">
					<template slot-scope='{row}'>
						<i class="fa fa-trash" @click="deleteArticle(row.id)"></i>
						<i class="fa fa-pencil" @click="toAddArticle(row)"></i>
					</template>
				</el-table-column>
			</el-table>
		</div>
		<!-- 数据区结束 -->
		<!-- 分页 -->
		<div class="page">
			<el-pagination
			    layout="prev, pager, next"
			    :page-size="params.pageSize"
			    @current-change="hadleCurrentChange"
			    :total="total">
			</el-pagination>
		</div>
		<!-- 分页结束 -->
		<!-- 模态框区 -->
		<div class="model">
			<el-dialog :title="articleDialog.title" :visible.sync="articleDialog.visible" fullscreen>
				<!-- {{articleDialog.form.fileIds}} -->
			  <el-form :model="articleDialog.form" :rules="rules" ref="articleDialog.form">
			    <el-form-item style="display: inline-block;" prop="title" label="招聘标题" label-width="120px">
			      <el-input style="width: 360px;" v-model="articleDialog.form.title" autocomplete="off"></el-input>
			    </el-form-item>
			    <el-form-item style="display: inline-block;" label="所属方向" label-width="120px" prop="categoryId">
			      <el-select v-model="articleDialog.form.categoryId" placeholder="请选择">
			        <el-option v-for="c in categories" :label="c.name" :key="c.id" :value="c.id"></el-option>
			      </el-select>
			    </el-form-item>
			    <el-form-item prop="liststyle" label="发布样式" label-width="120px">
			    	<el-radio v-model="articleDialog.form.liststyle" label="style-one">
			    		<img src="../images/style1.jpg" style="width: 120px; height: 40px;">
			    	</el-radio>
				    <el-radio v-model="articleDialog.form.liststyle" label="style-two">
				    	<img src="../images/style2.jpg" style="width: 120px; height: 40px;">
				    </el-radio>
			    </el-form-item>
			    <el-form-item label="缩略图" label-width="120px"> 
					  <!-- action="http://120.78.164.247:8888/manager/file/upload" -->
			    	<el-upload
					  action="http://39.108.188.49:8099/manager/file/upload"
					  :on-success='handleUploadSuccess'
					  :on-remove='handleUploadRemove'
					  :file-list="articleDialog.fileList"
					  list-type="picture">
					  <el-button size="small" type="primary">点击上传</el-button>
					  <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
					</el-upload>
			    </el-form-item>
			    <el-form-item prop="content" label="文章正文" label-width="120px">
			      <mavon-editor ref=md v-model="articleDialog.form.content"/>
			    </el-form-item>
			  </el-form>
			  <div slot="footer" class="dialog-footer">
			    <el-button size="small" @click="closeArticleDialog('articleDialog.form')">取 消</el-button>
			    <el-button type="primary" size="small" @click="saveOrUpdateArticle('articleDialog.form')">确 定</el-button>
			  </div>
			</el-dialog>
		</div>
		<!-- 模态框区结束 -->
	</div>
</template>
<script>
	import axios from '@/http/axios'
export default {
	data(){
		return{
			loading:false,
			ids:[],
			categories:[],
			articles:[],
			total:0,
			params:{
				page:0,
				pageSize:10,
			},
			articleDialog:{
				title:'',
				radio:1,
				form:{},
				visible:false,
				fileList:[],
			},
			rules:{
				title:[
					{ required: true, message: '请输入招聘标题', trigger: 'blur' },
				],
				content:[
					{ required: true, message: '请输入招聘正文信息', trigger: 'blur' },
				],
				categoryId:[
					{ required: true, message: '请选择父级方向', trigger: 'blur' },
				]
			}
		}
	},
	watch:{
		// 只要params中任意参数改变,就要刷新页面
		params:{
			handler:function(){
				this.findAllArticles();
			},
			deep:true
		}
	},
	created(){
		this.findAllArticles();
		this.findAllCategories();
	},
	methods:{
		//删除附件
		handleUploadRemove(file){
			// 调用接口删除图片
			axios.get('/manager/file/delete',{
				params:{
					id:file.name,
				}
			}).then(()=>{
				// 从fileIds中移除
				_.remove(this.articleDialog.form.fileIds,(id)=>{
					return id == file.name;
				});
				this.articleDialog.form.fileIds.push(1);
				this.articleDialog.form.fileIds.pop();
				this.$notify({title:'成功',message:'操作成功'});
			}).catch(()=>{
				this.$notify({title:'错误',message:'网络异常'});
			});
		},
		handleSelectionChange(val) {
			this.ids = [];
			val.forEach((item)=>{
				this.ids.push(item.id);
			});
	    },
		batchDeleteArticle(){
			this.$confirm('此操作将永久删除信息, 是否继续?', '提示', {
	            confirmButtonText: '确定',
	            cancelButtonText: '取消',
	            type: 'warning'
	        }).then(() => {
				let url = '/manager/article/batchDeleteArticle';
	        	axios.post(url,{ids:this.ids}).then(({data:result})=>{
	        		if(result.status == 200){
            			this.$notify({
	            			title:'成功',
		            		message: result.message,
		            		type: 'success',
		            	});
            		} else {
            			this.$notify({
	            			title: '发生错误',
	            			message: '后台数据异常',
		            	});
            		}
	            	this.findAllArticles();
	            });
	        }).catch(() => {
	            this.$message({
	                type: 'info',
	                message: '已取消删除'
	            });          
	        });
		},
		closeArticleDialog(formName){
			this.articleDialog.visible = false;
			if(formName){
				this.$refs[formName].resetFields();
			}
		},
		saveOrUpdateArticle(formName){
			this.$refs[formName].validate((valid) => {
	            if (valid) {
	            this.articleDialog.form.source = this.$refs.md.d_render;
				let url = '/manager/article/saveOrUpdateArticle';
					axios.post(url,this.articleDialog.form)
					.then(({data:result})=>{
						if(result.status == 200){
							this.closeArticleDialog();
							this.$notify({ 
								title: '成功',
					            message: result.message,
					            type: 'success'
					        });
							this.findAllArticles();
						} else {
							this.$notify({
					            title: '错误',
					            message: result.message
					        });
						}
					})
					.catch((error)=>{
						this.$notify({
							type:'error',
				            title: '错误',
				            message: '网络异常',
				        });
			            console.log(error);
					});
				}else {
					return false;
				}
			});
		},
		handleUploadSuccess(response,file,fileList){
			file.name = response.data.id;
			this.articleDialog.form.fileIds.push(response.data.id);
		},
		toAddArticle(row){
			if(row){
				// 处理表单数据
				let c = _.cloneDeep(row);
				// 显示默认缩略图
				this.articleDialog.fileList = c.articleFileVMs.map((item)=>{
					return {
						name: item.cmsFile.id,
						url: 'http://39.108.81.60:8888/group1/'+item.cmsFile.id,
					}
				});
				//处理表单数据
				// 依赖栏目   category -> categoryId
				c.categoryId = c.category.id;
				delete c.category;
				// 依赖文件  articleFileVMs -> fileIds
				c.fileIds = c.articleFileVMs.map(item=>item.cmsFile.id);
				delete c.articleFileVMs;
				// 取消默认空值
				for(let key in c){
					let val = c[key];
					if(!val){
						delete c[key];
					}
				}
				this.articleDialog.form = c;
				this.articleDialog.title = '修改招聘信息';
			} else {
				this.articleDialog.form = {
					liststyle:'style-one',
					fileIds:[],
					categoryId: this.params.categoryId,
				};
				this.articleDialog.fileList = [];
				this.articleDialog.title = '添加招聘信息';
			}
			this.articleDialog.visible = true;
		},
		hadleCurrentChange(page){
			this.params.page = page-1;
		},
		deleteArticle(id){
			this.$confirm('此操作将永久删除该信息, 是否继续?', '提示', {
	            confirmButtonText: '确定',
	            cancelButtonText: '取消',
	            type: 'warning'
	        }).then(() => {
	            let url = '/manager/article/deleteArticleById?id='+id;
	            axios.get(url).then(({data:result})=>{
            		if(result.status == 200){
            			this.$notify({
	            			title:'成功',
		            		message: result.message,
		            		type: 'success',
		            	});
            		} else {
            			this.$notify({
	            			title: '错误',
	            			message: '无法删除',
		            	});
            			console.log(result.message);
            		}
	            	this.findAllArticles();
	            });
	        }).catch(() => {
	            this.$message({
	                type: 'info',
	                message: '已取消删除'
	            });          
	        });
		},
		findAllArticles(){
			this.loading = true;
			let url = '/manager/article/findArticle';
			axios.get(url,{params:this.params})
			.then(({data:result})=>{
				if(result.status == 200){
					this.total = result.data.total;
					this.articles = result.data.list;
				} else {
					alert('后台数据异常');
				}
			}).catch((error)=>{
				this.$notify({
					title:'错误',
					message:'网络异常',
				});
			}).finally(()=>{
				this.loading = false;
			});
		},
		findAllCategories(){
			let url = '/manager/category/findAllCategory';
			axios.get(url).then(({data:result})=>{
				this.categories = result.data;
			}).catch((error)=>{
				this.$notify({
					title:'错误',
					message:'网络异常',
					type:'error',
				});
			});
		}
	}
}
</script>
<style scoped>
i.fa {
	margin: 0 .3em;
	cursor: pointer;
}
i.fa.fa-trash {
	color: #f56c6c;
}
.table {
	padding: 1em 0 0 0;
}
.page {
	position: absolute;
	bottom: 15%;
	right: 1%;
}
</style>