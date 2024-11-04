<template>
	<view class="content flex-col">
		<view class="header" style="height: 140rpx;">
			<view style="position: fixed;top: 0;width: 100%;z-index: 1;background-color: #fff;">
				<view class="">
					<scroll-view style="white-space: nowrap;" :scroll-x="true" :show-scrollbar="false" scroll-with-animation>
						<view class="options">
							<view 
								v-for="(title, index) in slideTitleList"
								class="obox"
								:class="slideIndex == index ? 'actobox' : ''"
								@tap="chooseSlideIndex(index)">
								<text style="">{{ title }}</text>
							</view>
						</view>
					</scroll-view>
				</view>
				<view class="options">
					<view class="obox" :class="dqxz == 'ALL' ? 'actobox' : ''" @tap="choptions('ALL')">
						ALL
					</view>
					<view class="obox" :class="dqxz == 'PPT' ? 'actobox' : ''" @tap="choptions('PPT')">
						PPT
					</view>
					<view class="obox" :class="dqxz == 'Question' ? 'actobox' : ''" @tap="choptions('Question')">
						Question
					</view>
					<view class="obox" :class="isSave == true ? 'actobox' : ''" @tap="save()">
						{{isSave==false?'SAVE':'UNSAVE'}}
					</view>
					<view class="obox" @tap="generatePDF()">
						PDF
					</view>
				</view>
			</view>
		</view>
		
		<view class="center">
			<block v-for="(slide, index) in slideList">
				<view class="pptBox" v-if="dqxz=='ALL' || (slide['problem'] && dqxz=='Question') || (!slide['problem'] && dqxz=='PPT')">
					<image :src="slide['cover']" mode="aspectFit" @click="imgShow(index)"></image>
					<block v-if="slide['problem'] && slide['problem']['answers'] && slide['problem']['answers']['length']>0">
						<text class="answer" style="font-size: 16px;font-weight: 800;">答案：{{slide['problem']['answers'].join(' ')}}</text>
					</block>
					<text style="font-size: 13px;font-weight: 600;margin-top: 10rpx;">第{{slide['index']}}张</text>
				</view>
			</block>
			<view class="tips">
				{{ jztips }}
			</view>
		</view>
	</view>
</template>

