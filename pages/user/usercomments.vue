<template>
	<view class="user">
		<view class="header" :style="[{height:CustomBar + 'px'}]">
			<view class="cu-bar bg-white" :style="{'height': CustomBar + 'px','padding-top':StatusBar + 'px'}">
				<view class="action" @tap="back">
					<text class="cuIcon-back"></text>
				</view>
				<view class="content text-bold" :style="[{top:StatusBar + 'px'}]">
					我的评论
				</view>
				<view class="action">
					
				</view>
			</view>
		</view>
		<view :style="[{padding:NavBar + 'px 10px 0px 10px'}]"></view>
		
		<view class="cu-card dynamic no-card" style="margin-top: 20upx;">
			
			<view class="cu-item">
				<view class="cu-list menu-avatar comment">
					<view class="no-data" v-if="commentsList.length==0">
						暂时没有评论
					</view>
					<view class="cu-card dynamic no-card" style="margin-top: 20upx;">
						<view class="cu-item" v-for="(item,index) in commentsList" :key="index" v-if="commentsList.length>0">
							<view class="cu-list menu-avatar comment">
								<view class="cu-item">
									<view class="cu-avatar round" :style="item.style"></view>
									<view class="content">
										<view class="text-grey">{{item.author}}</view>
										<view class="text-content text-df break-all">
											<rich-text :nodes="markHtml(item.text)"></rich-text>
										</view>
										<view class="bg-grey light padding-sm radius margin-top-sm  text-sm">
											<view class="flex" @tap="toInfo(item.cid,item.contenTitle)">
												<view class="break-all">{{item.contenTitle}}</view>
												
											</view>
										</view>
										<view class="margin-top-sm flex justify-between">
											<view class="text-gray text-df">{{formatDate(item.created)}}</view>
											<view>
												<text class="cuIcon-messagefill text-gray margin-left-sm" @tap="commentsAdd(item.author+'：'+item.text,item.coid,1)"></text>
											</view>
										</view>
									</view>
								</view>
					
								
							</view>
						</view>
					</view>
					
					<view class="load-more" @tap="loadMore" v-if="commentsList.length>0">
						<text>{{moreText}}</text>
					</view>
				</view>
			</view>
		</view>
		<!--加载遮罩-->
		<view class="loading" v-if="isLoading==0">
			<view class="loading-main">
				<image src="../../static/loading.gif"></image>
			</view>
		</view>
		<!--加载遮罩结束-->
	</view>
</template>

