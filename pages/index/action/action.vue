<template>
	<view >
		<lookerCountInfo ref="lookerCountInfo"></lookerCountInfo>
		<view class="cu-card dynamic " :class="pushList.pictures!=''?'no-card':''">
			<view class="cu-item shadow">
				<view class="cu-list menu-avatar">
					<view class="cu-item ">
						<view @tap="goUser(pushList.userId)" class="cu-avatar round lg" :style="{backgroundImage: 'url(' +pushList.userHead + ')'}" ></view>
						<view class="content flex-sub">
							<view @tap="goUser(pushList.userId)">{{pushList.userName}}</view>
						</view>
						<view >
							<view class="cu-tag line-orange radius" v-if="guanzhu.length > 0" @tap="tags">
								{{guanzhu}}
							</view>
						</view>
					</view>
					<view class="flex flex-wrap padding justify-between">
						<view class="widthtext " >
							<view class="flex flex-wrap justify-between">
								<view class="">
									<text class="text-orange" v-if="pushList.pushCardStatus==1">进行中...</text>
									<text class="text-gray" v-else-if="pushList.pushCardStatus==2">未达成</text>
									<text class="text-green" v-else-if="pushList.pushCardStatus==3">已达成</text>
									<view class="margin-left-xs cu-tag bg-grey radio">{{pushList.label}}</view>
								</view>
								<view class="text-xl" v-if="pushList.isopen==1">
									<text class="lg text-orange cuIcon-lock" ></text>
								</view>
							</view>
							
							<view class="text-gray text-sm ">
								阶段期限：{{pushList.createTime}}--{{pushList.endTime}}
							</view>
							<view class="text-gray text-sm  flex flex-wrap">
								<view class="">
									已达成天数：{{pushList.pushCardCount}}/{{pushList.targetDay}}
								</view>
								<view class="margin-left-sm"> 
								    可休假天数：{{pushList.kholidayDay}}/{{pushList.holidayDay}}
								</view>
							</view>
						</view>
						<view v-if="pushList.challengeRmb>0">
							<view class="cu-tag light bg-red radius" >
								保证金￥{{pushList.challengeRmb}}
							</view>
							<view style="display:inline-block; padding-left:5px;">
								<view v-if="!pushList.belongOwn "  @tap="goTOSponsor(0,pushList.id)" class="cu-tag light bg-yellow radius" >
									赞助 
								</view>{{!pushList.belongOwn && pushList.sponsorCount>0?pushList.sponsorCount:''}}
								<view v-if="pushList.belongOwn" @tap="goTOSponsor(0,pushList.id)" class="cu-tag light bg-yellow radius" >
									获赞助金 ￥{{pushList.challengeRmb}}
								</view>
							</view>
							
						</view>
					</view>
				</view>
				
				<view class="text-contents">
					<text class="contentext" >{{pushList.content}}
					</text>
				</view>
				<view class="grid flex-sub padding-lr"  >
					<image class="bg-img imgheit"  :src="pushList.pictures" mode="aspectFill"
					 @tap="goPageImg(pushList.pictures)" v-if="pushList.pictures!=''">
					</image>
					<image class="bg-img imgheit"  :src="audioPlaySrc" mode="aspectFill"
					 @tap="goPageImg(audioPlaySrc)" v-else @error="error">
					</image>
				</view>
				<view class="flex padding justify-between" >
					<view>
						<button class="cu-btn bg-light-blue sm round" v-if="pushList.userId==userId"  :id="index" open-type="share">分享邀请</button>
						<button class="cu-btn bg-orange sm round" v-else-if="pushList.onlooker"  :id="index" open-type="share">为TA打Call</button>
						<button class="cu-btn bg-green sm round  " v-else-if="pushList.userId!=userId && !pushList.onlooker&&pushList.challengeRmb<=0"  @tap="lookerClick(pushList,index)">围观</button>
						<button class="cu-btn bg-green sm round  " v-else  @tap="lookerClick(pushList,index)">围观分钱</button>
						<text class="text-gray text-df ">{{pushList.onlookerCount}}</text>
					</view>
					<!-- <view class="text-xxl"  >
						<button class="cu-btn line-green sm round  " @click="gothanl(pushList)" v-if="userId==pushList.userId" >我要感谢</button>
					</view> -->
					<view class="text-xxl"  >
						<button class="cu-btn line-green sm round  " @click="goSteps" v-if="userId==pushList.userId" >立即打卡</button>
						<button class="cu-btn line-green sm round  " @click="gostep" v-else>一起行动</button>
					</view>
				
				</view>
			</view>
			<scroll-view scroll-x class="bg-white nav">
				<view class="flex text-center">
					<view class="cu-item flex-sub" :class="index==TabCur?'text-orange cur':''" v-for="(items,index) in ['打卡内容','围观排行']" :key="index" @tap="tabSelect" :id="index" >
						{{items}}
					</view>
				</view>
			</scroll-view>
			<block v-for="(item,index) in pusCardList" :key="index" v-if="TabCur==0">
				<view class="cu-timeline">
					<view class="cu-time" v-if="index == 0 || compareDate(pusCardList[index-1],item)">{{item.createTime}}</view>
					<view class="cu-item">
						<view class="content">
							<view class="">
								<view class="cu-tag line-green">第{{pusCardList.length-index}}次打卡</view>
							</view>
							<view  class="margin-top-sm margin-bottom-sm margin-left-lg textcon" @tap="gocardComentList(item,0)">{{item.content}}</view>
							<view class="videheit" v-if="item.videos!=''&&item.videos!=undefined &&item.videos!=null ">
								<video class="videos"  :src="item.videos" controls></video>
							</view>
							<view v-else class="grid flex-sub padding-lr"   >
								<image class="bg-img imgheit"  :src="item.pictures[0]" mode="aspectFill"
								 @tap="goPageImg(item.pictures)" v-if="item.pictures.length!=''">
								</image>
							</view>
							<view class="text-xxl flex flex-wrap justify-end " @tap="gocardComentList(item,1)">
								<text class="text-gray cuIcon-comment  "></text>
								<text class="text-gray text-df">{{item.commentCount}}</text>
							</view>
						</view>
					</view>
				</view>
			</block>
			<block v-for="(attention,index) in lookerList" :key="index" v-if="TabCur==1">
				<view class="actionLi bg-white">
					<view class="ali-main">
						<view class="texticon" v-if="index==0">
							<text class="lg cuIcon-crown"> </text>
						</view>
						<view class="textnum" v-if="index>0">
							{{index+1}}
						</view>
						<view class="ali-main-img" @tap="goUser(attention.lookUserId)">
							
							<image class='xd-mag xd-box-shadow' :src="attention.userHead"></image>
							
						</view>
						<view class="lli-main-content xd-list-body" @tap="goUser(attention.lookUserId)">
							<view class="xd-list-title-text">
								<text>{{attention.userName}}</text>
								<text v-if="attention.sex==1" class="boy">♂</text>
								<text v-else-if="attention.sex==0" class="boy">♀</text>
								<text v-else class="boy">密</text>
							</view>
							<view class="moreInfoIn">
								<image class='address' src="/static/images/icon/address.png"></image>
								<text class="province">{{attention.province}}.{{attention.city}}</text>
							</view>
						</view>
						<view class="ali-main-list" @tap="showBanner(attention.lookUserId,attention.pushId)">
							<view class="ali-main-list-num">{{attention.lookerCount}}</view>
						</view>
						<!-- v-if="userId==pushList.userId" -->
						<view class="defaultthank" @click="gothank(attention)" >
							<button class="defaultbut" type="default">感谢</button>
						</view>
						
					</view>
				</view>
			</block>
		</view>
		<backTop :scrollTop="scrollTop"></backTop>
		<!-- 开始行动-加号 -->
		<view class="start-add" @tap="goPage('/pages/action/step1')" v-if="scrollTop<2000">
			<image src="../../../static/images/icon/add.png" mode="widthFix"></image>
		</view>
	</view>
