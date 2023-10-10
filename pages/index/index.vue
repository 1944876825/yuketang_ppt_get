<template>
	<view class="flex-col content">
		<view v-if="isLogin" class="" style="text-align: center;">
			<img style="width: 400rpx;height: 400rpx;margin: 20rpx;" :src="ticket" alt="" @tap="refresh_code">
			<view class="timeTips">
				请在60s内使用微信扫码（过期可点击二维码刷新）
			</view>
			<view class="banquan">
				雨课堂PPT获取工具
				<br />
				Q群：1093968466
				<br />
				By：流念、易仝
			</view>
		</view>
		
		<view v-else class="center">
			<view class="flex-col" style="padding: 15rpx 20rpx;" v-if="onClassList.length > 0">
				<view class="ykt_bgc flex-row justify-between"
					style="padding: 15rpx 20rpx;border-radius: 20rpx;color: #fff;margin-bottom: 15rpx;"
					v-for="onClass in onClassList"
					@tap="detail(onClass)">
					<view class="flex-col justify-center items-center" style="">
						<view class="ctitle line_1" style="padding: 10rpx;max-width: 400rpx;">
							<text style="font-size: 18px;font-weight: 800;">{{onClass['courseName']}}</text>
						</view>
						<view style="padding: 10rpx;">
							<image style="width: 80rpx;height: 80rpx;border-radius: 40rpx;" :src="onClass['teacherAvatar']" mode=""></image>
						</view>
					</view>
					<view class="flex-col justify-center" style="padding: 20rpx;">
						<text style="font-size: 15px;font-weight: 600;">点击查看</text>
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
				ticket: '',
				isLogin: true,
				onClassList: [],
			};
		},
		onLoad() {
			uni.getStorage({
				key: 'yktcookie',
				success:res=>{
					this.isLogin = false
					uni.setNavigationBarTitle({
						title: '一些课程'
					})
					getApp().globalData.Cookie = res.data
					this.getOnClass()
				},
				fail:err=>{
					this.isLogin = true
					this.initWebsocket()
				}
			})
		},
		async onPullDownRefresh() {
			this.getOnClass();
			setTimeout(function() {
				uni.stopPullDownRefresh();
			}, 500);
		},
		methods: {
			refresh_code() {
				var data = {
					"op": "requestlogin",
					"role": "web",
					"version": 1.4,
					"type": "qrcode",
					"from": "web"
				}
				this.websocket.send({
					data: JSON.stringify(data),
				});
			},
			getOnClass() {
				uni.request({
					url: 'https://www.yuketang.cn/api/v3/classroom/on-lesson',
					header: {
						'cookie': getApp().globalData.Cookie,
					},
					dataType: 'json',
					success:res=>{
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
								this.isLogin = true
								this.initWebsocket()
								uni.setNavigationBarTitle({
									title: '一些配置'
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
				uni.navigateTo({
					url: '/pages/detail/detail?courseName='+ ppt['courseName'] +'&source='+ ppt['role'] + '&lessonId=' + ppt['lessonId']
				})
			},
			initWebsocket() {
				// 创建websocket
				this.websocket = uni.connectSocket({
			        url: 'wss://www.yuketang.cn/wsapp/',
					success:res=> {
						console.log('连接成功')
					},
					fail:err=>{
						uni.showModal({
							title: '提示',
							content: '连接雨课堂失败，请下拉刷新或重启软件'
						})
					}
				});
				// 建立websocket连接
				this.websocket.onOpen((res) => {
			        // 注：只有连接正常打开中 ，才能正常成功发送消息
					this.refresh_code()
			        // 注：只有连接正常打开中 ，才能正常收到消息
			        this.websocket.onMessage((res) => {
						var json = JSON.parse(res.data)
						if (json['ticket']) {
							this.ticket = json['ticket']
						}
						if (json['Auth']) {
							this.websocket.close()
							this.login(json)
						}
			        });
				});
				// 监听webscket关闭的事件
				this.websocket.onClose(() => {
			        console.log("已经被关闭了");
				});
				// 监听发生错误时的时间
			    this.websocket.onError(() => {
			        console.log('连接雨课堂中断，如有异常，请下拉刷新或重启软件');
				})
			},
			login(userInfo) {
				uni.request({
					url: 'https://www.yuketang.cn/pc/web_login',
					method:'POST',
					data: {
						"UserID": userInfo['UserID'],
						"Auth": userInfo['Auth'],
					},
					success:res=>{
						const regex = /sessionid=.*?;/gm
						var headerTxt = JSON.stringify(res.header)
						var pres = headerTxt.match(regex)
						if (pres[0]) {
							uni.setStorage({
								key: 'yktcookie',
								data: pres[0],
								success:res=>{
									uni.showToast({
										icon:'none',
										title:'登录成功'
									})
									uni.setNavigationBarTitle({
										title: '一些课程'
									})
									getApp().globalData.Cookie = pres[0]
									this.isLogin = false
									this.getOnClass()
								},
								fail:err=>{
									uni.showModal({
										title: '提示',
										content: '配置写入本地失败'
									})
								}
							})
							
						} else {
							uni.showModal({
								title: '提示',
								content: '登录失败'
							})
						}
					}
				})
			}
		}
	}
</script>

<style lang="scss" scoped>
	.timeTips {
		color: $yuKeTang;
		font-size: 13px;
	}
	.banquan {
		width: 100%;
		padding: 30rpx 0;
		text-align: center;
		position: fixed;
		bottom: 90rpx;
		color: $yuKeTang;
		font-size: 13px;
	}
	.tips {
		text-align: center;
		color: $yuKeTang;
		font-size: 14px;
		font-weight: 600;
		margin-top: 100rpx;
	}
</style>
