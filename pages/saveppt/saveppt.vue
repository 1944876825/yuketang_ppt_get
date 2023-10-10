<template>
	<view class="flex-col content">
		<view class="header">
			<!-- PPT列表 -->
			<view class="">
				<view
					class="flex-col" style="padding: 15rpx 20rpx;"
					v-if="isShowPPT">
					<view class="ykt_bgc flex-row justify-between"
						style="padding: 15rpx 20rpx;border-radius: 20rpx;color: #fff;margin-bottom: 15rpx;"
						v-for="ppt in pptList"
						@tap="toPPT(ppt)">
						<view class="flex-col justify-center items-center">
							<view class="ctitle line_1" style="padding: 10rpx;max-width: 400rpx;">
								<text style="font-size: 18px;font-weight: 800;">{{ppt['courseName']}}</text>
							</view>
							<view class="ctitle line_1" style="padding: 10rpx;max-width: 400rpx;">
								<text style="font-size: 14px;font-weight: 800;">{{ppt['title']}}</text>
							</view>
							<view class="ctitle line_1" style="padding: 10rpx;max-width: 400rpx;">
								<text style="font-size: 13px;font-weight: 600;">{{ppt['time']}}</text>
							</view>
						</view>
						<view class="flex-col justify-center" style="padding: 20rpx;">
							<text style="font-size: 15px;font-weight: 600;">点击查看</text>
						</view>
					</view>
				</view>
				<view v-else class="tips">
					<text>你没有保存PPT哦 ~~</text>
				</view>
			</view>
		</view>
		<view class="center">
			
		</view>
		
		<view class="bottom">
			
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				type: 0,
				lessonId: '',
				pptList: [],
				isShowPPT: false,
			};
		},
		onShow() {
			this.getHistory()
		},
		onPullDownRefresh() {
			this.getHistory()
			setTimeout(function() {
				uni.stopPullDownRefresh();
			}, 500);
		},
		methods: {
			getHistory() {
				uni.getStorage({
					key: 'pptList',
					success:res=>{
						this.pptList = []
						var ks = Object.keys(res.data)
						if (ks && ks['length'] > 0) {
							ks.reverse()
							ks.forEach(e=>{
								this.pptList.push(res.data[e])
							})
							this.isShowPPT = true
						} else {
							this.isShowPPT = false
						}
					},
					fail:err=>{
						this.pptList = []
						this.isShowPPT = false
					}
				})
			},
			toPPT(ppt) {
				uni.navigateTo({
					url: '/pages/detail/detail?courseName='+ ppt['courseName'] +'&source='+ ppt['source'] + '&lessonId=' + ppt['lessonId'] + '&type=local' + '&title=' + ppt['title']
				})
			},
		}
	}
</script>

<style lang="scss" scoped>
	.tips {
		text-align: center;
		color: $yuKeTang;
		font-size: 14px;
		font-weight: 600;
		margin-top: 100rpx;
	}
</style>
