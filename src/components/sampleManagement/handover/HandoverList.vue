<template> 
	<div class='listpagewrap'>
		<!--面包屑-->
		<sinograin-breadcrumb :breadcrumb="breadcrumb" v-on:searchingfor="searchingfor"></sinograin-breadcrumb>
		<!--alert-->
		<!--<sinograin-prompt :alerts="alerts"></sinograin-prompt>-->
		<!--表格上的时间选框以及 创建-->
      	<list-header :listHeader="listHeader" v-on:dateChange="dateChange" v-on:statusChange="statusChange" @statusChange2="statusChange2" v-on:createSampling="createSampling" v-on:createlib="createlib" v-on:scanCode="scanCode" @connect='connect'></list-header>
		<!--表格-->
		<sinograin-list class="list" :tabledata="tabledatas" :list="list" :items="items" :actions="actions" v-on:getchecked="getchecked" :loading="loading" v-on:emptyCreate="emptyCreate">
		</sinograin-list>
		<!--分页-->
		<sinograin-pagination :page="page" v-on:paginationEvent="paginationEvent" v-on:getCurrentPage="getCurrentPage"></sinograin-pagination>
		<!--新建库典弹框-->
      	<sinograin-modal v-if="modalVisible"  :modal="modal" v-on:createlibitem="createlibitem" v-on:dialogClose="dialogClose" @modelSelectChange="modelSelectChange"></sinograin-modal>      	
	
	</div>
</template>

<style>

</style>

<script>
import SinograinList from '@/components/common/action/List.vue';
import SinograinPrompt from '@/components/common/prompt/Prompt.vue';
import SinograinBreadcrumb from '@/components/common/action/Breadcrumb.vue';
import SinograinPagination from '@/components/common/action/Pagination.vue';
import ListHeader from '@/components/common/action/ListHeader.vue';
import SinograinModal from '@/components/common/action/Modal.vue';
import "@/assets/style/common/list.css"
import { mapState, mapMutations, mapGetters, mapActions } from 'vuex';
//本地测试要用下面import代码
//import data from '@/util/mock';



