<template>
	<view class="action">
		<view class="actionInfo">
			<view class="tabbar bg-white">
				<view class="tab " :class="tab===0?'active':''" @click="tabs(0)">
					<text>行动 ({{total}})</text>
				</view>
				<view class="tab" :class="tab===1?'active':''" @click="tabs(1)">
					<text v-if="looktotal != ''">我围观行动({{looktotal}})</text>
					<text v-if="looktotal == ''">我围观行动</text>
				</view>
			</view>
			<view class="actionTabList">
				<view class="actionMy" v-show="tab===0">
					<actionlist v-for="(item,index) in cardList" :item="item" :key="index" :tab="tab" :index='index' v-on:toggleMask="toggleMask" :userId="userId"></actionlist>
				</view>
				<view class="actionLook" v-show="tab===1">
					<actionlist v-for="(item,index) in lookerList" :item="item" :key="index" :tab="tab" :index='index' v-on:toggleMask="toggleMask" :userId="userId"></actionlist>
				</view>
			</view>
		</view>	
		<!-- <view class="btn_bar">
			<view class="btns"><button class="btn" @click="goStep">制定新的行动项</button></view>
		</view> -->
		<backTop :scrollTop="scrollTop"></backTop>
		<view class="start-add" @click="goStep"  v-if="scrollTop<2000">
			<image src="../../static/images/icon/add.png" mode="widthFix"></image>
		</view>
		<view class="mask" :class="maskState===0 ? 'none' : maskState===1 ? 'show' : ''" @click="toggleMask">
			<view class="mask-content" >
				<view class="mask_top">
					<view class="mask_top_text" @tap="creatXd(1)">
						<text class="mask_text">重新编辑行动</text>
					</view>
					<view class="mask_top_text" @tap="creatXd(2)">
						<text class="mask_text">删除行动</text>
					</view>
				</view>
			</view>
		</view>
		
	</view>
</template>

<script>
	import{ mapState,mapMutations} from 'vuex'
import actionlist from "@/components/actionlist.vue";
import backTop from "@/components/backTop.vue"

