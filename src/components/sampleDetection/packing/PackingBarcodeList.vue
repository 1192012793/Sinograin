<template>
    <div class='listpagewrap'>
      <!--面包屑-->
      <sinograin-breadcrumb :breadcrumb="breadcrumb" v-on:searchingfor="searchingfor"></sinograin-breadcrumb>
      <!--alert-->
      <!--<sinograin-prompt :alerts="alerts"></sinograin-prompt>-->
      <!--表格上的时间选框以及 创建-->
      <list-header :listHeader="listHeader" v-on:dateChange="dateChange" v-on:statusChange="statusChange" v-on:createSampling="createSampling" v-on:createlib="createlib" v-on:scanCode="scanCode" ></list-header>
      <!--表格-->
      <sinograin-list class="list" :tabledata="tabledatas" :list="list" :items="items" :actions="actions" v-on:getchecked="getchecked" :loading="loading" v-on:emptyCreate="emptyCreate" > 
      </sinograin-list>
      <!--分页-->
      <sinograin-pagination :page="page" v-on:paginationEvent="paginationEvent" v-on:getCurrentPage="getCurrentPage"></sinograin-pagination>
      <!--新建库典弹框-->
      <sinograin-modal v-if="modalVisible"  :modal="modal" v-on:createlibitem="createlibitem" v-on:dialogClose="dialogClose"></sinograin-modal>      	
      <sinograin-message v-if="messageShow" :messages="messages" v-on:messageclick="messageclick" v-on:messageClose="messageClose" @getScanCode="getScanCode"></sinograin-message>
    
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
import SinograinMessage from "@/components/common/action/Message"
import "@/assets/style/common/list.css"
import { mapState,mapMutations,mapGetters,mapActions} from 'vuex';

//这里是打印控件
import {getLodop} from 'static/lodop/LodopFuncs'
let LODOP
//本地测试要用下面import代码
//import data from '@/util/mock';



export default {
  components: {
    SinograinList,SinograinPrompt,SinograinPagination,SinograinBreadcrumb,SinograinModal,ListHeader,SinograinMessage
  },
  computed:{
	...mapState(["modal_id_number","viewdata","editdata","aultdata","messions","mask"]),
	...mapGetters(["modal_id"]),
	tabledatasFilter(){

		if(this.filterStatus=="全部"){
			return this.tabledatas;
		}else{
			return this.tabledatas.filter((value,index)=>{
				return value.status==this.filterStatus
			})
		}
	}
  },
  created(){
//	console.log(this.$route.query)
//  获取列表数据（第一页）
	this.getlistdata(1)
//	移除监听事件
    this.$root.eventHub.$off('delelistitem')
    this.$root.eventHub.$off("viewlistitem")
    this.$root.eventHub.$off("printlistitem")
//	监听列表删除事件
    this.$root.eventHub.$on('delelistitem',function(rowid,list){
    	this.tabledatas=this.tabledatas.filter(function(item){
    		return item.id!==rowid;
    	})
    	this.sendDeleteId(rowid);
//  	console.log(rowid,list);
    }.bind(this)); 	
//	监听列表点击查看事件
  	this.$root.eventHub.$on("viewlistitem",function(id,state,row){  
		if(!this.$_ault_alert('cedingjilu:getBySmallSampleId')){
			return
		}
//		this.$router.push({path: '/index/sampleDetection/packingList/packingView',query:{libid:id}})
		var path=this.$route.name+'/打印条码'
		this.$router.push({name: path,params: {code:row.sampleNum,sort:row.sort,checkeds:row.checkeds,id:row.id,sampleState:row.sampleState,taskId:row.taskId}})
  	}.bind(this));
//	监听列表点击打印事件
  	this.$root.eventHub.$on("printlistitem",function(code){  
		this.printitem(code);	
  	}.bind(this));
  },
  destroy(){
  	this.$root.eventHub.$off("viewlistitem")
    this.$root.eventHub.$off("printlistitem")
  	this.$root.eventHub.$off('delelistitem')
  },
  methods: {
  	...mapMutations(['create_modal_id','is_mask','create_modal','close_modal']),
  	...mapActions(['addAction']),
//	列表头触发的事件
	dateChange(data){
		console.log(data);
	},
	statusChange(data){
//		console.log(data)
		this.filterStatus=data
	},
	createSampling(){
//		console.log('createSampling');
//		this.$router.push({path: '/index/sampling/samplingList/samplingListCreate'})
	},
	emptyCreate(){
//		this.scanCode();
	},
//	打开新建弹框
	createlib(){
		this.modalVisible=true;
	},
//	扫码新建样品
	scanCode(){
//		if(!this.$_ault_alert('sample:getBySampleNum')){
//			return
//		}
		this.messages.type='scaning';
		this.messageShow=true;
//		this.$router.push({path: '/index/sampleDetection/packingList/packingPrint'})
	},
//	填入新建数据
	createlibitem(form){
		console.log(form);
	},
//	关闭新建弹框
	dialogClose(){
		this.modalVisible=false;
	},
//	获取搜索数据
  	searchingfor(searching,page){
  		page?page:1;
  		this.searchText=searching.indexOf('监')==0?searching.slice(1):searching;
  		var params = {};
//		params.sampleNoOrSmallSampleNumLike = searching;
		params.sampleNumLike = searching;
		params.sampleState = 3;
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
				page:page,
			    rows:this.page.size,
			   	params:JSON.stringify(params)
			}
	    }).then(function (response) {
		  	this.tabledatas=response.data.rows;
			this.page.total = response.data.total;
			this.page.currentPage = page;
		}.bind(this)).catch(function (error) {
		    console.log(error);
		}.bind(this));
  	},
	