</template>

<script>
	import lookerCountInfo from "@/components/lookerCountInfo.vue"
	import{ mapState,mapMutations} from 'vuex'
	import backTop from "@/components/backTop.vue"
	export default {
		components:{
			lookerCountInfo,
			backTop
		},
		data() {
			return {
				TabCur: 0,
				pushComentList:[],
				pusCardList:[],
				pushList:{},
				modal: false,
				addrAnimation:'',
				audioPlaySrc:'../../../static/images/icon/img/titl.png',
				userId:uni.getStorageSync('id'),
				
				lookerList:[],
				looktotal:'',
				lookNextPageTwo:'',
				pushId:'',
				isShare:0,
				guanzhu:'关注',
				scrollTop:0,
				
				scrollTopinfo:true,
				
			};
		},
		onPageScroll(e) {
			this.scrollTop = e.scrollTop;
			// if(this.scrollTopinfo){
			// 	this.scrollTopinfo=false;
			// 	setTimeout(()=>{
			// 		this.scrollTop=0
			// 		this.scrollTopinfo=true;
			// 	},3000)
			// }
			
		},
		computed: {
		           ...mapState(['hasLogin'])  
		       },  
		onLoad(option) {
			//#ifdef MP-WEIXIN
			wx.showShareMenu({
			  menus: ['shareAppMessage', 'shareTimeline']
			})
			//#endif
			if(option.pushList==undefined){
				
				this.pushId=option.pushId;
				this.isShare=option.isopen?option.isopen:0;
				if(option.share!=undefined){
					try{												
					 uni.setStorageSync('share',option.share);
					}catch(e){
						console.log(Error)
					};
				}
				this.getpushList();
				this.getPushCardList();
				this.clickSaveShareInfo();
			}
			
		},
		/* watch: {
			hasLogin() {
				setTimeout(() => {
					this.clickSaveShareInfo();

				}, 100);
			},
		}, */
	
		onShareAppMessage(res) {
			let that = this;
			let tit=that.pushList.userId==that.userId? '第'+that.pushList.pushCardCishuCount+'次打卡:'+that.pusCardList[0].content:'我为@'+that.pushList.userName+'打Call：'+that.pusCardList[0].content;
			let path='/pages/index/action/action?pushId='+ that.pushList.id+'&share='+that.pushList.userId+'&isopen='+that.pushList.isopen;
			let img=that.pusCardList[0].pictures[0]?that.pusCardList[0].pictures[0]:'https://chucun2019.oss-cn-beijing.aliyuncs.com/dynamic/1595733463227.png';
			
		    let	tit2= that.pushList.userId==that.userId? '第'+that.pushList.pushCardCishuCount+'次打卡:'+that.pushList.content:'我为@'+that.pushList.userName+'打Call：'+that.pushList.content;
		    let	path2= '/pages/index/action/action?pushId='+ that.pushList.id+'&share='+that.pushList.userId+'&isopen='+that.pushList.isopen;
		    let	img2=that.pushList.pictures?that.pushList.pictures:'https://chucun2019.oss-cn-beijing.aliyuncs.com/dynamic/1595733463227.png';
			if(res.from=="menu"){
				
			if(that.pusCardList.length>0){
				that.setSaveShareInfo();
				return	that.xdUniUtils.xd_onShare(tit,path,img);
				
			}else{
				that.setSaveShareInfo();
				return	that.xdUniUtils.xd_onShare(tit2,path2,img2);	
			}
			}else{
				if(that.pusCardList.length>0){
					that.setSaveShareInfo();
					return	that.xdUniUtils.xd_onShare(tit,path,img);
					
				}else{
					that.setSaveShareInfo();
					return	that.xdUniUtils.xd_onShare(tit2,path2,img2);	
				}
			}		
		},
		//#ifdef MP-WEIXIN
		onShareTimeline(){
			let that = this;
			if(that.pusCardList.length>0){
				that.setSaveShareInfo();
				return {
					title: that.pushList.userId==that.userId? '第'+that.pushList.pushCardCishuCount+'次打卡:'+that.pusCardList[0].content:'我为@'+that.pushList.userName+'打Call：'+that.pusCardList[0].content,
					query: 'pushId='+ that.pushList.id+'&share='+that.pushList.userId+'&isopen='+that.pushList.isopen,
					imageUrl:that.pusCardList[0].pictures[0]?that.pusCardList[0].pictures[0]:'https://chucun2019.oss-cn-beijing.aliyuncs.com/dynamic/1595733463227.png',
				}
				
			}else{
				that.setSaveShareInfo();
				return {
					title: that.pushList.userId==that.userId? '第'+that.pushList.pushCardCishuCount+'次打卡:'+that.pushList.content:'我为@'+that.pushList.userName+'打Call：'+that.pushList.content,
					query: 'pushId='+ that.pushList.id+'&share='+that.pushList.userId+'&isopen='+that.pushList.isopen,
					imageUrl:that.pushList.pictures?that.pushList.pictures:'https://chucun2019.oss-cn-beijing.aliyuncs.com/dynamic/1595733463227.png',
				}
				
			}
		},
		//#endif
		methods:{
			gothank(e){
				if(!uni.getStorageSync('token')){
					uni.navigateTo({
						url: '../../login/login' 
					});
					return false
				}
				uni.navigateTo({
				url:'/pages/pageA/thankmoney/thankmoney?userId='+e.lookUserId+"&pushId="+e.pushId
			});
			},
			
			goPage(url){
				if(!this.hasLogin){
					uni.switchTab({
						url:'../index'
					})
					return false
				}
				uni.navigateTo({
					url
				});
			},
		    showBanner(lookUserId,pushId){
				this.$refs.lookerCountInfo.showBanner(lookUserId,pushId);
			},
			compareDate (d1, d2) {
				if(typeof d1 == 'undefined'){
					return true
				}
				if(typeof d2 == 'undefined'){
					return true
				}
				return this.xdUniUtils.xd_timestampToTime(d1.createTime,false,false,false) > this.xdUniUtils.xd_timestampToTime(d2.createTime,false,false,false)
			},
			gostep(){
				if(!uni.getStorageSync('token')){
					uni.switchTab({
						url:'../index'
					})
				}else{
					uni.showModal({
						 content: '是否要创建相同行动项',
						 confirmText: '新建',
						 confirmText:'继续',
						 success: (res) => {
						   if (res.confirm) {
							   uni.setStorageSync('pushData',this.pushList)
							 uni.navigateTo({
							 	url:'../../action/step1'
							 });
						   } else if (res.cancel) {
							 
						   }
						 }
					})
				}
				
			},
			tabSelect(e){
				this.TabCur=e.target.id;
				if(this.TabCur ==1){
					this.getLookerList()
				}
			},
			setSaveShareInfo(){
				this.xd_request_post(this.xdServerUrls.xd_saveShareInfo,{
					pushId:this.pushId,
					shareUserId:this.userId,
				},true
				   ).then(res => {
					  
					   })
			},
			clickSaveShareInfo(){
				if(uni.getStorageSync('share') != '' && this.userId != undefined){
					this.xd_request_post(this.xdServerUrls.xd_saveShareInfo,{
						pushId:this.pushId,
						shareUserId:uni.getStorageSync('share'),
						clickUserId:this.userId,
					},true
					   ).then(res => {
						  this.getpushList();
						   })
				}
			},
			goUser(e){
				if(!uni.getStorageSync('token')){
					uni.navigateTo({
						url: '../../login/login' 
					});
									return false
				}
				uni.navigateTo({
					url:'../../selfCenter/selfView?userId='+e
				})
			},
			//围观
			lookerClick:function(list,index){
				var that=this ;
				if(!uni.getStorageSync('token')){
					uni.navigateTo({
						url: '../../login/login' 
					});
									return false
				}
				that.userId=uni.getStorageSync('id');
				that.xd_request_post(that.xdServerUrls.xd_saveLooker,{
					
					pushId:list.id,
					lookUserId:that.userId,
				},true
				   ).then(res => {	
				
						   if(res.resultCode==0){
							   that.pushList.onlooker=true
							   that.pushList.onlookerCount++;
							  if(uni.getStorageSync(new Date().toLocaleDateString()+"dycwgKey") != 1){
								   uni.showModal({
										 content: that.xdCommon.gzsm_wgglts,
										 showCancel: false,
										 buttonText: '知道了',
										 success: (res) => {
										   if (res.confirm) {
											 uni.setStorageSync(new Date().toLocaleDateString()+'dycwgKey',1);
										   } else if (res.cancel) {
											 uni.setStorageSync(new Date().toLocaleDateString()+'dycwgKey',1);
										   }
										 }
									})
							  }else{
								  uni.showToast({
										title:'围观成功',
										 duration: 1000,
										 icon:'none',
								  }) 
							  }
						   }else{
							   uni.showToast({
								title:res.msg,
								 duration: 1000,
								 icon:'none',
							   })
						   }
						   
					
				})
			},
			gothanl(item){
				uni.showModal({
				    content: '感谢金设置',
					cancelText:'自定义',
					confirmText:'智能分配',
				    success: function (res) {
				        if (res.confirm) {
				
				        } else if (res.cancel) {
				            uni.navigateTo({
				            	url:'/pages/pageA/thankmoney/thankmoney?pushId='+item.id
				            });
				        }
				    },
				})
			},
			goSteps(){
				if(this.pushList.pushCardStatus==2||this.pushList.pushCardStatus==3){
					uni.showModal({
						 content: this.xdCommon.gzsm_clickCard,
						 confirmText: '新建',
						 success: (res) => {
						   if (res.confirm) {
							   uni.setStorageSync('pushData',this.pushList)
							 uni.navigateTo({
							 	url:'../../action/step1'
							 });
						   } else if (res.cancel) {
							 
						   }
						 }
					})
				}else{
					uni.navigateTo({
						url: '../../selfCenter/clockIn?pushId='+this.pushList.id
					});
				}
			},
			goPageImg(e,index){
				this.xdUniUtils.xd_showImg(e,index);
			},
			error: function() {
				this.audioPlaySrc=this.xdUniUtils.xd_randomImg();
			       }  ,
						
			gocardComentList(e,index){
				if(!uni.getStorageSync('token')){
					uni.navigateTo({
						url: '../../login/login' 
					});
									return false
				}
				uni.navigateTo({
					url: '../cardDetails/cardDetails?pushId='+e.pushId+'&cardId='+e.id+'&show='+index
				});
			},
			cardComentList(e){			
				var data=[];
				data.push(e);
				if(this.pushList.pushCardList.length<=0){
					var da={
						id:e.id,
						userId:e.userId
					};
					this.pushList.pushCardList.push(da);
				}
				
				var pushCard={
					pushList:this.pushList,
					pushCardList:data,
				};
				uni.navigateTo({
					url: '../cardDetails/cardDetails?pushCard='+encodeURIComponent(JSON.stringify(pushCard))
				});
			},
			timeStamp(res){
				let dataList=res.obj;
				for(var i=0;i <res.obj.length;i++){
				   var  time=this.xdUniUtils.xd_timestampToTime(res.obj[i].createTime,true);
					dataList[i].createTime=time;
				}
				return dataList;
			},
			openAddr() {
			// 	this.getPushComenList();
			
			//       this.modal=!this.modal;
			uni.navigateTo({
				url: '../cardDetails/cardDetails?pushList='+encodeURIComponent(JSON.stringify(this.pushList))
			});
			     
			  },
		async	getpushList(){
				if(this.isShare==1){
					if(!uni.getStorageSync('token')){
						uni.navigateTo({
							url: '../../login/login' 
						});
						return false
					}
				}
			  	this.xd_request_post(this.xdServerUrls.xd_pushDataByPushId,{
			  		pushId:this.pushId,
					isShare:this.isShare,
					lookUserId:uni.getStorageSync('id'),
			  	},true).then(res=>{	
					if(res.resultCode==0){
						var data=res.obj;
						data.createTime=this.xdUniUtils.xd_timestampToTime(res.obj.createTime)
						data.endTime=this.xdUniUtils.xd_timestampToTime(res.obj.endTime)
						data.challengeRmb=res.obj.challengeRmb/100;
						this.pushList=data;
						if(this.pushList.userId == uni.getStorageSync('id')){
							this.guanzhu =''
						}else{
							this.xd_request_post(this.xdServerUrls.xd_iSAttention,{
								userId:uni.getStorageSync('id'),
								attentionUserId:this.pushList.userId,
							},true)
							.then(res=>{
								if(res.obj){
									this.guanzhu ='已关注'
								}else{
									this.guanzhu ='关注'
								}
								
							})
						}
					}else{
						uni.showToast({
							title:res.msg,
							icon:'none',
						})
					}
					
			  	})
			  },
			goStep(){
				uni.navigateTo({
					url: `/pages/action/step1`
				});
			},
			getPushComenList(){
				this.xd_request_post(this.xdServerUrls.xd_showCommentAndReplayCommtent,{
					pushId:this.pushId,
						token:uni.getStorageSync('token')
				},false).then(res=>{
					
					this.pushComentList=this.timeStamp(res);
				})
			},
		async	getPushCardList(){
				this.xd_request_post(this.xdServerUrls.xd_pushCardListByPushId,{
					pushId:this.pushId,		
					
				},true).then(res=>{
					var data=res.obj.list;
					for(let i=0;i<res.obj.list.length;i++){
						data[i].createTime=this.xdUniUtils.xd_timestampToTime(data[i].createTime,false,false,false);
						if(res.obj.list[i].pictures!=""){
							data[i].pictures=res.obj.list[i].pictures.split(',')
						}
					}
					this.pusCardList=data;
				})
			},
			// 打卡
			cardFn: function () {
				this.active = 0;
			},
			// 详情
			detailFn : function(){
				this.active = 1;
			},
			// 围观的人
			lookFn : function(){
				this.active = 2;
				this.getLookerList();
			},
			tags(){
				if(!uni.getStorageSync('token')){
					uni.navigateTo({
						url: '../../login/login' 
					});
									return false
				}
				if(this.guanzhu =='已关注'){
					return
				}
				this.xd_request_post(this.xdServerUrls.xd_saveAttention,{
					userId:uni.getStorageSync('id'),
					attentionUserId:this.pushList.userId,		
					
				},true).then(res=>{
					if(res.resultCode == 0){
						 this.guanzhu="已关注"
						 uni.showToast({
						 	icon:'none',
						   title: '关注成功',
						 })
					}else{
						uni.showToast({
							icon:'none',
						  title: res.msg,
						})
					}
				})
			},
			goTOSponsor(index,pushId){
				console.log('goTOSponsor',index,pushId);
				uni.setStorageSync("pushId",pushId);
				uni.navigateTo({
					url:'../../sponsor/form'
				})
			},
			getLookerList(){
				this.xd_request_post(this.xdServerUrls.xd_getLookerByPushId,{
					pushId:this.pushId,
					pageNum: this.lookNextPageTwo,
					pageSize:10,
				},true)
				.then(res=>{
					this.lookerList=res.obj.list;
					this.looktotal=res.obj.total;
					this.lookNextPageTwo=res.obj.nextPage;
					this.lookerList.forEach(item =>{
						if(item.lookUserId == uni.getStorageSync('id')){
							this.guanzhu ='已关注'
						}
					})
					
				})
			}
		}
	}
	