export default {
	components:{
		actionlist,
		backTop
	},
	data() {
		return {
			vi:1,
			tab:0,//行动，围观，收藏
			userId:uni.getStorageSync('id'),
			cardList:[],
			lookerList:[],
			maskState:0,
			nextPage:1,//当前页数
			pageSize:10,//每页条数
			nextPageTwo:'',
			total:'',
			looktotal:'',
			pushId:'',
			index:'',
			scrollTop:0,
			scrollTopinfo:true,
			
			videoAd:'',
		};
	},
	onPageScroll(e) {
		this.scrollTop = e.scrollTop;
		if(this.scrollTopinfo){
			this.scrollTopinfo=false;
			setTimeout(()=>{
				this.scrollTop=0
				this.scrollTopinfo=true;
			},3000)
		}
		
	},
	watch:{
		hasLogin(){
			setTimeout(() => {
				this.inDada(this.tab);
				this.userId=uni.getStorageSync('id');
			
			}, 100);
			
		}
		
	},
	onShow() {	
		if(!this.hasLogin){
			return this.xdUniUtils.xd_login(this.hasLogin);
		}
	},
	onLoad() {
		if(!this.hasLogin){
			return this.xdUniUtils.xd_login(this.hasLogin);
		}
		this.inDada(this.tab);
		//#ifdef MP-WEIXIN
		wx.showShareMenu({
		  menus: ['shareAppMessage', 'shareTimeline']
		})
		//#endif
		
	},
	onReady() {
	},
	computed: {
	           ...mapState(['hasLogin'])  
	       }, 
	onShareAppMessage(res) {
		
		let that = this;
		if(res.from=="menu"){
		return	that.xdUniUtils.xd_onShare(
		'','/pages/selfCenter/selfView?userId='+uni.getStorageSync('id'),''
		);
		}else{
			if(that.tab==0){
				return {
					title:'第'+that.cardList[res.target.id].pushCardCishuCount+'次打卡:'+that.cardList[res.target.id].content,
					path: '/pages/index/action/action?pushId='+ that.cardList[res.target.id].id+'&share='+uni.getStorageSync('id')+'&isopen='+that.cardList[res.target.id].isopen,
					imageUrl:that.cardList[res.target.id].pictures?that.cardList[res.target.id].pictures:'https://chucun2019.oss-cn-beijing.aliyuncs.com/dynamic/1595733463227.png',
				}
			}else if(that.tab==1){
				return {
					title: '我为@'+that.lookerList[res.target.id].userName+'打Call：'+that.lookerList[res.target.id].content,
					path: '/pages/index/action/action?pushId='+ that.lookerList[res.target.id].id+'&share='+uni.getStorageSync('id')+'&isopen='+that.lookerList[res.target.id].isopen,
					imageUrl:that.lookerList[res.target.id].pictures?that.lookerList[res.target.id].pictures:'https://chucun2019.oss-cn-beijing.aliyuncs.com/dynamic/1595733463227.png',
				}
			}
			
		}		
	},//#ifdef MP-WEIXIN
		onShareTimeline(){
			let that = this;
			return {			
				query:'/pages/selfCenter/selfView?userId='+uni.getStorageSync('id'),
			}
				
			
		},
		//#endif
	methods: {
		tabs(e){
			
			this.tab=e
			if(e==0){
				this.inDada(0);
			}else if(e==1){
				this.inDada(1);
			}
		},
		setSaveShareInfo(res){
			this.xd_request_post(this.xdServerUrls.xd_saveShareInfo,{
				pushId:this.cardList[res.target.id].id,
				shareUserId:uni.getStorageSync('id'),
			},true
			   ).then(res => {
				 
				   })
		},
		creatXd(e){
			var that=this;
			var id=that.pushId;
			var i=that.index;
			if(e==1 && that.cardList[i].pushCardStatus!=2){
				uni.showModal({
				    title: '重新编辑行动',
				    content: '将删除原行动及打卡信息，在重新发布新行动',
				    success: function (res) {
						if (res.confirm) {
				           that.xd_request_post(that.xdServerUrls.xd_delPushDataByPushId,{pushid:id},true).then(res => {
				           	if (res.resultCode == 0) {
				           		uni.showToast({
				           		    title: '删除成功，将重新编辑',
									icon:'none',
				           		    duration: 1500,
									success() {
										try {
										    uni.setStorageSync('pushData',that.cardList[i] );
										} catch (e) {
										    // error
										}
										
										uni.navigateTo({
											url:'step1'
										})
									}
				           		});
				           	
				          
				        } else if (res.cancel) {
				           that.maskState=0;
				        }
						});
				    }
					}
				});
			}else if(e==2 ){
				if(that.cardList[i].pushCardStatus==2||that.cardList[i].pushCardStatus==3){
					uni.showModal({
					    title: '删除行动',
					    content: '删除未完成和已完成行动单次2元，月费6元，年费60元,观看视频免费删除 ',
						confirmText:'免费删除',
						cancelText:'取消',
					    success: function (res) {
					        if (res.confirm) {
									that.videoAd=that.xdUniUtils.videoAdUtil();
									if (that.videoAd) {
									   that.xdUniUtils.videoshowAd(that.videoAd);
									    
										 that.videoAd.onLoad(() => {
													uni.hideLoading();	 
										}) ;
									  that.videoAd.onClose((res) => {
										        that.videoAd.offClose()
												 if (res && res.isEnded || res === undefined) {
													 that.xd_request_post(that.xdServerUrls.xd_delPushDataByPushId,{
														 pushId:id,
														 tsbs:'jlsp',
													 },true).then(res => {	
														
														if (res.resultCode == 0) {
															uni.showToast({
																title: '删除成功',
																icon:'none',
																duration: 1500
															});
															that.cardList.splice(i,1);
														}else{
															uni.showModal({
																title: '该行动项发布已超过3天，不能删除，请继续',
																icon:'none',
										
															});
														}
																					
													  })
												 } else {
												    that.videoAd.offClose()
												   wx.showModal({
													 content: '广告未播放完成,无法删除',
													 showCancel: false
												   })
												 }
						 
									   }) ;
									   
					           
					        } else if (res.cancel) {
					            that.maskState=0;
					        }
					    }
						}
					})
					
				}else if(that.cardList[i].pushCardStatus==1){
					uni.showModal({
					    title: '删除行动',
					    content: '进行中行动项3天内删除将退回保证金',
					    success: function (res) {
					        if (res.confirm) {
					           that.xd_request_post(that.xdServerUrls.xd_delPushDataByPushId,{pushId:id},true).then(res => {
								  
					            	if (res.resultCode == 0) {
					            		uni.showToast({
					            		    title: '删除成功',
											icon:'none',
					            		    duration: 1500
					            		});
										that.cardList.splice(i,1);
					            	}else{
										uni.showModal({
										    title: '该行动项发布已超过3天，不能删除，请继续',
											icon:'none',
					
										});
									}
									
					            })
					        } else if (res.cancel) {
					            that.maskState=0;
					        }
					    }
					});
				}
			}
		},
		toggleMask(type,index){
			let timer = type >=0 ? 10 : 300;
			let	state = type >=0 ? 1 : 0;
			this.maskState = 2;
			this.pushId=type;
			this.index=index
			setTimeout(()=>{
				this.maskState = state;
			}, 10)
		},
		toAction(e){
			uni.navigateTo({
				url: '../index/action/action?pushId='+e
			});
		},
		goStep(){
			uni.navigateTo({
				url: `/pages/action/step1`
			});
		},
		
		inDada(tab){		
			let token='';
			let id='';
			let that=this;
			try{
				token=uni.getStorageSync('token');
				id=uni.getStorageSync('id');
			}catch(e){
				//TODO handle the exception
			}
			if(tab ==0){
				   that.xd_request_post(that.xdServerUrls.xd_pushByUserIdList,
				   {
				   token:token,
				   userId:id,
				   pageNum:1,
				   pageSize:10,
				   },
				   true
				   
					   ).then(res=>{
							 that.cardList=that.dataPaly(res);
							 that.nextPage=res.obj.nextPage;
							 that.total=res.obj.total;
					}).catch(Error=>{
						console.log(Error)
					})
			 }else{
					that.xd_request_post(that.xdServerUrls.xd_lookerPushListByUserId,
					{
						userId:uni.getStorageSync("id"),
						pageNum:1,
						pageSize:10,
					},
					true).then(res=>{
						
						that.lookerList=res.obj.list;
						that.nextPageTwo=res.obj.nextPage;
						that.looktotal=res.obj.total;
						that.lookerList.forEach(function (item) {
							if(typeof item.challengeRmb !='undefined' && item.challengeRmb != '' && item.challengeRmb != '0'){
								item.challengeRmb=Math.floor(item.challengeRmb/100);	
							}
							if(typeof item.pictures ==='undefined' || item.pictures == ''){
								item.pictures = that.xdUniUtils.xd_randomImg();
							}else{
								if(item.pictures.indexOf(",")> -1){
									item.pictures = item.pictures.split(",")[0]
								}
							}
						})			  
						
					}).catch(Error=>{
						console.log(Error)
					})
				}
			},
			getReachList(){
				let that=this;
				if(that.tab==0){
					if(that.nextPage==0){
						uni.showLoading(
						{
							title: '没有更多数据了'
						})
						setTimeout(function () {
							uni.hideLoading();
						}, 1000);
						return false
					}
					uni.showLoading(
					{
						title: '加载中..',
						mask:true
					})
					that.xd_request_post(that.xdServerUrls.xd_pushByUserIdList,
					{
						token:uni.getStorageSync('token'),
						userId:uni.getStorageSync('id'),
						pageNum:that.nextPage,
						pageSize:that.pageSize,
					},
					false
						   ).then(res=>{
							   that.nextPage=res.obj.nextPage;
							 var data=that.dataPaly(res);		
							that.cardList = that.cardList.concat(data);					
							setTimeout(function () {
								uni.hideLoading()
							}, 1000);
							// uni.hideNavigationBarLoading();//关闭加载动画
							
						})	
				 }else if(that.tab==1){
					 
					 if(that.nextPageTwo==0){
					 	uni.showLoading(
					 	{
					 		title: '没有更多数据了'
					 	})
					 	setTimeout(function () {
					 		uni.hideLoading();
					 	}, 1000);
					 	return false
					 }
					 uni.showLoading(
					 {
					 	title: '加载中..',
					 	mask:true
					 })
					 that.xd_request_post(that.xdServerUrls.xd_lookerPushListByUserId,
					 {
					 	userId:uni.getStorageSync("id"),
					 	pageNum:that.nextPageTwo,
					 	pageSize:10,
					 },
					 true).then(res=>{
					 	var data=res.obj.list;
					 	that.nextPageTwo=res.obj.nextPage;
					 	data.forEach(function (item) {
					 		if(typeof item.challengeRmb !='undefined' && item.challengeRmb != '' && item.challengeRmb != '0'){
					 			item.challengeRmb=Math.floor(item.challengeRmb/100);	
					 		}
					 		if(typeof item.pictures ==='undefined' || item.pictures == ''){
					 			item.pictures = that.xdUniUtils.xd_randomImg();
					 		}else{
					 			if(item.pictures.indexOf(",")> -1){
					 				item.pictures = item.pictures.split(",")[0]
					 			}
					 		}
					 	})			  
					 	that.lookerList = that.lookerList.concat(data);
					 	setTimeout(function () {
					 		uni.hideLoading()
					 	}, 1000);	
					 }).catch(Error=>{
					 	console.log(Error)
					 })
				 }
				},
				dataPaly(res){
					let dataList=res.obj.list;
					
					var date=new Date();
					
					date=date.getTime();
					for(var i=0;i <dataList.length;i++){
						
						// var num=dataList[i].targetDay-dataList[i].pushCardCount;
						// var num2=dataList[i].targetDay;
						// var num3=dataList[i].targetDay+dataList[i].holidayDay
						// var num4=dataList[i].pushCardCount;
						
						// let d = new Date(dataList[i].createTime);
						// let newD = new Date(d.setDate(d.getDate() + num3));
						
						// newD=newD.getTime()
						
						// let dd=Math.round((date-newD) / (1000 * 60 * 60 * 24));
						// if(num>0 && dd<=0 ){
						// 	dataList[i].btn=0//立即打卡
						// }else if(num2>num4 && dd>0){
						// 	dataList[i].btn=1}//未达成
						// 	else if(num==0&&num2==num4){
						// 		dataList[i].btn=2}	//已完成    
						
						dataList[i].challengeRmb=Math.floor(dataList[i].challengeRmb/100);		
						
					}
					return dataList;
				}
				
	},
	onReachBottom() {
		this.getReachList()
	},
	onPullDownRefresh() {
		this.inDada(this.tab);
		uni.stopPullDownRefresh();
	},
	components:{
		actionlist
	}
};
</script>

