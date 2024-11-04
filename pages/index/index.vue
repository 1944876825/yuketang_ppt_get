<template>
	<view class="flex-col">
		<view>
			<view class="flex-col" style="padding: 15rpx 20rpx;" v-if="onClassList.length > 0">
				<view class="ykt_bgc flex-row justify-between"
					style="padding: 15rpx 20rpx;border-radius: 20rpx;color: #fff;margin-bottom: 15rpx;"
					v-for="onClass in onClassList" @tap="detail(onClass)">
					<view class="flex-col justify-center items-center" style="">
						<view class="ctitle line_1" style="padding: 10rpx;max-width: 400rpx;">
							<text style="font-size: 30rpx;font-weight: 800;">{{onClass['courseName']}}</text>
						</view>
						<view style="padding: 10rpx;">
							<image style="width: 100rpx;height: 100rpx;border-radius: 50rpx;"
								:src="onClass['teacherAvatar']" mode=""></image>
						</view>
					</view>
					<view class="flex-col justify-center" style="padding: 20rpx;">
						<text style="font-size: 28rpx;font-weight: 600;">点击查看</text>
					</view>
				</view>
			</view>
			<view v-else class="tips">
				<text>你现在没有课哦 ~~</text>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				onClassList: [],
			};
		},
		onLoad() {
			this.Init()
		},
		onPullDownRefresh() {
			this.onClassList = []
			this.Init()
			setTimeout(function() {
				uni.stopPullDownRefresh()
			}, 500);
		},
		methods: {
			Init() {
				uni.setNavigationBarTitle({
					title: '一些课程'
				})
				this.getCookie()
			},
			getCookie() {
				uni.getStorage({
					key: 'yktcookie',
					success: res => {
						getApp().globalData.Cookie = res.data
						this.getOnClass()
					},
					fail: err => {
						uni.navigateTo({
							url: '/pages/login/login'
						})
					}
				})
			},
			getOnClass() {
				uni.request({
					url: 'https://www.yuketang.cn/api/v3/classroom/on-lesson',
					header: {
						'cookie': getApp().globalData.Cookie,
					},
					dataType: 'json',
					success: res => {
						if (res.data.code == 0) {
							if (res.data['data']['onLessonClassrooms']['length'] > 0) {
								this.onClassList = res.data.data.onLessonClassrooms
							} else {
								uni.showToast({
									icon: 'none',
									title: '你没课啦！'
								})
							}
						} else {
							if (res.data['msg'] == 'UNAUTHENTICATED') {
								uni.showToast({
									icon: 'none',
									title: 'Cookie失效'
								})
								uni.navigateTo({
									url: '/pages/login/login'
								})
							} else {
								uni.showToast({
									icon: 'none',
									title: res.data.msg
								})
							}
						}
					}
				})
			},
			detail(ppt) {
				uni.setStorage({
					key: 'current_class',
					data: ppt,
					success: res => {
						uni.navigateTo({
							url: '/pages/detail/detail'
						})
					},
					fail: err => {
						uni.showToast({
							icon: 'none',
							title: '缓存储存失败'
						})
					}
				})
			}
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