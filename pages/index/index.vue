<template>
	<view class="content">
		<view v-if="isOver" class="msg-over">没有更多消息了</view>
		<view class="pull-content" v-for="(item,index) in msgList" :key="index" :class="index % 2 == 0 ? 'msg-right' : ''" :id="'msg'+item.id">
			<view class="absolute-view" v-show="showIndex>index?'show-index':''"></view>
			<view class="img-box" v-if="item.type == 'img'">
				{{item.id}}<image  class="logo" src="/static/logo.png"></image>
			</view>
			<view v-else class="left-msg">
				{{item.id}}{{item.content}}
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				msgList:[],
				listPage:0,
				showIndex:-1,
				isOver:false
			}
		},
		onLoad() {
			let _this = this;
			_this.addList();  //产生十条消息
		},
		methods: {
			addList(){
				let _this = this;
				let type = null;
				for(let i = 0; i <10;i++){
					if(i % 3 == 1){            //消息列表为图片，增加图片消息，增加页面视觉辨识，如果不做处理，在滚动到下拉之前位置时，尤其在低端机上，是可以看见页面图片闪一下
						type = 'img'		   //文字消息不是很闪
					}else{
						type = 'msg'           //消息列表为文字消息 ，view循环渲染中判断
					}
					let msgObj={               //一条消息对象
						id:_this.listPage * 10 +i,
						content:'你有一条新的消息,请注意查收',
						type:type
					}
					_this.msgList.unshift(msgObj);  //插入列表
				}
				if(_this.listPage == 0){            //首次滚动到底部
					_this.$nextTick(function() {
						uni.pageScrollTo({
							scrollTop: 99999,
							duration: 0
						});
					});
					return;
				}
				if(_this.listPage>3){               //模拟消息取完了，拿出来40条消息
					_this.isOver = true; 			// 标记已无更多消息
					_this.endOver();     			// 设置禁用下拉加载
					return;
				}
				//最关键一步，在数据插入列表，渲染之后，没有滚动到下拉之前位置时，采用相对定位，设置同背景色遮罩层盖住消息
				//设置为9意思是，每次拿十条数据，遮住前十条，用数组循环下标判断从0开始，初始化为-1
				_this.showIndex = 9;                
				let scrollMsg = _this.msgList[9];  //数据合并之后，获取10条消息的元素
				_this.$nextTick(function() {
					let view = uni.createSelectorQuery().select('#msg'+scrollMsg.id);
					view.boundingClientRect(res => {
						console.log('#msg'+scrollMsg.id+'距离顶部'+res.top); //获取10条消息的元素距顶部的距离，这就是需要滚动的距离
						uni.pageScrollTo({         //回到下拉之前，多展示一条消息
							scrollTop: res.top,
							duration: 0
						});
						_this.showIndex = -1;      //去掉遮罩层
						uni.stopPullDownRefresh(); //结束下拉加载
					}).exec();
				});
			},
			endOver(){  //设置禁用下拉加载
				const pages = getCurrentPages();  
				const page = pages[pages.length - 1];  
				const currentWebview = page.$getAppWebview();
				currentWebview.setStyle({  
				  pullToRefresh: {  
				    support: false,  
				    style: plus.os.name === 'Android' ? 'circle' : 'default'  
				  }  
				});  
				console.log('没有更多数据了，禁用');
			},
		},
		onPullDownRefresh() {  //下拉加载
			let _this = this;
			console.log('refresh');
			_this.listPage ++;
			_this.addList();
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 30rpx;
		background-color: #FFFFFF;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin: 30rpx 0;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}
	.pull-content{
		position: relative;
		width: 100%;
		display: flex;
		align-items: center;
		margin: 30rpx 0;
	}
	.left-msg{
		width: 300rpx;
		border-radius: 10rpx;
		background-color: #9eea6b;
		color: #333;
		font-size: 30rpx;
		padding:20rpx;
	}
	.msg-right{
		justify-content: flex-end;
	}
	.img-box{
		display: flex;
		align-items: center;
		font-size: 28rpx;
	}
	.absolute-view{
		position: absolute;
		top: 0;
		left: 0;
		z-index: 10;
		width: 100%;
		height: 100%;
		background-color: #FFFFFF;
	}
	.show-index{
		display: none;
	}
	.msg-over{
		font-size: 24rpx;
		text-align: center;
		color: #999999;
	}
</style>