</script>
<style scoped lang="scss">
	page{background: #fcfcfc;}
	.imgheit{
		height: 320upx;
		width: 100%;
	}
	.contentext{
		
	}
	.cu-timeline .cu-time{
		width: 160rpx;
	}
	.commentCount{
		right: 0;
	}
	.actionLi{
		padding-bottom: 10upx;
	}
	.ali-main{
		display: flex;
		padding: 20rpx;
		.ali-main-list{
			color: #f37b1d;
			margin-left: 10%;
			line-height: 130rpx;
			width: 140rpx;
			.ali-main-list-num{
				margin-left: 40%;
			}
		}
		.ali-main-img .xd-mag{
			border-radius: 100%;
			height: 125rpx;
			width: 125rpx;
		}
		.lli-main-content {
			.boy{
				background:#66CCFF;
				color:#fff;
				display: inline-block;
				padding:0 12upx;
				border-radius: 100%;
				font-size: 25rpx;
				
				margin-left: 14rpx;
			}
			.lli-main-content-text{
				line-height: 90rpx;
				margin-right: 20rpx;
			}
			.moreInfoIn {
				.address {
					width: 30rpx;
					height: 30rpx;
				}
			
				.province {
					font-size: 28rpx;
					margin-left: 6rpx;
				}
			}
		}
	}

	.textcon{
		overflow: hidden;
		text-overflow:ellipsis;
		display:-webkit-box; 
		-webkit-box-orient:vertical;
		-webkit-line-clamp:2; ; 
	}

	.bg-light-blue{background-color: #007AFF;}
	
	.videos{
		width: 500upx;
		height: 325upx;
	}
	.widthtext{
		width: 100%;
	}
	.texticon{
		display: inline-flex;
		position: absolute;
		color: #f37b1d;
		font-size: 35upx;
		white-space: nowrap;
		-webkit-transform:rotate(-50deg);
		z-index: 10;

	}
	.textnum{
		display: inline-flex;
		position: absolute;
		color: #FFFFFF;
		background-color:#f37b1d;
		white-space: nowrap;
		z-index: 10;
		border-radius: 200rpx;
		padding: 0rpx 10rpx;
		height: 28rpx;
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
	.defaultthank{
		padding-top: 30upx;
	}
	.defaultbut{
		height: 60upx;
		line-height: 1.9;
	}

</style>