<script>
	import { localStorage } from '../../js_sdk/mp-storage/mp-storage/index.js'
	var API = require('../../utils/api')
	var Net = require('../../utils/net')
	// #ifdef APP-PLUS
	import owo from '../../static/app-plus/owo/OwO.js'
	// #endif
	// #ifdef H5
	import owo from '../../static/h5/owo/OwO.js'
	// #endif
	// #ifdef MP
	var owo = [];
	// #endif
	export default {
		data() {
			return {
				StatusBar: this.StatusBar,
				CustomBar: this.CustomBar,
				NavBar:this.StatusBar +  this.CustomBar,
				
				commentsList:[],
				
				moreText:"加载更多",
				page:1,
				
				isLoading:0,
				
				owo:owo,
				owoList:[],
				
			}
		},
		onPullDownRefresh(){
			var that = this;
			
		},
		onReachBottom() {
		    //触底后执行的方法，比如无限加载之类的
			var that = this;
			that.loadMore();
		},
		onShow(){
			var that = this;
			// #ifdef APP-PLUS
			
			plus.navigator.setStatusBarStyle("dark")
			// #endif
			that.page=1;
		},
		onLoad() {
			var that = this;
			// #ifdef APP-PLUS || MP
			that.NavBar = this.CustomBar;
			// #endif
			// #ifdef APP-PLUS || H5
			var owo = that.owo.data;
			var owoList=[];
			for(var i in owo){
				owoList = owoList.concat(owo[i].container);
			}
			that.owoList = owoList;
			// #endif
			that.getCommentsList(false);
		},
		methods:{
			back(){
				uni.navigateBack({
					delta: 1
				});
			},
			loadMore(){
				var that = this;
				that.moreText="正在加载中...";
				if(that.isLoad==0){
					that.getCommentsList(true);
				}
			},
			markHtml(text){
				var that = this;
				var owoList=that.owoList;
				for(var i in owoList){
				
					if(that.replaceSpecialChar(text).indexOf(owoList[i].data) != -1){
						text = that.replaceAll(that.replaceSpecialChar(text),owoList[i].data,"<img src='/"+owoList[i].icon+"' class='tImg' />")
						
					}
				}
				return text;
			},
			replaceAll(string, search, replace) {
			  return string.split(search).join(replace);
			},
			toInfo(cid,title){
				var that = this;
				
				uni.navigateTo({
				    url: '/pages/contents/info?cid='+cid+"&title="+title
				});
			},
			getCommentsList(isPage){
				var that = this;
				var userInfo = null;
				if(localStorage.getItem('userinfo')){
					
					var userInfo = JSON.parse(localStorage.getItem('userinfo'));
				}else{
					uni.showToast({
						title: "请先登录哦",
						icon: 'none'
					})
					return false;
				}
				var data = {
					"type":"comment",
					"authorId":userInfo.uid
				}
				var page = that.page;
				if(isPage){
					page++;
				}
				Net.request({
					url: API.getCommentsList(),
					data:{
						"searchParams":JSON.stringify(API.removeObjectEmptyKey(data)),
						"limit":5,
						"page":page,
					},
					header:{
						'Content-Type':'application/x-www-form-urlencoded'
					},
					method: "get",
					dataType: 'json',
					success: function(res) {
						that.isLoad=0;
						if(res.data.code==1){
							var list = res.data.data;
							if(list.length>0){
								var commentsList = [];
								for(var i in list){
									var arr = list[i];
									arr.style = "background-image:url("+list[i].avatar+");"
									commentsList.push(arr);
								}
								if(isPage){
									that.page++;
									that.commentsList = that.commentsList.concat(commentsList);
								}else{
									that.commentsList = commentsList;
								}
							}else{
								that.moreText="没有更多评论了";
							}
							
						}
						var timer = setTimeout(function() {
							that.isLoading=1;
							clearTimeout('timer')
						}, 300)
					},
					fail: function(res) {
						that.isLoad=0;
						that.moreText="加载更多";
						var timer = setTimeout(function() {
							that.isLoading=1;
							clearTimeout('timer')
						}, 300)
					}
				})
			},
			commentsAdd(title,coid,reply){
				var that = this;
				var cid = that.cid;
				uni.navigateTo({
				    url: '/pages/contents/commentsadd?cid='+cid+"&coid="+coid+"&title="+title+"&isreply="+reply
				});
			},
			
			formatDate(datetime) {
				var datetime = new Date(parseInt(datetime * 1000));
				// 获取年月日时分秒值  slice(-2)过滤掉大于10日期前面的0
				var year = datetime.getFullYear(),
					month = ("0" + (datetime.getMonth() + 1)).slice(-2),
					date = ("0" + datetime.getDate()).slice(-2),
					hour = ("0" + datetime.getHours()).slice(-2),
					minute = ("0" + datetime.getMinutes()).slice(-2);
				//second = ("0" + date.getSeconds()).slice(-2);
				// 拼接
				var result = year + "-" + month + "-" + date + " " + hour + ":" + minute;
				// 返回
				return result;
			},
			replaceSpecialChar(text) {
				if(!text){
					return false;
				}
				text = text.replace(/&quot;/g, '"');
				text = text.replace(/&amp;/g, '&');
				text = text.replace(/&lt;/g, '<');
				text = text.replace(/&gt;/g, '>');
				text = text.replace(/&nbsp;/g, ' ');
				return text;
			}
		}
	}
</script>

<style>
@import "../../static/base.css";
</style>