<style lang="scss">
.action {
	padding: 0 20rpx 0 20rpx;
	font-size: 30rpx;
}
.actionInfo{
	.tabbar{
		font-size: 28rpx;
		display: flex;
		justify-content: space-between;
		.tab{
			flex: 1;
			text-align: center;
			border-bottom: 1px solid #d9d9d9;
			padding:16rpx;
			&.active{
				border-bottom: 1px solid #fd5107;
				color:#fd5107;
			}
		}
	}
}
// .btn_bar{
// 	position: fixed;
// 	bottom: 0;
// 	left:25%;
// 	width: 50%;
// 	.btns {
// 		height: 100rpx;
		
// 		display: flex;
// 		align-items: center;
// 		justify-content: space-between;
// 		font-size: 28rpx;
// 		.btn {
// 			flex: 1;
// 			height: 64rpx;
// 			line-height: 64rpx;
// 			background: #ffa700;
// 			// color: #fff;
// 			font-size: 28rpx;
// 			border-radius: 40rpx;

// 		}
// 	}
// 	}
	
	.mask{
		display: flex;
		align-items: flex-end;
		position: fixed;
		left: 0;
		top: var(--window-top);
		bottom: 0;
		width: 100%;
		background: rgba(0,0,0,0);
		z-index: 9995;
		transition: .3s;
		
		.mask-content{
			width: 100%;
			// min-height: 30vh;
			max-height: 70vh;
			background: #f3f3f3;
			transform: translateY(100%);
			transition: .3s;
			overflow-y:scroll;
		}
		&.none{
			display: none;
		}
		&.show{
			background: rgba(0,0,0,.4);
			
			.mask-content{
				transform: translateY(0);
			}
		}
	}
	.mask_top{
		display: flex;
		flex-direction: column;
	}
	.mask_top_text{
		margin-bottom: 5upx;
		background-color: #FFFFFF;
		width: 100%;
		height: 100upx;
		text-align: center;
	}
	.mask_text{
		font-size: 28upx;
		font-weight: bold;
		line-height: 100upx;

	}
	.start-add{
		width: 100upx; height:100upx;
		display:flex; flex-direction:row; justify-content:center; align-items:center;
		background: #ffe66f;
		border: 2px solid #ffa700;
		border-radius: 50%;
		position: fixed; bottom: 100upx; right:30upx; z-index: 99;
	}
	.start-add image{
		width: 48upx; height:48upx;
	}
	
</style>