<script>
import loginVue from '../login/login.vue';
import PDFDocument from 'pdfkit';
import blobStream from 'blob-stream';

	export default {
		data() {
			return {
				type: '',
				source: '',
				lessonId: '',
				presentation_id_list: [],
				imgList: [],
				authHeader: '',
				websocket: {},
				userid: '',
				wssAuth: '',
				dqxz: 'ALL',
				isSave: false,
				courseName: '',
				pptList: {},
				baseInfo: {},// 当前课程信息
				isShowPPT: false,// 是否显示PPT
				
				slideIndex: 0,
				slideTitleList: [],
				slideList: [],
				// currentSlideList: [],
				jztips: '拼命加载中...'
			};
		},
		onLoad(opt) {
			this.type = opt['type'];
			this.load()
		},
		onPullDownRefresh() {
			this.slideTitleList = []
			this.slideList = []
			// this.currentSlideList = []
			
			this.getHistory()
			
			if (this.type != 'local') {
				if (this.authHeader && this.wssAuth && this.userid) {
					this.getAnswer()
				} else {
					this.getAuthHeader()	
				}
			}
			
			setTimeout(function() {
				uni.stopPullDownRefresh();
			}, 500);
		},
		// onReachBottom() {
		// 	const slen = this.slideList[this.slideIndex].length;
		// 	if (slen == 0) {
		// 		this.jztips = '暂时没有PPT ~'
		// 		return
		// 	}
		// 	if (this.currentSlideList.length >= slen) {
		// 		this.jztips = '没有啦 ~'
		// 		return
		// 	}
		// 	let end = slen - this.currentSlideList.length
		// 	if (end >= 9) {
		// 		end = 9
		// 	}
		// 	this.currentSlideList.push(...this.slideList[this.slideIndex].slice(this.currentSlideList.length, this.currentSlideList.length+9))
		// },
		methods: {
			async generatePDF() {
			    try {
			        const pdfDoc = new PDFDocument();
			        const stream = pdfDoc.pipe(blobStream());
			
			        for (const url of this.slideList[this.slideIndex]) {
			          const image = await this.downloadImage(url);
			          pdfDoc.image(image, {
			            fit: [500, 500], // 适当调整图像大小
			          });
			          pdfDoc.addPage();
			        }
			
			        pdfDoc.end();
			
			        stream.on('finish', async () => {
			          const blob = stream.toBlob('application/pdf');
			          const pdfPath = `${plus.io.convertLocalFileSystemURL('_doc/')}/images.pdf`;
			          const file = new File([blob], 'images.pdf', { type: 'application/pdf' });
			          await this.saveFile(pdfPath, file);
			          uni.showToast({
			            title: 'PDF已生成',
			            icon: 'success',
			          });
			        });
			      } catch (e) {
			        console.error(e);
			        uni.showToast({
			          title: '生成PDF失败',
			          icon: 'none',
			        });
				}
			},
			downloadImage(url) {
			  return new Promise((resolve, reject) => {
				downloadFile({
				  url,
				  success: (res) => {
					if (res.statusCode === 200) {
					  resolve(res.tempFilePath);
					} else {
					  reject(new Error('图片下载失败'));
					}
				  },
				  fail: (err) => reject(err),
				});
			  });
			},
			saveFile(path, file) {
			  return new Promise((resolve, reject) => {
				saveFile({
				  tempFilePath: file,
				  filePath: path,
				  success: () => resolve(),
				  fail: (err) => reject(err),
				});
			  });
			},
			load() {
				const option = uni.getStorageSync('current_class')
				if (!option) {
					uni.showToast({
						icon:'none',
						title:'缓存获取失败'
					})
					return
				}
				this.courseName = option['courseName']
				this.source = option['role']
				this.lessonId = option['lessonId']
				if (option['title'] && option['title']!='') {
					this.baseInfo['title'] = option['title']
					uni.setNavigationBarTitle({
						title: this.baseInfo['title']
					})
				}
				this.getHistory()
				if (this.type !== 'local') {
					this.getAuthHeader()
				}
			},
			getBasicInfo() {
				uni.request({
					url: 'https://www.yuketang.cn/api/v3/lesson/basic-info',
					method:'GET',
					header: {
						'Authorization': this.authHeader,
						'Cookie': getApp().globalData.Cookie,
					},
					dataType:'json',
					success:res=>{
						this.baseInfo = res.data['data']
						if (this.baseInfo['title'] && this.baseInfo['title']!='') {
							uni.setNavigationBarTitle({
								title: this.baseInfo['title']
							})
						} else {
							uni.setNavigationBarTitle({
								title: this.courseName
							})
						}
					},
					fail:err=>{
						uni.setNavigationBarTitle({
							title: this.courseName
						})
					}
				})
			},
			getHistory() {
				const res = uni.getStorageSync('pptList')
				if (res) {
					this.pptList = res.data
					if (this.type == 'local') {
						this.isSave = true
						this.slideTitleList = this.pptList[this.lessonId]['slideList']['title']
						this.slideList = this.pptList[this.lessonId]['slideList']['list']
						this.chooseSlideIndex(this.slideIndex)
						if (this.slideList.length > 0) {
							this.isShowPPT = true
							var imgList = []
							this.slideList.forEach(list=>{
								var all_cover = []
								list.forEach(e=>{
									all_cover.push(e['cover'])
								})
								imgList.push(all_cover)
							})
							this.imgList = imgList
						}
					} else {
						if (this.pptList[this.lessonId]) {
							this.isSave = true
						}
					}
					return
				}
			},
			save() {
				var pptList = this.pptList
				if (this.isSave == false) {
					var date = new Date()
					var year = date.getFullYear()
					var month = date.getMonth() + 1
					var day = date.getDate()
					month = (month > 9) ? month : ("0" + month);
					day = (day < 10) ? ("0" + day) : day;
					var time = year+'.'+month+'.'+day
					var lss = {
						title: this.baseInfo['title'],
						courseName: this.courseName,
						lessonId: this.lessonId,
						source: this.source,
						slideList: {
							title: this.slideTitleList,
							list: this.slideList
						},
						time: time
					}
					pptList[this.lessonId] = lss
					uni.setStorage({
						key: 'pptList',
						data: pptList,
						success:ress=>{
							this.isSave = true
							this.pptList = pptList
							uni.showToast({
								icon:'none',
								title:'保存成功'
							})
						},
						fail:errr=>{
							this.isSave = false
							uni.showToast({
								icon:'none',
								title:'保存失败'
							})
						}
					})
				} else {
					delete(pptList[this.lessonId])
					uni.setStorage({
						key: 'pptList',
						data: pptList,
						success:ress=>{
							this.isSave = false
							this.pptList = pptList
							uni.showToast({
								icon:'none',
								title:'取消成功'
							})
						},
						fail:errr=>{
							this.isSave = true
							uni.showToast({
								icon:'none',
								title:'取消失败'
							})
						}
					})
				}
			},
			// 切换ppt index
			chooseSlideIndex(index) {
				// this.currentSlideList = this.slideList[index].slice(0, 10)
				this.slideIndex = index
			},
			choptions(option) {
				this.dqxz = option
				uni.pageScrollTo({
					scrollTop: 0,
					duration: 300
				});
			},
			// 初始化 wss 发送 接收
			initWebsocket() {
				// 创建websocket
				this.websocket = uni.connectSocket({
			        url: 'wss://www.yuketang.cn/wsapp/',
					header: {
						'Cookie': getApp().globalData.Cookie,
					},
					success:res=> {
						console.log('连接成功')
					},
					fail:err=>{
						uni.showModal({
							title: '提示',
							content: '连接雨课堂失败'
						})
					}
				});
				// 建立websocket连接
				this.websocket.onOpen((res) => {
			        // 注：只有连接正常打开中 ，才能正常成功发送消息
			        this.websocket.send({
						data: `{"op":"hello","userid":"${this.userid}","role":"student","auth":"${this.wssAuth}","lessonid":"${this.lessonId}"}`,
			        });
			        // 注：只有连接正常打开中 ，才能正常收到消息
			        this.websocket.onMessage((res) => {
						res.data = JSON.parse(res.data)
						if (res.data['timeline'] && res.data['timeline']['length'] > 0) {
							const timeline = res.data['timeline']
							this.presentation_id_list = {}
							for (let tin in timeline) {
								if (timeline[tin]['type'] == 'slide') {
									var presentation_id = timeline[tin]['pres']
									if (!this.presentation_id_list[presentation_id]) {
										this.presentation_id_list[presentation_id] = {}
										this.getAnswer(presentation_id)
									}
								}
							}
							if (Object.keys(this.presentation_id_list)['length'] > 0) {
								this.websocket.close()
							}
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
			getAnswer(presentation_id) {
				uni.request({
					url: 'https://www.yuketang.cn/api/v3/lesson/presentation/fetch?presentation_id=' + presentation_id,
					header: {
						'Authorization': this.authHeader,
						'Cookie': getApp().globalData.Cookie,
					},
					dataType: 'json',
					success:res=>{
						if (res.data != '') {
							if (res.data['data'] && res.data['data']['slides']) {
								if (res.data['data']['slides']['length'] > 0) {
									this.slideTitleList.push(res.data['data']['title'])
									this.slideList.push(res.data['data']['slides'])
									var cover_arr = []
									res.data['data']['slides'].forEach(e=>{
										cover_arr.push(e['cover'])
									})
									this.imgList.push(cover_arr)
									if (this.currentSlideList.length == 0 && this.slideTitleList.length>0) {
										// this.chooseSlideIndex(this.slideIndex)
										this.isShowPPT = true
									}
								}
							}
						}
					}
				})
			},
			// 获取验证协议头
			getAuthHeader() {
				this.slideList = []
				this.imgList = []
				uni.request({
					url: 'https://www.yuketang.cn/api/v3/lesson/checkin',
					method: 'POST',
					data: {
						'source': parseInt(this.source),
						'lessonId': this.lessonId,
					},
					header: {
						'Content-Type': 'application/json;charset=UTF-8',
						'Cookie': getApp().globalData.Cookie,
					},
					dataType: 'json',
					success:res=>{
						if (res.data['msg'] == 'UNAUTHENTICATED') {
							uni.showModal({
								title: '提示',
								content: '账号过期，请重新登录',
								complete() {
									uni.navigateTo({
										url: '/pages/login/login'
									})
								}
							})
							return
						}
						if (res.data['msg'] == 'LESSON_END') {
							uni.showModal({
								title: '提示',
								content: '课程已经结束',
								complete() {
									uni.navigateBack()
								}
							})
							return
						}
						if (res.header['Set-Auth'] && res.data['data'] && res.data['data']['lessonToken'] && res.data['data']['identityId']) {
							this.authHeader = 'Bearer ' + res.header['Set-Auth']
							this.wssAuth = res.data['data']['lessonToken']
							this.userid = res.data['data']['identityId']
							this.getBasicInfo()
							this.initWebsocket()
							return
						}
						uni.showModal({
							title: '提示',
							content: 'Authorization获取失败，请刷新或重启软件'
						})
					}
				})
			},
			imgShow(cindex) {
				uni.previewImage({
					current: cindex,
					urls: this.imgList[this.slideIndex],
					loop: true
				})
			}
		}
	}
</script>

<style lang="scss" scoped>
	.header {
		.options {
			display: flex;
			flex-direction: row;
			flex-wrap: nowrap;
			.actobox {
				color: #fff !important;
				background-color: $yuKeTang;
			}
			.obox {
				padding: 10rpx 20rpx;
				margin: 10rpx;
				border: 1px solid $yuKeTang;
				border-radius: 8rpx;
				color: $yuKeTang;
				font-size: 13px;
			}
		}
	}
	.center {
		.pptList {
			padding: 15rpx 20rpx;
			.pptBox {
				display: flex;
				flex-direction: column;
				justify-content: center;
				align-items: center;
 				margin: 15rpx auto;
				border-radius: 20rpx;
				box-shadow: 1px 1px 1px 1px #DEEAF7;
				background-color: #fff;
				padding: 15rpx;
				.answer {
					color: $yuKeTang;
				}
			}
		}
	}
	.tips {
		text-align: center;
		color: $yuKeTang;
		font-size: 14px;
		font-weight: 600;
		margin: 30rpx 0;
	}
</style>
