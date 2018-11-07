<template>
	<div class="category">
		<!-- 按钮区 -->
		<div class="btns">
			<el-button size="small" @click="toAddCategory">添加</el-button>
			<el-button size="small" @click="batchDeleteCategory">批量删除</el-button>
		</div>
		<!-- 按钮区结束 -->
		<!-- 表格区 -->
		<div class="table" v-loading="loading">
			<el-table size="small" :data="categories" :border="true" stripe style="width: 100%" @selection-change="handleSelectionChange"> 
				<el-table-column type="selection" width="55"></el-table-column>
				<el-table-column prop="name" label="招聘方向" width="180"></el-table-column> 
				<el-table-column prop="parent.name" label="所属方向" width="180"></el-table-column>
				<el-table-column prop="comment" label="描述"></el-table-column>
				<el-table-column label="操作">
					<template slot-scope='{row}'>
						<i class="fa fa-trash" @click="deleteCategory(row.id)"></i>
						<i class="fa fa-pencil" @click="toAddCategory(row)"></i>
					</template>
				</el-table-column>
			</el-table>
		</div>
		<!-- 表格区结束 -->
		<!-- 模态框 -->
		<div class="model">
			<el-dialog :title="categoryDialog.title" :visible.sync="categoryDialog.visible">
			  <el-form :model="categoryDialog.form" :rules="rules" ref="categoryDialog.form">
			    <el-form-item prop="name" label="招聘方向" label-width="120px">
			      <el-input v-model="categoryDialog.form.name" autocomplete="off"></el-input>
			    </el-form-item>
			    <el-form-item label="所属方向" label-width="120px">
			      <el-select v-model="categoryDialog.form.parentId" clearable placeholder="一级">
			        <el-option v-for="c in categories" :label="c.name" :key="c.id" :value="c.id"></el-option>
			      </el-select>
			    </el-form-item>
			    <el-form-item prop="comment" label="描述信息" label-width="120px">
			      <el-input type="textarea" :rows="2" placeholder="请输入内容" v-model="categoryDialog.form.comment"></el-input>
			    </el-form-item>
			  </el-form>
			  <div slot="footer" class="dialog-footer">
			    <el-button size="small" @click="closeCategoryDialog('categoryDialog.form')">取 消</el-button>
			    <el-button type="primary" size="small" @click="saveOrUpdateCategory('categoryDialog.form')">确 定</el-button>
			  </div>
			</el-dialog>
		</div>
		<!-- 模态框结束 -->
	</div>
</template>
<script>
	// @为src目录
	import axios from '@/http/axios'
	export default {
		data(){
			return {
				categories:[],
				loading:false,
				ids:[],
				categoryDialog:{
					title:'',
					form:{},
					// comment:'',
					visible:false,
				},
				rules: {
		            name: [
			            { required: true, message: '请输入招聘方向', trigger: 'blur' },
		            ],
		            comment: [
		                { required: true, message: '请输入描述信息', trigger: 'blur' }
		            ],
		        },
			}
		},
		created(){
			// 加载所有栏目信息
			this.findAllCategories();
		},
		methods:{
			handleSelectionChange(val) {
				this.ids = [];
				val.forEach((item)=>{
					this.ids.push(item.id);
				});
		    },
			batchDeleteCategory(){
				this.$confirm('此操作将永久删除信息, 是否继续?', '提示', {
		            confirmButtonText: '确定',
		            cancelButtonText: '取消',
		            type: 'warning'
		        }).then(() => {
					let url = '/manager/category/batchDeleteCategory';
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
	            			console.log(result.message);
	            		}
		            	this.findAllCategories();
		            });
		        }).catch(() => {
		            this.$message({
		                type: 'info',
		                message: '已取消删除'
		            });          
		        });
			},
			deleteCategory(id){
				this.$confirm('此操作将永久删除该信息, 是否继续?', '提示', {
		            confirmButtonText: '确定',
		            cancelButtonText: '取消',
		            type: 'warning'
		        }).then(() => {
		            let url = '/manager/category/deleteCategoryById?id='+id;
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
	            			// console.log(result.message);
	            		}
		            	this.findAllCategories();
		            });
		        }).catch(() => {
		            this.$message({
		                type: 'info',
		                message: '已取消删除'
		            });          
		        });
			},
			saveOrUpdateCategory(formName){
				this.$refs[formName].validate((valid) => {
		            if (valid) {
		                let url = '/manager/category/saveOrUpdateCategory';
						axios.post(url,this.categoryDialog.form)
						.then(({data:result})=>{
							if(result.status == 200){
								this.closeCategoryDialog();
								this.$notify({ 
									title: '成功',
						            message: result.message,
						            type: 'success'
						        });
								this.findAllCategories();
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
					            message: '网络异常'
					        });
						});
		            } else {
		                return false;
		            }
		        });
			},
			closeCategoryDialog(formName){
				this.categoryDialog.visible = false;
				if(formName){
					this.$refs[formName].resetFields();
				}
			},
			toAddCategory(row){
				if(row){
					this.categoryDialog.form = {
						id: row.id,
						name: row.name,
						comment: row.comment,
						parentId: row.parent?row.parent.id:''
					};
					this.categoryDialog.title = '修改招聘方向';
				} else {
					this.categoryDialog.form = {};
					this.categoryDialog.title = '添加招聘方向';
				}
				this.categoryDialog.visible = true;
			},
			findAllCategories(){
				this.loading = true;
				let url = '/manager/category/findAllCategory';
				axios.get(url).then(({data:result})=>{
					this.categories = result.data;
				}).catch((error)=>{
					console.log('error:',error);
				}).finally(()=>{
					this.loading = false;
				});
			}
		}
	}
</script>

<!-- 局部样式 -->
<style scoped>
.btns {
	margin-bottom: .5em;
}
i.fa {
	margin: 0 .3em;
	cursor: pointer;
}
i.fa.fa-trash {
	color: #f56c6c;
}
</style>