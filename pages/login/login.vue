<template>
	<view class="flex-col">
		<view class="ykt_color" style="margin: 20rpx;">
			<span style="font-size: 28rpx;font-weight: bold;">Cookie</span>
			<input v-model="cookie" type="text" placeholder="扫码自动获取或手动填写 sessionid=xxxx;" style="background-color: #fff;font-size: 26rpx;padding: 20rpx;margin: 10rpx 0;border-radius: 8rpx;">
			
			<view class="flex-row" style="margin-top: 20rpx;">
				<view 
					@tap="openPopup"
					class="ykt_bgc" style="font-size: 24rpx;padding: 8rpx 20rpx;color: #fff;border-radius: 50rpx;">
					<span>扫码</span>
				</view>
				<view 
					@tap="showHelp"
					class="" style="font-size: 24rpx;padding: 8rpx 20rpx;color: #fff;border-radius: 50rpx;background-color: #f3a73f;margin-left: 10rpx;">
					<span>手动</span>
				</view>
			</view>
			<button class="ykt_bgc" @click="saveCk" style="color: #fff;font-size: 26rpx;margin-top: 20rpx;padding: 5rpx 0;">登录</button>
		</view>
		
		<uni-popup ref="popup" type="bottom">
			<view class="flex-col items-center" style="background-color: #fff;padding-bottom: 200rpx;padding-top: 20rpx;">
				<img style="width: 400rpx;height: 400rpx;margin: 20rpx;" :src="ticket" alt="" @tap="refresh_code">
				<view class="timeTips">
					请在60s内使用微信扫码（过期可点击二维码刷新）
				</view>
			</view>
		</uni-popup>
	</view>
</template>

<script>
import loginVue from './login.vue';
	export default {
		data() {
			return {
				websocket: {},
				ticket: '',
				cookie: uni.getStorageSync('yktcookie') || '',
			};
		},
		onLoad() {
			
		},
		methods: {
			showHelp() {
				uni.previewImage({
					urls: ['../../static/image/catck.png'],
				})
			},
			openPopup() {
				this.$refs.popup.open('bottom')
				if (this.websocket && this.websocket['readyState'] && this.websocket.readyState===1) {
					
				} else {
					this.initWebsocket()
				}
			},
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
							content: '连接雨课堂服务器失败，请下拉刷新或重启软件'
						})
					}
				});
				// 建立websocket连接
				this.websocket.onOpen((res) => {
			        // 注：只有连接正常打开中 ，才能正常成功发送消息
					this.refresh_code()
			        // 注：只有连接正常打开中 ，才能正常收到消息
			        this.websocket.onMessage((res) => {
						if (String(res.data).indexOf('reached the limit') != -1) {
							uni.showToast({
								icon: 'none',
								title: '获取二维码频繁'
							})
							return
						}
						this.ticket = ''
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
						if (pres && pres[0]) {
							this.cookie = pres[0];
							this.$refs.popup.close()
							uni.showToast({
								icon: 'none',
								title: '获取成功'
							})
						} else {
							uni.showModal({
								title: '提示',
								content: '登录失败'
							})
						}
					}
				})
			},
			saveCk() {
				uni.setStorage({
					key: 'yktcookie',
					data: this.cookie,
					success:res=>{
						uni.showToast({
							icon:'none',
							title:'登录成功'
						})
						getApp().globalData.Cookie = this.cookie
						uni.switchTab({
							url: '/pages/index/index'
						})
					},
					fail:err=>{
						uni.showModal({
							title: '提示',
							content: '配置写入本地失败'
						})
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
</style>