//	获取列表数据方法
  	getlistdata(page){
  		this.loading=false;
  		var params={};
//		params.taskId=this.$route.query.id;
  		params.sampleState=3;
  		// 获取列表数据（第？页）
		this.$http({
		    method: 'post',
			url: this.datalistURL,
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
			    page:page,
			    rows:this.page.size,
			    params:JSON.stringify(params),
			}
	    }).then(function (response) {
//			console.log(response)
			this.tabledatas = response.data.rows;
			this.page.total = response.data.total;
			this.loading = false;
		}.bind(this)).catch(function (error) {
		    console.log(error);
		}.bind(this));
  	},
  	//	发送删除id
  	sendDeleteId(id){
		this.$http({
		    method: 'post',
			url: this.deleteURL,
			headers: {'Content-Type': 'application/x-www-form-urlencoded'},
			data: {
			    listName: this.list,
			    id:id,
			}
	    }).then(function (response) {
		  	
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
  	paginationEvent(actiontype){
  		if(actiontype=='leading_out'){
  			console.log('leading_out')
  		}else if(actiontype=='refresh'){
  			// 获取列表数据（第一页）
			this.getlistdata(1)			
  		}
  	},
//	获取多选框选中数据的id(这是一个数组)
  	getchecked(checkedId){
  		this.checkedId=checkedId;
  	},
	messageclick(type){
  		if(type=="success"){
			console.log(type)
  		}else if(type=="error"){
  			console.log(type)  			
  		}
  	},
  	messageClose(){
  		this.messageShow=false;
  	},
  	getScanCode(code){
//		if(!this.$_ault_alert('sample:getBySmallSampleNum')){
//			return
//		}
  		
  		if(!code){
  			this.messageShow=false;
  		}else{  			
			this.messageShow=false;
			this.checkcode(code);			
		}
  	},
//	去验证条码有效否
	checkcode(code){
		var params={};
		params.sampleState=3;
		params.sampleNum=code;
		this.$http({
		    method: 'post',
			url: this.checkURL,
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
//				sampleNum:code,
				params:JSON.stringify(params)
			}
	    }).then(function (response) {
//			console.log(response)
			if(response.data.rows[0].checkeds) {
				var path=this.$route.name+'/打印条码'
//				this.$router.push({name: path,params: {code:response.data.sampleNum,sort:response.data.sort,checkeds:response.data.checkeds,id:response.data.id,sampleState:response.data.sampleState,taskId:this.$route.query.id}})
				this.$router.push({name: path,params: {code:response.data.rows[0].sampleNum,sort:response.data.rows[0].sort,checkeds:response.data.rows[0].checkeds,id:response.data.rows[0].id,sampleState:response.data.rows[0].sampleState,taskId:response.data.rows[0].taskId}})
			}else{

				this.$notify.error({
		          	title: '未获取到检验信息',
		          	message: '请重新核对条码信息！！！',
		        });
			}
		}.bind(this)).catch(function (error) {
//		    console.log(error);

		    this.$notify.error({
	          	title: '扫码失败',
	          	message: '请重新核对条码信息！！！',
	        });
		}.bind(this));
	},
	printitem(code){
		this.messageShow=true;
		this.messages.type="loading";
		this.printCodeBar(code)
	},
	printCodeBar(code){
		LODOP = getLodop();

		LODOP.PRINT_INIT("打印条码");
		LODOP.SET_PRINTER_INDEX("Godex G530");  
		LODOP.SET_PRINT_PAGESIZE(1, 700, 400, "USER");
		LODOP.ADD_PRINT_BARCODE(3,30,232,115,'128B',code);
//  			LODOP.PREVIEW(); 
		LODOP.PRINT(); 

	}
  },
  data() {
    return {
//    datalistURL: this.apiRoot +  '/grain/smallSample/data',
      datalistURL: this.apiRoot +  '/grain/sample/data',
      checkURL:this.apiRoot +'/grain/sample/data',
//    checkURL:'/saomiaofenxiaoyang',
//    searchURL:this.apiRoot +  '/grain/smallSample/data',
      searchURL:this.apiRoot +  '/grain/sample/data',
      deleteURL:'/liquid/role2/data/delete',
      searchText:'',
      checkedId:[],
      list:"samplinglist",
	  modalVisible:false,
	  modal:{
	  	title:'新建库点',
		formdatas:[
	  		{
	  			label:"单位名称",
	  			model:"unit",
	  		},
	  		{
	  			label:"库点名称",
	  			model:"lib",
	  		},
	  	],
	  	submitText:'确定',
	  },
      breadcrumb:{
      	search:false,   
      	searching:'',
      },
      loading:true,
      filterStatus:'全部',
//    分页数据
      page: {
        size: 10,
        total: 0,
        currentPage: 1,
        show:true,       
        tfootbtns:{
        	btns:false,//是否添加按钮组
        	leading_out:false,//导出按钮
        	refresh:false,//刷新按钮
        	delete:false, //删除按钮            	
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
      	status:false,
      	date:true,
      	scanCode:true,
      },
      tabledatas:[],
      items: [
      {
        id: 1,
        prop:'sampleNum',
//      prop:'sampleNo',
        label: "检验编号",
		status:true,
      },
//    {
//      id: 2,
//      prop:'smallSampleNum',
//      label: "小样编号",
//		status:true,
//    },
      {
        id: 3,
        prop:'checkeds',
        label:"检验项目",
		status:true,
      },
      {
      	id:4,
      	prop:'detectionState',
      	label:'检验状态',
      	status:true,
      }
//    {
//      id: 3,
//      prop:'checkPoint',
//      label:"检验项目",
//		status:true,
//    },
//    {
//      id: 4,
//      prop:'printTimes',
//      label: "打印条码数",
//
//    },
//    {
//      id: 5,
//      prop:'printDate',
//      label:"打印时间",
//      sort:true,
//    },
      ],
      actions:{
//    	noview:true,
      	selection:false,
      	number:false,
      	view:false,
      	edit:false,
      	dele:false,
      	print:false,
      	show:false,
      	manuscript:false,
      	safetyReport:false,
//    	sort:'smallSampleNum',
      },
      messageShow:false,
	  messages:{
	  	type:'scaning',
	  	scaning:{
	  		icon:'iconfont icon-iconset0255',
	  		messageTittle:'开始扫描',
	  		messageText:'不要点击键盘和鼠标请将扫码枪对准条形码，然后按下扫码枪按钮！',
	  	},
	  	success:{
	  		icon:'el-icon-success',
	  		messageTittle:'该样品已入库',
	  		messageText:'可点击编辑按钮修改入库信息！',
	  		buttonText:'编辑',
	  	},
	  	loading:{
	  		icon:'el-icon-printer',
	  		messageTittle:'正在打印中...',
	  		messageText:'请您耐心等待，正在打印中...',
	  	},
	  },
    }
  }
}
</script>