export default {
	components: {
		SinograinList, SinograinPrompt, SinograinPagination, SinograinBreadcrumb, SinograinModal, ListHeader
	},
	computed: {
		...mapState(["modal_id_number", "viewdata", "editdata", "aultdata", "messions", "mask"]),
		...mapGetters(["modal_id"]),
		tabledatasFilter() {

			if (this.filterStatus == "全部") {
				return this.tabledatas;
			} else {
				return this.tabledatas.filter((value, index) => {
					return value.status == this.filterStatus
				})
			}
		}
	},
	created() {
//		console.log(this.$route.query)
		//  获取列表数据（第一页）
		this.getlistdata(1)
		//	移除监听事件
		this.$root.eventHub.$off('delelistitem')
		this.$root.eventHub.$off("viewlistitem")
		this.$root.eventHub.$off("editlistitem")
		this.$root.eventHub.$off("returnPerson")
		//	监听列表删除事件
		this.$root.eventHub.$on('delelistitem', function(rowid, list) {
			if(!this.$_ault_alert('handover:remove')){
				return
			}
			this.$confirm('此操作将永久删除该交接单信息, 是否继续?', '提示', {
		     	confirmButtonText: '确定',
		      	cancelButtonText: '取消',
		      	type: 'warning'
		   }).then(() => {		    	
				this.sendDeleteId(rowid);
		    }).catch(() => {
		      	this.$message({
		        	type: 'info',
		        	message: '已取消删除'
		      	});          
		    });

			//  	console.log(rowid,list);
		}.bind(this));
		//	监听列表点击查看事件
		this.$root.eventHub.$on("viewlistitem", function(id) {
			if(!this.$_ault_alert('handover:getById')){
				return
			}
			//		console.log(id)
			this.$router.push({ path: '/index/sampleManagement/handover/handoverListView', query: { id: id } })

		}.bind(this));
		//监听列表点击编辑事件
		this.$root.eventHub.$on('editlistitem',function(id,row){
//			if(!this.$_ault_alert('handover:save')){
//				return
//			}
			var formdatas={};
				formdatas.id=row.id;
				formdatas.title='样品领取交接单';
		  		formdatas.loading=false,
		        formdatas.form={            	
		        	name:row.name,//交接单名称
		        	manager:row.sampleAdmin,//管理员名
		        	remarks:row.remark,//备注信息
		        	sort:row.sort,//品种
		        };
		        formdatas.checkList=row.checkeds.split(',');//检验项目数组
		        formdatas.items=row.sampleNums.split(',');//      检验样品数组
			var path=this.$route.name+'/编辑样品领取交接单'
			this.$router.push({name: path,params: {formdatas:formdatas}})
//			this.$router.push({ path: '/index/sampleManagement/handover/handoverListEdit', query: { id: id } })
		}.bind(this));
		//	监听列表点击归还事件
		this.$root.eventHub.$on("returnPerson", function(id) {
//			if(!this.$_ault_alert('handover:getById')){
				return
//			}
			this.returnId=id
			this.modalVisible=true;
		}.bind(this));
	},
	destroy() {
		this.$root.eventHub.$off("viewlistitem")
		this.$root.eventHub.$off('delelistitem')
		this.$root.eventHub.$off("editlistitem")
		this.$root.eventHub.$off("returnPerson")
	},
	methods: {
		...mapMutations(['create_modal_id', 'is_mask', 'create_modal', 'close_modal']),
		...mapActions(['addAction']),
		//	列表头触发的事件
		dateChange(data) {
			console.log(data);
		},
		scanCode(){
			
		},
		statusChange(data) {
			this.filterStatus = data;
			this.getlistdata(1)
//	  		this.page.currentPage=1;
		},
		statusChange2(data) {
			console.log(data)
		},
		createSampling() {
			//		console.log('createSampling');
			this.$router.push({ path: '/index/sampling/samplingList/samplingListCreate' })
		},
		emptyCreate() {
			this.connect();
		},
		//	打开新建弹框
		createlib() {
			this.modalVisible = true;
		},
		//	新建样品交接单
		connect() {
			if(!this.$_ault_alert('handover:save')){
				return
			}
			this.$router.push({ path: '/index/sampleManagement/handover/handoverListCreate' })
		},
		//	填入新建数据
		createlibitem(form) {
			console.log(form);
		},
		modelSelectChange(val,model){
		
		},
		//	关闭新建弹框
		dialogClose() {
			this.modalVisible = false;
		},
		//	获取搜索数据
		searchingfor(searching,page){
	  		page?page:1;
	  		this.searchText=searching;
	  		var params = {};
			params.nameLikeOrId = searching;
//			if(this.filterStatus==-1){
//				params.returnState=-1;
//			}else if(this.filterStatus==1){
//				params.returnState=1;	
//			}
	//		console.log(this.breadcrumb.searching);
	  		// 获取列表数据（第？页）
			this.$http({
			    method: 'post',
				url: this.searchURL,
				transformRequest: [function (data) {
					// Do whatever you want to transform the data
					let ret = ''
					for (let it in data) {
					ret += encodeURIComponent(it) + '=' + encodeURIComponent(data[it]) + '&'
					}
					return ret
				}],
				headers: {'Content-Type': 'application/x-www-form-urlencoded'},
				data: {
				   params:JSON.stringify(params)
				}
		    }).then(function (response) {
			 	this.tabledatas = response.data.rows;
				this.page.total = response.data.total;
				this.loading = false;
	  			this.page.currentPage=page;
	
			}.bind(this)).catch(function (error) {
			    console.log(error);
			}.bind(this));
	  	},
		//	获取列表数据方法
		getlistdata(page) {
			this.loading = true;
			var params = {};
			if(this.filterStatus==-1){
				params.returnState=-1;
			}else if(this.filterStatus==1){
				params.returnState=1;	
			}
			if(this.searchText){				
				params.nameLike = this.searchText;
			};
			// 获取列表数据（第？页）
			this.$http({
				method: 'post',
				url: this.datalistURL,
				transformRequest: [function (data) {
					let ret = ''
					for (let it in data) {
					ret += encodeURIComponent(it) + '=' + encodeURIComponent(data[it]) + '&'
					}
					return ret
				}],
				headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
				data: {
					page:page,
			    	rows:this.page.size,
				   	params:JSON.stringify(params)			    	
				}
			}).then(function(response) {
				this.tabledatas = response.data.rows;
				this.page.total = response.data.total;
				this.loading = false;
	  			this.page.currentPage=page;
			}.bind(this)).catch(function(error) {
				console.log(error);
			}.bind(this));
		},
		//	发送删除id
		sendDeleteId(id) {
			this.$http({
			    method: 'post',
				url: this.deleteURL,
				transformRequest: [function (data) {
					// Do whatever you want to transform the data
					let ret = ''
					for (let it in data) {
					ret += encodeURIComponent(it) + '=' + encodeURIComponent(data[it]) + '&'
					}
					return ret
				}],
				headers: {'Content-Type': 'application/x-www-form-urlencoded'},
				data: {
					id:id
				},
		    }).then(function (response) {
		    	if(response.data.success){
					this.tabledatas=this.tabledatas.filter(function(item){
		    			return item.id!==id;
		    		})		  
		    		this.$message({
			        	type: 'success',
			        	message: '删除成功!'
			      	});
		    	}else{
		    		this.$message.error('删除失败!');
		    	}
			}.bind(this)).catch(function (error) {
			    console.log(error);
			}.bind(this));
		},
		//	获取分页点击事件中及当前页码
	    getCurrentPage(currentPage){
			if(this.searchText){			
				this.searchingfor(this.searchText,currentPage)
			}else{			
				this.getlistdata(currentPage)
			}
		},
		//	映射分页触发的事件
		paginationEvent(actiontype) {
			if (actiontype == 'leading_out') {
//				导出扦样表事件已触发
				console.log('leading_out')
			}else if (actiontype == 'handoverPrint') {
//				打印样品领取交接单触发
				console.log('handoverPrint')
			}else if (actiontype == 'refresh') {
				// 获取列表数据（第一页）
				this.getlistdata(1)
			}
		},
		//	获取多选框选中数据的id(这是一个数组)
		getchecked(checkedId) {
			this.checkedId = checkedId;
		},

	},
	data() {
		return {
			datalistURL: this.apiRoot +'/grain/handover/data',
			searchURL: this.apiRoot +'/grain/handover/data',
			deleteURL: this.apiRoot +'/grain/handover/removeHandover',
      		searchText:'',
			checkedId: [],
			list: "samplinglist",
			modalVisible:false,
			modal:{
			  	title:'归还',
				formdatas:[
					{
			  			label:"归还人:",
			  			model:"returnPerson",
//			  			disabled:true,
			  			value:'',
			  			type:'input',
			  		},					
			  	],		
			  	submitText:'归还',
			},
			breadcrumb: {
				search: true,
				searching: '',
			},
			loading: true,
			filterStatus: '全部',
			//    分页数据
			page: {
				size: 10,
				total: 0,
				currentPage: 1,
				show: true,
				tfootbtns: {
					btns: false,//是否添加按钮组
					leading_out: false,//导出按钮
					refresh: false,//刷新按钮
					delete: false, //删除按钮 
					connect: false//打印样品交接单            	
				}
			},
			//    弹窗数据
			alerts: [{
				title: '温馨提示：此页面只展示本库信息!',
				type: 'info'
			}],
			//    表格数据
			listHeader:{
		      	createlib:false,
		      	createSampling:false,
		      	date:false,
		      	scanCode:false,
				connect: true,
//		      	status:true,
		      	statusitems:[
		      		{label:'全部',text:'全部'},
		      		{label:'-1',text:'已归还'},
		      		{label:'1',text:'未归还'},
		      	],
		      	status2:false,
		      	statusTitle:'交接单状态:',
		      	statusTitle2:[],
		    },
			tabledatas: [],
			items: [
				{
					id: 1,
					prop: 'id',
					label: "编号",
					sort: true,
					status:true,
				},
				{
					id: 2,
					prop: 'name',
					label: "样品领取交接单名称",
//					sort: true,
				},
				{
					id: 3,
					prop: 'checkeds',
					label: "检验项目",
					status:true,
					width:350,
//					sort: true,
				},
				{
					id: 4,
					prop: 'receiptor',
					label: "领取人",
//					sort: true,
				},
				{
					id: 5,
					prop: 'createTime',
					label: "领取时间",
					sort: true,
				},
//				{
//					id: 6,
//					prop: 'returnPerson',
//					label: "归还人",
////					sort: true,
//					status:true,
//				},
//				{
//					id: 7,
//					prop: 'returnTime',
//					label: "归还时间",
//					sort: true,
//					status:true,
//				},
			],
			actions: {
				selection: false,
				number: false,
				view: false,
				edit: true,
				dele: false,
				manuscript: false,
				safetyReport: false,
				show:true,
			}
		}
	}
}
</script>

