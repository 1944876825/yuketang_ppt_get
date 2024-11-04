<template>
	<view class="flex-col" style="padding: 20rpx;">
		<view class="flex-col copyable-text" style="font-size: 28rpx;line-height: 50rpx;">
			<span style="margin-top: 15rpx;">作者：BY易仝、流念</span>
			<span style="margin-top: 15rpx;">初衷：方便获取、下载雨课堂的PPT</span>
			<span style="margin-top: 15rpx;">Q群：1093968466</span>
			<span style="margin-top: 15rpx;">版本：v1.1.1</span>
			<span style="margin-top: 15rpx;">源代码：</span>
			<span>https://github.com/1944876825/yuketang_ppt_get</span>
			<span style="margin-top: 15rpx;"></span>
			<span>更新或分享：</span>
			<span>https://jmwl.lanzoub.com/b08o52p2h 密码：c79m</span>
		</view>
		
		<view class="flex-row" style="margin-top: 20rpx;">
			<view 
				@tap="toLogin"
				class="ykt_bgc" style="font-size: 24rpx;padding: 10rpx 20rpx;border-radius: 50rpx;color: #fff;">
				切换账号
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				
			};
		},
		methods: {
			toLogin() {
				uni.navigateTo({
					url: '/pages/login/login'
				})
			}
		}
	}
</script>

<style lang="scss">

</style>
